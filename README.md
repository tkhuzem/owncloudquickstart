# Quickstart guide to install and configure ownCloud


## Introduction

ownCloud is an open source software that provides cloud hosting solutions with file sharing capabilities allowing your team members to collaborate and contribute content from anywhere and with any device or platform.

This quickstart guide provides instructions on how to:

* Install and configure the ownCloud server
* Configure ownCloud to allow users to connect to the server using the server's IP address and port 8080
* Add a user
* Connect to the ownCloud server using a desktop client or a mobile phone (Android and iOS)

## Installing ownCloud

This section explains how to install ownCloud manually. For alternative methods of installing ownCloud, see the following links:
* [Installing ownCloud using `docker`](https://doc.owncloud.com/server/10.8/admin_manual/installation/docker/)
* 

### System requirements
Before installing ownCloud, ensure you have Ubuntu 20.04 LTS installed on your computer. Click [here](https://doc.owncloud.org/server/10.8/admin_manual/installation/system_requirements.html) for a complete list of supported operating systems and softwares.

### Prepare your computer for installation

1. Upgrade and update your operating system and packages. To do this, execute the following command:

	`apt update && apt upgrade -y`

2. Create the `occ` command helper script that would help in executing `occ` commands:

        FILE="/usr/local/bin/occ"
        /bin/cat <<EOM >$FILE
        #! /bin/bash
        cd /var/www/owncloud
        sudo -E -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
        EOM 

3. Make the `occ` script executable:

	`chmod +x /usr/local/bin/occ`.

### To install ownCloud

1. Install all required packages such as Apache server, the MariaDB database, openSSL, and PHP:

	`apt install -y \ ` <br>
	`apache2 \` <br>
	`libapache2-mod-php \ ` <br>
	`mariadb-server \` <br>
	`openssl \ ` <br>
	`php-imagick php-common php-curl \` <br>
  	`php-gd php-imap php-intl \` <br>
  	`php-json php-mbstring php-mysql \ ` <br>
  	`php-ssh2 php-xml php-zip \ ` <br>
  	`php-apcu php-redis redis-server \ ` <br>
  	`wget ` <br>

2. Install all the recommended packages such as `bzip2` file decompression tool, `ssh`, `curl` and other core internet utilities:

	`apt install -y \ ` <br> 
	`ssh bzip2 rsync curl jq \ ` <br>
  	`inetutils-ping coreutils`  <br>
	
3. Change the document root and restart the Apache service:

	`sed -i "s#html#owncloud#" /etc/apache2/sites-available/000-default.conf` <br>
	`systemctl restart apache2.service`
	
4. Create a virtual host configuration file:

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
	
5. Enable the host configuration file and restart the Apache service:

	`a2ensite owncloud.conf` <br>
	`systemctl restart apache2.service`

6. Configure the database:

	`mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; \ ` <br>
	`GRANT ALL PRIVILEGES ON owncloud.* \ ` <br>
  	`TO owncloud@localhost \ ` <br>
	`IDENTIFIED BY 'password'";`

7. Change the working directory to `/var/www/`, download the owncloud tar file, unzip, and change the ownership of the unzipped owncloud files:

	`cd /var/www/` <br>
	`wget https://download.owncloud.org/community/owncloud-10.8.0.tar.bz2 && \` <br>
	`tar -xjf owncloud-10.8.0.tar.bz2 && \ ` <br>
	`chown -R www-data. owncloud`

8. Install ownCloud, create a database, create user credentials for the database, and create admin credentials for the database:

	`occ maintenance:install \ ` <br>
	`--database "mysql" \ ` <br>
	`--database-name "owncloud" \ ` <br>
	`--database-user "owncloud" \ ` <br>
	`--database-pass "password" \ ` <br>
	`--admin-user "admin" \ ` <br>
	`--admin-pass "admin"` <br>

9. Configure ownCloud's trusted domains:

	`myip=$(hostname -I|cut -f1 -d ' ') ` <br>
	`occ config:system:set trusted_domains 1 --value="$myip"` <br>

> Note the IP address of the server displayed in the output of this command. This IP address is used to open the log in page of ownCloud server.

10. Finalize the ownCloud installation:

	`cd /var/www/ ` <br>
	`chown -R www-data. owncloud` <br>

ownCloud is now installed on your computer. 

### Logging in to ownCloud
In a supported browser, open the ownCloud log in page by entering the IP address of the server noted after executing the commands in step 9 of the previous section. The following screen is displayed:



## Configure the ownCloud server to allow incoming connections

This section provides instructions to set up the ownCloud server to allow incoming connections from users using the server's IP address. The first step is to enable SSL and then port fowarding to port 8080.

### To enable SSL on the ownCloud server

To allow external connections to the ownCloud server, you must enable SSL for secure access. 
Execute the following steps in a terminal window:

1. Execute the following command to enable SSL:

	`a2enmod ssl`

2. Create a directory for the self-signed certificate:

	`mkdir /etc/apache2/ssl`

3. Create the self-signed certificate and the key and save both in the newly created directory:

	`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/owncloud.key -out /etc/apache2/ssl/owncloud.crt`

> This self-signed certificate is valid for 365 days, as mentioned in the command.

4. Execute the following command to modify some values in the certificate:

	`nano /etc/apache2/sites-available/default-ssl­.conf`

5. Modify the following values:
	
        ServerName IP :443
        SSLEngine on
        SSLCertificateFile /etc/apache2/ssl/owncloud.crt
        SSLCertificateKeyFile /etc/apache2/ssl/owncloud.key 

6. Execute the following command to activate the new virtual host:

	`a2ensite default-ssl`

7. Restart the Apache service:

	`systemctl restart apache2.service`

### To configure port forwarding to port 8080

1. Log in to your router and note the WAN IP address.
2. Open the owncloud config file to add the WAN IP address:

	`nano /var/www/html/owncloud/config/config.php`

3. In the owncloud config file, in the trusted domains array, add an entry for the WAN IP address.
4. Change the value of `overwrite.cli.url` with the WAN IP address.
5. Log in to your router and navigate to the port forwarding section.
6. Forward the SSL port 443 to the Ubuntu server running the ownCloud instance internal IP (LAN IP) address and save settings.

## Adding a user

You can add users to your ownCloud server who can share their files and content with the server adminsitrator and other users who are subscribing to the services in the ownCloud server.

### To add a user

1. In a supported browser, open the admin log in page and enter the credentials.
2. In the upper left corner of the screen, click **Admin -> Users**. A list of users is displayed.
3. Enter the **Username** and **E-mail** address of the user you want to add.
4. From the dropdown list, select the user group to which you want to add the user. To create a new one click **+ add group**.
5. Click **Create**. 
   A new user is now added to the ownCloud server.
   
## Accessing the ownCloud server from a desktop client
After you have configured the ownCloud server to be accessed from externally, you can configure your desktop client and also your mobile phones to connect to the ownCloud server.

### Accessing ownCloud server from a desktop client

1. Download the ownCloud desktop client from [here](https://owncloud.com/desktop-app).
2. In the **Setup ownCloud Server** page, enter the IP address of the ownCloud server and click **Next**.
3. Enter the credentials and click **Next**.
4. In the **Setup local folder options** dialog, select what to sync from the server. You can sync all the contents of the server or limited set of files and folders.
5. Select the local folder on your computer that will be synced with the ownCloud server and click **Connect**.

### Accessing ownCloud server from a mobile phone
1. Download the ownCloud app from the Google Playstore or Apple store and install the app.
2. On first launch, the app prompts for the server IP and the credentials. Enter the required information and click **Connect**.
 
