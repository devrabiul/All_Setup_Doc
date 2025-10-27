# 🛠️ Docker Troubleshooting & Debugging

---

## 🧠 ১️⃣ কেন Troubleshooting দরকার?

Docker container ও network অনেক শক্তিশালী, কিন্তু কখনো কখনো সমস্যা দেখা দেয়:  

- Container চালু হচ্ছে না  
- Network connect হচ্ছে না  
- Service crash / database connect fail  
- Image build slow বা fail  

Debugging শেখা মানে তুমি নিজে থেকে solution খুঁজে বের করতে পারবে।  

---

## 🔍 ২️⃣ Container না চালু হলে কি করো?

### Step 1: Container লিস্ট চেক করা
```bash
docker ps -a
````

* Status দেখো (`Exited` হলে container fail হয়েছে)

### Step 2: Logs দেখা

```bash
docker logs container_name
docker logs -f container_name
```

### Step 3: Interactive shell দিয়ে check করা

```bash
docker exec -it container_name bash
# অথবা sh
```

---

## 🧱 ৩️⃣ Common Container Errors

| Error               | Possible Cause         | Fix                               |
| ------------------- | ---------------------- | --------------------------------- |
| Exit 1 / Exit 137   | Out of memory / crash  | Increase memory / resource limit  |
| Port already in use | Host port conflict     | Change `-p` port mapping          |
| Image not found     | Wrong image name/tag   | `docker pull correct_image:tag`   |
| Permission denied   | Root vs non-root issue | USER setting / volume permissions |

---

## 🌐 ৪️⃣ Network Connectivity Issues

### Step 1: Network inspect

```bash
docker network inspect network_name
```

### Step 2: Ping between containers

```bash
docker exec -it container1 ping container2
```

### Common fixes:

* Containers same custom network-এ আছে কিনা চেক করো
* Network alias ব্যবহার করো
* Firewall / host rules block করছে কিনা check করো

---

## 🐳 ৫️⃣ Docker Image Build Failures

| Cause                      | Fix                                                 |
| -------------------------- | --------------------------------------------------- |
| Cache problem              | `docker build --no-cache -t myapp .`                |
| Missing files              | `.dockerignore` ভুল check করো                       |
| Dependency error           | Dockerfile RUN command ঠিক করো, versions verify করো |
| Network issue during build | Host internet / proxy check করো                     |

---

## 💾 ৬️⃣ Volume / Data Issues

* Container restart-এর পরে data missing → Check volume mapping

```bash
docker inspect container_name
docker volume ls
```

* Permission denied → Correct ownership inside container

```bash
chown -R www-data:www-data /var/lib/mysql
```

---

## 📊 ৭️⃣ Performance Debugging

1. CPU / Memory usage

```bash
docker stats
```

2. Slow container → Check logs, optimize image, reduce unnecessary layers
3. Network lag → Check bridge/host network, bandwidth, overlay issues

---

## ⚡ ৮️⃣ Docker Compose Specific Debug

* Container not starting → `docker-compose logs service_name`
* Build issues → `docker-compose build --no-cache`
* Dependency order → `depends_on` ঠিক আছে কিনা চেক করো

---

## 🧠 ৯️⃣ Tips & Tricks

* Use descriptive container names → Easy debug
* Always define healthcheck → Detect failures early
* Keep logs centralized → ELK/Grafana
* Inspect container → `docker inspect container_name`
* Rebuild from scratch → `docker-compose down --volumes && docker-compose up -d --build`

---

## 🛠️  🔟 Quick Debug Commands

| Task                    | Command                                                      |
| ----------------------- | ------------------------------------------------------------ |
| Container logs          | `docker logs -f container_name`                              |
| Exec shell              | `docker exec -it container_name bash`                        |
| Inspect container       | `docker inspect container_name`                              |
| List networks           | `docker network ls`                                          |
| Ping container          | `docker exec -it container1 ping container2`                 |
| Stop / remove container | `docker stop container_name` <br> `docker rm container_name` |
| Prune unused resources  | `docker system prune -a --volumes`                           |

---

## ✅ উপসংহার

Docker শেখার শেষে troubleshooting জানা অত্যন্ত গুরুত্বপূর্ণ।

* Container crash → Logs + exec shell
* Build fail → Cache + dependency check
* Network issue → Inspect network + ping
* Volume / data → Check mapping + permissions
* Performance → stats + optimize image

> 🧠 মনে রাখো: “যে Debug জানে, সে Production-ready।” 🐳