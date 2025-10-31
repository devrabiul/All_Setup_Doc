# Project Backup, Restore, and Monitoring on Ubuntu Server

Proper backup and monitoring strategies ensure your web application remains reliable, recoverable, and performant. This guide covers database and file backups, restoration, and basic monitoring.

---

## 1. Backing Up MySQL Databases

### Backup a Single Database

```bash
mysqldump -u myuser -p myapp > myapp_backup.sql
````

* `myuser` → MySQL username
* `myapp` → Database name
* `myapp_backup.sql` → Output backup file

### Backup All Databases

```bash
mysqldump -u root -p --all-databases > all_databases_backup.sql
```

### Automate Daily Backup (Optional)

Create cron job:

```bash
crontab -e
```

Add line (backup at 2 AM daily):

```bash
0 2 * * * mysqldump -u myuser -p'yourpassword' myapp > /home/ubuntu/backups/myapp_$(date +\%F).sql
```

---

## 2. Restoring MySQL Databases

### Restore Single Database

```bash
mysql -u myuser -p myapp < myapp_backup.sql
```

### Restore All Databases

```bash
mysql -u root -p < all_databases_backup.sql
```

---

## 3. Backing Up Web Project Files

Copy project files to a backup directory:

```bash
cp -r /var/www/html/myproject /home/ubuntu/backups/myproject_$(date +%F)
```

Or compress as a zip:

```bash
tar -czvf myproject_backup_$(date +%F).tar.gz /var/www/html/myproject
```

---

## 4. Restoring Web Project Files

Unpack backup:

```bash
tar -xzvf myproject_backup_2025-10-31.tar.gz -C /var/www/html/
```

Set permissions:

```bash
sudo chown -R www-data:www-data /var/www/html/myproject
sudo chmod -R 755 /var/www/html/myproject
```

---

## 5. Server Monitoring Basics

### Disk Usage

```bash
df -h        # disk usage of partitions
du -sh /var/www/html   # folder size
```

### Memory and CPU Usage

```bash
free -h      # memory usage
top          # interactive process monitor
htop         # better interface (if installed)
```

### Apache & MySQL Logs

Check logs for errors:

```bash
tail -f /var/log/apache2/error.log
tail -f /var/log/mysql/error.log
```

### Monitoring Active Connections

```bash
netstat -tulnp
```

---

## 6. Automating Backups (Optional)

1. Create backup directory:

```bash
mkdir -p /home/ubuntu/backups
```

2. Create a cron script `/home/ubuntu/backup.sh`:

```bash
#!/bin/bash
# Backup MySQL
mysqldump -u myuser -p'yourpassword' myapp > /home/ubuntu/backups/myapp_$(date +%F).sql
# Backup Project Files
tar -czvf /home/ubuntu/backups/myproject_$(date +%F).tar.gz /var/www/html/myproject
```

3. Make it executable:

```bash
chmod +x /home/ubuntu/backup.sh
```

4. Add cron job:

```bash
crontab -e
0 3 * * * /home/ubuntu/backup.sh
```

This will backup daily at 3 AM automatically.

---

## 7. Summary

* Regularly backup your **database** and **project files**.
* Set up automated cron jobs to reduce manual work.
* Monitor **disk space**, **memory**, **CPU**, and **logs** for potential issues.
* Restoring from backups ensures minimal downtime in case of failure.

Proper backup and monitoring maintain your web application’s **reliability, stability, and security**.

```