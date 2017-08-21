# Linux-based Server Configuration
## Purpose:
-------------------
Deploying web applications to a publicly accessible server and  secure the application

## Description:
------------------
Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

##  IP address, URL and SSH port
IP address: <http://54.242.103.25/>  
Port: 2200  
URL:<http://ec2-54-242-103-25.compute-1.amazonaws.com/>  
Username: `grader`


## Configuration
### 1 - SSH access to the Udacity Development Environment
-------------------

1. create AWS EC2 mico instance of ubuntu
2. Download Private Key   
3. Move the private key file into the local user folder ~/.ssh  		
4. Log into instance

		$ ssh -i ~/.ssh/<private key>.rsa ubuntu@<micro instance ip address>


### 2 - User Management & Security
-----------------

1. Create a new user name grader

		$ sudo adduser grader

2. Grant **grader** the permission to sudo

	In the /etc/sudoers.d create a new file called grader
		
		$ sudo vi /etc/sudoers.d/grader        
		
	In the file put in:  `grader ALL=(ALL) NOPASSWD:ALL`
	Save and quit.
 	
3. Create a ssh key for **grader** to log in
	In your own computer terminal, generate key pairs locally
	
		$ ssh-keygen
		
	Input the key name and passphrase.  
	See the content in the public key
	
		$ cat ~/.ssh/keyname.pub
	
	Copy all the content.
	
	In the Develop Environment, change to the user **grader**
	Make a directory called **.ssh**
	
		$ sudo mkdir ~/.ssh
		
	Create a new file called **authorized_keys** and edit it
	
		$ sudo vi ~/.ssh/authorized_keys
	
	Paste the public key content to it, save and quit.
	
	Change the permision of the file/directory
	
		$ sudo chmod 751 ~/.ssh
		$ sudo chmod 644 ~/.ssh/authorized_keys
		
4. Change the SSH port from 22 to 2200, and disabled remote login of the root user 
	
	Edit configue file
		
		$ sudo vi /etc/ssh/sshd-config
		
	In hte file, change the `Port 22` to  `Port 2200`, and `PermitRootLogin without-password` to `PermitRootLogin no`(Because in the configue file `PasswordAuthentication no` is already set to disable password login, doesn't need to change it.), save and qiut.
	
	Restart ssh service
	
		$ sudo service sshd restart
		
5. login with new user "grader"


### 3 - Updated application to most recent updates
---------------

1. Update package source list:

		$ sudo apt-get update

2. Upgrade all the application:

		$ sudo apt-get upgrade
		

### 4 - Configure the Uncomplicated Firewall(UFW)
-----------------
1. Check the ufw status

		$ sudo ufw status
2. Allow incoming connections for SSH (port 2200)

		$ sudo ufw allow 2200/tcp
	
3. Allow incoming connections for HTTP (port 80)

		$ sudo ufw allow www
	
4. Allow incoming connections for NTP (port 123)

		$ sudo ufw allow 123/udp
		
5. Eable the ufw

		$ sudo ufw enable
		


### 5 - Configure the local timezone to UTC
--------------------
1. Run the command below:

		$ sudo dpkg-reconfigure tzdata
		
2. select "None of the above" in first screen.

3. select "UTC" in second screen

### 6 - Install Apache, mod_wsgi application, Git
--------------
1. Install  Apache

		$ sudo apt-get install apache2
	
	
2. Install mod_wsgi 

		$ sudo apt-get install libapache2-mod-wsgi-py3
		
3. Install git 

		$ sudo apt-get install git
		
### 7 - Configure Apache to serve a Python mod_wsgi application
-----------

1. Clone the CatalogApp on Github:

		$ cd /var/www
		$ sudo mkdir ItemCatalog
		$ cd ItemCatalog
		$ git clone git@github.com:csundeep/CatalogApp.git
		
2. To make  .git directory is not publicly accessible via a browser. Create a .htaccess file in the .git folder and put the following in this file:
		
		RedirectMatch 404 /\.git
		
	 Reference:[serverfault](serverfault)
3. Make the `project.py` file name to `__init__.py`  
4. Install pip 
	
	
		$ sudo apt-get -qqy install python3 python3-pip
		$ sudo pip3 install --upgrade pip
		
		
5. Using pip install following nessary packages
	
		$ sudo pip3 install flask packaging oauth2client redis passlib flask-httpauth
		$ sudo pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach
		$ sudo pip3 install requests
		
6. Configure and Enable a New Virtual Host

		$ sudo vi /etc/apache2/sites-available/catalog.conf
	
	Add these content:
		
		<VirtualHost *:80>
			ServerName localhost
			ServerAdmin admin@eynaksara.com
			WSGIDaemonProcess myapp python-path=/var/www/ItemCatalog/CatalogApp
			WSGIScriptAlias / /var/www/ItemCatalog/catalogapp.wsgi process-group=myapp application-group=%{GLOBAL}
			<Directory /var/www/ItemCatalog>
			<Files catalogapp.wsgi>
				Order allow,deny
				Allow from all
			</Files>
			</Directory>
			Alias /static /var/www/ItemCatalog/CatalogApp/static
			<Directory /var/www/ItemCatalog/CatalogApp/static/>
				Order allow,deny
				Allow from all
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined

		</VirtualHost>

				
	Enable  virtual host:
	
		$ sudo a2dissite 000-default
		$ sudo a2ensite catalog
		
7. Create the .wsgi File
	 Create a file named catalogapp.wsgi with following commands:
		
		cd /var/www/ItemCatalog/
		sudo vi catalogapp.wsgi 
	
	Add these content:
	
	
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/ItemCatalog/")
		from CatalogApp import app as application
		application.secret_key = 'Add your secret key'

		
	Right now my directory tree(In the /var/www/):
	
		ItemCatalog
		----catalogapp.wsgi
		----CatalogApp
		------static
		------__init__.py
	
8. In Facebook developer page, Google developer console, change the applicaiton oauth path to `http://ec2-54-242-103-25.compute-1.amazonaws.com` 



### 8 - Install and configure PostgreSQL
-----------

1. Install PostgreSQL
	
		$ sudo apt-get install postgresql

2. Log into PostgreSQL
	
		$ sudo su - postgres
		$ psql
		
3. Create a new role

		CREATE ROLE catalog WITH CREATEDB;
		ALTER USER "catalog" WITH PASSWORD 'password';
		ALTER ROLE "catalog" WITH LOGIN;
	
	using `\du` can see the roles list
	
4. Create database

		CREATE DATABASE itemcatalog WITH OWNER catalog;
			


### 9 - Make the app run

1. Go to the Catalog app directory

		$ cd /var/www/ItemCatalog/CatalogApp/

2. Run this command for CatalogApp seed database
		
		$ sudo python3 insert_records.py
		
3. Restart Apache

		$ sudo service apache2 restart 
		
		
Now can visit the page thourgh <http://ec2-54-242-103-25.compute-1.amazonaws.com/>.



I learn a lot from error log (`/var/log/apache2/`)

