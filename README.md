
### Installation

Install Terminator with...

```bash
sudo apt update
sudo apt install terminator
```

### Installation

Install VLC Player with...

```bash
sudo apt update
sudo apt install vlc
```

### Installation

Install Github Desktop with...

```bash
sudo wget https://github.com/shiftkey/desktop/releases/download/release-3.1.1-linux1/GitHubDesktop-linux-3.1.1-linux1.deb
sudo apt-get install gdebi-core 
sudo gdebi GitHubDesktop-linux-3.1.1-linux1.deb
```

### Installation

Setup SSH Key with...

```bash
ssh-keygen
ls
cd /
ls
cd home/
ls
cd username/.ssh/
ls
cat id_rsa.pub
```

Git SSH Connetion

```bash
git config --get remote.origin.url
ssh -T git@github.com

git remote -v
git remote set-url origin git@github.com:username/repositoryname.git
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```


### Installation

Install Node.js with Ubuntu Software...


### Installation

Download [xampp](https://www.apachefriends.org/download.html) for ubuntu

If you are using XAMPP, it is likely that the Apache web server that comes with XAMPP is conflicting with the Apache web server that is pre-installed on your Linux system. To stop the default Apache web server on your Linux system, you can follow these steps:

Run the following command to stop the Apache web server:
```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
```

-> Install Xampp from https://www.apachefriends.org/

```bash
cd /
cd Downloads/
sudo chmod -R 777 .
```

```bash
sudo ./xampp-linux-x64-***-installer.run
```

```bash
sudo /opt/lampp/lampp start
```

```bash
sudo apt install net-tools
sudo apt install git
```
try ...
```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install php8.2
php -v
```

```bash
sudo apt-get install php-curl
```

```bash
sudo apt-get install php-gd
```

```bash
sudo apt-get install php-init
```

```bash
sudo apt-get install php-cli
```


```bash
sudo apt-get install php-xml
```

```bash
sudo apt-get install php-mbstring
```
```bash
sudo apt-get install php-pgsql
```

```bash
sudo apt-get install php-zip
```

```bash
sudo apt-get install php-mysql
```

```bash
sudo apt-get install php-init
```

```bash
sudo apt-get install php-dom
```


```bash
sudo apt-get install php-xml
```

```bash
sudo apt-get install php-mbstring
```

### OR Can try


```bash
sudo apt-get install php-curl
sudo apt-get install php-gd
sudo apt-get install php-init
sudo apt-get install php-cli
sudo apt-get install php-xml
sudo apt-get install php-mbstring
sudo apt-get install php-pgsql
sudo apt-get install php-zip
sudo apt-get install php-mysql
sudo apt-get install php-init
sudo apt-get install php-dom
sudo apt-get install php-xml
sudo apt-get install php-mbstring
sudo apt-get install php-bcmath
sudo apt-get install php-bz2
sudo apt-get install php-calendar
sudo apt-get install php-Core
sudo apt-get install php-ctype
sudo apt-get install php-curl
sudo apt-get install php-date
sudo apt-get install php-dom
sudo apt-get install php-exif
sudo apt-get install php-FFI
sudo apt-get install php-fileinfo
sudo apt-get install php-filter
sudo apt-get install php-ftp
sudo apt-get install php-gd
sudo apt-get install php-gettext
sudo apt-get install php-hash
sudo apt-get install php-iconv
sudo apt-get install php-intl
sudo apt-get install php-json
sudo apt-get install php-ldap
sudo apt-get install php-libxml
sudo apt-get install php-mbstring
sudo apt-get install php-mysqli
sudo apt-get install php-mysqlnd
sudo apt-get install php-openssl
sudo apt-get install php-pcntl
sudo apt-get install php-pcre
sudo apt-get install php-PDO
sudo apt-get install php-pdo_mysql
sudo apt-get install php-pdo_pgsql
sudo apt-get install php-pgsql
sudo apt-get install php-Phar
sudo apt-get install php-posix
sudo apt-get install php-readline
sudo apt-get install php-Reflection
sudo apt-get install php-session
sudo apt-get install php-shmop
sudo apt-get install php-SimpleXML
sudo apt-get install php-soap
sudo apt-get install php-sockets
sudo apt-get install php-sodium
sudo apt-get install php-SPL
sudo apt-get install php-standard
sudo apt-get install php-sysvmsg
sudo apt-get install php-sysvsem
sudo apt-get install php-sysvshm
sudo apt-get install php-tokenizer
sudo apt-get install php-xml
sudo apt-get install php-xmlreader
sudo apt-get install php-xmlwriter
sudo apt-get install php-xsl
sudo apt-get install php-zip
sudo apt-get install php-zlib
```

### Installation

Install Composer with...

```bash
sudo apt-get install composer
```

Instead of removing all your history entries, type these commands in your terminal:

```bash
history
history -c
```

### Additional

If need check system information 

```bash
sudo dmidecode -t system
```
