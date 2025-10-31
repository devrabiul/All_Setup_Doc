# WordPress Deployment on LAMP Server

WordPress is the most popular content management system (CMS) used to build blogs, websites, and e-commerce stores. This guide explains how to deploy WordPress on an Ubuntu server with LAMP stack.

---

## 1. Prerequisites

- Ubuntu server with **Apache, MySQL, PHP** installed  
- PHP version >= 7.4 (check with `php -v`)  
- MySQL database ready  

---

## 2. Download WordPress

Download the latest WordPress package:

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
````

Move WordPress to Apache root:

```bash
sudo mv wordpress /var/www/html/mywebsite
```

---

## 3. Set Permissions

```bash
sudo chown -R www-data:www-data /var/www/html/mywebsite
sudo chmod -R 755 /var/www/html/mywebsite
```

* Ensures Apache can read and write necessary files

---

## 4. Create MySQL Database

Login to MySQL:

```bash
sudo mysql -u root -p
```

Create database and user:

```sql
CREATE DATABASE wordpress_db;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'strongpassword';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 5. Configure WordPress

1. Copy sample config:

```bash
cd /var/www/html/mywebsite
cp wp-config-sample.php wp-config.php
```

2. Edit `wp-config.php`:

```php
define('DB_NAME', 'wordpress_db');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'strongpassword');
define('DB_HOST', 'localhost');
```

Optional: Add security keys from [WordPress Salt Generator](https://api.wordpress.org/secret-key/1.1/salt/).

---

## 6. Apache Virtual Host (Optional)

Create virtual host configuration:

```bash
sudo nano /etc/apache2/sites-available/mywebsite.conf
```

Content:

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html/mywebsite

    <Directory /var/www/html/mywebsite>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/mywebsite_error.log
    CustomLog ${APACHE_LOG_DIR}/mywebsite_access.log combined
</VirtualHost>
```

Enable site and rewrite module:

```bash
sudo a2ensite mywebsite.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 7. Complete WordPress Installation

1. Open browser:

```
http://your-server-ip
```

or

```
http://example.com
```

2. Follow WordPress setup wizard:

   * Select language
   * Enter site title, username, password, email
   * Login to admin dashboard

---

## 8. Enabling HTTPS (Optional but Recommended)

Install Certbot:

```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache -d example.com
```

* This sets up free SSL for secure HTTPS

---

## 9. Security Tips for WordPress

* Use strong admin password
* Keep WordPress, plugins, and themes updated
* Disable unused plugins
* Regularly backup database and files
* Use security plugins like Wordfence

---

## 10. Summary

By following these steps:

* WordPress is installed on Apache and connected to MySQL
* Permissions are configured for security
* Optional virtual host allows custom domains
* Optional SSL ensures HTTPS
* You can now manage your website from the WordPress admin dashboard

This setup enables you to host blogs, portfolios, or e-commerce sites efficiently on a LAMP server.

```