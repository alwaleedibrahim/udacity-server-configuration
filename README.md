# Linux Server Configuration

This is a project for [Full Stack Developer Nanodegree - Udacity](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd0044)
## Project Overview
> You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

# Access details 

| **Public IP:** | [3.126.217.230](http://3.126.217.230) |
| ------ | ------- |
| **SSH Port:** | 2200 |
| **User name:** | ubuntu |

# Configuration steps
## Get your server.
1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
2. Follow the instructions provided to SSH into your server.

## Secure your server.
3. Update all currently installed packages.
```
sudo apt-get update
sudo apt-get upgrade
```
4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
`sudo nano /etc/ssh/sshd_config` and change `Port 22` to `Port 2200`
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
sudo ufw enable
```

`sudo ufw status` to check
```
sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/tcp                    ALLOW       Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/tcp (v6)               ALLOW       Anywhere (v6) 
```

## Give grader access.
6. Create a new user account named grader.
`sudo adduser grader`
7. Give grader the permission to sudo.
`sudo nano /etc/sudoers.d/grader` and type `grader ALL=(ALL:ALL) ALL`
8. Create an SSH key pair for grader using the ssh-keygen tool.
- Generate key pair on your local machine using `ssh-keygen`
- On server machine
```
su - grader
mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
```
- Copy the public key from your local machine and paste it in this file and save.
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
sudo chown -R grader.grader /home/grader/.ssh
sudo service ssh restart
```

## Prepare to deploy your project.
9. Configure the local timezone to UTC.
- already UTC
10. Install and configure Apache to serve a Python mod_wsgi application.
```
sudo apt-get install apache
sudo apt-get install python-setuptools libapache2-mod-wsgi
sudo apt-get install libapache2-mod-wsgi-py3
```
11. Install and configure PostgreSQL:
`sudo apt-get install postgresql`
  - Do not allow remote connections
  check `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
  - Create a new database user named catalog that has limited permissions to your catalog application database.
  Login as user "postgres" `sudo su - postgres`

Get into postgreSQL shell `psql`

Create a new database named catalog and create a new user named catalog in postgreSQL shell
```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```
Set a password for user catalog
```
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
```
Give user "catalog" permission to "catalog" application database
```
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
Quit postgreSQL `postgres=# \q`

Exit from user "postgres" `exit`

12. Install git.
`sudo apt-get install git`

## Deploy the Item Catalog project.
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
```
cd /var/www
sudo mkdir FlaskApp
cd FlaskApp
git clone https://github.com/alwaleedibrahim/Item-catalog.git
sudo mv ./Item_Catalog_UDACITY ./FlaskApp
cd FlaskApp
sudo mv website.py __init__.py
```
change `engine = create_engine('sqlite:///toyshop.db')` to 
`engine = create_engine('postgresql://catalog:password@localhost/catalog')`
```
sudo apt-get install python-pip
sudo pip install Flask
sudo pip install sqlalchemy sqlalchemy_utils
sudo pip install httplib2 oauth2client requests
sudo apt-get -qqy install postgresql python-psycopg2
sudo python database_setup.py
```

Configure and Enable a New Virtual Host
`sudo nano /etc/apache2/sites-available/FlaskApp.conf`

Add the following lines of code to the file to configure the virtual host.
```
<VirtualHost *:80>
	ServerName 3.126.217.230
	ServerAdmin elwaleed315@gmail.com
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

Create the .wsgi File
Create the .wsgi File under `/var/www/FlaskApp`:
```
cd /var/www/FlaskApp
sudo nano flaskapp.wsgi 
```
Add the following lines of code to the flaskapp.wsgi file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```
Restart Apache
`sudo service apache2 restart`
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!

