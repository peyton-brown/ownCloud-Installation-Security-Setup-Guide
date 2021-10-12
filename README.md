# ownCloud Installation & Setup Guide
Ubuntu Server 20.04 on VirtualBox VM

## Prerequisite Setup

### Static IP Address (If using VirtualBox, make sure Bridged Adapters are being used.)
For steps on setting up a static ip, use [this Linuxize guide](https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/#netplan).

### Update Ubuntu Packages
```
sudo apt-get update -y && sudo apt-get upgrade -y
```

### Install Apache, PHP, MariaDB, and other Dependencies
```
sudo apt-get install apache2 libapache2-mod-php7.4 openssl php-imagick php7.4-common php7.4-curl php7.4-gd php7.4-imap php7.4-intl php7.4-json php7.4-ldap php7.4-mbstring php7.4-mysql php7.4-pgsql php-ssh2 php7.4-sqlite3 php7.4-xml php-redis php7.4-zip p7zip p7zip-full unrar redis-server mariadb-server mariadb-client unzip smbclient openssh-server curl wget -y; sudo apt-get remove certbot -y; sudo snap install core; sudo snap refresh core; sudo snap install --classic certbot
```

### Allow Port 443 Through Firewall & Start Apache on Startup
```
sudo ufw allow 'Apache Secure'
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

## Domains
A domain will be needed for secure access outside of your network, as well as, HTTPS. I use Google Domains but any provider will work. Create a custom record inside of "DNS". The hostname can be any name, but for most people, the first record should be "owncloud". This will set the URL to *owncloud.example*.com. The type needs to be "A" unless you are using IPv6, in that case use "AAAA". The TTL default of "3600" is fine. For Data, enter your public IPv4 address, ***not the local Ubuntu-Server IP address***. If using IPv6, enter that instead. Do not give your public IP address to anyone you do not trust. This is why a domain is important.

![Google Domains DNS Setup](https://i.imgur.com/FhMaV0c.png)

## MariaDB for Database

### Setup MariaDB
```
sudo mysql_secure_installation

	Set root password? [Y/n] y
		New password: (any password will work)
	Remove anonymous users? [Y/n] y
	Disallow root login remotely? [Y/n] y
	Remove test database and access to it? [Y/n] y
	Reload privilege tables [Y/n] y
```

### Create the ownCloud Database
```
sudo mysql -u root -p
```

#### Enter the following into MariaDB. Make sure you change the default password from "qwe" to something else.
```
CREATE DATABASE owncloud_db;
GRANT ALL ON owncloud_db.* TO 'owncloud_db_user'@'localhost' IDENTIFIED BY 'qwe';
FLUSH PRIVILEGES;
EXIT;
```

## Download & Set Permissions for ownCloud

### ownCloud [Download](https://owncloud.com/older-versions/#server)
```
cd /tmp
sudo wget https://download.owncloud.org/community/owncloud-complete-20210721.zip
sudo unzip owncloud-complete-20210721.zip -d /var/www/
```

### ownCloud Permissions
```
sudo chown -R www-data:www-data /var/www/owncloud/
sudo chmod -R 755 /var/www/owncloud/
```

## Apache Configuration 

### Disable the Default Apache Configuration
```
sudo a2dissite 000-default
```

### Apache owncloud.conf File
This file will control how users access ownCloud content. Copy the configuation code from [owncloud.conf](https://github.com/peyton-brown/ownCloud-installation-guide/blob/main/owncloud.conf) and paste into the following file. You will need to change example.com to whichever domain name you have.
```
sudo vim /etc/apache2/sites-available/owncloud.conf
```

### Enable the new owncloud.conf Apache Configuration
```
sudo a2ensite owncloud.conf
```

### Enable Required Apache Modules
```
sudo a2enconf owncloud; sudo a2enmod rewrite; sudo a2enmod headers; sudo a2enmod env; sudo a2enmod dir; sudo a2enmod mime; sudo a2enmod ssl
sudo systemctl restart apache2
```

## Memory Caching / Transactional File Locking

### Copy the configuation code from [redis-config](https://github.com/peyton-brown/ownCloud-Installation-Security-Setup-Guide/blob/main/redis-config) and paste at the bottom of config.php. You should change the password in this file.
```
sudo vim /var/www/owncloud/config/config.php
```

## Strict Transport Security HTTP Header

### Access the SSL.conf file. This should be named owncloud-ssl.conf or owncloud-le-ssl.conf
```
sudo vim /etc/apache2/sites-available/owncloud-le-ssl.conf
```

### Paste the following below "ServerName"
```
Header always add Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"
```

## SSL / Let's Encrypt

### Run this command to receive your certificate. Enter your personal information into the prompt.
```
sudo certbot --apache

	Enter email address (used for urgent renewal and security notices)
 		(Enter 'c' to cancel): (YOUR EMAIL)


	Please read the Terms of Service at https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must agree in order to register with the ACME server. Do you agree?
		(Y)es/(N)o: y


	Would you be willing, once your first certificate is successfully issued, to
	share your email address with the Electronic Frontier Foundation, a founding
	partner of the Let's Encrypt project and the non-profit organization that
	develops Certbot? We'd like to send you email about our work encrypting the web,
	EFF news, campaigns, and ways to support digital freedom.
		(Y)es/(N)o: y


	Which names would you like to activate HTTPS for?
	- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
	1: owncloud.example.com
	- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
		Select the appropriate numbers separated by commas and/or spaces, or leave input blank to select all options shown (Enter 'c' to cancel): 1
```

### Turn on Automatic Renewal with the following command.
```
sudo certbot renew --dry-run; sudo systemctl restart apache2
```

[Source](https://certbot.eff.org/lets-encrypt/ubuntufocal-apache)

## Finalizing the ownCloud Installation

### Go to a Browser and Type URL into the Address Bar
```
Example: 
	https://owncloud.example.com/
```

Enter a Username & Password for the main adminstator
![username and password](https://i.imgur.com/LOKsV74.png)


Select "Storage & database", select "MySQL/MariaDB", fill in the information, and select "Finish setup"

![storage and database](https://i.imgur.com/PK8ooYs.png)

## Useful Market Apps
```
Activity
Announcement Center
Calendar
Contacts
Custom Groups
E2EE File Sharing
Extract
Files clipboard
Gallery
Impersonate
Music
Password Policy
PDF Viewer
Text Editor
Text File Viewer
Wallpaper 

--- 
	DO NOT USE IF OUTLOOK PLUGIN IS OR WILL BE INSTALLED
		2-Factor Authentication  
		Two factor backup codes
---

```


## Backing up ownCloud

### Make Backup Directory for DB & Config Files
```
mkdir -p /owncloud-backups/owncloud-db-backups; mkdir -p /owncloud-backups/config-data
```

### Backup Config & Data Folders
```
cd /var/www/owncloud
rsync -Aax config data /owncloud-backups/config-data
```

### Database Backup
If you changed any of the default database information (*which you should've*), you will also need to change the following.
```
cd /owncloud-backups/owncloud-db-backups/
mysqldump --single-transaction -h localhost -u owncloud_db_user -p qwe owncloud_db > owncloud-db-backup_`date +"%Y-%m-%d"`.bak
```

[Source](https://doc.owncloud.com/server/10.7/admin_manual/maintenance/backup.html)


## Restoring ownCloud Backup

Needs to be added

[Source](https://doc.owncloud.com/server/10.7/admin_manual/maintenance/restore.html)


## Updating ownCloud

### Review & Disable Third-Party Apps via Browser 
Review any installed third-party apps for compatibility with all new ownCloud release. Ensure that they are all disabled before beginning the upgrade.
```
Go to Settings -> Admin -> Apps and disable all third-party apps.
```

### Enable Maintenance Mode
```
cd /var/www/owncloud/
sudo -u www-data php occ maintenance:mode --on
sudo systemctl stop apache2
```

### Backup ownCloud Directories
Follow these steps of backing up ownCloud [here](https://github.com/peyton-brown/ownCloud-Installation-Security-Setup-Guide#backing-up-owncloud).


### Move Current ownCloud Directory
```
sudo mv /var/www/owncloud /var/www/backup_owncloud
```

### Download Latest Version
Download the latest [ownCloud server release](https://owncloud.com/older-versions/#server) to where your previous installation was (e.g. /var/www/). Replace the following url and zip name with the newest version at the time of reading.
```
cd /tmp; sudo wget https://download.owncloud.org/community/owncloud-complete-20210721.zip
sudo unzip owncloud-complete-20210721.zip -d /var/www/
```

### Copy the Old Configuration Files to the Updated ownCloud Download
```
sudo cp /var/www/backup_owncloud/config/config.php /var/www/owncloud/config/config.php; sudo cp /var/www/backup_owncloud/data /var/www/owncloud/data; sudo cp -r /var/www/backup_owncloud/apps/ /var/www/owncloud/apps/; sudo cp -r /var/www/backup_owncloud/apps-external/ /var/www/owncloud/apps-external/
```

### Set Permissions
```
sudo chown -R www-data:www-data /var/www/owncloud
```

### Start the Upgrade Process
```
cd /var/www/owncloud
sudo -u www-data php occ upgrade
```

The upgrade can take anywhere from a few seconds to a few minutes, depending on the size of your installation. When it is finished you will see either a success message or an error message that indicates why the update did not complete.

### Assuming your upgrade succeeded, disable maintenance mode.
```
sudo -u www-data php occ maintenance:mode --off
sudo service apache2 start
```

### Check if the Update Applied
Check that the version number reflects the new installation. 
```
It can be reviewed at the bottom of Settings -> Admin -> General.
```

[Source](https://doc.owncloud.com/server/10.7/admin_manual/maintenance/manual_upgrade.html)


## Local External Storage / VirtualBox Shared Folder

Needs to be added

https://tolotra.com/2018/07/28/how-to-install-ubuntu-server-on-virtualbox-with-shared-folder-and-ssh/

https://doc.owncloud.com/server/next/admin_manual/configuration/files/external_storage/local.html


## Outlook Intergration

For a basic overview of Outlook intergration with ownCloud, read [this](https://owncloud.com/features/outlook-plugin/).

Go to the [FAQ](https://www.epikshare.com/faq/) page on epiKshare's website. This will give you all the information that is needed to get the plugin to work.

```
You can find the download for the plugin under "How is the Outlook plugin installed?". 		

Click "Download the Outlook plugin"
Click ownCloud - Microsoft Outlook AddIn
Click the three dots next to "Office 2013 - 2016 - 2019 - O365" and select "Download" 
Save the zip folder
Extract the zip file.
Run the .exe file and follow the prompts to install the Outlook plugin

Once finished, restart Outlook.
```

Once Outlook has restarted, click "ownCloud" on the ribbon bar. Click "Settings" and follow the prompts. You will need to have DNS and HTTPS setup for this to work, if you do not have this, follow [Domain/DNS Setup](https://github.com/peyton-brown/ownCloud-Installation-Security-Setup-Guide/tree/production#domains) and/or [SSL/HTTPS Setup](https://github.com/peyton-brown/ownCloud-Installation-Security-Setup-Guide/tree/production#ssl--lets-encrypt). Once both are compeleted, enter your URL and click next. Select your type of authentication or if you just have none, select "Username and Password". Once signed-in you will need to restart Outlook for the changes to take effect. 

From here, you can change the settings to include templates, share settings, and share duration defaults. To test if the plugin is working, email yourself with an attachment. You can either drag and drope a file or by clicking "Create Upload-Link". The ladder is preferable as you can change settings (like share duration) individually rather than leaving it to the defaults.

This plugin is free for the first 30 days, after that you can purchase a license [here](https://oc.oem-cloud.com/en/owncloud-outlook-plugin-annual-license). 

```
Click English at the top
Select "Product-License"
Choose which plan you would like and click Next
Enter your domain name below "1-Year Subscription per User", for example, https://owncloud.example.com. 

Note: This is the price for each email address conntected to the plugin. If you add a different email address, it will charge you the same amount as the first license (e.g. €10,00 / Year).
```