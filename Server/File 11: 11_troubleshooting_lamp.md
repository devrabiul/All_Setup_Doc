# Basic Troubleshooting and Common Errors on LAMP Server

Even after setting up LAMP and deploying projects, you may encounter errors. This guide covers common issues and how to troubleshoot them on Ubuntu servers.

---

## 1. Apache Errors

### Common Issue: 500 Internal Server Error

Check Apache error log:

```bash
tail -f /var/log/apache2/error.log
````

* Often caused by wrong **permissions** or **.htaccess** rules
* Fix permissions:

```bash
sudo chown -R www-data:www-data /var/www/html/myproject
sudo chmod -R 755 /var/www/html/myproject
```

* Ensure `.htaccess` syntax is correct

---

### Common Issue: Apache Not Starting

Check status:

```bash
sudo systemctl status apache2
```

Check for port conflicts:

```bash
sudo netstat -tulnp | grep :80
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

---

## 2. MySQL Errors

### Cannot Connect to Database

* Verify credentials in `.env` or `wp-config.php`
* Test manually:

```bash
mysql -u myuser -p myapp
```

* Ensure MySQL service is running:

```bash
sudo systemctl status mysql
```

---

### Error: “Access denied for user”

* Check username, password, and host (`localhost`)
* Re-grant privileges:

```sql
GRANT ALL PRIVILEGES ON myapp.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
```

---

## 3. PHP Errors

### 500 Error on Laravel or PHP App

* Check Apache error log:

```bash
tail -f /var/log/apache2/error.log
```

* Check PHP modules:

```bash
php -m
```

* Common missing modules:

```
php-mysql, php-curl, php-mbstring, php-xml, php-zip
```

Install missing modules:

```bash
sudo apt install php-mysql php-curl php-mbstring php-xml php-zip -y
sudo systemctl restart apache2
```

---

### Syntax Error in PHP

Check file for mistakes:

```bash
php -l filename.php
```

---

## 4. File Permissions Issues

* Web applications often fail due to incorrect permissions
* Ensure Apache user owns project files:

```bash
sudo chown -R www-data:www-data /var/www/html/myproject
sudo chmod -R 755 /var/www/html/myproject
```

* Directories that need write access (Laravel):

```
storage, bootstrap/cache
```

---

## 5. SSH & Connection Errors

### Timeout or Refused

* Check server IP and firewall:

```bash
sudo ufw status
ping server_ip
```

* Ensure SSH service is running:

```bash
sudo systemctl status ssh
```

---

### Permission Denied (Key Issues)

* Correct permissions:

```bash
chmod 400 botkey.pem
```

* Correct user:

```bash
ssh -i botkey.pem ubuntu@server_ip
```

---

## 6. Troubleshooting Tips

1. **Check logs first** (Apache, MySQL, PHP, system)
2. **Test components individually** (PHP script, DB connection, Apache)
3. **Check firewall and ports**
4. **Verify permissions**
5. **Restart services after changes**

---

## 7. Summary

Troubleshooting is an essential skill for server management. By learning to:

* Read and interpret logs
* Check service status
* Verify configuration files
* Correct permissions

…you can quickly identify and fix issues on your LAMP server, ensuring your web applications remain stable and accessible.

```