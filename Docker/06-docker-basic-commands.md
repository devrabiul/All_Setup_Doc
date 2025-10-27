# 💻 Docker-এর মূল কমান্ডগুলো (Basic Commands)

Docker শিখতে গেলে সবচেয়ে গুরুত্বপূর্ণ হলো এর **কমান্ডগুলো ভালোভাবে বোঝা**।  
এই ডকুমেন্টে আমরা শেখব Docker-এর প্রাথমিক (basic) কমান্ডগুলোর কাজ,  
উদাহরণসহ ব্যাখ্যা এবং ছোট কিছু টিপস।

---

## 🧩 ১. Docker Version ও System তথ্য

### 🔹 Docker Version দেখার জন্য
```bash
docker --version
````

👉 Docker-এর বর্তমান ভার্সন দেখাবে।

### 🔹 Docker Info

```bash
docker info
```

👉 Docker Engine-এর বিস্তারিত তথ্য (containers, images, storage driver ইত্যাদি) দেখাবে।

---

## 🐳 ২. Docker Images সম্পর্কিত কমান্ড

### 🔹 লোকাল Image লিস্ট করা

```bash
docker images
```

👉 সিস্টেমে থাকা সব Docker image-এর তালিকা দেখাবে।

### 🔹 Registry থেকে Image ডাউনলোড করা

```bash
docker pull nginx
```

👉 Docker Hub থেকে `nginx` image নামিয়ে আনে।

### 🔹 Image মুছে ফেলা

```bash
docker rmi image_name
```

👉 নির্দিষ্ট image মুছে ফেলে।

### 🔹 অব্যবহৃত Image মুছে ফেলা (clean-up)

```bash
docker image prune
```

👉 যেসব image আর দরকার নেই সেগুলো ডিলিট করবে।

---

## 📦 ৩. Container তৈরি ও চালানো

### 🔹 নতুন Container চালানো

```bash
docker run hello-world
```

👉 “Hello World” টেস্ট container চালায়।

### 🔹 Detached Mode-এ চালানো (background-এ)

```bash
docker run -d nginx
```

### 🔹 Port Mapping সহ চালানো

```bash
docker run -d -p 8080:80 nginx
```

👉 Host-এর 8080 পোর্ট → Container-এর 80 পোর্টে ফরোয়ার্ড হবে।

### 🔹 Container-কে নাম দিয়ে চালানো

```bash
docker run -d --name my-nginx nginx
```

---

## 🔄 ৪. চলমান ও বন্ধ থাকা Container ম্যানেজ করা

### 🔹 চলমান Container লিস্ট করা

```bash
docker ps
```

### 🔹 সব Container (চলমান + বন্ধ) দেখানো

```bash
docker ps -a
```

### 🔹 Container বন্ধ করা

```bash
docker stop container_id
```

### 🔹 Container চালু করা

```bash
docker start container_id
```

### 🔹 Container মুছে ফেলা

```bash
docker rm container_id
```

### 🔹 সব Container একসাথে বন্ধ/মুছে ফেলা

```bash
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

---

## 🧠 ৫. Container-এ ঢোকা / কমান্ড চালানো

### 🔹 Container-এর ভিতরে ঢোকা (interactive)

```bash
docker exec -it container_id bash
```

👉 Container-এর shell-এ ঢুকতে পারবে।

### 🔹 Container-এর ভিতরে একবার কমান্ড চালানো

```bash
docker exec container_id ls
```

👉 শুধু ওই কমান্ডটি রান করবে, shell খোলবে না।

---

## 📜 ৬. Container লগ দেখা

### 🔹 Container-এর লগ দেখা

```bash
docker logs container_id
```

### 🔹 লাইভ লগ দেখা (follow mode)

```bash
docker logs -f container_id
```

---

## 🧱 ৭. Docker Build ও Custom Image

### 🔹 Dockerfile থেকে Image তৈরি করা

