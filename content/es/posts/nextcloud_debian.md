---
title: "NEXTCLOUD DEBIAN SERVER AND CLIENT"
date: 2017-06-24
draft: false
description: "Servidor NEXTCLOUD DEBIAN y CLIENTE"
hideToc: false
enableToc: false
enableTocContent: false
author: HotspotCepeda 
authorEmoji: "🗻"
tags: 
- docker
- nextcloud
- debian
- linux
image: images/feature1/nextcloud_logo.png
---
![Nextcloud custom login server](/images/post/nextcloud_custom.png)

ADD NEXTCLOUD REPOSITORY
Add the Nextcloud repository in the /etc/apt/sources.d/nextcloud.list
``` bash
echo 'deb http://apt.jurisic.org/debian/ jessie main contrib non-free' >> /etc/apt/sources.list.d/nextcloud.list
```
Install release key of Nextcloud repository:
``` bash
wget -q http://apt.jurisic.org/Release.key -O- | apt-key add -
```
And run apt-get update to download the list of packages.
``` bash
apt-get update
```
INSTALL NEXTCLOUD CLIENT
Install Nextcloud server with:
``` bash
apt-get install nextcloud-client
```
INSTALL NEXTCLOUD SERVER
INSTALL POSTGRESQL AND SETUP NEW USER & DATABASE
Install PostgreSQL with:
``` bash
apt-get install postgresql
```
- The PostgreSQL package will install all required dependencies.
- Choose «y» to start the installation.
- Add a PostgreSQL database
- The next step is to create a PostgreSQL database for nextcloud.
- Login to PostgreSQL on the commandline by running this command:
``` bash
su - postgres
psql
```
Enter the following commands to create a database user with the name «nextuser» and and a database name «nextcloud». Replace the word «StorngPasswordHere» with your own password in the commands.
``` bash
CREATE DATABASE nextcloud;
CREATE USER nextuser WITH PASSWORD 'StorngPasswordHere';
GRANT ALL PRIVILEGES ON DATABASE nextcloud to nextuser;
```
Install Nextcloud server with:
``` bash
apt-get install nextcloud-server
```
When the shell part of the installation is finished, proceed by opening the Nextcloud web installer in your browser. The URL is http://[YOURIP]/nextcloud. In my case the IP is 192.168.1.100 so I enter http://192.168.1.100/nextcloud in my browser to get the installer:

Enter the desired administrator username and password in the login fields. Please choose a secure password and also a username that is not «admin» or «administrator» might be a good choice to make it less easy for attackers to guess your admin login.

Nextcloud is using sqlite as storage engine by default. This is not a good choice performace wise, so I will choose PostgreSQL as database backend. We have created a PostgreSQL database above, enter the details of that database now:

Username:       nextuser
Password:       The password that you have choosen for the database.
Database name:  nextcloud
Hostname:       localhost

ACCESS NEXTCLOUD WITH SSL (HTTPS)
The default installation of Nextcloud is not secured by SSL. To enable SSL in your webserver, run these commands:
``` bash
a2enmod ssl
a2ensite default-ssl
service apache2 restart
```
Now you can access the web interface with https://[YOURIP]/nextcloud. You will probably get a SSL warning, this warning should be accepted. To avoid such warnings, get a free officially signed SSL certificate e.g. from Startssl (or check LetsEncrypt).

LETSENCRYPT
STEP 1 — DOWNLOAD THE LET’S ENCRYPT CLIENT
The first step to using Let’s Encrypt to obtain an SSL certificate is to install the certbot software on your server. The Certbot developers maintain their own Ubuntu software repository with up-to-date versions of the software. Because Certbot is in such active development it’s worth using this repository to install a newer Certbot than provided by Ubuntu.

First, add the repository:
``` bash
sudo add-apt-repository ppa:certbot/certbot
```
You’ll need to press ENTER to accept. Afterwards, update the package list to pick up the new repository’s package information:
``` bash
sudo apt-get update
```
And finally, install Certbot from the new repository with apt-get:
``` bash
sudo apt-get install python-certbot-apache
```
The certbot Let’s Encrypt client is now ready to use.

STEP 2 — SET UP THE SSL CERTIFICATE
``` bash
letsencrypt --apache
```