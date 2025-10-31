# Laravel Deployment on LAMP Server

Laravel is a modern PHP framework used for building complex web applications. This guide explains how to deploy a Laravel project on an Ubuntu server with LAMP stack.

---

## 1. Prerequisites

- Ubuntu server with **Apache, MySQL, PHP** installed  
- PHP version >= 8.0 (check with `php -v`)  
- Composer installed (dependency manager for PHP)

Install Composer:

```bash
sudo apt update
sudo apt install composer -y
````

Check version:

```bash
composer -V
```

---

## 2. Uploading Laravel Project

Upload your Laravel project using **SCP**, **SFTP**, or **Git**:

```bash
scp -i botkey.pem -r /local/laravel_project ubuntu@server_ip:/var/www/html/laravel_project
```

---

## 3. Set Directory Permissions

Laravel requires **storage** and **bootstrap/cache** directories to be writable:

```bash
sudo chown -R www-data:www-data /var/www/html/laravel_project
sudo chmod -R 775 /var/www/html/laravel_project/storage
sudo chmod -R 775 /var/www/html/laravel_project/bootstrap/cache
```

---

## 4. Install Dependencies

Navigate to project folder:

```bash
cd /var/www/html/laravel_project
```

Install PHP dependencies:

```bash
composer install --no-dev
```

Generate application key:

```bash
php artisan key:generate
```

---

## 5. Configure Environment

Update `.env` file with your database and server info:

```
APP_NAME=MyApp
APP_ENV=production
APP_KEY=base64:generated_key
APP_DEBUG=false
APP_URL=http://your-server-ip

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=myuser
DB_PASSWORD=mypassword
```

---

## 6. Database Migration & Seeding

Run migrations to create tables:

```bash
php artisan migrate
```

Optional: seed database with initial data:

```bash
php artisan db:seed
```

---

## 7. Apache Virtual Host for Laravel

1. Create virtual host configuration:

```bash
sudo nano /etc/apache2/sites-available/laravel.conf
```

Content:

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html/laravel_project/public

    <Directory /var/www/html/laravel_project/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/laravel_error.log
    CustomLog ${APACHE_LOG_DIR}/laravel_access.log combined
</VirtualHost>
```

2. Enable site and rewrite module:

```bash
sudo a2ensite laravel.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 8. Testing Laravel Application

Open browser:

```
http://your-server-ip
```

* You should see Laravel welcome page
* If you get 500 error, check:

```bash
tail -f /var/log/apache2/error.log
```

* Ensure **storage** and **bootstrap/cache** directories are writable

---

## 9. Optional: Enabling HTTPS

Install Certbot for SSL:

```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache -d example.com
```

This enables HTTPS with a free SSL certificate.

---

## 10. Summary

By following these steps:

* Laravel project is uploaded and dependencies installed
* `.env` configured for database connection
* Apache virtual host points to `public` directory
* Database migrations and optional seeders applied
* Optional SSL setup for secure HTTPS

You now have a fully functional Laravel application running on a LAMP server.

```