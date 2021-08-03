# Quickstart guide to install and configure ownCloud


## Introduction

ownCloud is an open source software that provides cloud hosting solutions with file sharing capabilities allowing your team members to collaborate and contribute content from anywhere and with any device or platform.

This quickstart guide provides instructions on how to:

* Install and configure the ownCloud server
* Configure ownCloud server to allow incoming connections using the server's IP address and port 8080
* Add a user
* Connect to the ownCloud server using a desktop client or a mobile phone (Android and iOS)

## Installing ownCloud

This section explains how to install ownCloud manually. For alternative methods of installing ownCloud, see the following links:
* LINK1
* LINK2

### System requirements
Before installing ownCloud, ensure you have Ubuntu 20.04 LTS installed on your computer. Click [here](https://doc.owncloud.org/server/10.8/admin_manual/installation/system_requirements.html) for a complete list of supported operating systems and softwares.

### Prepare your computer for installation

1. Upgrade and update your operating system and packages. To do this, execute the following command:
'apt update && apt upgrade -y' or
'sudo apt update && upgrade -y
2. Create the 'occ' command helper script that would help in executing 'occ' commands.
        FILE="/usr/local/bin/occ"
        /bin/cat <<EOM >$FILE
        #! /bin/bash
        cd /var/www/owncloud
        sudo -E -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
        EOM
3. Make the 'occ' script executable:
'chmod +x /usr/local/bin/occ'.

### To install ownCloud

1. Install all required packages such as Apache server, the MariaDB database, openSSL, and PHP.
'apt install -y \
  apache2 \
  libapache2-mod-php \
  mariadb-server \
  openssl \
  php-imagick php-common php-curl \
  php-gd php-imap php-intl \
  php-json php-mbstring php-mysql \
  php-ssh2 php-xml php-zip \
  php-apcu php-redis redis-server \
  wget'
2. Install the 'bzip2' file decompression tool as well as other core internet utilities.
'apt install -y \
  ssh bzip2 rsync curl jq \
  inetutils-ping coreutils'
3. Change the document root and restart the Apache service.
'sed -i "s#html#owncloud#" /etc/apache2/sites-available/000-default.conf
service apache2 restart'
4. Create a virtual host configuration file.
        FILE="/etc/apache2/sites-available/owncloud.conf"
	/bin/cat <<EOM >$FILE
	Alias /owncloud "/var/www/owncloud/"

	<Directory /var/www/owncloud/>
  	Options +FollowSymlinks
  	AllowOverride All

	<IfModule mod_dav.c>
  	Dav off
 	</IfModule>

 	SetEnv HOME /var/www/owncloud
 	SetEnv HTTP_HOME /var/www/owncloud
	</Directory>
	EOM
5. Enable the host configuration file and restart the Apache service.
'a2ensite owncloud.conf
service apache2 reload'
6. Configure the database:
'mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; \
GRANT ALL PRIVILEGES ON owncloud.* \
  TO owncloud@localhost \
  IDENTIFIED BY 'password'";'
7. Change the working directory to '/var/www/', download the owncloud tar file, unzip the tar file, and change the ownership of the unzipped owncloud files.
'cd /var/www/
wget https://download.owncloud.org/community/owncloud-10.8.0.tar.bz2 && \
tar -xjf owncloud-10.8.0.tar.bz2 && \
chown -R www-data. owncloud'
8. Install ownCloud, create a database, create user credentials for the database, and create admin credentials for the database.
'occ maintenance:install \
    --database "mysql" \
    --database-name "owncloud" \
    --database-user "owncloud" \
    --database-pass "password" \
    --admin-user "admin" \
    --admin-pass "admin"'
9. Configure ownCloud's trusted domains.
'myip=$(hostname -I|cut -f1 -d ' ')
occ config:system:set trusted_domains 1 --value="$myip"'
Note the IP address of the server displayed in the output of this command. This IP address is used to open the log in page of ownCloud server.
10. Finalize ownCloud installation.
'cd /var/www/
chown -R www-data. owncloud'

ownCloud is now installed on your computer. 



### Logging in to ownCloud
In a supported browser, open the ownCloud log in page by entering the IP address of the server noted after executing the commands in step 9 of the previous section. The following screen is displayed:



## Configure the ownCloud server to allow incoming connections

This section provides instructions to set up the ownCloud server to allow incoming connections from users using the server's IP address.

### To configure the ownCloud server


6. 



