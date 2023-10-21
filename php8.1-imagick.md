Install Imagick:

To install the Imagick PHP extension, open a terminal and use the following commands:

bash
Copy code
sudo apt update
sudo apt install php-imagick
Restart Apache:

After installing Imagick, you need to restart the Apache web server that comes with XAMPP to make the changes take effect:

bash
Copy code
sudo /opt/lampp/lampp restart
Verify Installation:

You can verify that Imagick is installed and enabled by creating a PHP file with the following content and accessing it through your web browser:

php
Copy code
<?php
phpinfo();
?>
Look for a section in the phpinfo() output that confirms the presence of the Imagick extension.

That's it! You should now have Imagick installed and working in XAMPP on your Ubuntu system.

## Install PHP-FPM with Apache and php8.1-imagick on Ubuntu 22.04

#### Getting Started

```bash
sudo apt update
sudo apt upgrade
```
#### Step 1: Add PPA for PHP 8.1

Add the ondrej/php which has PHP 8.1 package and other required PHP extensions.

```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

#### Step 2: Install PHP 8.1 FPM
Now we shall install PHP 8.1-FPM and some common modules to run a PHP application like WordPress.

```bash
sudo apt install php8.1-fpm php8.1-common php8.1-mysql php8.1-xml php8.1-xmlrpc php8.1-curl php8.1-gd php8.1-imagick php8.1-cli php8.1-dev php8.1-imap php8.1-mbstring php8.1-soap php8.1-zip php8.1-bcmath -y
```

Once you have added the PPA you can install PHP 8.1.

In this tutorial we learn how to install php8.1-imagick on Ubuntu 22.04. php8.1-imagick is Provides a wrapper to the ImageMagick library

#### Introduction
In this tutorial we learn how to install php8.1-imagick on Ubuntu 22.04.

##### What is php8.1-imagick
php8.1-imagick is:

Imagick is a native php extension to create and modify images using the ImageMagick API.

There are three methods to install php8.1-imagick on Ubuntu 22.04. We can use apt-get, apt and aptitude. In the following sections we will describe each method. You can choose one of them.

#### Install php8.1-imagick Using apt-get
Update apt database with apt-get using the following command.

```bash
sudo apt-get update
```

After updating apt database, We can install php8.1-imagick using apt-get by running the following command:

```bash
sudo apt-get -y install php8.1-imagick
Install php8.1-imagick Using apt
```

Update apt database with apt using the following command.

```bash
sudo apt update
```

After updating apt database, We can install php8.1-imagick using apt by running the following command:

```bash
sudo apt -y install php8.1-imagick
```

#### Install php8.1-imagick Using aptitude
If you want to follow this method, you might need to install aptitude first since aptitude is usually not installed by default on Ubuntu. Update apt database with aptitude using the following command.

```bash
sudo aptitude update
```

After updating apt database, We can install php8.1-imagick using aptitude by running the following command:

```bash
sudo aptitude -y install php8.1-imagick
```

#### How To Uninstall php8.1-imagick on Ubuntu 22.04
To uninstall only the php8.1-imagick package we can use the following command:

```bash
sudo apt-get remove php8.1-imagick
```
