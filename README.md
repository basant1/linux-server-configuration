# Project 6: Linux Server Configuration
### by Basant Singh

Sixth and final project of Udacity's [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004) program

## What it does

The Linux Server Configuration project consists of configuring a secure Ubuntu server to host a Python Flask app using Amazon Lightsail with PostgreSQL as the database.

The app develops a CRUD application that provides a list of items within a variety of categories, as well as provide a user registration and authentication system.

## Required Libraries and Dependencies

* Python 2.7.12
* Flask - http://flask.pocoo.org
* SQLAlchemy - http://www.sqlalchemy.org
* OAuth 2.0 Client - http://github.com/google/oauth2client

## 1 - Create an Amazon Lightsail instance

Create an account at https://aws.amazon.com/lightsail/ and setup you instance. Choose the plain Ubuntu Linux image, and "OS Only" and choose Ubuntu as the operating system. For the instance plan, the lowest tier of $3.50 per month is good with the first 30 days free. Name the instance and then create it. 

## 2 - Update all packages 

Connect to your instance from the AWS browser-based ssh client using the ```Connect using SSH``` button and run commands: ```sudo apt-get update``` and ```sudo apt-get upgrade``` 

## 3 - Configure Uncomplicated Firewall (UFW)

Connect using the ```Connect using SSH``` button on your account page and only allow incoming connections for SSH (Port 2200), HTTP (Port 80), and NTP (Port 123):

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
sudo ufw status
```

## 4 - Change SSH port from 22 to 2200

On your local machine edit the sshd_config file and change port to 2200: ```sudo nano /etc/ssh/sshd_config```

## 5 - Connect from your local terminal to your instance

Download the ssh key (.pem file) from your account page and store it in ```/user/.ssh/``` on your local machine. Open a terminal on your local machine and enter ```ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem -p 2200 ubuntu@52.207.68.245``` to connect to your instance. Now you can disable port 22: ```sudo ufw deny 22```

## 6 - Create new user account named grader

Connect to your instance and enter command: ```sudo adduser grader```

## 7 - Give grader permission to sudo 

Create grader file in directory ```sudo nano /etc/sudoers.d/grader``` and type ```grader ALL=(ALL) NOPASSWD:ALL``` to grant grader sudo permissions.

## 8 - Create an SSH key pair for grader using the ssh-keygen tool

On your local machine create a new key pair using command ```ssh-keygen``` Save the key pair in ```/user/ssh/``` directory.

## 9 - Save public key on your server

On your local machine, enter command ```cat .ssh/keyPairName.pub``` and your public key will be outputted. Copy it.

Connect to your server, make directory .ssh ```mkdir .ssh``` and create authorized_keys file ```touch .ssh/authorized_keys``` in that directory which will have 1 line for each public key you want to authorize. Open the file ```nano .ssh/authorized_keys``` and paste in the public key you copied from your local machine and save.

## 10 - Set secure permissions to your newly created ssh folder and file

```chmod 700 /home/grader/.ssh```
```chmod 644 /home/grader/.ssh/authorized_keys```

## 11 - Connect to your instance using your new user

```ssh -i ~/.ssh/keyPairName -p 2200 grader@52.207.68.245```

## 12 - Configure local timezone to UTC

```sudo timedatectl set-timezone UTC```

## 13 - Install and configure Apache to serve a Python mod_wsgi application

```sudo apt-get install libapache2-mod-wsgi```

## 14 - Install and configure PostgreSQL

```sudo apt-get install postgresql```

## 14 - Do not allow remote connections

To disable remote connections enter ```sudo nano /etc/ssh/sshd_config``` and change PermitRootLogin to ```PermitRootLogin no``` and the PasswordAuthentication to ```PasswordAuthentication no```

Then restart SSH service ```sudo service ssh restart```

## 15 - Create a new database user named catalog

```
sudo -u postgres createuser -P catalog
sudo -u postgres createdb -O catalog catalog
```

## 16 - Install git

```sudo apt-get install git```

## 17 - Clone Item Catalog App from GitHub

Clone app from GitHub and make .git inaccessible

```
cd /var/www
sudo mkdir catalog
cd /var/www/catalog
sudo mkdir catalog
git clone https://github.com/basant1/item-catalog.git catalog
sudo nano .htaccess
```

Add line ```RedirectMatch 404 /\.git```

## 18 - Make changes to Item Catalog app to accomodate deployment requirements

1. Change all instances where sqlite database engine is being used to ```engine = create_engine('postgresql://catalog:my_password@localhost/catalog')``` 
2. To change project.py file name to __init__.py cd into the directory where the file is located and do ```sudo mv application.py __init.py__```
3. Update the absolute path for all references to client_secrets.json which will now be ```/var/www/catalog/catalog/client_secrets.json```

## 19 - Create and activate virtual environment

```
sudo apt-get install python-pip
sudo pip install virtualenv
virtualenv -p /usr/bin/python2.7 my_env
source my_env/bin/activate
```

## 20 - Intall app dependencies

In your virtual environment run the following commands:
```
sudo pip install flask
sudo pip install oauth2client
sudo pip install sqlalchemy
sudo pip install requests
sudo pip install httplib2
sudo pip install psycopg2
```

## 21 - Update Google OAuth urls

Add .xip.io at the end of your authorized urls in the Google Developers Console credentials page so in this case it will be ```http://52.207.68.245.xip.io```

## 22 - Configure a New Virtual Host

```sudo nano /etc/apache2/sites-available/catalog.conf```

and add the following the code:

```
<VirtualHost *:80>
    ServerName http://52.207.68.245/
    ServerAlias HOSTNAME http://52.207.68.245/
    ServerAdmin grader@52.207.68.245
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

Save and close the file.

## 23 - Enable the Virtual Host

Enable the virtual host with the following command:

```sudo a2ensite catalog```

## 24 - Create the .wsgi File

```
cd /var/www/catalog
sudo nano catalog.wsgi
```

and add the following code:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```

## 25 - Directory Structure

Now your directory structure should look like this:

```
|--------catalog
|----------------catalog
|-----------------------static
|-----------------------templates
|-----------------------venv
|-----------------------__init__.py
|----------------catalog.wsgi
```

## 26 - Restart Apache to apply the changes

```sudo service apache2 restart```

## 27 - DONE!!

View the app at http://52.207.68.245.xip.io/