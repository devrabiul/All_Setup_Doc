# Ubuntu সার্ভারে LAMP Stack (Apache, MySQL, PHP) ইনস্টল করার গাইড

এই গাইডে দেখানো হয়েছে কিভাবে Ubuntu সার্ভারে LAMP stack ইনস্টল এবং কনফিগার করবেন।  
**LAMP** = Linux, Apache, MySQL, PHP। এটি ডায়নামিক ওয়েবসাইট ও অ্যাপ্লিকেশন হোস্ট করার সবচেয়ে জনপ্রিয় স্ট্যাকগুলোর মধ্যে একটি।

---

## প্রয়োজনীয়তা

- Ubuntu সার্ভার (18.04, 20.04 বা 22.04)  
- `sudo` প্রিভিলেজ সহ একটি ইউজার  
- সার্ভারে SSH অ্যাক্সেস  
- টার্মিনাল কমান্ডের বেসিক জ্ঞান

---

## ধাপ ১: সিস্টেম আপডেট করুন

প্যাকেজ ইনডেক্স আপডেট এবং ইনস্টল করা প্যাকেজ আপগ্রেড করুন:

```bash
sudo apt update
sudo apt upgrade -y
```

---

## ধাপ ২: Apache ওয়েব সার্ভার ইনস্টল

Apache একটি জনপ্রিয় ওয়েব সার্ভার। ইনস্টল করুন:

```bash
sudo apt install apache2 -y
```

Apache চালু এবং সক্রিয় করুন:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

ব্রাউজারে আপনার সার্ভারের IP খুলে টেস্ট করুন:

```
http://your-server-ip
```

আপনি **“Apache2 Ubuntu Default Page”** দেখতে পাবেন।

---

## ধাপ ৩: MySQL ডাটাবেস সার্ভার ইনস্টল

MySQL একটি রিলেশনাল ডাটাবেস। ইনস্টল করুন:

```bash
sudo apt install mysql-server -y
```

MySQL নিরাপত্তা সেটআপ করুন:

```bash
sudo mysql_secure_installation
```

প্রম্পট অনুযায়ী root পাসওয়ার্ড সেট করুন এবং অ-নিরাপদ ডিফল্ট অপশনগুলো সরান।

---

## ধাপ ৪: PHP ইনস্টল

PHP সার্ভার-সাইড স্ক্রিপ্টিং ভাষা। MySQL এক্সটেনশনসহ ইনস্টল করুন:

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

PHP টেস্ট করতে `phpinfo` ফাইল তৈরি করুন:

```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

ব্রাউজারে ভিজিট করুন:

```
http://your-server-ip/info.php
```

আপনি PHP তথ্য পেজ দেখতে পাবেন।

---

## ধাপ ৫: Apache রিস্টার্ট

PHP ইনস্টল করার পরে Apache রিস্টার্ট করুন:

```bash
sudo systemctl restart apache2
```

---

## ধাপ ৬: ইনস্টলেশন যাচাই

ইনস্টল করা কম্পোনেন্টগুলোর ভার্সন চেক করুন:

```bash
apache2 -v
mysql --version
php -v
```

---

## ঐচ্ছিক: ফায়ারওয়াল কনফিগারেশন

UFW চালু থাকলে HTTP এবং HTTPS ট্রাফিক অনুমোদন করুন:

```bash
sudo ufw allow in "Apache Full"
sudo ufw enable
sudo ufw status
```

---

## সারাংশ

আপনার Ubuntu সার্ভারে এখন সম্পূর্ণ কার্যকর **LAMP স্ট্যাক** রয়েছে।  
এখন আপনি PHP অ্যাপ্লিকেশন ডিপ্লয় করতে পারবেন, MySQL ডাটাবেসের সাথে সংযোগ করতে পারবেন এবং Apache এর মাধ্যমে ওয়েব কনটেন্ট সার্ভ করতে পারবেন।

---

## রেফারেন্স

* [DigitalOcean: How To Install LAMP Stack on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-lamp-stack-on-ubuntu)