# 💾 Docker Volumes & Data Persistence

---

## 🧠 ১️⃣ কেন দরকার Docker Volume?

Docker container হলো **temporary environment**।  
অর্থাৎ, container বন্ধ বা মুছে ফেললে তার ভিতরের data-ও মুছে যায়।

🧩 উদাহরণ:
```bash
docker run -d --name test-mysql mysql:8
````

যদি container মুছে দাও 👇

```bash
docker rm -f test-mysql
```

👉 Database-এর সব data হারিয়ে যাবে।

⚠️ তাই — **Persistent Storage** দরকার, যাতে container delete হলেও data থেকে যায়।

---

## 📦 ২️⃣ Docker Volume কী?

Docker Volume হলো container-এর বাইরে stored data-এর জায়গা,
যা Docker নিজেই ম্যানেজ করে।

🔹 এটি host machine-এ থাকে
🔹 container শুধু access করে
🔹 container delete হলেও volume থাকে

---

## 🗂️ ৩️⃣ Volume তৈরি করা

```bash
docker volume create mydata
```

দেখতে:

```bash
docker volume ls
```

📋 আউটপুট:

```
DRIVER    VOLUME NAME
local     mydata
```

---

## 🏗️ ৪️⃣ Volume ব্যবহার করা (Attach করা)

```bash
docker run -d \
  --name mymysql \
  -v mydata:/var/lib/mysql \
  mysql:8
```

🧩 এখানে:

* `-v mydata:/var/lib/mysql` → `mydata` volume টি container-এর `/var/lib/mysql` path-এ মাউন্ট হচ্ছে।

👉 ফলে container মুছে ফেললেও database ফাইল থাকবে।

---

## 🔍 ৫️⃣ Volume Inspect করা

```bash
docker volume inspect mydata
```

📋 আউটপুটে দেখাবে কোথায় data আসলে stored আছে:

```
"Mountpoint": "/var/lib/docker/volumes/mydata/_data"
```

---

## 🧱 ৬️⃣ Container Delete করলেও Data থাকবে

```bash
docker rm -f mymysql
```

এখন volume চেক করো:

```bash
docker volume ls
```

👉 `mydata` এখনো আছে।

---

## 🔄 ৭️⃣ একই Data নতুন Container-এ ব্যবহার করা

```bash
docker run -d \
  --name newmysql \
  -v mydata:/var/lib/mysql \
  mysql:8
```

🔹 পুরানো data এখানেও থাকবে!
✅ অর্থাৎ **Data Persistent**.

---

## 💡 ৮️⃣ Bind Mount বনাম Volume

| বিষয়        | Volume                      | Bind Mount                     |
| ----------- | --------------------------- | ------------------------------ |
| Location    | Docker দ্বারা ম্যানেজ করা   | Host machine-এর নির্দিষ্ট path |
| Backup      | সহজ                         | manual                         |
| Performance | ভালো                        | ফাইলসিস্টেমের উপর নির্ভরশীল    |
| Security    | নিরাপদ (Docker control করে) | সরাসরি access                  |

---

## 🧩 ৯️⃣ Bind Mount উদাহরণ

```bash
docker run -d \
  --name webapp \
  -v $(pwd)/app:/usr/share/nginx/html \
  nginx
```

🔹 এখানে host-এর `app` ফোল্ডার container-এর ভিতরের `/usr/share/nginx/html` হিসেবে কাজ করছে।

> ✅ Bind Mount developer-friendly — live code changes container-এ reflect হয়।

---

## 🧰 🔟 সব Volume দেখা ও ম্যানেজ করা

### লিস্ট:

```bash
docker volume ls
```

### Remove একক Volume:

```bash
docker volume rm mydata
```

### Remove সব অপ্রয়োজনীয় Volume:

```bash
docker volume prune
```

⚠️ সতর্কতা: prune করলে unused সব volume ডিলিট হয়ে যাবে।

---

## 🧱 ১১️⃣ Docker Compose-এ Volume ব্যবহার

`docker-compose.yml` উদাহরণ 👇

```yaml
version: "3.8"
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - dbdata:/var/lib/mysql

volumes:
  dbdata:
```

🔹 এখানে `dbdata` volume Docker নিজে তৈরি করবে।
🔹 container restart বা rebuild করলেও data থাকবে।

---

## 💾 ১২️⃣ Named vs Anonymous Volumes

| ধরন              | Example           | Description                                               |
| ---------------- | ----------------- | --------------------------------------------------------- |
| Named Volume     | `-v mydata:/data` | নাম দেওয়া volume, পুনরায় ব্যবহারযোগ্য                     |
| Anonymous Volume | `-v /data`        | random নাম দেওয়া volume, container delete করলে হারিয়ে যায় |

👉 তাই, production-এ সর্বদা **named volume** ব্যবহার করো।

---

## 🧠 ১৩️⃣ Volume Inspect করে Data দেখা

Volume path খুঁজে নিয়ে manual ফাইল দেখার উদাহরণ:

```bash
docker volume inspect mydata
# Path: /var/lib/docker/volumes/mydata/_data
cd /var/lib/docker/volumes/mydata/_data
ls
```

---

## 📦 ১৪️⃣ Volume Backup & Restore

### 🔹 Backup:

```bash
docker run --rm \
  -v mydata:/volume \
  -v $(pwd):/backup \
  busybox tar cvf /backup/mydata-backup.tar /volume
```

### 🔹 Restore:

```bash
docker run --rm \
  -v mydata:/volume \
  -v $(pwd):/backup \
  busybox tar xvf /backup/mydata-backup.tar
```

---

## 🧹 ১৫️⃣ Cleanup Tips

✅ নিয়মিত চালাও:

```bash
docker system prune
docker volume prune
```

✅ পুরনো volume delete করার আগে backup রাখো।
✅ named volume ব্যবহার করলে সহজে ম্যানেজ করা যায়।

---

## 🧾 ১৬️⃣ Quick Reference Cheatsheet

| কাজ                 | কমান্ড                       |
| ------------------- | ---------------------------- |
| Volume তৈরি         | `docker volume create name`  |
| Volume লিস্ট        | `docker volume ls`           |
| Volume inspect      | `docker volume inspect name` |
| Volume ডিলিট        | `docker volume rm name`      |
| Unused Volume ডিলিট | `docker volume prune`        |
| Compose-এ Volume    | `volumes: - name:/path`      |

---

## ✅ উপসংহার

Docker-এর শক্তি শুধু container-এ না,
বরং **data নিরাপদে সংরক্ষণে (persistence)**।

> “Container আসে যায়, কিন্তু Volume থাকে।” 💾