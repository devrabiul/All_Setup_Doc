# ⚙️ Docker Images এবং Containers ম্যানেজমেন্ট

---

## 📦 ১️⃣ Docker Image কীভাবে কাজ করে?

একটি **Docker Image** হচ্ছে read-only template,  
যেটি থেকে container তৈরি হয়।  
প্রতিটি image অনেকগুলো **layer** নিয়ে তৈরি হয় — যেমন:
- OS layer
- Runtime (PHP, Node, Python ইত্যাদি)
- App files
- Configuration

👉 যখন তুমি `docker run` দাও, Docker এই image থেকে একটি **container** তৈরি করে।

---

## 🧱 ২️⃣ Image লিস্ট দেখা

```bash
docker images
````

📋 আউটপুট:

```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    c316d5a335a5   2 days ago      142MB
mysql        8         6b5d55b3c1b4   1 week ago      500MB
```

---

## 🧩 ৩️⃣ Image ডাউনলোড (Pull)

```bash
docker pull nginx
```

👉 Docker Hub (বা private registry) থেকে image নামিয়ে আনে।

নির্দিষ্ট ভার্সন চাইলে:

```bash
docker pull mysql:8.0.40
```

---

## 🏷️ ৪️⃣ Image Tagging (Rename/Versioning)

Tag দেওয়া মানে image-কে version বা নাম দেওয়া।

```bash
docker tag myapp:latest myusername/myapp:v1.0
```

👉 এতে image সহজে সনাক্ত করা যায় ও push করা যায় Docker Hub-এ।

---

## ☁️ ৫️⃣ Image Push করা (Docker Hub-এ)

```bash
docker push myusername/myapp:v1.0
```

> 🔐 আগে লগইন করতে হবে:
>
> ```bash
> docker login
> ```

---

## 🧹 ৬️⃣ অব্যবহৃত Image পরিষ্কার করা

### 🔹 একক image ডিলিট করা

```bash
docker rmi image_id
```

### 🔹 অব্যবহৃত (dangling) image মুছে ফেলা

```bash
docker image prune
```

### 🔹 সব অপ্রয়োজনীয় image মুছে ফেলা

```bash
docker image prune -a
```

---

## 🧠 ৭️⃣ Docker Container — Overview

Container হলো চলমান instance of an image.
প্রত্যেক container image থেকে তৈরি হয় এবং run-time environment দেয়।

---

## 🏃 ৮️⃣ Container চালানো

```bash
docker run -d -p 8080:80 nginx
```

### ব্যাখ্যা:

* `-d` → detached mode
* `-p` → port mapping
* `nginx` → image name

---

## 🔍 ৯️⃣ Container লিস্ট দেখা

### 🔹 চলমান container:

```bash
docker ps
```

### 🔹 সব container (চলমান + বন্ধ):

```bash
docker ps -a
```

---

## ⏸️ 🔁 🔥 ১০️⃣ Container Control

| কাজ            | কমান্ড                        |
| -------------- | ----------------------------- |
| Container বন্ধ | `docker stop container_id`    |
| Container চালু | `docker start container_id`   |
| Restart        | `docker restart container_id` |
| Remove         | `docker rm container_id`      |
| Force remove   | `docker rm -f container_id`   |

---

## 📜 ১১️⃣ Container লগ দেখা

```bash
docker logs container_id
```

লাইভ লগ দেখতে:

```bash
docker logs -f container_id
```

---

## 🧠 ১২️⃣ Container-এর ভিতরে ঢোকা

```bash
docker exec -it container_id bash
```

অথবা shell না থাকলে:

```bash
docker exec -it container_id sh
```

---

## 📦 ১৩️⃣ Container Inspect করা

```bash
docker inspect container_id
```

👉 এটি JSON ফরম্যাটে container-এর বিস্তারিত তথ্য দেয়:
IP, Volume, Network, Mounts ইত্যাদি।

---

## 🧰 ১৪️⃣ Useful Combined Commands

### 🔹 সব container বন্ধ ও ডিলিট করা

```bash
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

### 🔹 সব image ডিলিট করা

```bash
docker rmi $(docker images -q)
```

---

## 🧹 ১৫️⃣ Full Clean-up (System-level)

### 🔹 সব অব্যবহৃত জিনিস ডিলিট করা

```bash
docker system prune
```

### 🔹 জোরপূর্বক সব কিছু মুছে ফেলা (images + containers + volumes)

```bash
docker system prune -a --volumes
```

⚠️ সাবধান — এটা পুরো সিস্টেম clean করে ফেলবে।

---

## 📊 ১৬️⃣ Resource ব্যবহার দেখা

```bash
docker stats
```

👉 Live CPU, Memory, Network usage দেখাবে।

---

## 🧱 ১৭️⃣ Container Rename / Copy

### Rename container:

```bash
docker rename old_name new_name
```

### Container থেকে ফাইল কপি করা:

```bash
docker cp container_id:/path/in/container ./local_folder
```

---

## 🧠 ১৮️⃣ Container Commit (Custom Image তৈরি করা)

Container থেকে নতুন image বানাতে পারো:

```bash
docker commit container_id myapp:v2
```

👉 এতে container-এর বর্তমান অবস্থা একটি image হিসেবে সংরক্ষিত হবে।

---

## 🧾 ১৯️⃣ Container Inspect + Logs + Network একত্রে বোঝা

একটি সার্ভিস বিশ্লেষণের সময় এই ৩টি কমান্ড অপরিহার্য:

```bash
docker inspect container_id
docker logs container_id
docker exec -it container_id bash
```

👉 এগুলোর মাধ্যমে বোঝা যায় container সঠিকভাবে চলছে কিনা।

---

## 🧭 ২০️⃣ Lifecycle Flow (চক্রাকারে বোঝো)

```
docker pull → docker run → docker ps → docker exec → docker stop → docker rm → docker system prune
```

---

## 🧱 ২১️⃣ Advanced: Container Backup ও Restore

### Backup Container-এর volume

```bash
docker run --rm --volumes-from mycontainer -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /data
```

### Restore

```bash
docker run --rm --volumes-from newcontainer -v $(pwd):/backup ubuntu tar xvf /backup/backup.tar
```

---

## 💡 Pro Tips

✅ **Image ছোট রাখো:** Alpine base ব্যবহার করো
✅ **Tag ব্যবহার করো:** Version ট্র্যাক করা সহজ হয়
✅ **.dockerignore** ফাইল রাখো
✅ **Prune নিয়মিত চালাও:** Storage clean থাকে
✅ **Inspect করে বোঝো:** Problem solve করা সহজ হয়

---

## ✅ উপসংহার

Docker শেখার পর সবচেয়ে গুরুত্বপূর্ণ হলো — **ম্যানেজ করতে জানা**।
Image আর Container ঠিকভাবে ম্যানেজ করতে পারলেই তুমি পুরো system automation করতে পারবে।

> 🧠 মনে রাখো: “Docker চালানো সহজ, ম্যানেজ করা কৌশল।” ⚙️