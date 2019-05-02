# ***Linux-Server-Config*** 
> Linux-Server-Config is one of the udacity's Full Stack Web Developer Nanodegree projects.
This project is linked to the Configuring Linux Web Servers course, which teaches you to secure and set up a Linux server. 
By the end of this project, you will have one of your web applications running live on a secure web server.

## Why this Project? ##
> A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. 
In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.

## Server Information : ##
  > * Public IP Address : `18.185.105.26`
  > * Private IP Address : `172.26.10.157`
  > * Accessible SSH Port : `2200`
  > * Root user-name : `ubuntu`
  > * SSH user-name : `grader`

## Steps Required :
### 1- Start a new Ubuntu Linux server instance on Amazon Lightsail. 
* Create an [AWS account](https://aws.amazon.com/)
* Create an instance [Amazone Lightsail](https://lightsail.aws.amazon.com)
    * Click `Create` an instance 
    * Select `Ubuntu/Linux` platform
    * Select `OS Only` and `Ubuntu` blueprint
    * Identify Instance 
    * Click `Create Instance`
### 2- Instructions for SSH access to the instance
* Download the Private Key of the instance 
* Save the key as key.rsa in the working directory
* Open terminal in the working directory
* Craete `.ssh` directory
  > `sudo mkdir .ssh`
* Move the key to the `~/.ssh` directory 
  > `sudo mv key.rsa ~/.ssh`
* Connect to the instance
  > `ssh -i ~/.ssh/key.rsa [user-name]@[public-ip]`
### 3- Update all currently installed packages.
* Update Packages
  > `sudo apt-get update`
* Upgrade Packages
  > `sudo apt-get upgrade`
### 4- Change the SSH port from `22` to `2200`
* go to the intance and add the port `2200` to the networking 
* `sudo nano /etc/ssh/sshd_config`  change `Port 22` to `Port 2200` save and close file
* `sudo service ssh restart`
### 5- Configure the Uncomplicated Firewall (UFW) 
* SSH (port 2200) `sudo ufw allow 2200/tcp`
* HTTP (port 80) `sudo ufw allow 80/tcp`
* NTP (port 123) `sudo ufw allow 123/udp`
* `sudo ufw enable` 
* `sudo service ssh restart`
### 6- Give grader access.
* Create a new user account named `grader`
  > `sudo adduser grader`
* Give `grader` the permission to `sudo`
  > `sudo nano /etc/sudoers.d/grdaer` and add this line `grader ALL=(ALL) NOPASSWD:ALL` save and close 
* Access using `grader`
  > `su -l grader`
* Create an SSH key pair for `grader` using the `ssh-keygen` tool
  * Open new terminal in the working directory
  * Create SSH key pair using `ssh-keygen`
    > `ssh-keygen` give it name `grader_key`
  * open the key file `sudo cat ~/.ssh/grader_key` and take it copy
  * Go to the grader terminal and create `.ssh` directory
    * `sudo mkdir .ssh` then `cd .shh`
    * `sudo nano authorized_key` and pest the key in it the save and close
    * `sudo chmod 700 .ssh`
    * `sudo chmod 644 .ssh/authorized_keys`
    * `sudo service ssh restart`
  * to login to the server with the new user
    > `ssh -i ~/.ssh/grader_key grader@[public-ip] -p 2200`
### 7- Prepare to deploy Item-Catalog project
* Configure the local timezone to `UTC`.
  > `sudo dpkg-reconfigure tzdata` Choose `None of the above` then Choose `UTC`
* Install and configure `Apache` to serve a Python `mod_wsgi` application.
  > * Install Apache2 `sudo apt-get install apache2`
  > * Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi-py3`
  > * Check whether mod_wsgi is enabled `sudo a2enmod wsgi`
* Install and configure `PostgreSQL`
  > * `sudo apt-get install postgresql`
  > * Switch to postgres, PostgreSQL User `sudo su - postgres`
  > * Connect to your own database `psql`
  > * Create database user named catalog `postgres=# CREATE DATABASE catalog;`
  > * Create a new user `postgres=# CREATE USER catalog;`
  > * Set user catalog password `password Item-Catalog`
  > * Exit `psql` by pressing `Ctrl+D`
* Install `git`
  > `sudo apt-get install git`
### 8- Deploy the Item Catalog project
* Change directory
  > `cd /var/www`
* Clone Item Catalog project repository
  > sudo git clone https://github.com/Reem24Hagrss/Item-Catalog.git
* Change ownership of `catalog` directory to ubuntu user
  > `sudo chown -R ubuntu:ubuntu Item-Catalog/`
* Change current directory to project directory
  > `cd Item-Catalog`
* Create `Item-Catalog.wsgi` file using nano
  > `sudo nano Item-Catalog.wsgi`
* Add the following into the file created
  > ````sql
  > #!/usr/bin/python3
  > import sys
  > sys.stdout = sys.stderr
  > activate_this = '/var/www/Item-Catalog/env/bin/activate_this.py'
  > with open(activate_this) as file_:
  > exec(file_.read(), dict(__file__=activate_this))
  > sys.path.insert(0,"/var/www/Item-Catalog/")
  > from project import app as application
  > ````
* Install Python3 
  > `sudo apt-get install python3`
* Install pip3 
  > `sudo apt-get install Python3-pip`
* Install dependencies
  > * ` sudo pip3 install httplib2`
  > * ` sudo pip3 install requests`
  > * ` sudo pip3 install --upgrade oauth2client`
  > * ` sudo pip3 install sqlalchemy`
  > * ` sudo pip3 install flask`
  > * ` sudo pip3 install psycopg2`
### 9- set up the server configuration with apache
* Configure Apache Server by editing `000-default.conf` file
  > `sudo nano /etc/apache2/sites-enabled/000-default.conf`
* Add the following to `000-default.conf` file
  > ````sql
  > <VirtualHost *:80>
  >          ServerName 172.26.10.157
  >          WSGIScriptAlias / /var/www/Item-Catalog/Itam-Catalog.wsgi
  >          <Directory /var/www/Item-Catalog/>
  >                Order allow,deny
  >                Allow from all
  >                Options -Indexes
  >          </Directory>
  >          Alias /static /var/www/Item-Catalog/static
  >          <Directory /var/www/Item-Catalog/static/>
  >                Order allow,deny
  >                Allow from all
  >                Options -Indexes
  >          </Directory>
  >          ErrorLog ${APACHE_LOG_DIR}/error.log
  >          LogLevel warn
  >          CustomLog ${APACHE_LOG_DIR}/access.log combined
  > </VirtualHost>
 * Reload Apache Server
  > `sudo service apahce2 restart`
 * Run `database_setup.py`
  > `python3 database_setup.py`
 * Restart Apache Server
  > `sudo service apache2 restart`
  ## References :
  * [AWS Lightsail](https://lightsail.aws.amazon.com)
  * [SSH: How to access a remote server and edit files](https://www.youtube.com/watch?v=HcwK8IWc-a8)
  * [Deploying a Flask App with Heroku](https://www.youtube.com/watch?v=5UNAy4GzQ5E)
