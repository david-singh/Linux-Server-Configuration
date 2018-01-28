# Linux-Server-Configuration

Configuring a Linux server to host a web app securely using flask application on to AWS Light Sail. Installation of a Linux distribution on a virtual machine and prepare it to host web application(Item Catalog). It includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- IP address: ~~52.221.231.17

- Accessible SSH port: ~~2200

- Application URL: http://ec2-52-221-231-17.ap-southeast-1.compute.amazonaws.com/

## Configuration Steps:
### Step 1 : Create new user named grader and give it the permission to sudo

  - SSH into the server through `ssh -i ~/.ssh/udacity_key.pem unbuntu@52.221.231.17`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`
  - Run `sudo nano /etc/hosts`
   
### Step 2 : Update all currently installed packages
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

### Step 3 : Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - Confirm by running `ssh -i ~/.ssh/udacity_key.pem -p 2200 unbuntu@52.221.231.17`
  
### Step 4 : Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
  
### Step 5 : Configure the local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC
 
### Step 6 : Configure key-based authentication for grader user
  - generate key-pair with ssh-keygen
  - Save keygen file into (/home/user/.ssh/keypair).and fill the password . 2 keys will be generated,  public key (keypair.pub) and       identification key(keypair).
  - Login into grader account using `sudo login grader`.  type the password that you have fill during user creation
    (`sudo adduser grader` step 3) .
    `grader@52.221.231.17 password :`
  - if the password is correct , you will login as grader account:
     `grader@52.221.231.17:~$`
  - make a directory in grader account : `mkdir .ssh`
  - make a authorized_keys file using `touch .ssh/authorized_keys`
  - from your local machine,copy the contents of public key(keypair.pub).
  - paste that contents on authorized_keys of grader account using `nano authorized_keys` and save it .
  - give the permissions : `chmod 700 .ssh`    and `chmod 644 .ssh/authorized_keys`.
  - do `nano /etc/ssh/sshd_config` , change `PasswordAuthentication` to  no .
  - `sudo service ssh restart`.
  -  `ssh grader@52.221.231.17 -p 2200 -i ~/.ssh/keypair` in new terminal .A pop-up window will open for authentication. just fill the      password that    you have fill during ssh-keygen creation.

  **Resources -** [initial server setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04),      [udacity course videos](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4331066009/concepts/48010894750923#)

### Step 7 : Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh -i ~/.ssh/udacity_key.pem -p 2200 grader@52.221.231.17`
 
### Step 8 : Install Apache
  - `sudo apt-get install apache2`

### Step 9 : Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

  
### Step 10 : Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - Clone your project from github `git clone https://github.com/rrjoson/udacity-item-catalog.git catalog`
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
  - Rename application.py to __init__.py `mv application.py __init__.py`
  
### Step 11 : Install virtual environment
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

### Step 12 : Install Flask and other dependencies
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

### Step 13 : Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
  
### Step 14 : Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  <VirtualHost *:80>
      ServerName 52.221.231.17
      ServerAlias http://ec2-52-221-231-17.ap-southeast-1.compute.amazonaws.com
      ServerAdmin admin@52.221.231.17
      WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
      WSGIProcessGroup catalog
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
      <Directory /var/www/catalog/catalog/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/catalog/catalog/static
      <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite catalog`

### Step 15 : Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - Change create engine line in your `__init__.py` , `database_setup.py` and `lotsofitems.py` to: 
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - Run `sudo python database_setup.py`
  
  ```
  
### Step 16 : Restart Apache 
  - `sudo service apache2 restart`
  
17. Visit site at [http://ec2-52-221-231-17.ap-southeast-1.compute.amazonaws.com/](http://ec2-52-221-231-17.ap-southeast-1.compute.amazonaws.com/)
