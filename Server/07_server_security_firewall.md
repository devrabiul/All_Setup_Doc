# Server Security and Firewall Configuration on Ubuntu

Securing your server is essential before deploying any web application. This guide covers basic security practices and firewall setup on an Ubuntu server.

---

## 1. Update and Upgrade Server

Keeping your server updated prevents vulnerabilities:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
````

* `apt update` → updates package list
* `apt upgrade` → upgrades installed packages
* `apt autoremove` → removes unnecessary packages

---

## 2. Create a New User

Avoid using `root` directly for daily tasks:

```bash
sudo adduser rabiul
```

Grant administrative privileges:

```bash
sudo usermod -aG sudo rabiul
```

Test new user by logging in:

```bash
ssh rabiul@your-server-ip
```

---

## 3. Configure SSH

### Change Default Port

Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Change:

```
Port 2222
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

### Disable Root Login

In the same file (`/etc/ssh/sshd_config`):

```
PermitRootLogin no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

### Enable Key-Based Authentication

1. Generate key on local machine:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

2. Copy public key to server:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub rabiul@your-server-ip
```

3. Now you can log in without password. Disable password authentication:

```
PasswordAuthentication no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## 4. Firewall Setup with UFW

### Enable UFW

```bash
sudo ufw enable
```

### Allow SSH

If you changed SSH port:

```bash
sudo ufw allow 2222/tcp
```

Default port 22:

```bash
sudo ufw allow 22/tcp
```

### Allow Web Traffic (HTTP/HTTPS)

```bash
sudo ufw allow in "Apache Full"
```

### Check UFW Status

```bash
sudo ufw status
```

---

## 5. Fail2Ban (Optional but Recommended)

Install Fail2Ban to protect against brute-force attacks:

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Check status:

```bash
sudo fail2ban-client status
```

---

## 6. Additional Security Tips

* Disable unused services:

```bash
sudo systemctl disable service_name
```

* Keep software updated:

```bash
sudo apt update && sudo apt upgrade -y
```

* Monitor logs regularly:

```bash
tail -f /var/log/auth.log
tail -f /var/log/apache2/error.log
```

* Use strong passwords for all users and databases.
* Backup important files regularly.

---

## 7. Summary

By following these steps:

* You reduce the risk of unauthorized access
* Your SSH connection becomes secure
* Your web server is protected with a firewall
* Fail2Ban can prevent brute-force attacks

A secure server is essential before hosting any live application.

```
