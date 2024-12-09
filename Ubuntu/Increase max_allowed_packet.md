### Increase max_allowed_packet

If you're using XAMPP on Linux, the process to increase the max_allowed_packet value is slightly different. Follow these steps:

1. Locate the my.cnf File for XAMPP
In XAMPP, the MySQL configuration file is typically located at:

```bash
/opt/lampp/etc/my.cnf
```

2. Edit the Configuration File
Open the my.cnf file using a text editor:

```bash
sudo nano /opt/lampp/etc/my.cnf
```

3. Increase max_allowed_packet
Search for the [mysqld] section and add or update the following line:

```bash
[mysqld]
max_allowed_packet=256M
```

If it doesn’t exist, add it manually.

You can also add the same setting under the [client] section if needed:

```bash
[client]
max_allowed_packet=256M
```

4. Restart XAMPP
After saving the changes (Ctrl + O to save, Ctrl + X to exit), restart XAMPP to apply the changes:

```bash
sudo /opt/lampp/lampp restart
```

5. Verify the Setting
Log in to the MySQL console:

```bash
/opt/lampp/bin/mysql -u root -p
```

Check the value of max_allowed_packet:

```sql
SHOW VARIABLES LIKE 'max_allowed_packet';
```

It should display the updated value, e.g., 268435456 for 256 MB.

