# ownCloud Installation Guide
### Detailed intructions for installing ownCloud server on Ubuntu Server 20.04

---

## Introduction:

### Update & Reboot
- sudo apt-get update && sudo apt-get upgrade -y && sudo reboot

---

## Install Apache2:

- sudo apt-get install apache2 apache2-utils

- sudo systemctl enable apache2
- sudo systemctl start apache2
- sudo systemctl status apache2

---

## Install MariaDB:
