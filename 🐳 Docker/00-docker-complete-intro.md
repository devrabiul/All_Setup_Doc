# 🐳 Docker Complete Introduction (বাংলায়)

---

## ১️⃣ Docker কী?

**Docker** হলো একটি open-source platform যা ব্যবহার করা হয় **applications containerize করতে**।  
Container হলো একটি lightweight, isolated environment যেখানে তোমার app এবং তার সব dependencies চলে।  

> সহজভাবে: Docker হচ্ছে “app + environment” এক জায়গায় bundle করার tool।  
> ফলে তুমি একই app যে কোনো machine-এ চালাতে পারো, same behavior সহ।  

---

## ২️⃣ Docker কেন ব্যবহার করতে হবে?

Docker ব্যবহারের মূল সুবিধা:

1. **Consistency across environments**  
   - Developer machine → Testing → Production  
   - সব জায়গায় app একইভাবে চলে  

2. **Lightweight & Fast**  
   - Virtual Machine এর চেয়ে অনেক কম resource ব্যবহার করে  
   - Quick startup  

3. **Isolation**  
   - App আলাদা container-এ চলে, host বা অন্য container interfere করে না  

4. **Portability**  
   - Container যেকোনো OS-এ চলে যেখানে Docker support আছে  

5. **Microservices friendly**  
   - ছোট ছোট service আলাদা container-এ চালানো যায়  

---

## ৩️⃣ Docker vs Virtual Machine (VM)

| Feature | Docker | VM |
|---------|--------|----|
| Size | Lightweight | Heavyweight |
| Boot Time | Seconds | Minutes |
| Resource Usage | Low | High |
| Isolation | Process-level | OS-level |
| Use Case | Microservices, Dev/Test | Legacy apps, Full OS |
| Image | Docker Image | VM Image (Full OS) |

### Resource requirement comparison

| Environment | RAM | CPU | Storage |
|-------------|-----|-----|--------|
| Docker | 512MB-2GB | 1 CPU | ~1-2GB per image |
| VM | 2-8GB | 2+ CPU | 10+GB per VM |

> অর্থাৎ, Docker কম resource ব্যবহার করে, দ্রুত start হয় এবং lightweight।  

---

## ৪️⃣ Docker-এর মূল Concept

### ৪.১ Container

- Container হলো **running instance of a Docker image**  
- Container isolated, lightweight, fast  
- Stop/Delete করলে container gone, কিন্তু volume থাকলে data safe  

**Example:**
```bash
docker run -d -p 8080:80 nginx
````

---

### ৪.২ Image

* Image হলো **read-only template**, যেটি থেকে container তৈরি হয়
* Multiple containers একই image থেকে তৈরি করা যায়
* Image build করা যায় Dockerfile দিয়ে

**Example:**

```bash
docker build -t myapp:1.0 .
```

---

### ৪.৩ Docker Engine

* Docker Engine হলো **Docker run-time environment**
* এটি ৩টি অংশ নিয়ে গঠিত:

    1. **Server (dockerd)** → background service, container run করে
    2. **REST API** → communication interface
    3. **CLI (docker command)** → user interface

**Docker Engine workflow:**

```
Docker CLI → Docker REST API → Docker Daemon → Container/Image
```

---

### ৪.৪ Volume

* Container-এর data safe রাখার জন্য Volume ব্যবহার হয়
* Container delete হলেও Volume data থেকে যায়

**Example:**

```bash
docker run -v mydata:/var/lib/mysql mysql
```

---

### ৪.৫ Network

* Container isolation কিন্তু একে অপরের সাথে communicate করতে পারে custom network দিয়ে
* Default bridge network বা user-defined network ব্যবহার করা যায়

---

## ৫️⃣ Docker Architecture Summary

* **Images** → Templates
* **Containers** → Running apps
* **Docker Engine** → Runtime
* **Volumes** → Persistent storage
* **Network** → Communication

### Diagram (ASCII style)

```
+-------------------------+
|      Docker CLI         |
+-------------------------+
            |
            v
+-------------------------+
|      Docker Daemon      |
|  (Engine / dockerd)     |
+-------------------------+
      |           |
      v           v
  +-------+   +--------+
  | Image |   | Volume |
  +-------+   +--------+
      |
      v
  +-------------+
  | Container   |
  +-------------+
```

---

## ৬️⃣ Resource Planning

* Docker কম resource use করে, তবে large apps বা multiple containers হলে:

    * **RAM:** 1-4GB minimum recommended
    * **CPU:** 1-2 core minimum
    * **Storage:** 5GB+ for images & volumes

* VM তুলনায় কম resource লাগে, বেশি scalable

---

## ৭️⃣ Quick Commands Overview

| Task             | Command                          |
| ---------------- | -------------------------------- |
| List containers  | `docker ps -a`                   |
| Run container    | `docker run -d -p 8080:80 nginx` |
| Stop container   | `docker stop container_name`     |
| Remove container | `docker rm container_name`       |
| List images      | `docker images`                  |
| Build image      | `docker build -t myapp:1.0 .`    |
| Remove image     | `docker rmi image_name`          |

---

## ✅ উপসংহার

Docker হলো lightweight, portable, fast, এবং consistent runtime environment।

* VM এর চেয়ে resource কম লাগে
* Container + Image + Engine + Volume + Network মিলেই Docker workflow complete হয়
* Modern development এবং production-ready apps-এর জন্য Docker অপরিহার্য

> 🐳 “Docker = Portable + Lightweight + Isolated + Consistent Application Environment”