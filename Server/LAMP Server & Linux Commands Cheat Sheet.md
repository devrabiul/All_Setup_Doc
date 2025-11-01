# LAMP ও ওয়েব ডেপ্লয়মেন্ট কমান্ড চিটশিট (Bengali)

এই চিটশিটে Ubuntu সার্ভার, LAMP (Linux, Apache, MySQL, PHP) এবং Laravel/WordPress ডেপ্লয়মেন্টের সব গুরুত্বপূর্ণ কমান্ড ও ব্যবহার বিস্তারিতভাবে দেওয়া হয়েছে।  

---

## ১. সার্ভার আপডেট এবং প্যাকেজ ম্যানেজমেন্ট

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `sudo apt update` | প্যাকেজ লিস্ট আপডেট করে |
| `sudo apt upgrade -y` | সব ইনস্টলড প্যাকেজ আপগ্রেড করে |
| `sudo apt autoremove -y` | অপ্রয়োজনীয় প্যাকেজ রিমুভ করে |
| `sudo apt install package-name -y` | নতুন প্যাকেজ ইনস্টল করে |
| `sudo apt remove package-name -y` | প্যাকেজ রিমুভ করে |

---

## ২. ইউজার এবং পারমিশন

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `sudo adduser username` | নতুন ইউজার তৈরি করে |
| `sudo usermod -aG sudo username` | ইউজারকে sudo গ্রুপে যোগ করে |
| `sudo chown -R www-data:www-data /var/www/html/project` | ফাইলের মালিক www-data করে দেয় (Apache) |
| `sudo chmod -R 755 /var/www/html/project` | ডিরেক্টরি/ফাইলের পারমিশন সেট করে |

---

## ৩. SSH সিকিউরিটি

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `ssh-keygen -t rsa -b 4096 -C "email@example.com"` | SSH key pair জেনারেট করে |
| `ssh-copy-id -i ~/.ssh/id_rsa.pub username@server_ip` | পাবলিক কী সার্ভারে কপি করে |
| `chmod 400 key.pem` | PEM key পারমিশন সিকিউর করে |
| `ssh -i key.pem username@server_ip` | সার্ভারে লগইন করে |

**SSHD Config** (`/etc/ssh/sshd_config`) পরিবর্তন:  
- `Port 2222` → SSH ডিফল্ট পোর্ট পরিবর্তন  
- `PermitRootLogin no` → রুট লগিন নিষিদ্ধ  
- `PasswordAuthentication no` → পাসওয়ার্ড লগিন নিষিদ্ধ  

---

## ৪. Firewall (UFW) Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `sudo ufw enable` | UFW চালু করে |
| `sudo ufw allow 22/tcp` | SSH পোর্ট খোলে (ডিফল্ট) |
| `sudo ufw allow 2222/tcp` | SSH পোর্ট যদি পরিবর্তন করা থাকে |
| `sudo ufw allow in "Apache Full"` | HTTP & HTTPS খুলে দেয় |
| `sudo ufw status` | অবস্থা দেখায় |

---

## ৫. Apache Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `sudo apt install apache2 -y` | Apache ইনস্টল করে |
| `sudo systemctl start apache2` | Apache শুরু করে |
| `sudo systemctl stop apache2` | Apache বন্ধ করে |
| `sudo systemctl restart apache2` | পুনরায় শুরু করে |
| `sudo systemctl reload apache2` | কনফিগ রিলোড করে |
| `sudo systemctl status apache2` | সার্ভিস স্ট্যাটাস দেখায় |
| `/var/log/apache2/error.log` | Apache এর এরর লগ |
| `/var/log/apache2/access.log` | Apache অ্যাক্সেস লগ |

**Virtual Host Configuration**  
`/etc/apache2/sites-available/example.conf`:

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html/project
    <Directory /var/www/html/project>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Enable site & reload:

```bash
sudo a2ensite example.conf
sudo a2enmod rewrite
sudo systemctl reload apache2
```

---

