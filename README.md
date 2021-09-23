# ownCloud Installation Guide
#### Detailed intructions for installing ownCloud server on Ubuntu Server 20.04

---

## Update Ubuntu System Packages
- sudo apt-get update -y && sudo apt-get upgrade -y

---

## Install Apache, PHP, and MariaDB
- sudo apt-get install apache2 libapache2-mod-php7.4 openssl php-imagick php7.4-common php7.4-curl php7.4-gd php7.4-imap php7.4-intl php7.4-json php7.4-ldap php7.4-mbstring php7.4-mysql php7.4-pgsql php-ssh2 php7.4-sqlite3 php7.4-xml php7.4-zip mariadb-server unzip curl wget -y

### Start and Enable Apache to run on Startup
	- sudo systemctl start apache2
	- sudo systemctl enable apache2
	- sudo systemctl status apache2

---

## Run MariaDB
- sudo mysql_secure_installation

#### Promt Answers:
	- Set root password? [Y/n] y
		- New password: pwd
	- Remove anonymous users? [Y/n] y
	- Disallow root login remotely? [Y/n] y
	- Remove test database and access to it? [Y/n] y
	- Reload privilege tables [Y/n] y

### Create the ownCloud database from the MySQL console:
- sudo mysql -u root -p
#
	- CREATE DATABASE owncloud_db;
	- GRANT ALL ON owncloud_db.* TO 'owncloud_db_user'@'localhost' IDENTIFIED BY 'db_password';
	- FLUSH PRIVILEGES;
	- EXIT;

---

## Download & Install OwnCloud
#### [ownCloud Downloads](https://owncloud.com/download-server/)
- cd /tmp ; sudo wget https://download.owncloud.org/community/owncloud-complete-20210721.zip ; sudo unzip owncloud-complete-20210721.zip -d /var/www/
