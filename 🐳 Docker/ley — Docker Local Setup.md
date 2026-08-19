# 6Valley — Docker Local Setup

> Bilingual guide — **[English](#english)** · **[বাংলা](#বাংলা)**

---

# English

## 0. Which demo do you want? (`demo-default-production` / `demo-aster-production`)

Each demo lives on its own branch and ships its **own** compose project, port, volumes,
network and database dump. Nothing is shared, so **both can run at the same time**.

| | `demo-default-production` | `demo-aster-production` |
|---|---|---|
| URL | `http://localhost:8300` | `http://localhost:8301` |
| `APP_PORT` | `8300` | `8301` |
| `WEB_THEME` | `default` | `theme_aster` |
| `CONTAINER_NAME_PREFIX` / `COMPOSE_PROJECT_NAME` | `6valley-default` | `6valley-aster` |
| Code volume | `6valley-default_6valley-default-data` | `6valley-aster_6valley-aster-data` |
| DB volume | `6valley-default_mysql-data` | `6valley-aster_mysql-data` |
| Network | `6valley-default-network` | `6valley-aster-network` |
| Image | `6valley-default-app:latest` | `6valley-aster-app:latest` |
| DB dumps | `.docker/db/*.sql` (default demo data) | `.docker/db/*.sql` (**different** aster demo data) |

Everything in this guide applies to both — only the branch, port and
`6valley-default` / `6valley-aster` names change.

### Run one demo

```bash
cd /Applications/MAMP/htdocs/6Valley
git checkout demo-default-production     # or: demo-aster-production
cp .env.example .env                     # branch-specific: brings the right port + theme
docker compose up -d --build
```

Then apply the **required post-build fix** in §4 Step 4 — without it every page 404s.

### Run both demos side by side

They only coexist if each stack is built **from its own branch**, because
`docker-compose.yml`, `.env.example` and the SQL dumps all differ per branch. Build one,
then switch and build the other:

```bash
# 1) default demo → localhost:8300
git checkout demo-default-production
cp .env.example .env
docker compose up -d --build

# 2) aster demo → localhost:8301
git checkout demo-aster-production
cp .env.example .env
docker compose up -d --build
```

Both stacks keep running after you switch branches again — the containers hold their own
copy of the code in their own volume, so the checked-out branch only matters at
**build** time.

```bash
docker ps --format '{{.Names}}\t{{.Ports}}' | grep 6valley
```

> **Always `cp .env.example .env` right after switching branches.** `.env` is gitignored,
> so it does **not** change when you checkout. If you build aster while `.env` still says
> `COMPOSE_PROJECT_NAME=6valley-default`, compose files the aster volumes under the
> *default* project and you get an orphan like `6valley-default_6valley-aster-data`.
> Harmless but confusing — remove with
> `docker volume rm 6valley-default_6valley-aster-data`.

### Switching a demo off / on

```bash
docker compose -p 6valley-default stop     # or start
docker compose -p 6valley-aster  stop
```

`-p` targets a stack by project name, so it works no matter which branch is checked out.

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

### Step 4 — **Required:** patch the container's `.env` (otherwise every page 404s)

`.env.example` — which the Dockerfile bakes into the image as the container's `.env` —
ships two values that are correct for the public demo server but break any localhost run:

| Key | Shipped value | Why it breaks localhost |
|---|---|---|
| `APP_HOST_DOMAIN` | `6valley.6amtech.com` (aster: `6valley-aster.6amtech.com`) | `RouteServiceProvider` wraps **all** host routes (web, admin, vendor, api v1–v3) in `Route::domain(...)` when this is set. Nothing matches `localhost`, so `/`, `/login/admin` and `/admin` all return **404** — while static assets still return 200. |
| `FORCE_HTTPS` | `true` | `AppServiceProvider` forces `https://` for every client except `127.0.0.1`. Behind nginx the client *is* the Docker bridge, so the exemption never applies. Asset URLs become `https://localhost:8300`, which this nginx does not serve — no CSS/JS, and the browser tries a TLS handshake against a plain-HTTP port. |

Run once per stack, after the first `up`:

```bash
# default demo (8300)
docker compose exec app sed -i \
  -e 's|^APP_HOST_DOMAIN=.*|APP_HOST_DOMAIN=|' \
  -e 's|^APP_HOST_BASE_DOMAIN=.*|APP_HOST_BASE_DOMAIN=|' \
  -e 's|^FORCE_HTTPS=.*|FORCE_HTTPS=false|' \
  -e 's|^APP_URL=.*|APP_URL=http://localhost:8300|' .env
docker compose exec app php artisan optimize:clear
docker compose restart app
```

For the **aster** demo use `-p 6valley-aster` and port `8301`:

```bash
docker compose -p 6valley-aster exec app sed -i \
  -e 's|^APP_HOST_DOMAIN=.*|APP_HOST_DOMAIN=|' \
  -e 's|^APP_HOST_BASE_DOMAIN=.*|APP_HOST_BASE_DOMAIN=|' \
  -e 's|^FORCE_HTTPS=.*|FORCE_HTTPS=false|' \
  -e 's|^APP_URL=.*|APP_URL=http://localhost:8301|' .env
docker compose -p 6valley-aster exec app php artisan optimize:clear
docker compose -p 6valley-aster restart app
```

> **Re-apply this after any rebuild that drops the code volume.** The fix lives in the
> container's `.env` inside the volume; recreating the volume restores the baked
> `.env.example` values and the 404 comes straight back.

### Step 5 — Watch progress

```bash
docker compose ps
docker compose logs -f app
docker compose logs -f db
```

MySQL is ready when the log shows `ready for connections` twice (init pass, then real start). Importing the two ~4.4 MB dumps takes a minute or two.

### Step 6 — Open the app

| Demo | Storefront | Admin login |
|---|---|---|
| `demo-default-production` | `http://localhost:8300` | `http://localhost:8300/login/admin` |
| `demo-aster-production` | `http://localhost:8301` | `http://localhost:8301/login/admin` |

Credentials come from the imported demo dump (the login screen shows them in demo mode).
`/admin` itself returns 404 until you are logged in — that is deliberate, the admin
middleware hides the panel rather than redirecting.

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

To ship code changes, rebuild **and** drop the app volume — from the branch you want built:

```bash
# default demo
docker compose down
docker volume rm 6valley-default_6valley-default-data
docker compose up -d --build

# aster demo
docker compose -p 6valley-aster down
docker volume rm 6valley-aster_6valley-aster-data
docker compose up -d --build
```

Dropping the code volume also throws away the §4 Step 4 `.env` patch — re-apply it, or you
are back to 404 on every page.

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
# default demo
docker compose down && docker volume rm 6valley-default_mysql-data && docker compose up -d

# aster demo
docker compose -p 6valley-aster down && docker volume rm 6valley-aster_mysql-data && docker compose up -d
```

The two demos ship **different** dumps, so each stack must be re-imported from its own
branch — aster's data will not appear in the default stack, or vice versa.

**Never run `php artisan migrate` on an empty database.** Per the project rules, the SQL dump must be imported first — the compose setup already does this via the init scripts. Only run `migrate` afterwards to apply pending migrations.

**`docker compose down -v` deletes both volumes** — code volume *and* your database. That is a full reset, not a restart.

**`.dockerignore` excludes `vendor/`, `node_modules/`, `.git/`, and `.env`.** They are generated or created inside the image; do not expect the host copies to be used.

**Port conflict.** If `8300` (default demo) or `8301` (aster demo) is busy, change `APP_PORT` in `.env`, then `docker compose up -d` — no rebuild needed, the port only affects the `webserver` service. Also update `APP_URL` in the **container's** `.env` to match (§4 Step 4), or generated links keep pointing at the old port.

**Apple Silicon (M1–M4).** `php:8.3-fpm` and `mysql:8.0` both publish `arm64` images, so it runs natively. If you hit a platform error, add `platform: linux/amd64` to the failing service and expect a slower emulated run.

## 8. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| **404 on every page** (`/`, `/login/admin`) but CSS/images load | `APP_HOST_DOMAIN` still set to the demo domain | apply §4 Step 4 |
| **No CSS/JS**, browser tries `https://localhost:8300` | `FORCE_HTTPS=true` — the `127.0.0.1` exemption never applies behind nginx | apply §4 Step 4 |
| 404 came back after a rebuild | code volume recreated → baked `.env` restored | re-apply §4 Step 4 |
| `dependency failed to start: container … is unhealthy` + log says `MYSQL_USER="root" … cannot be used for the root user` | root `.env` has `DB_USERNAME=root` (e.g. local MAMP creds); compose feeds it to MySQL | set `DB_USERNAME=6valley_user` / `DB_PASSWORD=Te?tu2)gs4=q(oz?rT5!oz` in the root `.env`, or `cp .env.example .env` |
| Orphan volume `6valley-default_6valley-aster-data` | built aster while `.env` still had the default `COMPOSE_PROJECT_NAME` | `cp .env.example .env` after checkout; `docker volume rm` the orphan |
| Aster shows the default theme | built with the wrong branch's `.env` | `git checkout demo-aster-production && cp .env.example .env` (`WEB_THEME=theme_aster`), rebuild |
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

Run this from the branch whose demo you are resetting.

```bash
# default demo — from demo-default-production
docker compose down -v                      # containers AND both volumes
docker image rm 6valley-default-app:latest
docker compose up -d --build

# aster demo — from demo-aster-production
docker compose -p 6valley-aster down -v
docker image rm 6valley-aster-app:latest
docker compose up -d --build
```

Re-apply the §4 Step 4 `.env` patch afterwards.

## 10. Two demos — quick reference

| Task | `demo-default-production` | `demo-aster-production` |
|---|---|---|
| Build | `git checkout demo-default-production && cp .env.example .env && docker compose up -d --build` | `git checkout demo-aster-production && cp .env.example .env && docker compose up -d --build` |
| Open | `http://localhost:8300` | `http://localhost:8301` |
| Logs | `docker compose -p 6valley-default logs -f` | `docker compose -p 6valley-aster logs -f` |
| Shell | `docker compose -p 6valley-default exec app bash` | `docker compose -p 6valley-aster exec app bash` |
| Stop | `docker compose -p 6valley-default stop` | `docker compose -p 6valley-aster stop` |
| Reset DB | `docker volume rm 6valley-default_mysql-data` | `docker volume rm 6valley-aster_mysql-data` |
| Reset code | `docker volume rm 6valley-default_6valley-default-data` | `docker volume rm 6valley-aster_6valley-aster-data` |

`-p <project>` targets a stack regardless of which branch is currently checked out.
Commands **without** `-p` act on whatever `COMPOSE_PROJECT_NAME` your current `.env` says —
which is why you should `cp .env.example .env` immediately after every checkout.

---
---

# বাংলা

## ০. কোন ডেমো চালাবেন? (`demo-default-production` / `demo-aster-production`)

প্রতিটি ডেমো আলাদা ব্রাঞ্চে থাকে এবং **নিজস্ব** compose প্রজেক্ট, পোর্ট, ভলিউম, নেটওয়ার্ক ও
ডাটাবেস ডাম্প নিয়ে আসে। কিছুই শেয়ার হয় না, তাই **দুইটি একসাথে চালানো যায়**।

| | `demo-default-production` | `demo-aster-production` |
|---|---|---|
| URL | `http://localhost:8300` | `http://localhost:8301` |
| `APP_PORT` | `8300` | `8301` |
| `WEB_THEME` | `default` | `theme_aster` |
| `CONTAINER_NAME_PREFIX` / `COMPOSE_PROJECT_NAME` | `6valley-default` | `6valley-aster` |
| কোড ভলিউম | `6valley-default_6valley-default-data` | `6valley-aster_6valley-aster-data` |
| DB ভলিউম | `6valley-default_mysql-data` | `6valley-aster_mysql-data` |
| নেটওয়ার্ক | `6valley-default-network` | `6valley-aster-network` |
| ইমেজ | `6valley-default-app:latest` | `6valley-aster-app:latest` |
| DB ডাম্প | `.docker/db/*.sql` (ডিফল্ট ডেমো ডেটা) | `.docker/db/*.sql` (**ভিন্ন** aster ডেমো ডেটা) |

এই গাইডের সবকিছু দুইটির জন্যই প্রযোজ্য — শুধু ব্রাঞ্চ, পোর্ট এবং
`6valley-default` / `6valley-aster` নামগুলো বদলায়।

### একটি ডেমো চালানো

```bash
cd /Applications/MAMP/htdocs/6Valley
git checkout demo-default-production     # অথবা: demo-aster-production
cp .env.example .env                     # ব্রাঞ্চ-অনুযায়ী সঠিক পোর্ট ও থিম আনে
docker compose up -d --build
```

এরপর §৪ ধাপ ৪ এর **বাধ্যতামূলক ফিক্স** চালান — না করলে প্রতিটি পেজ ৪০৪ দেবে।

### দুইটি ডেমো একসাথে চালানো

প্রতিটি স্ট্যাক অবশ্যই **নিজের ব্রাঞ্চ থেকে** বিল্ড করতে হবে, কারণ `docker-compose.yml`,
`.env.example` ও SQL ডাম্প — সবই ব্রাঞ্চভেদে আলাদা। একটি বিল্ড করে, তারপর ব্রাঞ্চ বদলে
অন্যটি বিল্ড করুন:

```bash
# ১) default ডেমো → localhost:8300
git checkout demo-default-production
cp .env.example .env
docker compose up -d --build

# ২) aster ডেমো → localhost:8301
git checkout demo-aster-production
cp .env.example .env
docker compose up -d --build
```

পরে আবার ব্রাঞ্চ বদলালেও দুইটি স্ট্যাকই চলতে থাকে — কন্টেইনার নিজের ভলিউমে কোডের নিজস্ব
কপি রাখে, তাই চেকআউট করা ব্রাঞ্চ শুধু **বিল্ডের সময়** গুরুত্বপূর্ণ।

```bash
docker ps --format '{{.Names}}\t{{.Ports}}' | grep 6valley
```

> **ব্রাঞ্চ বদলানোর সাথে সাথেই সবসময় `cp .env.example .env` চালান।** `.env` gitignore করা,
> তাই checkout করলে এটি বদলায় **না**। `.env` এ `COMPOSE_PROJECT_NAME=6valley-default`
> থাকা অবস্থায় aster বিল্ড করলে compose সেটির ভলিউম *default* প্রজেক্টের নিচে বানায় এবং
> `6valley-default_6valley-aster-data` এর মতো একটি orphan তৈরি হয়। ক্ষতিকর নয়, তবে
> বিভ্রান্তিকর — মুছতে: `docker volume rm 6valley-default_6valley-aster-data`।

### কোনো ডেমো বন্ধ / চালু করা

```bash
docker compose -p 6valley-default stop     # অথবা start
docker compose -p 6valley-aster  stop
```

`-p` দিয়ে প্রজেক্টের নাম ধরে স্ট্যাক টার্গেট করা যায়, তাই কোন ব্রাঞ্চ চেকআউট আছে তাতে
কিছু যায় আসে না।

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

### ধাপ ৪ — **বাধ্যতামূলক:** কন্টেইনারের `.env` ঠিক করুন (না করলে প্রতিটি পেজ ৪০৪ দেবে)

Dockerfile বিল্ডের সময় `.env.example` কে কন্টেইনারের `.env` হিসেবে বসায়। সেই ফাইলে দুইটি মান
আছে যেগুলো পাবলিক ডেমো সার্ভারের জন্য ঠিক, কিন্তু localhost এ চালালে ভাঙে:

| কী | ফাইলে যা আছে | localhost এ কেন ভাঙে |
|---|---|---|
| `APP_HOST_DOMAIN` | `6valley.6amtech.com` (aster: `6valley-aster.6amtech.com`) | এটি সেট থাকলে `RouteServiceProvider` **সব** host রুট (web, admin, vendor, api v1–v3) `Route::domain(...)` দিয়ে মুড়ে দেয়। `localhost` এর সাথে কিছুই মেলে না, তাই `/`, `/login/admin`, `/admin` সব **৪০৪** — অথচ static অ্যাসেট ঠিকই ২০০ দেয়। |
| `FORCE_HTTPS` | `true` | `AppServiceProvider` `127.0.0.1` ছাড়া সব ক্লায়েন্টের জন্য `https://` জোর করে। nginx এর পেছনে ক্লায়েন্ট আসলে Docker bridge, তাই ছাড়টি কখনো কাজ করে না। অ্যাসেটের URL হয়ে যায় `https://localhost:8300`, যা এই nginx সার্ভ করে না — CSS/JS আসে না, ব্রাউজার প্লেইন-HTTP পোর্টে TLS handshake চেষ্টা করে। |

প্রথম `up` এর পরে প্রতি স্ট্যাকে একবার চালান:

```bash
# default ডেমো (8300)
docker compose exec app sed -i \
  -e 's|^APP_HOST_DOMAIN=.*|APP_HOST_DOMAIN=|' \
  -e 's|^APP_HOST_BASE_DOMAIN=.*|APP_HOST_BASE_DOMAIN=|' \
  -e 's|^FORCE_HTTPS=.*|FORCE_HTTPS=false|' \
  -e 's|^APP_URL=.*|APP_URL=http://localhost:8300|' .env
docker compose exec app php artisan optimize:clear
docker compose restart app
```

**aster** ডেমোর জন্য `-p 6valley-aster` ও পোর্ট `8301`:

```bash
docker compose -p 6valley-aster exec app sed -i \
  -e 's|^APP_HOST_DOMAIN=.*|APP_HOST_DOMAIN=|' \
  -e 's|^APP_HOST_BASE_DOMAIN=.*|APP_HOST_BASE_DOMAIN=|' \
  -e 's|^FORCE_HTTPS=.*|FORCE_HTTPS=false|' \
  -e 's|^APP_URL=.*|APP_URL=http://localhost:8301|' .env
docker compose -p 6valley-aster exec app php artisan optimize:clear
docker compose -p 6valley-aster restart app
```

> **কোড ভলিউম মুছে রিবিল্ড করলে এটি আবার চালাতে হবে।** ফিক্সটি ভলিউমের ভেতরের কন্টেইনার
> `.env` এ থাকে; ভলিউম নতুন করে তৈরি হলে `.env.example` এর মান ফিরে আসে এবং ৪০৪ আবার শুরু হয়।

### ধাপ ৫ — অগ্রগতি দেখুন

```bash
docker compose ps
docker compose logs -f app
docker compose logs -f db
```

লগে `ready for connections` দুইবার এলে MySQL প্রস্তুত (একবার init পর্যায়ে, একবার আসল স্টার্টে)। দুইটি ~৪.৪ MB ডাম্প ইমপোর্ট হতে এক-দুই মিনিট লাগে।

### ধাপ ৬ — অ্যাপ খুলুন

| ডেমো | স্টোরফ্রন্ট | অ্যাডমিন লগইন |
|---|---|---|
| `demo-default-production` | `http://localhost:8300` | `http://localhost:8300/login/admin` |
| `demo-aster-production` | `http://localhost:8301` | `http://localhost:8301/login/admin` |

লগইন তথ্য ইমপোর্ট করা ডেমো ডাম্প থেকে আসে (ডেমো মোডে লগইন স্ক্রিনেই দেখানো থাকে)।
লগইন না করা অবস্থায় `/admin` নিজে ৪০৪ দেয় — এটি ইচ্ছাকৃত, অ্যাডমিন middleware প্যানেলটি
রিডাইরেক্ট না করে লুকিয়ে রাখে।

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

কোড পরিবর্তন কার্যকর করতে রিবিল্ড **এবং** app ভলিউম মুছুন — যেই ব্রাঞ্চ বিল্ড করতে চান সেখান থেকে:

```bash
# default ডেমো
docker compose down
docker volume rm 6valley-default_6valley-default-data
docker compose up -d --build

# aster ডেমো
docker compose -p 6valley-aster down
docker volume rm 6valley-aster_6valley-aster-data
docker compose up -d --build
```

কোড ভলিউম মুছলে §৪ ধাপ ৪ এর `.env` ফিক্সও চলে যায় — আবার চালান, নাহলে প্রতিটি পেজ আবার ৪০৪ দেবে।

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
# default ডেমো
docker compose down && docker volume rm 6valley-default_mysql-data && docker compose up -d

# aster ডেমো
docker compose -p 6valley-aster down && docker volume rm 6valley-aster_mysql-data && docker compose up -d
```

দুইটি ডেমোর ডাম্প **আলাদা**, তাই প্রতিটি স্ট্যাক নিজের ব্রাঞ্চ থেকেই আবার ইমপোর্ট করতে হবে —
aster এর ডেটা default স্ট্যাকে আসবে না, উল্টোটাও নয়।

**খালি ডাটাবেসে কখনো `php artisan migrate` চালাবেন না।** প্রজেক্টের নিয়ম অনুযায়ী আগে SQL ডাম্প ইমপোর্ট করতে হবে — compose সেটআপ init স্ক্রিপ্ট দিয়ে এটি এমনিতেই করে। এর পরেই কেবল pending migration চালাতে `migrate` ব্যবহার করুন।

**`docker compose down -v` দুইটি ভলিউমই মুছে দেয়** — কোড ভলিউম *এবং* আপনার ডাটাবেস। এটি রিস্টার্ট নয়, সম্পূর্ণ রিসেট।

**`.dockerignore` বাদ দেয় `vendor/`, `node_modules/`, `.git/`, `.env`।** এগুলো ইমেজের ভেতরে তৈরি হয়; হোস্টের কপি ব্যবহৃত হবে না।

**পোর্ট কনফ্লিক্ট।** `8300` (default ডেমো) বা `8301` (aster ডেমো) ব্যস্ত থাকলে `.env` এ `APP_PORT` বদলে `docker compose up -d` দিন — রিবিল্ড লাগবে না, পোর্ট শুধু `webserver` সার্ভিসে প্রভাব ফেলে। সাথে **কন্টেইনারের** `.env` এ `APP_URL` ও মিলিয়ে দিন (§৪ ধাপ ৪), নাহলে জেনারেট হওয়া লিংক পুরোনো পোর্টেই যেতে থাকবে।

**Apple Silicon (M1–M4)।** `php:8.3-fpm` ও `mysql:8.0` দুইটিরই `arm64` ইমেজ আছে, তাই নেটিভভাবেই চলে। প্ল্যাটফর্ম এরর এলে ঐ সার্ভিসে `platform: linux/amd64` যোগ করুন — তবে এমুলেশনে ধীর চলবে।

## ৮. সমস্যা সমাধান

| সমস্যা | কারণ | সমাধান |
|---|---|---|
| **প্রতিটি পেজে ৪০৪** (`/`, `/login/admin`) কিন্তু CSS/ছবি ঠিকই আসে | `APP_HOST_DOMAIN` এখনো ডেমো ডোমেইনে সেট | §৪ ধাপ ৪ চালান |
| **CSS/JS আসছে না**, ব্রাউজার `https://localhost:8300` এ যাচ্ছে | `FORCE_HTTPS=true` — nginx এর পেছনে `127.0.0.1` ছাড়টি কাজ করে না | §৪ ধাপ ৪ চালান |
| রিবিল্ডের পর আবার ৪০৪ | কোড ভলিউম নতুন হওয়ায় baked `.env` ফিরে এসেছে | §৪ ধাপ ৪ আবার চালান |
| `dependency failed to start: container … is unhealthy` এবং লগে `MYSQL_USER="root" … cannot be used for the root user` | রুট `.env` এ `DB_USERNAME=root` (যেমন লোকাল MAMP এর তথ্য); compose সেটিই MySQL কে দেয় | রুট `.env` এ `DB_USERNAME=6valley_user` / `DB_PASSWORD=Te?tu2)gs4=q(oz?rT5!oz` দিন, অথবা `cp .env.example .env` |
| orphan ভলিউম `6valley-default_6valley-aster-data` | `.env` এ default `COMPOSE_PROJECT_NAME` থাকা অবস্থায় aster বিল্ড হয়েছে | checkout এর পরে `cp .env.example .env`; orphan টি `docker volume rm` দিয়ে মুছুন |
| aster এ default থিম দেখাচ্ছে | ভুল ব্রাঞ্চের `.env` দিয়ে বিল্ড হয়েছে | `git checkout demo-aster-production && cp .env.example .env` (`WEB_THEME=theme_aster`), তারপর রিবিল্ড |
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

যেই ডেমো রিসেট করছেন, তার ব্রাঞ্চ থেকে চালান।

```bash
# default ডেমো — demo-default-production থেকে
docker compose down -v                      # কন্টেইনার এবং দুইটি ভলিউমই মুছে ফেলে
docker image rm 6valley-default-app:latest
docker compose up -d --build

# aster ডেমো — demo-aster-production থেকে
docker compose -p 6valley-aster down -v
docker image rm 6valley-aster-app:latest
docker compose up -d --build
```

এরপর §৪ ধাপ ৪ এর `.env` ফিক্স আবার চালান।

## ১০. দুই ডেমো — সংক্ষিপ্ত রেফারেন্স

| কাজ | `demo-default-production` | `demo-aster-production` |
|---|---|---|
| বিল্ড | `git checkout demo-default-production && cp .env.example .env && docker compose up -d --build` | `git checkout demo-aster-production && cp .env.example .env && docker compose up -d --build` |
| খুলুন | `http://localhost:8300` | `http://localhost:8301` |
| লগ | `docker compose -p 6valley-default logs -f` | `docker compose -p 6valley-aster logs -f` |
| শেল | `docker compose -p 6valley-default exec app bash` | `docker compose -p 6valley-aster exec app bash` |
| বন্ধ | `docker compose -p 6valley-default stop` | `docker compose -p 6valley-aster stop` |
| DB রিসেট | `docker volume rm 6valley-default_mysql-data` | `docker volume rm 6valley-aster_mysql-data` |
| কোড রিসেট | `docker volume rm 6valley-default_6valley-default-data` | `docker volume rm 6valley-aster_6valley-aster-data` |

`-p <project>` দিয়ে কোন ব্রাঞ্চ চেকআউট আছে তা নির্বিশেষে নির্দিষ্ট স্ট্যাক টার্গেট করা যায়।
`-p` **ছাড়া** কমান্ডগুলো বর্তমান `.env` এর `COMPOSE_PROJECT_NAME` অনুযায়ী কাজ করে — এজন্যই
প্রতিবার checkout এর পরে `cp .env.example .env` চালানো জরুরি।


## Env File

> নিচেরটি **default** ডেমোর (`demo-default-production`) `.env`। **aster** ডেমোর জন্য
> `APP_NAME=6valley-aster`, `WEB_THEME=theme_aster`, `APP_PORT=8301`,
> `CONTAINER_NAME_PREFIX=6valley-aster`, `COMPOSE_PROJECT_NAME=6valley-aster`,
> `APP_URL=http://localhost:8301` — বাকি সব একই। সবচেয়ে নিরাপদ উপায়: ব্রাঞ্চ checkout করে
> `cp .env.example .env` চালানো।
>
> The block below is the **default** demo's `.env`. For the **aster** demo change
> `APP_NAME`, `WEB_THEME=theme_aster`, `APP_PORT=8301`, both `*_PREFIX` / `*_PROJECT_NAME`
> to `6valley-aster`, and `APP_URL=http://localhost:8301`. Simplest: checkout the branch
> and run `cp .env.example .env`.
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


## The post-build fix, in one place

Run after every build that recreates the code volume. See §4 Step 4 for why.
প্রতিবার কোড ভলিউম নতুন করে তৈরি হলে চালান — কারণ §৪ ধাপ ৪ এ আছে।

```bash
# default demo → http://localhost:8300
docker compose -p 6valley-default exec app sed -i \
  -e 's|^APP_HOST_DOMAIN=.*|APP_HOST_DOMAIN=|' \
  -e 's|^APP_HOST_BASE_DOMAIN=.*|APP_HOST_BASE_DOMAIN=|' \
  -e 's|^FORCE_HTTPS=.*|FORCE_HTTPS=false|' \
  -e 's|^APP_URL=.*|APP_URL=http://localhost:8300|' .env
docker compose -p 6valley-default exec app php artisan optimize:clear
docker compose -p 6valley-default restart app

# aster demo → http://localhost:8301
docker compose -p 6valley-aster exec app sed -i \
  -e 's|^APP_HOST_DOMAIN=.*|APP_HOST_DOMAIN=|' \
  -e 's|^APP_HOST_BASE_DOMAIN=.*|APP_HOST_BASE_DOMAIN=|' \
  -e 's|^FORCE_HTTPS=.*|FORCE_HTTPS=false|' \
  -e 's|^APP_URL=.*|APP_URL=http://localhost:8301|' .env
docker compose -p 6valley-aster exec app php artisan optimize:clear
docker compose -p 6valley-aster restart app
```

Check both are up / দুইটিই ঠিক আছে কিনা দেখুন:

```bash
curl -s -o /dev/null -w "8300 -> %{http_code}\n" http://localhost:8300/login/admin
curl -s -o /dev/null -w "8301 -> %{http_code}\n" http://localhost:8301/login/admin
# both should print 200
```
