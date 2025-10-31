# PHP Basics and PHP Deployment on Ubuntu

PHP is a popular server-side scripting language used to build dynamic web applications. This guide explains PHP basics and how to deploy PHP applications on an Ubuntu server with Apache and MySQL.

---

## 1. What is PHP?

PHP (Hypertext Preprocessor) is a scripting language that:

- Runs on the server to generate dynamic content  
- Integrates with databases (like MySQL)  
- Handles forms, sessions, cookies, and authentication  
- Works with most web servers, especially Apache  

---

## 2. Installing PHP on Ubuntu

Update packages:

```bash
sudo apt update
sudo apt upgrade -y
````

Install PHP along with Apache module and MySQL extension:

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

Check version:

```bash
php -v
```

---

## 3. Testing PHP

Create a `phpinfo.php` file to check installation:

```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/phpinfo.php
```

Open in a browser:

```
http://your-server-ip/phpinfo.php
```

You should see a PHP information page with all installed modules.

---

## 4. PHP Basics

### Variables and Data Types

```php
<?php
$name = "Rabiul";
$age = 22;
$isStudent = true;
$grades = array(90, 85, 78);
?>
```

### Conditional Statements

```php
<?php
if ($age > 18) {
    echo "Adult";
} else {
    echo "Minor";
}
?>
```

### Loops

```php
<?php
for ($i = 1; $i <= 5; $i++) {
    echo $i;
}

foreach ($grades as $grade) {
    echo $grade;
}
?>
```

### Functions

```php
<?php
function greet($name) {
    return "Hello, $name!";
}
echo greet("Rabiul");
?>
```

### Working with Forms

```php
<form method="post" action="submit.php">
  Name: <input type="text" name="name">
  <input type="submit" value="Submit">
</form>
```

`submit.php`:

```php
<?php
$name = $_POST['name'];
echo "Hello, $name!";
?>
```

---

## 5. Connecting PHP to MySQL

```php
<?php
$servername = "localhost";
$username = "myuser";
$password = "mypassword";
$database = "myapp";

$conn = new mysqli($servername, $username, $password, $database);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```

---

## 6. Deploying PHP Applications on Ubuntu

1. Place project files in Apache document root:

```bash
sudo cp -r /local/project /var/www/html/myproject
sudo chown -R www-data:www-data /var/www/html/myproject
sudo chmod -R 755 /var/www/html/myproject
```

2. Test in a browser:

```
http://your-server-ip/myproject
```

3. Optional: Configure virtual host for custom domain:

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html/myproject
    <Directory /var/www/html/myproject>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Enable site and reload Apache:

```bash
sudo a2ensite myproject.conf
sudo systemctl reload apache2
```

---

## 7. PHP Security Tips

* Never display errors on a live site: `display_errors = Off` in `php.ini`
* Validate all user input
* Use prepared statements for database queries
* Keep PHP and modules updated

---

## 8. Summary

By learning PHP basics and deployment:

* You can create dynamic web pages
* Connect to MySQL databases
* Deploy projects on Ubuntu servers with Apache
* Understand security best practices for live applications

```