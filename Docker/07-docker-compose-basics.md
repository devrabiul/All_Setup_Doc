# 🧱 Docker Compose পরিচিতি

## 📖 Docker Compose কী?

**Docker Compose** হলো একটি টুল যা দিয়ে একাধিক container (যেমন Laravel, MySQL, Redis, Nginx) একসাথে **একটি configuration file** দিয়ে চালানো যায়।

👉 এটি সাধারণত **`docker-compose.yml`** নামে একটি YAML ফাইলে configuration রাখে।

---

## 💡 কেন Docker Compose দরকার?

যদি তোমার প্রজেক্টে একাধিক সার্ভিস থাকে (যেমন):

- Laravel (PHP container)
- MySQL (Database container)
- Redis (Cache container)
- Nginx (Web server container)

তাহলে প্রতিটি container আলাদা করে চালাতে হবে অনেকগুলো কমান্ড দিয়ে।

যেমন:
```bash
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=secret mysql:8
docker run -d --name nginx -p 8080:80 nginx
docker run -d --name app --link mysql:mysql php:8.2
````

😓 এটা অনেক ঝামেলার কাজ!

**Docker Compose** দিয়ে তুমি শুধু একটি ফাইল লেখো, তারপর একটি কমান্ড দাও:

```bash
docker compose up -d
```

এবং সব container একসাথে চলবে!

---

## ⚙️ Docker Compose-এর স্থাপত্য (Architecture)

Docker Compose তিনটি ধাপে কাজ করে:

1. **Define (সংজ্ঞায়িত করা)** → YAML ফাইলে সার্ভিস, নেটওয়ার্ক ও ভলিউম নির্ধারণ করা হয়।
2. **Build (নির্মাণ করা)** → প্রয়োজনীয় image তৈরি বা pull করা হয়।
3. **Run (চালানো)** → একসাথে সব container চালানো হয়।

---

## 🧩 একটি সাধারণ docker-compose.yml উদাহরণ

ধরো তুমি Laravel + MySQL চালাতে চাও:

```yaml
version: '3.8'

services:
  app:
    image: laravelphp/php-fpm
    container_name: laravel_app
    ports:
      - "8000:8000"
    volumes:
      - ./:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:8
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

---

## 🔍 ব্যাখ্যা

| কী               | অর্থ                                                |
| ---------------- | --------------------------------------------------- |
| `version`        | Docker Compose ফাইলের সংস্করণ                       |
| `services`       | অ্যাপ্লিকেশনের বিভিন্ন অংশ (app, db, redis ইত্যাদি) |
| `image`          | কোন Docker image ব্যবহার হবে                        |
| `container_name` | Container-এর নাম                                    |
| `ports`          | Host ও Container পোর্ট ম্যাপ                        |
| `volumes`        | লোকাল ফোল্ডার container-এ মাউন্ট করা                |
| `environment`    | environment variables (যেমন পাসওয়ার্ড)              |
| `depends_on`     | কোন সার্ভিস আগে চালু হবে তা নির্ধারণ করে            |

---

## ⚡ সাধারণ Docker Compose কমান্ডসমূহ

| কমান্ড                   | কাজ                                 |
| ------------------------ | ----------------------------------- |
| `docker compose up`      | ফাইল অনুযায়ী সব container চালানো    |
| `docker compose up -d`   | Background-এ চালানো (detached mode) |
| `docker compose down`    | সব container বন্ধ ও মুছে ফেলা       |
| `docker compose ps`      | চলমান সার্ভিস লিস্ট                 |
| `docker compose logs`    | লগ দেখা                             |
| `docker compose restart` | সার্ভিস রিস্টার্ট করা               |
| `docker compose build`   | Dockerfile থেকে image build করা     |
| `docker compose stop`    | সার্ভিস বন্ধ করা                    |

---

## 🧱 উদাহরণ: Laravel Development Setup

**প্রজেক্ট স্ট্রাকচার:**

```
laravel-project/
├── docker-compose.yml
├── app/
│   └── (Laravel code)
```

**docker-compose.yml**

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: laravel_app
    ports:
      - "8000:8000"
    volumes:
      - ./app:/var/www/html
    depends_on:
      - mysql

  mysql:
    image: mysql:8
    container_name: mysql_db
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

এখন চালাও:

```bash
docker compose up -d
```

👉 Laravel ও MySQL একসাথে চালু হবে।

---

## 🧠 কিভাবে Docker Compose প্রজেক্ট হ্যান্ডেল করে

1. **Configuration পড়া** → `docker-compose.yml`
2. **Network তৈরি** → স্বয়ংক্রিয়ভাবে network তৈরি করে
3. **Container চালানো** → সব সার্ভিস parallel-এ চালু করে
4. **Lifecycle manage** → বন্ধ, restart, rebuild — সব একটি কমান্ডেই

---

## 🧹 Clean-up

সব সার্ভিস, নেটওয়ার্ক ও ভলিউম বন্ধ ও মুছে ফেলতে:

```bash
docker compose down -v
```

---

## 💬 Real-Life Use Cases

* Laravel বা Django ডেভেলপমেন্ট environment তৈরি
* WordPress + MySQL সেটআপ
* Redis, Nginx, PHP একসাথে orchestration
* Microservices অ্যাপ্লিকেশন

---

## ✅ উপসংহার

Docker Compose হলো **multi-container management-এর backbone**।
এটি তোমাকে সময় বাঁচায়, কনফিগারেশন সহজ করে এবং consistency বজায় রাখে।

> 🧠 মনে রাখো: “একটা `docker-compose.yml`-ই তোমার পুরো সার্ভার এনভায়রনমেন্ট!”