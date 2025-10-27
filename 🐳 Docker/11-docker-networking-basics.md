# 🌐 Docker Networking Basics

---

## 🧠 ১️⃣ Docker Networking কেন দরকার?

Docker container গুলো সাধারণত isolated থাকে।  
কোনো container অন্য container বা host-এর সাথে যোগাযোগ করতে চাইলে **networking setup করতে হয়**।  

উদাহরণ:
- Laravel container → MySQL container  
- Web server container → Redis container  

---

## 🏗️ ২️⃣ Docker Networking-এর ধরন

Docker মূলত ৪ ধরনের network দেয়:

| Network Type | Description | Use Case |
|--------------|------------|----------|
| bridge | Default network, single host | Simple container communication |
| host | Container host-এর network ব্যবহার করে | High-performance, no isolation |
| none | No network | Isolation-only container |
| overlay | Multi-host network (Swarm mode) | Microservices across hosts |

---

## 🌉 ৩️⃣ Default Bridge Network

- প্রতিটি container default `bridge` network-এ চলে।  
- Container IP random দেওয়া হয়।  
- অন্য container access করতে `--link` ব্যবহার করা যায় (deprecated)।  

### উদাহরণ:
```bash
docker run -d --name web nginx
docker run -it --rm --link web:web busybox ping web
````

---

## 🧩 ৪️⃣ Custom Bridge Network তৈরি করা

Custom network তৈরি করলে container সহজে একে অপরকে নাম দিয়ে access করতে পারে।

```bash
docker network create my-network
```

লিস্ট দেখতে:

```bash
docker network ls
```

---

## 🔗 ৫️⃣ Container কাস্টম Network-এ attach করা

```bash
docker run -d --name db --network my-network mysql:8
docker run -d --name app --network my-network myapp
```

👉 এখন `app` container সহজেই `db` container-কে নাম দিয়ে access করতে পারবে:

```bash
ping db
```

---

## 🔍 ৬️⃣ Network Inspect করা

```bash
docker network inspect my-network
```

📋 আউটপুটে দেখাবে কোন container কোন IP address পেয়েছে।

---

## 🧱 ৭️⃣ Container একে অপরের সাথে কথা বলার উদাহরণ

* Laravel container → MySQL container

`docker-compose.yml` উদাহরণ:

```yaml
version: '3.8'
services:
  app:
    image: laravelphp/php-fpm
    networks:
      - appnet

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
    networks:
      - appnet

networks:
  appnet:
```

👉 `app` container থেকে host হিসেবে `db` ব্যবহার করা যায়।

```env
DB_HOST=db
```

---

## 🔄 ৮️⃣ Container Network Management

### Disconnect container

```bash
docker network disconnect my-network mycontainer
```

### Connect container

```bash
docker network connect my-network mycontainer
```

### Remove network

```bash
docker network rm my-network
```

⚠️ একটি network থেকে সব container remove করতে হবে আগে।

---

## 🔐 ৯️⃣ Host Network ব্যবহার করা

```bash
docker run --network host -d nginx
```

👉 Container host-এর network ব্যবহার করে।

* Zero network isolation
* Performance boost

---

## 🌐 🔟 Overlay Network (Swarm mode)

* Multi-host container communication
* Microservices architecture-এর জন্য দরকার

উদাহরণ (Swarm mode):

```bash
docker swarm init
docker network create -d overlay my-overlay
```

---

## 🧠 ১১️⃣ Quick Tips

✅ Container নাম দিয়ে connect করা সহজ
✅ Custom network দিয়ে alias ব্যবহার করতে পারো
✅ Host network high-performance দরকার হলে ব্যবহার করা যায়
✅ Overlay network multi-host deployment-এর জন্য

---

## 💡 ১২️⃣ Cheatsheet

| কাজ                  | কমান্ড                                      |
| -------------------- | ------------------------------------------- |
| Network list         | `docker network ls`                         |
| Network create       | `docker network create mynet`               |
| Network inspect      | `docker network inspect mynet`              |
| Connect container    | `docker network connect mynet container`    |
| Disconnect container | `docker network disconnect mynet container` |
| Remove network       | `docker network rm mynet`                   |

---

## ✅ উপসংহার

Docker Networking হলো container-to-container এবং host-to-container communication-এর মূল।

* Bridge → Single host
* Host → Performance
* Overlay → Multi-host

> “Networking ঠিক থাকলে, container একসাথে seamless কাজ করে।” 🌐