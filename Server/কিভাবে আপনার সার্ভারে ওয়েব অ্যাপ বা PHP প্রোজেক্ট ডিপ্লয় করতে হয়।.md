কিভাবে আপনার সার্ভারে **ওয়েব অ্যাপ বা PHP প্রোজেক্ট ডিপ্লয়** করতে হয়।

## ১️⃣ সার্ভারের root/public directory

Ubuntu সার্ভারে Apache-এর ডিফল্ট ওয়েব ফোল্ডার হলো:

```
/var/www/html
```

* এখানে সব ওয়েব ফাইল রাখতে হবে।
* ফাইলগুলো সাধারণত **index.php** বা **index.html** থেকে শুরু হয়।

---

## ২️⃣ প্রোজেক্ট ফাইল আপলোড করা

আপনি দুইভাবে ফাইল আপলোড করতে পারেন:

### a) SCP (Local থেকে Server-এ)

```bash
scp -i botkey.pem -r /local/path/to/project ubuntu@your-server-ip:/var/www/html
```

* `/local/path/to/project` = আপনার কম্পিউটারের প্রোজেক্ট ফোল্ডার
* `/var/www/html` = সার্ভারের ফোল্ডার

### b) FTP / SFTP (GUI ব্যবহার করে)

* FileZilla বা WinSCP দিয়ে **SFTP** কানেকশন ব্যবহার করতে পারেন
* Host = server IP
* Username = ubuntu
* Authentication = `.pem` key

---

## ৩️⃣ ফাইল permissions ঠিক করা

Apache কে ফাইল access করতে হলে permission ঠিক করতে হবে:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

* `www-data` = Apache user
* `755` = directories & files read/executable by Apache

---

## ৪️⃣ Database (যদি প্রয়োজন হয়)

1. MySQL-এ নতুন database তৈরি করুন:

```bash
sudo mysql -u root -p
CREATE DATABASE myapp;
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON myapp.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

2. আপনার প্রোজেক্টে `.env` বা config ফাইল আপডেট করুন:

```text
DB_HOST=localhost
DB_DATABASE=myapp
DB_USERNAME=myuser
DB_PASSWORD=password
```

---

## ৫️⃣ Apache Config (Optional)

যদি আপনার প্রোজেক্ট কোনো subdomain বা custom domain ব্যবহার করে:

1. Virtual host ফাইল তৈরি করুন:

```bash
sudo nano /etc/apache2/sites-available/myapp.conf
```

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html
    <Directory /var/www/html>
        AllowOverride All
    </Directory>
</VirtualHost>
```

2. Enable site & rewrite module:

```bash
sudo a2ensite myapp.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## ৬️⃣ ব্রাউজারে চেক করা

* ব্রাউজারে লিখুন:

```
http://your-server-ip
```

* বা যদি domain সেট করা থাকে:

```
http://example.com
```

আপনার প্রোজেক্ট চলে আসবে।

---

💡 **Tip:** Laravel বা অন্য framework হলে `.env` সেট করা, `composer install` করা এবং `php artisan migrate` চালানোও করতে হবে।