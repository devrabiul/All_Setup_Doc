# How To Install LAMP Stack (Apache, MySQL, PHP) on Ubuntu

This guide explains how to install and configure the LAMP stack on an Ubuntu server. LAMP stands for **Linux, Apache, MySQL, and PHP**. It is one of the most popular web service stacks for hosting dynamic websites and applications.

---

## Prerequisites

- An Ubuntu server (18.04, 20.04, or 22.04)
- A user with `sudo` privileges
- Access to the server via SSH
- Basic knowledge of terminal commands

---

## Step 1: Update the System

Always start by updating your package index and upgrading installed packages:

```bash
sudo apt update
sudo apt upgrade -y
````

---

## Step 2: Install Apache Web Server

Apache is a widely used web server software. Install it using:

```bash
sudo apt install apache2 -y
```

Enable and start the Apache service:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

Test Apache by opening your server IP in a browser:

```
http://your-server-ip
```

You should see the "Apache2 Ubuntu Default Page".

---

## Step 3: Install MySQL Database Server

MySQL is a popular relational database system. Install it using:

```bash
sudo apt install mysql-server -y
```

Secure MySQL installation:

```bash
sudo mysql_secure_installation
```

Follow the prompts to set a root password and remove insecure defaults.

---

## Step 4: Install PHP

PHP is a server-side scripting language used for dynamic web pages. Install PHP along with the MySQL extension:

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

Test PHP by creating a `phpinfo` file:

```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

Visit in browser:

```
http://your-server-ip/info.php
```

You should see the PHP information page.

---

## Step 5: Restart Apache

After installing PHP, restart Apache to load PHP module:

```bash
sudo systemctl restart apache2
```

---

## Step 6: Verify Installation

Check versions of installed components:

```bash
apache2 -v
mysql --version
php -v
```

---

## Optional: Firewall Configuration

Allow HTTP and HTTPS traffic if UFW is enabled:

```bash
sudo ufw allow in "Apache Full"
sudo ufw enable
sudo ufw status
```

---

## Conclusion

You now have a fully functional **LAMP stack** on your Ubuntu server. You can start deploying PHP applications, connect them with MySQL databases, and serve web content via Apache.

---

## References

* [DigitalOcean: How To Install LAMP Stack on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-lamp-stack-on-ubuntu)

```