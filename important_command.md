### Get IP information

```bash
ifconfig
```

### Update Chrome in Ubuntu

```bash
sudo apt-get update
sudo apt-get --only-upgrade install google-chrome-stable
```

### Make a folder shortcut in Ubuntu

#### Example

Help me to create a shortcut, in Windows OS. I copy a folder and paste as a shortcut
I need this in Linux, I want to create a shortcut, give the instruction and terminal command.

want to copy /opt/lampp/htdocs/endpoint/resources/themes folder into /opt/lampp/htdocs/endpoint/public

To achieve this, you can use the cp command to copy the themes folder to the desired location (/opt/lampp/htdocs/endpoint/public). However, since you want to create a shortcut (symbolic link) instead of copying, you'll use the ln command with the -s option to create the symbolic link. Here's the command:

```bash
ln -s /opt/lampp/htdocs/endpoint/resources/themes /opt/lampp/htdocs/endpoint/public/themes
```

This command creates a symbolic link named themes in the /opt/lampp/htdocs/endpoint/public directory, which points to the themes folder located at /opt/lampp/htdocs/endpoint/resources/themes.

After running this command, you'll have a symbolic link named themes in the public directory, which will allow you to access the contents of the themes folder without duplicating the files.
