{% include warning.html content="This guide is out-dated as it refers to MariaDB, which is no longer supported." %}

# AzerothCore Debian 12 Quick Install Guide

This is a streamlined guide for installing AzerothCore to a Debian 12 server, securing it, and enabling one-command maintenance from your Windows PC.
## Table of Contents

- [AzerothCore Debian 12 Quick Install Guide](#azerothcore-debian-12-quick-install-guide)
  - [Table of Contents](#table-of-contents)
  - [Requirements](#requirements)
        - [PuTTY](#putty)
        - [FileZilla](#filezilla)
        - [HeidiSQL](#heidisql)
        - [Debian 12](#debian-12)
  - [Debian Setup](#debian-setup)
    - [First Login](#first-login)
    - [Change Default Password](#change-default-password)
    - [Change Default SSH Port](#change-default-ssh-port)
    - [Setup Firewall](#setup-firewall)
    - [Get Dependencies](#get-dependencies)
    - [Install SQL Database](#install-sql-database)
  - [SSH Setup](#ssh-setup)
    - [Key Generation](#key-generation)
      - [Debian Public Key](#debian-public-key)
      - [Windows Private Key](#windows-private-key)
    - [Key-based Login Setup](#key-based-login-setup)
      - [PuTTY](#putty-1)
      - [FileZilla](#filezilla-1)
      - [HeidiSQL](#heidisql-1)
    - [Disable Password Logins](#disable-password-logins)
  - [AzerothCore Installation](#azerothcore-installation)
    - [Clone Repository](#clone-repository)
    - [Add Anticheat Module](#add-anticheat-module)
    - [Get Data Files](#get-data-files)
    - [Build Core](#build-core)
    - [Edit Configs](#edit-configs)
    - [Set Realm IP](#set-realm-ip)
    - [Launch Server](#launch-server)
    - [Create GM account](#create-gm-account)
  - [Maintenance](#maintenance)
    - [Create Alias Command](#create-alias-command)
    - [Update AzerothCore](#update-azerothcore)
  - [Finish!](#finish)
  - [Common Problems](#common-problems)
      - [Auth/Worldserver wont even start.](#authworldserver-wont-even-start)
      - [Successful login but cant enter the realm.](#successful-login-but-cant-enter-the-realm)
        - [Good things to know that this guide does not cover.](#good-things-to-know-that-this-guide-does-not-cover)
  - [Other Resources](#other-resources)

## Requirements

##### [PuTTY](https://www.putty.org/)
- A Windows program for sending commands to the server.
##### [FileZilla](https://filezilla-project.org/download.php)
- A Windows program for transferring files to/from the server.
##### [HeidiSQL](https://www.heidisql.com/)
- A Windows program for connecting to the servers SQL database.
##### [Debian 12](https://www.ovhcloud.com/en-ca/vps/)
- A server with Debian 12 installed. (ex: 4gb/4core VPS from OVH)

---
## Debian Setup
### First Login

- Use **PuTTY** to connect to your Debian server using the IP address and login credentials supplied by the hosting provider.
- Copy the following code blocks and paste them into the PuTTY terminal with right click, then hit enter.
### Change Default Password
```bash
passwd
```
### Change Default SSH Port
```bash
sudo sed -i 's/^#Port 22\+$/Port 55022/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```
- Remember to use 55022 as the SSH port for subsequent connections.
### Setup Firewall
```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 55022
sudo ufw allow 3724
sudo ufw allow 8085
sudo ufw enable
```
### Get Dependencies
```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install git cmake make gcc g++ clang libssl-dev libbz2-dev libreadline-dev libncurses-dev libboost-all-dev mariadb-server mariadb-client libmariadb-dev libmariadb-dev-compat build-essential p7zip-full screen fail2ban -y
```
### Install SQL Database
```bash
sudo mysql_secure_installation
```
- **-/N/Y/Y/Y/Y/Y**
```bash
sudo mysql -u root -p
```
- Enter the root password you just set up.
- This root user has remote access disabled, so we will create a new "acore" user for the SQL database.
```bash
DROP USER IF EXISTS 'acore'@'localhost';
CREATE USER 'acore'@'%' IDENTIFIED BY 'SQLPASSWORD';
GRANT ALL PRIVILEGES ON * . * TO 'acore'@'%';
exit
```
- Change **SQLPASSWORD** to something more secure.
---
## SSH Setup

### Key Generation
#### Debian Public Key
```bash
ssh-keygen -t ed25519 -C "Debian12"
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```
#### Windows Private Key
- Use **Filezilla** to connect to the server and navigate to `home/USERNAME/ssh/`
- Copy the `id_ed25519` file to your PC and load it into **puttygen.exe** (located in the PuTTY folder) 
- Generate a private key `.ppk` file. Store this file somewhere safe and make a backup.
### Key-based Login Setup
#### PuTTY
![PuTTY1](https://github.com/azerothcore/wiki/assets/61268368/6210d43d-a7c4-4444-b896-4f23a2ee415f)
![PuTTY2](https://github.com/azerothcore/wiki/assets/61268368/e39e5a4f-f93f-4a69-9d9c-785bd98afdbd)
#### FileZilla
![FileZilla](https://github.com/azerothcore/wiki/assets/61268368/d45e952a-4f3b-4c38-9cdb-b72f5bc76651)
#### HeidiSQL
![HeidiSQL1](https://github.com/azerothcore/wiki/assets/61268368/9d693ba8-bd49-448c-92b4-2206b7e04e41)
![HeidiSQL2](https://github.com/azerothcore/wiki/assets/61268368/4043857a-2d1e-4c5b-bb61-2d76ed8a5514)
### Disable Password Logins
- **After confirming that key-based login works**, disable password logins to enhance SSH security.
```bash
sudo sed -i -E 's/#?PasswordAuthentication yes/PasswordAuthentication no/' ~/etc/ssh/sshd_config
sudo rm /etc/ssh/sshd_config.d/*
sudo service ssh restart
```

---

## AzerothCore Installation

### Clone Repository
```bash
git -C ~/ clone https://github.com/azerothcore/azerothcore-wotlk.git --branch master --single-branch azerothcore
```
### Add Anticheat Module
```bash
git -C ~/azerothcore/modules clone https://github.com/azerothcore/mod-anticheat
cp -n ~/azerothcore/modules/mod-anticheat/conf/Anticheat.conf.dist ~/server/etc/Anticheat.conf 
```
### Get Data Files
```bash
mkdir -p ~/server/data && cd ~/server/data
wget https://github.com/wowgaming/client-data/releases/download/v16/data.zip
7z x data.zip && rm data.zip
```
### Build Core
```bash
mkdir -p ~/azerothcore/build && cd ~/azerothcore/build
cmake ../ -DCMAKE_INSTALL_PREFIX=$HOME/server/ -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DWITH_WARNINGS=1 -DTOOLS_BUILD=db-only -DSCRIPTS=static -DMODULES=static
make -j $(nproc) install
```
### Edit Configs
```bash
cp -n ~/server/etc/authserver.conf.dist ~/server/etc/authserver.conf
cp -n ~/server/etc/worldserver.conf.dist ~/server/etc/worldserver.conf
sudo sed -i -E 's|^DataDir = .*|DataDir = "/home/USERNAME/server/data"|' ~/server/etc/worldserver.conf
sudo sed -i -E 's/= "127.0.0.1;3306;acore;acore;/= "127.0.0.1;3306;acore;SQLPASSWORD;/' ~/server/etc/*.conf
```
- Change **USERNAME** to your Debian user.
- Change **SQLPASSWORD** to the password for the acore database user.
### Set Realm IP
```bash
sudo mysql -u acore -p
```
- Enter the password for the acore database user.
```sql
UPDATE acore_auth.realmlist SET address = '0.0.0.0' WHERE id = 1;
```
- Change **0.0.0.0** to the public IP address of your Debian12 server.
### Launch Server
```bash
screen -AmdS auth ~/server/bin/authserver
screen -AmdS world ~/server/bin/worldserver
screen -r world
```

### Create GM account
```bash
account create USERNAME PASSWORD
account set gmlevel USERNAME 3 -1
```

- **To exit screen:** Ctrl+A -> Ctrl+D
- **To kill process:** Ctrl+C

---
## Maintenance
### Create Alias Command
```bash
touch ~/.bash_aliases
echo "alias acoreupdate='
screen -S world -p 0 -X stuff "saveall^m";
screen -X -S "world" quit;
git -C ~/azerothcore/modules/mod-anticheat pull;
git -C ~/azerothcore pull;
cd ~/azerothcore/build;
cmake ../ -DCMAKE_INSTALL_PREFIX=$HOME/server/ -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -DWITH_WARNINGS=1 -DTOOLS_BUILD=db-only -DSCRIPTS=static -DMODULES=static;
make -j $(nproc) install;
screen -AmdS world ~/server/bin/worldserver;
screen -r world;'" > ~/.bash_aliases
source ~/.bashrc
```

- Now we can **save/exit** the worldserver, **pull** the latest changes from GitHub, **build** the updated core, and **restart** the worldserver all with one command.
### Update AzerothCore
```
acoreupdate
```

## Finish!

- You should now be able to connect to the AzerothCore server by setting your realmlist to the public IP address of the Debian12 server. ex: `set realmlist 0.0.0.0`

---
## Common Problems

#### Auth/Worldserver wont even start.
- Make sure you matched the password of the acore [SQL user](#install-sql-database) with the one in [the configs](#edit-configs)
#### Successful login but cant enter the realm.
- Double check the [realm address.](#set-realm-ip)

---
##### Good things to know that this guide does not cover.
- Domain name and DNS setup for *"set realmlist logon.server.com"*
- Wordpress registration site & acore-cms plugin SOAP connection.
- Automated database backups to Google Drive using cron and rclone.

## Other Resources
- [Official AzerothCore Installation Guide](installation)
- [heyaapl's Debian Tutorial](digital-ocean-video-tutorial)
- [Digital Scriptorium's Video](https://www.youtube.com/watch?v=k4i4za1Scgg)