```bash
docker build -t myapp:1.0 .
```

> * `-t` = Tag (image name ও version)
> * `.` = বর্তমান ফোল্ডার (যেখানে Dockerfile আছে)

### 🔹 Image লিস্ট দেখা

```bash
docker images
```

### 🔹 Image Push করা (Docker Hub-এ)

```bash
docker push username/myapp:1.0
```

---

## 🌐 ৮. Docker Network

### 🔹 Network তৈরি করা

```bash
docker network create mynetwork
```

### 🔹 Network লিস্ট দেখা

```bash
docker network ls
```

### 🔹 Network inspect করা

```bash
docker network inspect mynetwork
```

### 🔹 Container Network-এ চালানো

```bash
docker run -d --network=mynetwork --name=myapp nginx
```

---

## 💾 ৯. Docker Volume

### 🔹 Volume তৈরি করা

```bash
docker volume create mydata
```

### 🔹 Volume লিস্ট দেখা

```bash
docker volume ls
```

### 🔹 Volume attach করে container চালানো

```bash
docker run -d -v mydata:/app/data nginx
```

### 🔹 Volume inspect করা

```bash
docker volume inspect mydata
```

---

## 🧹 ১০. Clean-up কমান্ডসমূহ

### 🔹 অব্যবহৃত সব কিছু মুছে ফেলা

```bash
docker system prune
```

### 🔹 আরও জোরালোভাবে সব image, container, volume মুছে ফেলা

```bash
docker system prune -a --volumes
```

> ⚠️ সাবধান: এটি সবকিছু (images, containers, volumes) মুছে দেবে!

---

## 🧭 ১১. Useful Combined Examples

### 🧱 Example 1 — nginx চালানো

```bash
docker run -d -p 8080:80 nginx
```

### 🐘 Example 2 — MySQL container চালানো

```bash
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=secret mysql:8
```

### 🐍 Example 3 — Python app চালানো

```bash
docker run -it --rm python:3.12 python
```

---

## 🧠 ১২. কমান্ড মনে রাখার ট্রিক

| মনে রাখার টিপ                | উদাহরণ                       |
| ---------------------------- | ---------------------------- |
| `run` → নতুন চালাও           | `docker run nginx`           |
| `ps` → চলমান container       | `docker ps`                  |
| `exec` → ভিতরে কমান্ড চালাও  | `docker exec -it nginx bash` |
| `logs` → লগ দেখো             | `docker logs nginx`          |
| `stop` / `start` → চালু/বন্ধ | `docker stop nginx`          |
| `rm` → মুছে ফেলো             | `docker rm nginx`            |

---

## 🧩 ১৩. Command Lifecycle Flow

```
docker pull → image নামাও
docker run → container চালাও
docker ps → দেখো চলমান container
docker exec → ভিতরে ঢুকো
docker stop → বন্ধ করো
docker rm → মুছে ফেলো
```

---

## ✅ সংক্ষেপে

| ক্যাটাগরি | গুরুত্বপূর্ণ কমান্ড                |
| --------- | ---------------------------------- |
| Info      | `docker --version`, `docker info`  |
| Image     | `pull`, `images`, `rmi`, `build`   |
| Container | `run`, `ps`, `stop`, `start`, `rm` |
| Exec/Logs | `exec`, `logs`                     |
| Volume    | `volume create`, `volume ls`       |
| Network   | `network create`, `network ls`     |
| Clean-up  | `system prune`                     |

---

## 💡 উপসংহার

Docker শেখার মূল চাবিকাঠি হলো **এই কমান্ডগুলো আয়ত্ত করা**।
প্রতিদিন ১০–১৫ মিনিট প্র্যাকটিস করলেই তুমি দ্রুত এগিয়ে যাবে।

> 🧠 মনে রাখো: “কমান্ড হাতে মানে Docker তোমার নিয়ন্ত্রণে!” 🐳