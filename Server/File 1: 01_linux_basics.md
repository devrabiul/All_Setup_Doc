# Linux Basics for Beginners

This guide covers the fundamentals of Linux, focusing on Ubuntu, which is widely used in servers. It is designed for beginners who want to become confident in using the command line.

---

## 1. Introduction to Linux

Linux is an open-source operating system based on Unix. Most servers, including AWS EC2 instances, run Linux. Key features:  

- Open-source and free  
- Highly stable and secure  
- Supports multi-user and multitasking  
- Command-line interface (CLI) and GUI

On servers, most tasks are performed via **CLI** because it's faster and more flexible.

---

## 2. Connecting to a Linux Server

You usually access a server remotely using **SSH**:

```bash
ssh -i your_key.pem username@server_ip
````

* `your_key.pem` → your private key file
* `username` → typically `ubuntu` for Ubuntu servers
* `server_ip` → public IP of your server

Example:

```bash
ssh -i botkey.pem ubuntu@54.82.111.59
```

---

## 3. File and Directory Management

### Listing Files

```bash
ls       # list files in current directory
ls -l    # detailed list with permissions, owner, size, date
ls -a    # include hidden files
```

### Changing Directory

```bash
cd /path/to/directory
cd ..       # move up one level
cd ~        # go to home directory
```

### Creating Directories

```bash
mkdir new_folder          # create a directory
mkdir -p /a/b/c           # create nested directories
```

### Creating Files

```bash
touch file.txt            # create an empty file
nano file.txt             # create/edit file using nano editor
```

### Copying and Moving

```bash
cp source.txt destination.txt      # copy file
cp -r folder1 folder2             # copy folder recursively

mv oldname.txt newname.txt        # rename or move file
```

### Deleting Files and Folders

```bash
rm file.txt            # delete a file
rm -r folder           # delete folder and its contents recursively
```

---

## 4. File Permissions

Linux uses three types of permissions: **read (r), write (w), execute (x)**.

Check permissions:

```bash
ls -l
```

Change permissions:

```bash
chmod 755 file.sh     # Owner: read/write/execute, Others: read/execute
chmod 644 file.txt    # Owner: read/write, Others: read
```

Change owner:

```bash
sudo chown user:group file.txt
```

---

## 5. Processes and Services

Check running processes:

```bash
ps aux
top                # interactive process monitoring
htop               # (optional) better interface if installed
```

Manage services:

```bash
sudo systemctl start apache2      # start service
sudo systemctl stop apache2       # stop service
sudo systemctl restart apache2    # restart service
sudo systemctl status apache2     # check status
```

---

## 6. Package Management (APT)

Ubuntu uses **APT** to install, update, and remove software.

Update package list:

```bash
sudo apt update
sudo apt upgrade -y
```

Install software:

```bash
sudo apt install package_name
```

Remove software:

```bash
sudo apt remove package_name
```

---

## 7. Useful Commands

| Command  | Description            |
| -------- | ---------------------- |
| pwd      | Show current directory |
| whoami   | Show current user      |
| hostname | Show server hostname   |
| df -h    | Show disk usage        |
| free -h  | Show memory usage      |
| uname -a | Show OS details        |
| history  | Show command history   |
| clear    | Clear terminal screen  |

---

## 8. Tips for Beginners

* Always use **tab autocomplete** to avoid typing mistakes.
* Use `man <command>` to see the manual of a command. Example: `man ls`
* Use `sudo` for administrative tasks.

---

## Summary

Linux is the backbone of most servers. Mastering basic commands, file management, permissions, and package management is the first step to becoming a server-ready developer.
With these basics, you can confidently navigate your Ubuntu server and prepare for web deployment.

```