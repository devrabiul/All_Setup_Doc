# SSH and Remote Access for Ubuntu Servers

SSH (Secure Shell) is the standard way to securely access and manage Linux servers remotely. This guide explains everything you need to know to connect, manage, and transfer files securely.

---

## 1. What is SSH?

SSH is a network protocol that allows encrypted communication between your local computer and a remote server. It is widely used for:

- Remote server login  
- File transfer  
- Executing commands remotely  
- Managing server configurations

---

## 2. Connecting to a Server via SSH

### Using PEM Key (AWS EC2)

1. Download the `.pem` key provided by your server administrator.
2. Set proper permissions for security:

```bash
chmod 400 botkey.pem
````

3. Connect using SSH:

```bash
ssh -i botkey.pem ubuntu@server_ip
```

* `ubuntu` → default username on Ubuntu EC2 servers
* `server_ip` → public IP of your server

Example:

```bash
ssh -i botkey.pem ubuntu@54.82.111.59
```

After running this, you will be logged into your server’s terminal.

---

### Using Password Authentication

If the server uses password authentication:

```bash
ssh username@server_ip
```

Then enter the password when prompted.

> Note: PEM key authentication is more secure than password login.

---

## 3. Basic SSH Commands

Once logged in via SSH, you can use all Linux commands:

* `pwd` → show current directory
* `ls` → list files
* `cd /path` → change directory
* `nano file.txt` → edit files
* `sudo` → execute commands as administrator

---

## 4. Exiting SSH

To logout from SSH session:

```bash
exit
```

or press `Ctrl + D`.

---

## 5. SSH Key Management

### Generate Your Own Key Pair

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

* Saves private key in `~/.ssh/id_rsa` and public key in `~/.ssh/id_rsa.pub`

### Copy Public Key to Server

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@server_ip
```

Now you can log in without a password.

---

## 6. Transferring Files

### Using SCP (Secure Copy)

Copy files from local to server:

```bash
scp -i botkey.pem localfile.txt ubuntu@server_ip:/remote/path/
```

Copy entire folder:

```bash
scp -i botkey.pem -r /local/folder ubuntu@server_ip:/remote/path/
```

Copy files from server to local:

```bash
scp -i botkey.pem ubuntu@server_ip:/remote/path/file.txt /local/path/
```

### Using SFTP (Interactive)

```bash
sftp -i botkey.pem ubuntu@server_ip
```

* Commands inside SFTP: `ls`, `cd`, `get`, `put`, `exit`

---

## 7. SSH Configuration Tips

* Use strong keys, not just passwords
* Disable root login via SSH for security
* Consider changing the default SSH port (22)
* Use `ssh -v` for debugging connection issues

---

## 8. Summary

SSH is essential for working with remote servers. By mastering SSH, you can:

* Access servers securely
* Transfer files
* Execute commands remotely
* Deploy applications

This is the foundation for all server-related work, including LAMP stack setup and PHP deployment.

```