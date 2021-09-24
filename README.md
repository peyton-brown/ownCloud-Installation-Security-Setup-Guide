# ownCloud Installation Guide
#### Detailed intructions for installing ownCloud server on Ubuntu Server 20.04

---

## Update Ubuntu System Packages
sudo apt-get update -y && sudo apt-get upgrade -y

---

## Install Apache, PHP, and MariaDB
sudo apt-get install apache2 libapache2-mod-php7.4 openssl php-imagick php7.4-common php7.4-curl php7.4-gd php7.4-imap php7.4-intl php7.4-json php7.4-ldap php7.4-mbstring php7.4-mysql php7.4-pgsql php-ssh2 php7.4-sqlite3 php7.4-xml php7.4-zip mariadb-server unzip smbclient certbot curl wget -y

### Start and Enable Apache to run on Startup
	sudo systemctl start apache2
	sudo systemctl enable apache2
	sudo systemctl status apache2

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
### [ownCloud Downloads](https://owncloud.com/download-server/)
	cd /tmp
	sudo wget https://download.owncloud.org/community/owncloud-complete-20210721.zip 
	sudo unzip owncloud-complete-20210721.zip -d /var/www/

### ownCloud Permissions
	sudo chown -R www-data:www-data /var/www/owncloud/
	sudo chmod -R 755 /var/www/owncloud/

---

## Configure Apache for ownCloud
### Copy the configuation code from [owncloud.conf](https://github.com/peyton-brown/ownCloud-installation-guide/blob/main/owncloud.conf) and paste into the following file
sudo vim /etc/apache2/conf-available/owncloud.conf

### Enable Required Apache Modules
	sudo a2enconf owncloud
	sudo a2enmod rewrite
	sudo a2enmod headers
	sudo a2enmod env
	sudo a2enmod dir
	sudo a2enmod mime
	sudo a2enmod ssl

	sudo systemctl restart apache2

--- 

## Finalizing the ownCloud Installation

### Go to your browser and type your IP into the address bar followed by /owncloud
Enter a Username & Password for the main adminstator
![username and password](https://i.imgur.com/LOKsV74.png)


Select "Storage & database", select "MySQL/MariaDB", fill in the information, and select "Finish setup"

![storage and database](https://i.imgur.com/PK8ooYs.png)

---

## SSL / Let's Encrypt

### A domain will be needed for access outside of your network. I use Google Domains but any provider will work. 
**Important:** Create a custom record inside of DNS. Hostname can be any name, for this example, I used "owncloud". Type ***has*** to be "A". TTL's default of "3600" is fine. For Data, enter your public IPv4 address, ***not your local Ubuntu-Server ip***. Do not give your public ip address to anyone you do not trust. This is why the domain is important. Save when completed.

&nbsp;

![Google Domains DNS Setup](https://i.imgur.com/AbOyF9f.png)

&nbsp;

### Change Default Apache Root Directory
**Important:** This method assumes that ownCloud is the only Apache program being used on the server. If you have a different program that uses apache, either create a new virtual machine or skip this step and type "owncloud.yourdomain.com/owncloud" into the address bar. With this method, you will simply type "owncloud.yourdomain.com".

	1. Change default directory:
		- sudo vim /etc/apache2/sites-available/000-default.conf
	
	2. Edit the DocumentRoot option:
		- DocumentRoot /var/www/owncloud

	3. Restart Apache:
		- sudo service apache2 restart

---