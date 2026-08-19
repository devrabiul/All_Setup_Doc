# 6Valley — Docker Local Setup

> Bilingual guide — **[English](#english)** · **[বাংলা](#বাংলা)**

---

# English

## 1. What this setup gives you

`docker-compose.yml` runs three containers on a private bridge network:

| Service | Image | Role |
|---|---|---|
| `app` | built from `.docker/Dockerfile` (`php:8.3-fpm`) | Laravel app + PHP-FPM (port 9000) + queue worker + scheduler, all supervised by `supervisord` |
| `webserver` | `nginx:1.24.0-alpine` | Serves the site on `http://localhost:${APP_PORT}`, proxies `.php` to `app:9000` |
| `db` | `mysql:8.0` | MySQL 8, auto-imports `database1` and `database2` on first boot |

Two named volumes hold state:

- `6valley-default-data` → the app code inside the container (`/var/www/html`), shared read-only with nginx
- `mysql-data` → MySQL data directory

Inside `app`, `supervisord` keeps three processes alive (`.docker/supervisord.conf`):

1. `php-fpm`
2. `queue-worker` — `queue:work database --queue=auctions,auction_notifications`
3. `scheduler` — `schedule:work` (replaces a system cron)

## 2. Prerequisites

- **Docker Desktop** (macOS/Windows) or **Docker Engine + Compose v2** (Linux) — install from https://docs.docker.com/get-docker/
- Verify:
  ```bash
  docker --version
  docker compose version
  ```
- **Resources:** give Docker Desktop at least **4 GB RAM** and **~15 GB free disk**. The image installs PHP extensions, Node 20, Composer deps, runs a Vite build, and unzips a 69 MB demo asset archive — it is a large image.
- **Free port:** whatever you set as `APP_PORT` (default `8300`).
- **macOS + MAMP note:** this project lives under `/Applications/MAMP/htdocs`, but Docker does **not** use MAMP. Stop MAMP's Apache/MySQL if they conflict with your chosen port.

## 3. Files involved

```
.docker/
├── Dockerfile              # app image build
├── docker-nginx.conf       # nginx vhost (root = /var/www/html, uses root index.php)
├── supervisord.conf        # php-fpm + queue worker + scheduler
├── create_databases.sql    # creates database1, database2 + grants to 6valley_user
└── db/
    ├── database1.sql       # dump, starts with `USE database1;`
    └── database2.sql
docker-compose.yml
.env.example                # template for the root .env
.dockerignore
demo/public.zip             # storage/app/public demo assets (required by the build)
```

## 4. Setup steps

### Step 1 — Create the root `.env`

Compose reads `.env` from the project root to substitute `${APP_PORT}`, `${CONTAINER_NAME_PREFIX}`, `${DB_USERNAME}`, `${DB_PASSWORD}`. Without it, `docker compose` fails or builds unnamed containers.

```bash
cd /Applications/MAMP/htdocs/6Valley
cp .env.example .env
```

Keys that matter for Docker (already correct in `.env.example`):

```dotenv
DB_HOST=db                 # the compose service name — NOT 127.0.0.1
DB_PORT=3306
DB_DATABASE=database1
DB_USERNAME=6valley_user
DB_PASSWORD=Te?tu2)gs4=q(oz?rT5!oz

APP_PORT=8300              # host port → nginx:80
CONTAINER_NAME_PREFIX=6valley-default
COMPOSE_PROJECT_NAME=6valley-default
APP_URL=http://localhost:8300
```

Change `APP_PORT` if `8300` is taken. Keep `APP_URL` in sync with it.

> The **container's** `.env` is a separate copy: the Dockerfile does `cp .env.example .env` during the build. So edits you make to the host `.env` after building are **not** reflected inside the container — see §7 for how to change it live.

### Step 2 — Verify the demo assets exist

The build hard-fails without this file:

```bash
ls -lh demo/public.zip     # expect ~69M
```

### Step 3 — Build and start

```bash
docker compose up -d --build
```

First run takes **10–25 minutes** — it installs apt packages, compiles GD/intl/pdo_mysql, installs Node 20, runs `composer update`, `npm install && npm run build`, and unzips the demo assets.

Compose waits for the DB healthcheck (`mysqladmin ping`) before starting `app`. The MySQL init scripts run **only on first creation** of the `mysql-data` volume, in alphabetical order: `create_databases.sql` → `database1.sql` → `database2.sql`.

### Step 4 — Watch progress

```bash
docker compose ps
docker compose logs -f app
docker compose logs -f db
```

MySQL is ready when the log shows `ready for connections` twice (init pass, then real start). Importing the two ~4.4 MB dumps takes a minute or two.

### Step 5 — Open the app

```
http://localhost:8300
```

Admin panel: `http://localhost:8300/admin` (credentials come from the imported demo dump).

## 5. Verify the install

```bash
# app container health
docker compose exec app php -v
docker compose exec app php artisan --version

# supervised processes — expect php-fpm, queue-worker, scheduler all RUNNING
docker compose exec app supervisorctl status

# database reachable from the app
docker compose exec app php artisan db:show

# tables actually imported
docker compose exec db mysql -u6valley_user -p'Te?tu2)gs4=q(oz?rT5!oz' -e "SHOW TABLES IN database1;" | head
```

## 6. Everyday commands

```bash
docker compose up -d              # start
docker compose stop               # stop, keep data
docker compose down               # remove containers, keep volumes
docker compose restart app        # restart just the app
docker compose exec app bash      # shell into the app container
docker compose logs -f            # all logs

# artisan
docker compose exec app php artisan optimize:clear
docker compose exec app php artisan migrate
docker compose exec app php artisan queue:restart

# nginx config reload after editing docker-nginx.conf
docker compose restart webserver
```

`.docker/docker-nginx.conf` is bind-mounted, so editing it needs only a `webserver` restart — no rebuild.

## 7. Important gotchas

**Your local code edits do NOT appear in the browser.**
The `app` service mounts the **named volume** `6valley-default-data` at `/var/www/html`, not your project folder. Code is baked into the image at build time and copied into the volume the first time it is created. Two consequences:

1. Editing a file on your Mac changes nothing in the container.
2. Rebuilding the image alone changes nothing either — Docker **will not** overwrite a non-empty named volume.

To ship code changes, rebuild **and** drop the app volume:

```bash
docker compose down
docker volume rm 6valley-default_6valley-default-data
docker compose up -d --build
```

For quick one-off edits, edit inside the container instead (`nano` is installed):

```bash
docker compose exec app bash
nano .env
exit
docker compose restart app
```

If you want true live-reload development, add a bind mount to the `app` and `webserver` services in `docker-compose.yml` (e.g. `- ./:/var/www/html`) — but note that overrides the built `vendor/`, `node_modules/`, `public/build`, and the demo `storage/app/public`, so you must install and build on the host yourself.

**Database dumps only import once.** `docker-entrypoint-initdb.d` runs on first creation of `mysql-data`. To re-import from scratch:

```bash
docker compose down
docker volume rm 6valley-default_mysql-data
docker compose up -d
```

**Never run `php artisan migrate` on an empty database.** Per the project rules, the SQL dump must be imported first — the compose setup already does this via the init scripts. Only run `migrate` afterwards to apply pending migrations.

**`docker compose down -v` deletes both volumes** — code volume *and* your database. That is a full reset, not a restart.

**`.dockerignore` excludes `vendor/`, `node_modules/`, `.git/`, and `.env`.** They are generated or created inside the image; do not expect the host copies to be used.

**Port conflict.** If `8300` is busy (MAMP, another stack), change `APP_PORT` in `.env`, then `docker compose up -d` — no rebuild needed, the port only affects the `webserver` service.

**Apple Silicon (M1–M4).** `php:8.3-fpm` and `mysql:8.0` both publish `arm64` images, so it runs natively. If you hit a platform error, add `platform: linux/amd64` to the failing service and expect a slower emulated run.

## 8. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `variable is not set` warnings on `up` | root `.env` missing | `cp .env.example .env` |
| Build fails at `COPY demo/public.zip` | file missing | restore `demo/public.zip` from the release package |
| `502 Bad Gateway` | php-fpm not up in `app` | `docker compose logs app`; `docker compose exec app supervisorctl status` |
| `SQLSTATE[HY000] [2002] Connection refused` | `DB_HOST` wrong inside container | must be `db`, not `127.0.0.1`; fix the container `.env` (§7) |
| `Access denied for user '6valley_user'` | DB volume created before the current password | `docker compose down && docker volume rm 6valley-default_mysql-data && docker compose up -d` |
| `SHOW TABLES` returns nothing | init scripts skipped on an existing volume | same as above |
| Blank page / missing CSS-JS | Vite build or demo assets missing | `docker compose exec app ls public/build storage/app/public` |
| Code changes not visible | named-volume behaviour | see §7 |
| Queue jobs never process | worker crashed | `docker compose exec app supervisorctl restart queue-worker` |
| `permission denied` on `storage/` | ownership drift | `docker compose exec app chown -R www-data:www-data storage bootstrap/cache` |

## 9. Full reset

```bash
docker compose down -v            # removes containers AND both volumes
docker image rm 6valley-default-app:latest
docker compose up -d --build
```

---
---

# বাংলা

## ১. এই সেটআপে কী কী চলে

`docker-compose.yml` একটি প্রাইভেট ব্রিজ নেটওয়ার্কে তিনটি কন্টেইনার চালায়:

| সার্ভিস | ইমেজ | কাজ |
|---|---|---|
| `app` | `.docker/Dockerfile` থেকে বিল্ড (`php:8.3-fpm`) | Laravel অ্যাপ + PHP-FPM (পোর্ট 9000) + queue worker + scheduler — সবগুলো `supervisord` দিয়ে নিয়ন্ত্রিত |
| `webserver` | `nginx:1.24.0-alpine` | `http://localhost:${APP_PORT}` এ সাইট সার্ভ করে, `.php` রিকোয়েস্ট `app:9000` এ পাঠায় |
| `db` | `mysql:8.0` | MySQL 8 — প্রথম বুটেই `database1` ও `database2` অটো-ইমপোর্ট করে |

দুইটি named volume ডেটা ধরে রাখে:

- `6valley-default-data` → কন্টেইনারের ভেতরের কোড (`/var/www/html`), nginx এটি read-only হিসেবে পড়ে
- `mysql-data` → MySQL এর ডেটা ডিরেক্টরি

`app` কন্টেইনারের ভেতরে `supervisord` তিনটি প্রসেস চালু রাখে (`.docker/supervisord.conf`):

১. `php-fpm`
২. `queue-worker` — `queue:work database --queue=auctions,auction_notifications`
৩. `scheduler` — `schedule:work` (সিস্টেম cron এর বিকল্প)

## ২. পূর্বশর্ত

- **Docker Desktop** (macOS/Windows) অথবা **Docker Engine + Compose v2** (Linux) — https://docs.docker.com/get-docker/
- যাচাই করুন:
  ```bash
  docker --version
  docker compose version
  ```
- **রিসোর্স:** Docker Desktop এ কমপক্ষে **৪ GB RAM** এবং **~১৫ GB খালি ডিস্ক** দিন। ইমেজটি PHP এক্সটেনশন, Node 20, Composer ডিপেন্ডেন্সি ইনস্টল করে, Vite বিল্ড চালায়, এবং ৬৯ MB এর ডেমো অ্যাসেট আনজিপ করে — ইমেজটি বড়।
- **খালি পোর্ট:** `APP_PORT` এ যা দেবেন (ডিফল্ট `8300`)।
- **macOS + MAMP নোট:** প্রজেক্টটি `/Applications/MAMP/htdocs` এ থাকলেও Docker কিন্তু MAMP ব্যবহার করে **না**। পোর্ট কনফ্লিক্ট হলে MAMP এর Apache/MySQL বন্ধ করুন।

## ৩. সংশ্লিষ্ট ফাইলগুলো

```
.docker/
├── Dockerfile              # অ্যাপ ইমেজ বিল্ড
├── docker-nginx.conf       # nginx vhost (root = /var/www/html, রুটের index.php ব্যবহার করে)
├── supervisord.conf        # php-fpm + queue worker + scheduler
├── create_databases.sql    # database1, database2 তৈরি + 6valley_user কে গ্রান্ট
└── db/
    ├── database1.sql       # ডাম্প, শুরুতেই `USE database1;` আছে
    └── database2.sql
docker-compose.yml
.env.example                # রুট .env এর টেমপ্লেট
.dockerignore
demo/public.zip             # storage/app/public এর ডেমো অ্যাসেট (বিল্ডের জন্য বাধ্যতামূলক)
```

## ৪. সেটআপ ধাপসমূহ

### ধাপ ১ — রুটে `.env` তৈরি করুন

Compose প্রজেক্ট রুটের `.env` পড়ে `${APP_PORT}`, `${CONTAINER_NAME_PREFIX}`, `${DB_USERNAME}`, `${DB_PASSWORD}` বসায়। এটি না থাকলে `docker compose` ব্যর্থ হবে বা নামবিহীন কন্টেইনার বানাবে।

```bash
cd /Applications/MAMP/htdocs/6Valley
cp .env.example .env
```

Docker এর জন্য গুরুত্বপূর্ণ কী-গুলো (`.env.example` এ ইতিমধ্যেই ঠিক আছে):

```dotenv
DB_HOST=db                 # compose সার্ভিসের নাম — 127.0.0.1 নয়
DB_PORT=3306
DB_DATABASE=database1
DB_USERNAME=6valley_user
DB_PASSWORD=Te?tu2)gs4=q(oz?rT5!oz

APP_PORT=8300              # হোস্ট পোর্ট → nginx:80
CONTAINER_NAME_PREFIX=6valley-default
COMPOSE_PROJECT_NAME=6valley-default
APP_URL=http://localhost:8300
```

`8300` দখলে থাকলে `APP_PORT` বদলান। `APP_URL` ও একই পোর্টে রাখুন।

> **কন্টেইনারের ভেতরের** `.env` আলাদা একটি কপি: Dockerfile বিল্ডের সময় `cp .env.example .env` চালায়। তাই বিল্ডের পরে হোস্টের `.env` এডিট করলে কন্টেইনারে তার কোনো প্রভাব পড়ে **না** — §৭ দেখুন।

### ধাপ ২ — ডেমো অ্যাসেট আছে কিনা দেখুন

এই ফাইল ছাড়া বিল্ড ফেইল করবে:

```bash
ls -lh demo/public.zip     # প্রায় ~69M হওয়া উচিত
```

### ধাপ ৩ — বিল্ড ও চালু

```bash
docker compose up -d --build
```

প্রথমবার **১০–২৫ মিনিট** লাগে — apt প্যাকেজ ইনস্টল, GD/intl/pdo_mysql কম্পাইল, Node 20 ইনস্টল, `composer update`, `npm install && npm run build`, এবং ডেমো অ্যাসেট আনজিপ হয়।

Compose প্রথমে DB এর healthcheck (`mysqladmin ping`) পাস করার অপেক্ষা করে, তারপর `app` চালু করে। MySQL এর init স্ক্রিপ্টগুলো **শুধু `mysql-data` ভলিউম প্রথমবার তৈরি হওয়ার সময়** চলে, বর্ণানুক্রমে: `create_databases.sql` → `database1.sql` → `database2.sql`।

### ধাপ ৪ — অগ্রগতি দেখুন

```bash
docker compose ps
docker compose logs -f app
docker compose logs -f db
```

লগে `ready for connections` দুইবার এলে MySQL প্রস্তুত (একবার init পর্যায়ে, একবার আসল স্টার্টে)। দুইটি ~৪.৪ MB ডাম্প ইমপোর্ট হতে এক-দুই মিনিট লাগে।

### ধাপ ৫ — অ্যাপ খুলুন

```
http://localhost:8300
```

অ্যাডমিন প্যানেল: `http://localhost:8300/admin` (লগইন তথ্য ইমপোর্ট করা ডেমো ডাম্প থেকে আসে)।

## ৫. ইনস্টল যাচাই

```bash
# app কন্টেইনার ঠিক আছে কিনা
docker compose exec app php -v
docker compose exec app php artisan --version

# supervised প্রসেস — php-fpm, queue-worker, scheduler তিনটিই RUNNING হওয়া উচিত
docker compose exec app supervisorctl status

# অ্যাপ থেকে ডাটাবেস কানেকশন
docker compose exec app php artisan db:show

# টেবিল আসলেই ইমপোর্ট হয়েছে কিনা
docker compose exec db mysql -u6valley_user -p'Te?tu2)gs4=q(oz?rT5!oz' -e "SHOW TABLES IN database1;" | head
```

## ৬. প্রতিদিনের কমান্ড

```bash
docker compose up -d              # চালু
docker compose stop               # বন্ধ, ডেটা থাকবে
docker compose down               # কন্টেইনার মুছে ফেলে, ভলিউম থাকবে
docker compose restart app        # শুধু app রিস্টার্ট
docker compose exec app bash      # app কন্টেইনারে শেল
docker compose logs -f            # সব লগ

# artisan
docker compose exec app php artisan optimize:clear
docker compose exec app php artisan migrate
docker compose exec app php artisan queue:restart

# docker-nginx.conf এডিটের পর
docker compose restart webserver
```

`.docker/docker-nginx.conf` bind-mount করা, তাই এটি এডিট করলে শুধু `webserver` রিস্টার্ট করলেই হয় — রিবিল্ড লাগে না।

## ৭. গুরুত্বপূর্ণ সতর্কতা

**আপনার লোকাল কোড এডিট ব্রাউজারে দেখা যাবে না।**
`app` সার্ভিস `/var/www/html` এ আপনার প্রজেক্ট ফোল্ডার নয়, বরং **named volume** `6valley-default-data` মাউন্ট করে। কোড বিল্ড টাইমে ইমেজে ঢোকে এবং ভলিউম প্রথমবার তৈরি হওয়ার সময় সেখানে কপি হয়। এর দুইটি ফল:

১. Mac এ ফাইল এডিট করলে কন্টেইনারে কিছুই বদলায় না।
২. শুধু ইমেজ রিবিল্ড করলেও কিছু বদলায় না — Docker খালি নয় এমন named volume ওভাররাইট করে **না**।

কোড পরিবর্তন কার্যকর করতে রিবিল্ড **এবং** app ভলিউম মুছুন:

```bash
docker compose down
docker volume rm 6valley-default_6valley-default-data
docker compose up -d --build
```

ছোটখাটো এডিটের জন্য কন্টেইনারের ভেতরেই এডিট করুন (`nano` ইনস্টল করা আছে):

```bash
docker compose exec app bash
nano .env
exit
docker compose restart app
```

সত্যিকারের live-reload ডেভেলপমেন্ট চাইলে `docker-compose.yml` এর `app` ও `webserver` সার্ভিসে bind mount যোগ করুন (যেমন `- ./:/var/www/html`) — তবে মনে রাখবেন এটি বিল্ড করা `vendor/`, `node_modules/`, `public/build`, এবং ডেমো `storage/app/public` ঢেকে দেবে, তাই হোস্টে নিজেই ইনস্টল ও বিল্ড করতে হবে।

**ডাটাবেস ডাম্প একবারই ইমপোর্ট হয়।** `docker-entrypoint-initdb.d` শুধু `mysql-data` প্রথমবার তৈরি হওয়ার সময় চলে। নতুন করে ইমপোর্ট করতে:

```bash
docker compose down
docker volume rm 6valley-default_mysql-data
docker compose up -d
```

**খালি ডাটাবেসে কখনো `php artisan migrate` চালাবেন না।** প্রজেক্টের নিয়ম অনুযায়ী আগে SQL ডাম্প ইমপোর্ট করতে হবে — compose সেটআপ init স্ক্রিপ্ট দিয়ে এটি এমনিতেই করে। এর পরেই কেবল pending migration চালাতে `migrate` ব্যবহার করুন।

**`docker compose down -v` দুইটি ভলিউমই মুছে দেয়** — কোড ভলিউম *এবং* আপনার ডাটাবেস। এটি রিস্টার্ট নয়, সম্পূর্ণ রিসেট।

**`.dockerignore` বাদ দেয় `vendor/`, `node_modules/`, `.git/`, `.env`।** এগুলো ইমেজের ভেতরে তৈরি হয়; হোস্টের কপি ব্যবহৃত হবে না।

**পোর্ট কনফ্লিক্ট।** `8300` ব্যস্ত থাকলে (MAMP বা অন্য স্ট্যাক) `.env` এ `APP_PORT` বদলে `docker compose up -d` দিন — রিবিল্ড লাগবে না, পোর্ট শুধু `webserver` সার্ভিসে প্রভাব ফেলে।

**Apple Silicon (M1–M4)।** `php:8.3-fpm` ও `mysql:8.0` দুইটিরই `arm64` ইমেজ আছে, তাই নেটিভভাবেই চলে। প্ল্যাটফর্ম এরর এলে ঐ সার্ভিসে `platform: linux/amd64` যোগ করুন — তবে এমুলেশনে ধীর চলবে।

## ৮. সমস্যা সমাধান

| সমস্যা | কারণ | সমাধান |
|---|---|---|
| `up` দিলে `variable is not set` ওয়ার্নিং | রুটে `.env` নেই | `cp .env.example .env` |
| `COPY demo/public.zip` এ বিল্ড ফেইল | ফাইলটি নেই | রিলিজ প্যাকেজ থেকে `demo/public.zip` ফিরিয়ে আনুন |
| `502 Bad Gateway` | `app` এ php-fpm চালু হয়নি | `docker compose logs app`; `docker compose exec app supervisorctl status` |
| `SQLSTATE[HY000] [2002] Connection refused` | কন্টেইনারে `DB_HOST` ভুল | `127.0.0.1` নয়, `db` হতে হবে; কন্টেইনারের `.env` ঠিক করুন (§৭) |
| `Access denied for user '6valley_user'` | বর্তমান পাসওয়ার্ডের আগে DB ভলিউম তৈরি হয়েছে | `docker compose down && docker volume rm 6valley-default_mysql-data && docker compose up -d` |
| `SHOW TABLES` খালি আসে | পুরোনো ভলিউম থাকায় init স্ক্রিপ্ট চলেনি | উপরেরটাই প্রযোজ্য |
| সাদা পেজ / CSS-JS নেই | Vite বিল্ড বা ডেমো অ্যাসেট নেই | `docker compose exec app ls public/build storage/app/public` |
| কোড পরিবর্তন দেখা যাচ্ছে না | named volume এর আচরণ | §৭ দেখুন |
| Queue জব প্রসেস হচ্ছে না | worker ক্র্যাশ করেছে | `docker compose exec app supervisorctl restart queue-worker` |
| `storage/` এ `permission denied` | ownership বদলে গেছে | `docker compose exec app chown -R www-data:www-data storage bootstrap/cache` |

## ৯. সম্পূর্ণ রিসেট

```bash
docker compose down -v            # কন্টেইনার এবং দুইটি ভলিউমই মুছে ফেলে
docker image rm 6valley-default-app:latest
docker compose up -d --build
```


## Env File
```bash
APP_NAME=6valley-default
APP_ENV=local
APP_KEY=base64:1vaU3dfc+sWjx8TuDXzginRsEa2dp2SBL+Ujs6QCb5c=
APP_DEBUG=false
APP_MODE=demo
APP_URL=http://localhost:8300

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=database1
DB_USERNAME=6valley_user
DB_PASSWORD=Te?tu2)gs4=q(oz?rT5!oz

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=database
SESSION_DRIVER=file
SESSION_LIFETIME=120
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
AWS_ENDPOINT=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1
MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
NEXMO_KEY=
NEXMO_SECRET=
PURCHASE_CODE=
BUYER_USERNAME=
SOFTWARE_ID=MzE0NDg1OTc=
SOFTWARE_VERSION=16.5
WEB_THEME=default

#docker configs
APP_PORT=8300
CONTAINER_NAME_PREFIX=6valley-default
COMPOSE_PROJECT_NAME=6valley-default
FORCE_HTTPS=false

LOG_CHANNEL=single
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

# The single hostname where the admin/vendor panel + all host routes live.
# Sub-domains and custom domains in tenant_domain_configs serve the
# storefront at root. Leave both unset for single-tenant / legacy installs.
APP_HOST_DOMAIN=
APP_HOST_BASE_DOMAIN=
APP_PUBLIC_IP=
```
