# Linux-Server-Configuration-UDACITY
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. 

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

The main Idea is to deploy your website in a server so the calint can use it. There are alot of compnies that provied this servec such as Amazon, google, ALipapa, and DigitalOcean. I have chosen DigitalOcean for this project. Since your website will be on the web now where everyone can use it. you should create a firalwall between the world and the server first. Then insde the server you need to install some progrmaes so the serve can be aple to run your website.
		

You can visit http://104.248.243.155 for the website deployed.

## Tasks
1. Create a Server
2. Follow the instructions provided by the server compeny to SSH into your server
3. Create a new user named grader
4. Give the grader the permission to sudo
5. Update all currently installed packages
6. Change the SSH port from 22 to 2200
7. Configure the Firewall to only allow incoming connections for SSH (2200/tcp), HTTP (80/tcp), and NTP (123/udp)
9. Install and configure Apache to serve a Python mod_wsgi application
10. Install and configure PostgreSQL and create a new user named catalog that has limited permissions to your catalog application database
11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser.
12. Change your cloned project in your server from using sqlite to PostgreSQL

## Instructions for SSH access to the instance
1. Using Liuncix, Type ssh-keygen either in your localmeachen or your server 
2. make a copy of the privete key in any folder and remember the path by heart and name it key
3. open the public key file and copy it content in your clipbored. you're going to use it later
4. Enter as root to create the grader user `root@104.248.243.155`
5. `adduser grader` password `grader`
6. `sudo nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## Set ssh login using keys
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ nano .ssh/authorized_keys
	```
	paste what you haved copied from the public key and paste here and save the file
    
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `service ssh restart`
4. now you can use ssh to login with the new user you created

	`ssh -i [privateKeyFilename] grader@104.248.243.155`

    The ssh passphrase is the same as the root password: Ll112233
    
## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
 
## Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" `exit`
 
## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/AlkhorayefAbdulellah/CatlogApp`
6. Move to the inner FlaskApp directory using `cd FlaskApp`
7. Rename `website.py` to `__init__.py` using `sudo mv website.py __init__.py`
8. Edit `database_setup.py`, `app.py` and `seeder.py` and change `engine = create_engine('sqlite:///toyshop.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
9. adding http://104.248.243.155.xip.io:80 to google APIs

then downloding the new json and replace it with the cloned project in the server
change the clint id if neccery in all three places:
1-Login.html 2-the json itself 3-app.py(two clint id parameter)

10. Install pip `sudo apt-get install python-pip`
11. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
12. Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
1. Create CatlogApp.conf to edit: `sudo nano /etc/apache2/sites-available/CatlogApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
       ServerName 104.248.243.155
       ServerAdmin iiiLohy@gmail.com
       WSGIDaemonProcess CatlogApp python-path=/var/www/FlaskApp:/var/www/FlaskApp/venv/lib/python2.7/site-pac$
       WSGIProcessGroup CatlogApp
       WSGIScriptAlias / /var/www/FlaskApp/catlogapp.wsgi
       <Directory /var/www/FlaskApp/CatlogApp/>
               Order allow,deny
               Allow from all
       </Directory>
       Alias /static /var/www/FlaskApp/CatlogApp/static
       <Directory /var/www/FlaskApp/CatlogApp/static/>
               Order allow,deny
               Allow from all
       </Directory>
       ErrorLog ${APACHE_LOG_DIR}/error.log
       LogLevel warn
       CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
	
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano catlogapp.wsgi 
	```
2. Add the following lines of code to the catlogapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")
	from FlaskApp import app as application
	application.secret_key = 'Lohy'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

https://github.com/kongling893/Linux-Server-Configuration-UDACITY/blob/master/README.md

https://github.com/chuanqin3/udacity-linux-configuration
