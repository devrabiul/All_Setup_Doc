# LAMP Stack Integration and Full Project Deployment

LAMP stands for **Linux, Apache, MySQL, PHP**. It is the most common stack for hosting dynamic web applications. This guide explains how to integrate all components and deploy a full project on an Ubuntu server.

---

## 1. Overview of LAMP Components

| Component | Role |
|-----------|------|
| Linux     | Operating system |
| Apache    | Web server |
| MySQL     | Database server |
| PHP       | Server-side scripting language |

Each component works together to serve dynamic websites:

1. Apache serves web pages.  
2. PHP generates dynamic content and interacts with the database.  
3. MySQL stores and retrieves application data.  
4. Linux provides a stable environment and command-line tools.

---

## 2. Preparing the Server

Before deploying a project:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql -y
````

Check that all services are running:

```bash
sudo systemctl status apache2
sudo systemctl status mysql
```

---

## 3. Securing MySQL

Run:

```bash
sudo mysql_secure_installation
```

Steps:

* Remove anonymous users → y
* Disallow remote root login → y
* Remove test database → y
* Reload privileges → y

---

## 4. Uploading Your Project

Use **SCP**, **SFTP**, or Git to upload your project to Apache root:

```bash
sudo cp -r /local/project /var/www/html/myproject
sudo chown -R www-data:www-data /var/www/html/myproject
sudo chmod -R 755 /var/www/html/myproject
```

* `www-data` is the Apache user
* Ensure correct permissions for security and functionality

---

## 5. Database Setup

1. Login to MySQL:

```bash
sudo mysql -u root -p
```

2. Create a database and user:

```sql
CREATE DATABASE myapp;
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON myapp.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

3. Import database schema (if you have one):

```bash
mysql -u myuser -p myapp < /path/to/schema.sql
```

---

## 6. Configuring PHP Project

* Update `.env` or config files with database credentials:

```
DB_HOST=localhost
DB_DATABASE=myapp
DB_USERNAME=myuser
DB_PASSWORD=mypassword
```

* Test the connection with a simple PHP script:

```php
<?php
$conn = new mysqli("localhost", "myuser", "mypassword", "myapp");
if ($conn->connect_error) die("Connection failed: " . $conn->connect_error);
echo "Database connected successfully";
?>
```

---

## 7. Apache Virtual Hosts (Optional)

To host multiple projects or a custom domain:

1. Create configuration:

```bash
sudo nano /etc/apache2/sites-available/myproject.conf
```

Content:

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html/myproject
    <Directory /var/www/html/myproject>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/myproject_error.log
    CustomLog ${APACHE_LOG_DIR}/myproject_access.log combined
</VirtualHost>
```

2. Enable site and reload Apache:

```bash
sudo a2ensite myproject.conf
sudo a2enmod rewrite
sudo systemctl reload apache2
```

---

## 8. Testing the Deployment

* Open browser:

```
http://your-server-ip
```

or

```
http://example.com
```

* Verify dynamic pages are loading and database connection works.
* Check Apache logs if there are errors:

```bash
tail -f /var/log/apache2/error.log
```

---

## 9. Security Best Practices

* Disable root SSH login
* Set correct permissions on `/var/www/html`
* Keep LAMP packages updated:

```bash
sudo apt update
sudo apt upgrade -y
```

* Enable UFW firewall:

```bash
sudo ufw allow in "Apache Full"
sudo ufw enable
sudo ufw status
```

* Consider SSL (HTTPS) with Certbot:

```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache
```

---

## 10. Summary

By integrating all LAMP components:

* Your PHP project can serve dynamic content
* MySQL stores your application data
* Apache serves the web pages
* Linux ensures stable and secure hosting

This process allows you to fully deploy, manage, and maintain web applications on an Ubuntu server.

```