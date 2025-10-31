# MySQL Database Management for Web Applications

MySQL is a widely used relational database management system (RDBMS) that stores and organizes data for web applications. In this guide, you will learn how to install, secure, and manage MySQL on Ubuntu, as well as connect it with PHP applications.

---

## 1. What is MySQL?

MySQL is an open-source RDBMS that allows:

- Storing structured data in tables  
- Querying data using SQL (Structured Query Language)  
- Connecting databases with web applications (PHP, Laravel, WordPress, etc.)  
- Managing multiple users with privileges  

---

## 2. Installing MySQL on Ubuntu

Update packages:

```bash
sudo apt update
sudo apt upgrade -y
````

Install MySQL server:

```bash
sudo apt install mysql-server -y
```

Check status:

```bash
sudo systemctl status mysql
```

---

## 3. Securing MySQL Installation

Run the security script:

```bash
sudo mysql_secure_installation
```

You will be prompted to:

1. Set a root password (if not already set)
2. Remove anonymous users → Press **y**
3. Disallow remote root login → Press **y**
4. Remove test database → Press **y**
5. Reload privilege tables → Press **y**

This ensures your MySQL server is production-ready.

---

## 4. Basic MySQL Commands

### Logging in as root

```bash
sudo mysql -u root -p
```

Enter your root password when prompted.

### Creating a Database

```sql
CREATE DATABASE myapp;
```

### Creating a User and Granting Privileges

```sql
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON myapp.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
```

### Listing Databases

```sql
SHOW DATABASES;
```

### Selecting a Database

```sql
USE myapp;
```

### Creating Tables

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    password VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Inserting Data

```sql
INSERT INTO users (name, email, password) VALUES ('Rabiul', 'rabiul@example.com', 'hashedpassword');
```

### Querying Data

```sql
SELECT * FROM users;
SELECT name, email FROM users WHERE id = 1;
```

### Updating Data

```sql
UPDATE users SET name='Rabiul Ahmed' WHERE id=1;
```

### Deleting Data

```sql
DELETE FROM users WHERE id=1;
```

---

## 5. Connecting PHP with MySQL

Use **mysqli** or **PDO** in PHP:

```php
<?php
$servername = "localhost";
$username = "myuser";
$password = "mypassword";
$database = "myapp";

// Create connection
$conn = new mysqli($servername, $username, $password, $database);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```

---

## 6. Backup and Restore

### Backup a database

```bash
mysqldump -u myuser -p myapp > myapp_backup.sql
```

### Restore a database

```bash
mysql -u myuser -p myapp < myapp_backup.sql
```

---

## 7. Summary

MySQL is essential for storing data in web applications. By mastering:

* Database and user creation
* CRUD operations (Create, Read, Update, Delete)
* Connecting with PHP
* Backup and restore

…you can confidently manage the data layer for your web projects.

```