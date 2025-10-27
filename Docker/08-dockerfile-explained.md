# 🧾 Dockerfile Explained — কাস্টম Docker Image তৈরির গাইড

---

## 🧠 Dockerfile কী?

**Dockerfile** হলো একটি সাধারণ **টেক্সট ফাইল**, যেখানে নির্দেশনা (instructions) লেখা থাকে  
কীভাবে একটি Docker image তৈরি হবে।

> সহজভাবে বললে — Dockerfile হচ্ছে **image তৈরির রেসিপি** 🍲  
> যেমন রান্না করতে রেসিপি লাগে, তেমনি Docker image বানাতে লাগে Dockerfile।

---

## 🧩 Dockerfile-এর কাজ

- Custom image তৈরি করে (নিজস্ব অ্যাপসহ)
- নির্দিষ্ট environment সেটআপ করে
- ডিপেন্ডেন্সি, প্যাকেজ, কোড সব ইনস্টল করে
- image build করার সময় সবকিছু automate করে

---

## 📂 Dockerfile কোথায় থাকে?

সাধারণত তোমার প্রজেক্টের root ডিরেক্টরিতে থাকে:

```

my-app/
├── Dockerfile
├── docker-compose.yml
├── app/
└── requirements.txt

````

---

## 🧱 Dockerfile-এর মৌলিক Structure

একটি Dockerfile সাধারণত এমনভাবে লেখা হয়:

```dockerfile
# 1️⃣ কোন base image থেকে শুরু হবে
FROM ubuntu:22.04

# 2️⃣ author তথ্য (ঐচ্ছিক)
LABEL maintainer="yourname@example.com"

# 3️⃣ প্রয়োজনীয় প্যাকেজ ইনস্টল
RUN apt-get update && apt-get install -y nginx

# 4️⃣ কাজের ডিরেক্টরি নির্ধারণ
WORKDIR /var/www/html

# 5️⃣ ফাইল কপি করা
COPY . .

# 6️⃣ পোর্ট ওপেন করা
EXPOSE 80

# 7️⃣ container শুরু হলে কোন কমান্ড চলবে
CMD ["nginx", "-g", "daemon off;"]
````

---

## 🔍 ব্যাখ্যা

| নির্দেশনা    | কাজ                                                   |
| ------------ | ----------------------------------------------------- |
| `FROM`       | কোন base image থেকে শুরু হবে (যেমন ubuntu, node, php) |
| `LABEL`      | meta তথ্য, যেমন author বা version                     |
| `RUN`        | container build সময় চালানো কমান্ড                     |
| `WORKDIR`    | container-এর ভিতরে কাজের ফোল্ডার                      |
| `COPY`       | লোকাল ফাইল container-এ কপি করা                        |
| `ADD`        | ফাইল কপি + URL বা tar ফাইল extract করতে পারে          |
| `EXPOSE`     | কোন পোর্ট ওপেন থাকবে তা জানায়                         |
| `CMD`        | container চালু হলে ডিফল্ট কমান্ড                      |
| `ENTRYPOINT` | CMD-এর মতো, তবে override করা যায় না                   |
| `ENV`        | environment variable সেট করা                          |
| `ARG`        | build সময় variable পাঠানো যায়                         |

---

## 🧠 `CMD` বনাম `ENTRYPOINT`

| পার্থক্য      | CMD                                  | ENTRYPOINT             |
| ------------- | ------------------------------------ | ---------------------- |
| কাজ           | Default command                      | Mandatory command      |
| পরিবর্তনযোগ্য | সহজে override করা যায়                | override কঠিন          |
| উদাহরণ        | `CMD ["nginx", "-g", "daemon off;"]` | `ENTRYPOINT ["nginx"]` |

---

## 🧩 Example 1: Node.js App-এর জন্য Dockerfile

```dockerfile
# 1️⃣ Node base image
FROM node:18

# 2️⃣ working directory
WORKDIR /usr/src/app

# 3️⃣ package.json কপি ও ইনস্টল
COPY package*.json ./
RUN npm install

# 4️⃣ সব কোড কপি
COPY . .

# 5️⃣ পোর্ট 3000 ওপেন
EXPOSE 3000

# 6️⃣ অ্যাপ চালানো
CMD ["npm", "start"]
```

**চালানোর ধাপ:**

```bash
docker build -t my-node-app .
docker run -d -p 3000:3000 my-node-app
```

---

## 🧩 Example 2: PHP + Apache (Laravel বা Custom PHP) App

```dockerfile
FROM php:8.2-apache

# Laravel বা অন্যান্য PHP extension ইনস্টল
RUN docker-php-ext-install pdo pdo_mysql

# সোর্স কোড কপি
COPY . /var/www/html

# Apache config fix
WORKDIR /var/www/html
EXPOSE 80

CMD ["apache2-foreground"]
```

---

## 🧩 Example 3: Python Flask App

```dockerfile
FROM python:3.12

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

EXPOSE 5000
CMD ["python", "app.py"]
```

---

## 🧩 Example 4: Multi-stage Build (Optimized Image)

```dockerfile
# Stage 1 — Build
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Stage 2 — Serve
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

👉 এতে final image ছোট হয় কারণ build tools বাদ যায়।

---

## 🧠 Build করার কমান্ড

```bash
docker build -t myapp:1.0 .
```

> `-t` = tag name
> `.` = বর্তমান ডিরেক্টরি (যেখানে Dockerfile আছে)

---

## 🔍 Build Logs দেখা

Build করার সময় Docker প্রতিটি লেয়ার (layer) আলাদা করে cache করে রাখে।

যেমন:

```
Step 1/7 : FROM node:18
Step 2/7 : WORKDIR /app
Step 3/7 : COPY package*.json ./
Step 4/7 : RUN npm install
...
```

👉 যদি কিছু না বদলাও, Docker cache ব্যবহার করবে এবং build দ্রুত হবে।

---

## 🧱 Docker Image তৈরি ও চালানো

```bash
# Build image
docker build -t myapp .

# Run container
docker run -d -p 8080:80 myapp
```

---

## ⚡ Dockerfile Best Practices

1. ✅ কম লেয়ার রাখো (RUN কম্বাইন করো)
2. ✅ `.dockerignore` ব্যবহার করো (অপ্রয়োজনীয় ফাইল বাদ দিতে)
3. ✅ ছোট base image ব্যবহার করো (যেমন `alpine`)
4. ✅ ENV ও ARG ব্যবহার করে flexible রাখো
5. ✅ Multi-stage build ব্যবহার করো
6. ✅ পোর্ট স্পষ্টভাবে EXPOSE করো
7. ✅ Root user ব্যবহার না করে `USER` সেট করো

---

## 🧩 .dockerignore উদাহরণ

`.dockerignore` ফাইল দিয়ে build context থেকে অপ্রয়োজনীয় ফাইল বাদ দিতে পারো:

```
node_modules
.git
.env
*.log
```

---

## 🧠 Dockerfile Lifecycle Summary

```
Dockerfile → docker build → Docker Image → docker run → Container
```

---

## ✅ উপসংহার

**Dockerfile** শেখা মানে তোমার নিজের custom environment তৈরি করার ক্ষমতা অর্জন।
এটা Docker শেখার backbone।

> 🧠 মনে রাখো: “Dockerfile ছাড়া Docker incomplete।” 🐳