## ৬. MySQL Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `sudo apt install mysql-server -y` | MySQL ইনস্টল করে |
| `sudo mysql_secure_installation` | সার্ভার সিকিউর করে |
| `sudo mysql -u root -p` | MySQL এ লগইন |
| `CREATE DATABASE dbname;` | নতুন ডাটাবেস তৈরি |
| `CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';` | নতুন ইউজার তৈরি |
| `GRANT ALL PRIVILEGES ON dbname.* TO 'user'@'localhost';` | ইউজারকে প্রিভিলেজ দেয় |
| `FLUSH PRIVILEGES;` | প্রিভিলেজ রিলোড |
| `SHOW DATABASES;` | ডাটাবেস দেখায় |
| `USE dbname;` | ডাটাবেস সিলেক্ট করে |
| `mysqldump -u user -p dbname > backup.sql` | ব্যাকআপ তৈরি |
| `mysql -u user -p dbname < backup.sql` | ব্যাকআপ রিস্টোর |

---

## ৭. PHP Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `sudo apt install php libapache2-mod-php php-mysql -y` | PHP ইনস্টল |
| `php -v` | ভার্সন দেখায় |
| `php -m` | ইনস্টলড মডিউল দেখায় |
| `php -l file.php` | সিনট্যাক্স চেক করে |
| `echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php` | PHP info ফাইল তৈরি |

**Laravel Required Modules:**  
`php-mysql php-curl php-mbstring php-xml php-zip php-bcmath php-intl php-gd`

---

## ৮. Laravel Deployment Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `composer install --no-dev` | ডিপেন্ডেন্সি ইনস্টল |
| `php artisan key:generate` | APP_KEY জেনারেট |
| `php artisan migrate` | ডাটাবেস মাইগ্রেশন |
| `php artisan db:seed` | সিডার রান |
| `php artisan config:cache` | কনফিগ ক্যাশ |
| `php artisan route:cache` | রাউট ক্যাশ |
| `php artisan view:clear` | ভিউ ক্লিয়ার |

**Permissions:**

```bash
sudo chown -R www-data:www-data project
sudo chmod -R 775 project/storage project/bootstrap/cache
```

---

## ৯. WordPress Deployment Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `wget https://wordpress.org/latest.tar.gz` | WordPress ডাউনলোড |
| `tar -xzvf latest.tar.gz` | এক্সট্রাক্ট |
| `sudo mv wordpress /var/www/html/mywebsite` | ফোল্ডার মুভ |
| `sudo chown -R www-data:www-data /var/www/html/mywebsite` | পারমিশন সেট |
| MySQL Database creation (WP) | `CREATE DATABASE wordpress_db;` |
| User creation | `CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';` |
| Grant privileges | `GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wpuser'@'localhost';` |

---

## ১০. Backup & Restore Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `tar -czvf project_backup.tar.gz /var/www/html/project` | প্রজেক্ট ব্যাকআপ |
| `tar -xzvf project_backup.tar.gz -C /var/www/html/` | প্রজেক্ট রিস্টোর |
| `mysqldump -u user -p dbname > backup.sql` | ডাটাবেস ব্যাকআপ |
| `mysql -u user -p dbname < backup.sql` | ডাটাবেস রিস্টোর |

**Cron Job Example** (`daily backup at 3AM`):

```bash
0 3 * * * /home/ubuntu/backup.sh
```

---

## ১১. Monitoring Commands

| কমান্ড | ব্যাখ্যা |
|--------|---------|
| `df -h` | ডিস্ক ব্যবহার দেখায় |
| `du -sh /var/www/html` | ফোল্ডার সাইজ দেখায় |
| `free -h` | মেমোরি ব্যবহার দেখায় |
| `top` | CPU & Processes |
| `htop` | উন্নত Process Monitor |
| `tail -f /var/log/apache2/error.log` | অ্যাপ্লিকেশন লগস |
| `netstat -tulnp` | Active network connections |

---

## ১২. Troubleshooting Tips

- সব সময় **লগস চেক করুন** (Apache, MySQL, PHP, System)  
- সার্ভিস স্ট্যাটাস যাচাই: `sudo systemctl status apache2/mysql/ssh`  
- পোর্ট এবং ফায়ারওয়াল চেক করুন  
- পারমিশন সঠিক আছে কিনা যাচাই করুন  
- পরিবর্তন করার পর সার্ভিস রিস্টার্ট করুন  

---

## ১৩. SSL Setup (HTTPS)

```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache -d example.com
```

- SSL Certificate স্বয়ংক্রিয়ভাবে Renewal হবে
- Apache VirtualHost স্বয়ংক্রিয়ভাবে HTTPS রিডাইরেকশন করবে
