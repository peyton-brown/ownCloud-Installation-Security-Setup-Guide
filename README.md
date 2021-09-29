# ownCloud Installation Guide
#### Detailed intructions for installing ownCloud server on Ubuntu Server 20.04
---

## Prerequisite Setup

### Static IP Address
For steps on setting up a static ip, use [this Linuxize guide](https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/#netplan).

### Update Ubuntu Packages
	sudo apt-get update -y && sudo apt-get upgrade -y

### Install Apache, PHP, MariaDB, and Dependencies
	sudo apt-get install apache2 libapache2-mod-php7.4 openssl php-imagick php7.4-common php7.4-curl php7.4-gd php7.4-imap php7.4-intl php7.4-json php7.4-ldap php7.4-mbstring php7.4-mysql php7.4-pgsql php-ssh2 php7.4-sqlite3 php7.4-xml php7.4-zip mariadb-server unzip smbclient openssh-server curl wget -y; sudo apt-get remove certbot -y; sudo snap install core; sudo snap refresh core; sudo snap install --classic certbot

### Start and Enable Apache to run on Startup
	sudo ufw allow 'Apache Secure'
	sudo systemctl start apache2
	sudo systemctl enable apache2
	sudo systemctl status apache2

### Virtual Hosts
Follow [DigitalOcean's guide](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04#step-5-%E2%80%94-setting-up-virtual-hosts-recommended) for setting up virtual hosts with Apache. The steps are also included in the [virtual_hosts.md](https://github.com/peyton-brown/ownCloud-Installation-Security-Setup-Guide/blob/production/virtual_hosts.md) file. This is only really needed if you plan on having multiple sites on one domain and/or server. For example, a portfolio website and an ownCloud server on the same www.example.com website.

---

## Domains

#### A domain will be needed for access outside of your network. I use Google Domains but any provider will work. 
Create a custom record inside of DNS. The hostname can be any name, but for most people, the first record should be left blank. This will auto-fill to *yourdomain*.com. The type ***has*** to be "A" unless you are using IPv6; in that case use "AAAA". The TTL default of "3600" is fine. For Data, enter your public IPv4 address, ***not your local Ubuntu-Server IP address***. If using IPv6, enter that instead. Do not give your public IP address to anyone you do not trust. This is why domains are important. Create a second record and follow the same previous steps; however, for the hostname, enter "www". Save when completed.

![Google Domains DNS Setup](https://i.imgur.com/bpuxroA.png)

---

## Run MariaDB
	sudo mysql_secure_installation

#### Promt Answers:
	Set root password? [Y/n] y
		New password: pwd
	Remove anonymous users? [Y/n] y
	Disallow root login remotely? [Y/n] y
	Remove test database and access to it? [Y/n] y
	Reload privilege tables [Y/n] y

### Create the ownCloud database from the MySQL console:
	sudo mysql -u root -p
######
	CREATE DATABASE owncloud_db;
	GRANT ALL ON owncloud_db.* TO 'owncloud_db_user'@'localhost' IDENTIFIED BY 'qwe';
	FLUSH PRIVILEGES;
	EXIT;

---

## Download & Install ownCloud
### [ownCloud Downloads](https://owncloud.com/older-versions/#server)
	cd /tmp
	sudo wget https://download.owncloud.org/community/owncloud-complete-20210721.zip
	sudo unzip owncloud-complete-20210721.zip -d /var/www/html/

### ownCloud Permissions
	sudo chown -R www-data:www-data /var/www/html/owncloud/
	sudo chmod -R 755 /var/www/html/owncloud/

---

## Backing up ownCloud
### When you backup your ownCloud server, there are four things that you need to copy:
	config/ directory
	data/ directory
	ownCloud database
	theme files (if applicable)

### Make a Backup Directory
	mkdir -p /owncloud-backups/owncloud-db-backups; mkdir -p /owncloud-backups/config-data

### Backup config/ and data/ Directories: 
Simply copy your config/ and data/ folder to a place outside of your ownCloud environment. This example uses rsync to copy the two directories to /oc-backupdir. rsync is just one method, use whatever you like.
	cd /var/www/html/owncloud
	rsync -Aax config data /owncloud-backups/config-data

### Backup the Database
	cd /owncloud-backups/owncloud-db-backups/
	mysqldump --single-transaction -h localhost -u owncloud_db_user -p qwe owncloud_db > owncloud-dbbackup_`date +"%Y%m%d"`.bak

[Source](https://doc.owncloud.com/server/10.7/admin_manual/maintenance/backup.html)

---

## Restoring ownCloud Backup

[Source](https://doc.owncloud.com/server/10.7/admin_manual/maintenance/restore.html)

---

## Updating ownCloud
### Enable Maintenance Mode
	cd /var/www/html/owncloud/
	sudo -u www-data php occ maintenance:mode --on

### Stop Apache
	sudo service apache2 stop

### Backup ownCloud
Follow the previous steps of backing up ownCloud [here]().

### Review Third-Party Apps
Review any installed third-party apps for compatibility with any new ownCloud release. Ensure that they are all disabled before beginning the upgrade.
#### Disable via Browser
	Go to Settings -> Admin -> Apps and disable all third-party apps.

### Move Current ownCloud Directory
Although you have already made a backup, move your current ownCloud directory to a different location for easy access later:
#### Move ownCloud Directory
	sudo mv /var/www/html/owncloud /var/www/html/backup_owncloud

### Download Latest Version
#### Download the latest [ownCloud server](https://owncloud.com/older-versions/#server) release to where your previous installation was (e.g. /var/www/). Replace the following url and zip folder name with the newest version at the time of reading.
	sudo wget https://download.owncloud.org/community/owncloud-complete-20210721.zip
	sudo unzip owncloud-complete-20210721.zip -d /var/www/html/

### Copy Old Configuration Files to Updated ownCloud Install & Set Permissions
	sudo cp /var/www/html/backup_owncloud/config/config.php /var/www/owncloud/config/config.php
	sudo mv /var/www/html/backup_owncloud/data /var/www/html/owncloud/data
	sudo chown -R www-data:www-data /var/www/html/owncloud






[Source](https://doc.owncloud.com/server/10.7/admin_manual/maintenance/manual_upgrade.html)
---

## Configure Apache for ownCloud

### Disable Default Apache Configuration
	sudo a2dissite 000-default

### This file will control how users access ownCloud content. Copy the configuation code from [owncloud.conf](https://github.com/peyton-brown/ownCloud-installation-guide/blob/main/owncloud.conf) and paste into the following file. You will need to change example.com to whichever domain name you have.
	sudo vim /etc/apache2/sites-available/owncloud.conf

### Enable the new Apache Configuration
	sudo a2ensite owncloud.conf

### Enable Required Apache Modules
	sudo a2enconf owncloud; sudo a2enmod rewrite; sudo a2enmod headers; sudo a2enmod env; sudo a2enmod dir; sudo a2enmod mime; sudo a2enmod ssl

	sudo systemctl restart apache2

--- 

## SSL / Let's Encrypt

### Now that Certbot is installed, run this command to receive your certificate. Enter your personal information into the prompt.
	sudo certbot --apache

### Automatic Renewal
	sudo certbot renew --dry-run

---

## Finalizing the ownCloud Installation

### Go to your browser and type your IP into the address bar followed by /owncloud
#### Example: 
	http://www.yourdomain.com/owncloud
	192.168.1.254/owncloud
Enter a Username & Password for the main adminstator
![username and password](https://i.imgur.com/LOKsV74.png)


Select "Storage & database", select "MySQL/MariaDB", fill in the information, and select "Finish setup"

![storage and database](https://i.imgur.com/PK8ooYs.png)

---