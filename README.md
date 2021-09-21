# ownCloud Installation Guide
#### Detailed intructions for installing ownCloud server on Ubuntu Server 20.04

---

## Introduction:

### Update & Reboot
	- sudo apt-get update && sudo apt-get upgrade -y && sudo reboot

---

## Install Apache2:
	- sudo apt-get install apache2 apache2-utils

### Start Apache
	- sudo systemctl enable apache2
	- sudo systemctl start apache2
	- sudo systemctl status apache2

---

## Install MariaDB:

	- sudo apt-get install mariadb-server

### Start MariaDB
	- sudo systemctl status mariadb
	- sudo systemctl enable mariadb
	- sudo systemctl start mariadb

### Run MariaDB
	- mysql_secure_installation

#### Promt Answers:
	- Set root password? [Y/n] y
	- Remove anonymous users? [Y/n] y
	- Disallow root login remotely? [Y/n] y
	- Remove test database and access to it? [Y/n] y
	- Reload privilege tables [Y/n] y

#### Create the ownCloud database from the MySQL console:
	- mysql -u root -p

##### Create the MySQL ownCloud database:
	- CREATE DATABASE ownclouddb;

##### Create a new user. Replace password by the appropriate value:
	- CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'password';

##### Grant the new user with the necessary privileges:
	- GRANT ALL ON ownclouddb.* TO 'ownclouduser'@'localhost' WITH GRANT OPTION;

##### Enable changes by flushing the privileges:
	- FLUSH PRIVILEGES;

##### Finally, exit the MySQL console:
	- exit