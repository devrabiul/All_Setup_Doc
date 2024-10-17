### Symbolic link from the Modules directory

To create a symbolic link from the Modules directory in /var/www/project to the public directory, you can use the ln -s command as follows:

Open your terminal.

Navigate to the public directory:

```bash
cd /var/www/project/public
```

Create the symbolic link:

```bash
ln -s ../Modules Modules
```

This command creates a symbolic link named Modules in the public directory that points to the Modules directory located at /var/www/project/Modules.

If you want to ensure that the symbolic link is created successfully, you can list the contents of the public directory:

```bash
ls -l
```

You should see the Modules link pointing to the correct directory.