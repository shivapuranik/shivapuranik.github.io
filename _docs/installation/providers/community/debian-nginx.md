---
layout: single
type: docs
permalink: /docs/installation/providers/community/debian-nginx/
redirect_from:
  - /theme-setup/
last_modified_at: 2023-12-19
last_modified_by: Mohammad_Asif
toc: true
title: Installing Faveo Helpdesk Community Edition on Debian With Nginx Webserver
---

#  <!-- omit in toc -->


<img alt="Debian" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4a/Debian-OpenLogo.svg/109px-Debian-OpenLogo.svg.png" width="120" height="120" />

Faveo can run on Debian 11(Bullseye), Debian 12 (Bookworm).

- [<strong>Installation steps :</strong>](#installation-steps-)
    - [<strong>1. Update your package list</strong>](#1-update-your-package-list)
    - [<strong>2. Upload Faveo</strong>](#2-upload-faveo)
    - [<strong>3. Setup the database</strong>](#3-setup-the-database)
    - [<strong>4. Configure Nginx webserver</strong>](#4-configure-nginx-webserver)
    - [<strong>5. Configure cron job</strong>](#5-configure-cron-job)
    - [<strong>6. Redis Installation</strong>](#6-redis-installation)
    - [<strong>7. SSL Installation</strong>](#7-ssl-installation)
    - [<strong>8. Install Faveo</strong>](#8-install-faveo)
    - [<strong>9. Faveo Backup</strong>](#9-faveo-backup)
    - [<strong>10. Final step</strong>](#10-final-step)

<a id="installation-steps-" name="installation-steps-"></a>

# <strong>Installation steps :</strong>


Faveo depends on the following:

-   **Apache** (with mod_rewrite enabled) 
-   **PHP 8.1+** with the following extensions: curl, dom, gd, json, mbstring, openssl, pdo_mysql, tokenizer, zip
-   **MySQL 8.0+** or **MariaDB 10.6+**
-   **SSL** ,Trusted CA Signed or Slef-Signed SSL


<a id="1-update-your-package-list" name="1-update-your-package-list"></a>

### <strong>1. Update your package list</strong>

Run the following commands as sudoers or Login as root user by typing the command below

```sh
sudo su
```

```sh
apt update && apt upgrade -y
```

<b>1.a. Nginx should come pre-installed with your server. If it's not, install it with:</b>


```sh
apt install nginx
systemctl start nginx
systemctl enable nginx
```
<b>1.b. Install some Utility packages</b>

```sh
apt install -y git wget curl unzip nano zip
```

<b>1.c. PHP 8.1+</b>

Note: In Debian upon installing PHP packages apache2 will be automatically installed and started. It needs to be removed after installing PHP8.1

Before we install PHP 8.1, it’s important to make sure your system is up to date by running the following apt commands.

```sh
sudo apt update
sudo apt install apt-transport-https lsb-release ca-certificates
```

Add the Ondřej Surý PHP repository.

```sh
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
```
Now Install PHP 8.1 and extensions.

```sh
sudo apt update
apt install -y php8.1 libapache2-mod-php8.1 php8.1-mysql \
    php8.1-cli php8.1-common php8.1-fpm php8.1-soap php8.1-gd \
    php8.1-opcache  php8.1-mbstring php8.1-zip \
    php8.1-bcmath php8.1-intl php8.1-xml php8.1-curl  \
    php8.1-imap php8.1-ldap php8.1-gmp php8.1-redis
```
Now remove the apache2 package
```sh
apt remove apache2
```
After installing PHP 8.1, run the commands below to open PHP default config file for Nginx…

```sh
nano /etc/php/8.1/fpm/php.ini
```

Then make the changes on the following lines below in the file and save. The value below are great settings to apply in your environments.

```
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
cgi.fix_pathinfo = 0
upload_max_filesize = 100M
max_execution_time = 360
```

<b>1.d.Setting Up ionCube</b>
```sh
wget http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz 
tar xvfz ioncube_loaders_lin_x86-64.tar.gz 
```
Make the note of path and directory from the above command.

Copy ion cube loader to Directory. Replace your *yourpath* below with actual path that was shown with the first command below.
```sh
php -i | grep extension_dir
cp ioncube/ioncube_loader_lin_8.1.so /usr/lib/php/'replaceyourpath'
sed -i '2 a zend_extension = "/usr/lib/php/'replaceyourpath'/ioncube_loader_lin_8.1.so"' /etc/php/8.1/fpm/php.ini
sed -i '2 a zend_extension = "/usr/lib/php/'replaceyourpath'/ioncube_loader_lin_8.1.so"' /etc/php/8.1/cli/php.ini
systemctl restart nginx 
systemctl restart php8.1-fpm
```

<b>1.e. Mysql</b>

The official Faveo installation uses Mysql/MariaDB as the database system and **this is the only official system we support**. While Laravel technically supports PostgreSQL and SQLite, we can't guarantee that it will work fine with Faveo as we've never tested it. Feel free to read [Laravel's documentation](https://laravel.com/docs/database#configuration) on that topic if you feel adventurous.

Install Mysql 8.0 or MariaDB 10.6. Note that this only installs the package, but does not setup Mysql. This is done later in the instructions:

<b> For Debian 11 </b>

```sh
sudo apt update
sudo apt-get install curl software-properties-common dirmngr
curl -LsS -O https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
sudo bash mariadb_repo_setup --mariadb-server-version=10.6
sudo apt-get update
sudo apt-get install mariadb-server mariadb-client
sudo systemctl start mariadb
sudo systemctl enable mariadb
```
<b> For Debian 12 </b>

```sh
sudo apt install dirmngr software-properties-common apt-transport-https curl lsb-release ca-certificates -y
curl -fsSL https://repo.mysql.com/RPM-GPG-KEY-mysql-2022 | gpg --dearmor | sudo tee /usr/share/keyrings/mysql.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/mysql.gpg] http://repo.mysql.com/apt/debian $(lsb_release -sc) mysql-8.0" | sudo tee /etc/apt/sources.list.d/mysql.list

sudo apt update
sudo apt install mysql-community-server
sudo systemctl start mysql --now
sudo systemctl enable mysql --now
```

Secure your MySql installation by executing the below command. Set Password for mysql root user, remove anonymous users, disallow remote root login, remove the test databases and finally reload the privilege tables.
```sh
mysql_secure_installation 
```

**phpMyAdmin(Optional):** Install phpMyAdmin. This is optional step. phpMyAdmin gives a GUI to access and work with Database

```sh
apt install phpmyadmin
```

<b>1.f. Install wkhtmltopdf</b>


Wkhtmltopdf is an open source simple and much effective command-line shell utility that enables user to convert any given HTML (Web Page) to PDF document or an image (jpg, png, etc). 

It uses WebKit rendering layout engine to convert HTML pages to PDF document without losing the quality of the pages. Its is really very useful and trustworthy solution for creating and storing snapshots of web pages in real-time.

**For Debian 11**

```sh
apt-get -y install wkhtmltopdf
```
**For Debian 12**

```sh
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.bookworm_amd64.deb
sudo apt install ./wkhtmltox*bookworm_amd64.deb
```

Once the softwares above are installed:


<a id="2-upload-faveo" name="2-upload-faveo"></a>

### <strong>2. Upload Faveo</strong>

You may install Faveo by simply cloning the repository. In order for this to work with Apache, you need to clone the repository in a specific folder:

```sh
mkdir -p /var/www/
cd /var/www/
git clone https://github.com/ladybirdweb/faveo-helpdesk.git faveo
```
You should check out a tagged version of Faveo since `master` branch may not always be stable. Find the latest official version on the [release page](https://github.com/ladybirdweb/faveo-helpdesk/releases)

<a id="3-setup-the-database" name="3-setup-the-database"></a>

### <strong>3. Setup the database</strong>

Log in with the root account to configure the database.

```sh
mysql -u root -p
```

Create a database called 'faveo'.

```sql
CREATE DATABASE faveo;
```

Create a user called 'faveo' and its password 'strongpassword'.

```sql
CREATE USER 'faveo'@'localhost' IDENTIFIED BY 'strongpassword';
```

We have to authorize the new user on the faveo db so that he is allowed to change the database.

```sql
GRANT ALL ON faveo.* TO 'faveo'@'localhost';
```

And finally we apply the changes and exit the database.

```sql
FLUSH PRIVILEGES;
exit
```

> **NOTE** :
> Please refrain from making direct MySQL/MariaDB modifications. Contact our support team for assistance.

<a id="4-configure-nginx-webserver" name="4-configure-nginx-webserver"></a>

### <strong>4. Configure Nginx webserver</strong>

 <b> 4.a. Give proper permissions to the project directory by running:</b>

```sh
chown -R www-data:www-data /var/www/faveo
cd /var/www/faveo/
find . -type f -exec chmod 644 {} \;
find . -type d -exec chmod 755 {} \;
```

 <b>4.b. Create a copy of Nginx default config file</b>
Finally, configure Nginx site configuration file for Faveo. This file will control how users access Faveo content. Run the commands below to create a new configuration file called faveo

```
nano /etc/nginx/sites-available/faveo
```
Then copy and paste the content below into the file and save it. Replace the highlighted line with your own domain name and directory root location.
```
server {
    listen 80;
    listen [::]:80;
    root /var/www/faveo/public;
    index  index.php index.html index.htm;
    server_name  example.com www.example.com;

     client_max_body_size 100M;

    location / {
        try_files $uri $uri/ /index.php?$query_string;       
    }

    location ~ \.php$ {
               include snippets/fastcgi-php.conf;
               fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
               fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
               include fastcgi_params;
    }
}
```
Save the file and exit.

<b> 4.c. Enable the Faveo and remove the default site</b>
After configuring the VirtualHost above delete the deafult Virtualhost and  enable the Faveo Virtualhost by running the commands below

```sh
rm -f /etc/nginx/sites-available/default /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/faveo /etc/nginx/sites-enabled/
systemctl restart nginx
systemctl restart php8.1-fpm
```

<a id="5-configure-cron-job" name="5-configure-cron-job"></a>

### <strong>5. Configure cron job</strong>

Faveo requires some background processes to continuously run. 
Basically those crons are needed to receive emails
To do this, setup a cron that runs every minute that triggers the following command `php artisan schedule:run`.

[comment]: <Create a new `/etc/cron.d/faveo` file with:>

```sh
(sudo -u www-data crontab -l 2>/dev/null; echo "* * * * * /usr/bin/php /var/www/faveo/artisan schedule:run 2>&1") | sudo -u www-data crontab -
```

<a id="6-redis-installation" name="6-redis-installation"></a>

### <strong>6. Redis Installation</strong>

Redis is an open-source (BSD licensed), in-memory data structure store, used as a database, cache and message broker.

This will improve system performance and is highly recommended.

[Redis installation documentation](/docs/installation/providers/enterprise/debian-redis)

<a id="7-ssl-installation" name="7-ssl-installation"></a>

### <strong>7. SSL Installation</strong>

Secure Sockets Layer (SSL) is a standard security technology for establishing an encrypted link between a server and a client. Let's Encrypt is a free, automated, and open certificate authority.

This is an optional step and will improve system security and is highly recommended.

[Let’s Encrypt SSL installation documentation](/docs/installation/providers/enterprise/debian-nginx-ssl)

[Self Signed SSL Certificate Documentation](/docs/installation/providers/enterprise/self-signed-ssl-debian-nginx/)

<a id="8-install-faveo" name="8-install-faveo"></a>

### <strong>8. Install Faveo</strong>

At this point if the domainname is propagated properly with your server’s IP you can open Faveo in browser just by entering your domainname. You can also check the Propagation update by Visiting this site www.whatsmydns.net.

Now you can install Faveo via [GUI](/docs/installation/installer/gui) Wizard or [CLI](/docs/installation/installer/cli)

<a id="9-faveo-backup" name="9-faveo-backup"></a>

### <strong>9. Faveo Backup</strong>


At this stage, Faveo has been installed, it is time to setup the backup for Faveo File System and Database. [Follow this article](/docs/helper/backup) to setup Faveo backup.

<a id="10-final-step" name="10-final-step"></a>

### <strong>10. Final step</strong>

The final step is to have fun with your newly created instance, which should be up and running to `http://localhost` or the domain you have configured Faveo with.
