# Linux Server Configuration

This is a project for Udacity's full stack developer nano degree.

# Access details 



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

## Deploy the Item Catalog project.
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!
