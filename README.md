###Linux-Server-Configuration-Project
###Project OverView

A baseline installation of a Linux server and prepare it to host web applications. 
As well as Securing server from a number of attack vectors,
install and configure a database server,
and deploy one of existing web applications onto it.

#Why this Project?

A deep understanding of exactly what your web applications are doing,
how they are hosted,and the interactions between multiple systems are what define you as
a Full Stack Web Developer.In this project, you’ll be responsible for turning a brand-new,
bare bones,Linux server into the secure and efficient web application host your applications need.

###Server details

Private IP:172.26.15.54
Public IP:18.196.238.100
SSH port: 2200

###Getting a Server

###1. Start a new Ubuntu Linux server instance on Amazon Lightsail. 
###There are full details on setting up your Lightsail instance on the next page.

Amazon Lightsail.

###2. Follow the instructions provided to SSH into your server.
Create a directory.


mkdir -p ~/.ssh

Move the downloaded .pem file to .ssh directory we just created.

mv DefaultKey.pem ~/.ssh
chmod +x ~/.ssh/DefaultKey.pem

Connect to instance.

ssh -i ~/.ssh/DefaultKey.pem -p 2200 ubuntu@172.26.15.54

###Secure your server.

###3. Update all currently installed packages.

sudo apt-get update
sudo apt-get upgrade

###4. Change the SSH port from 22 to 2200. 
###Make sure to configure the Lightsail firewall to allow it.
Edit sshd_config file using GNU Nano.

sudo nano /etc/ssh/sshd_config
Change line #5 from 22 to 2200.

Restart SSH service.

sudo service ssh restart

###5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections 
###for SSH (port 2200),
###HTTP (port 80), and NTP (port 123).

Allow incoming connections for SSH (port 2200)

sudo ufw allow 2200/tcp

Allow incoming connections for HTTP (port 80)

sudo ufw allow 80/tcp

Allow incoming connections for NTP (port 123)

sudo ufw allow 123/tcp

###Give grader access.

###6. Create a new user account named grader.

sudo adduser grader

###7.Give grader the permission to sudo

Super user configuration

sudo nano /etc/sudoers.d/grader

Add the following.

grader ALL=(ALL) NOPASSWD:ALL

###8. Create an SSH key pair for grader using the ssh-keygen tool.

ssh-keygen -t rsa

Move generated key

sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys
mv grader_key ~/.ssh/

Move content of grader_key.pub to that of authorized_keys

mv grader_key.pub ~/.ssh/authorized_keys
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys

Connect to grader

ssh -i ~/.ssh/grader_key grader@172.26.15.54 -p 2200

###Prepare to deploy your project.

###9. Configure the local timezone to UTC.

sudo dpkg-reconfigure tzdata

Choose None of the above.

Choose UTC

###10. Install and configure Apache to serve a Python mod_wsgi application.
###Install Apache2.

sudo apt-get install apache2

Install mod_wsgi.

sudo apt-get install libapache2-mod-wsgi-py3

Check whether mod_wsgi is enabled.

sudo a2enmod wsgi

###11. Install and configure PostgreSQL:

sudo apt-get install postgresql

Switch to postgres, PostgreSQL User.

sudo su - postgres

Connect to your own database

psql

Create database user named catalog.

CREATE ROLE CatalogItemProject WITH LOGIN;

Limit permissions of user.

ALTER ROLE CatalogItemProject CREATEDB;

Set user catalog password

\password CatalogItemProject

Exit psql by pressing Ctrl+D
Switch back to ubuntu user by running exit

###12. Install git.

sudo apt-get install git

###Deploy the Item Catalog project.

###13. Clone and setup your Item Catalog project from 
###the Github repository you created earlier in this Nanodegree program.

Change directory
cd /var/www

Clone Item Catalog project repository
sudo git clone https://github.com/fatimagamal/CatalogItemProject

Change ownership of catalog directory to ubuntu user
sudo chown -R ubuntu:ubuntu CatalogItemProject/

Change current directory to project directory
cd CatalogItemProject

Create catalog.wsgi file using GNU Nano
sudo nano CatalogItemProject.wsgi

Add the following into the file created

#!/usr/bin/python3
import sys
sys.stdout = sys.stderr

activate_this = '/var/www/CatalogItemProject/env/bin/activate_this.py'
with open(activate_this) as file_:
exec(file_.read(), dict(__file__=activate_this))

sys.path.insert(0,"/var/www/catalog")

from app.py import app as application
Install Python 3

sudo apt-get install python3-pip

Install Virtual Enviroment
sudo -H pip3 install virtualenv

Change current directory to catalog directory
cd CatalogItemProject/

Create virtual enviroment called env
virtualenv env

Activate env virtual enviroment
source env/bin/activate

Install dependencies
pip3 install httplib2
pip3 install requests
pip3 install --upgrade oauth2client
pip3 install sqlalchemy
pip3 install flask
pip3 install psycopg2

###14. Set it up in your server so that it functions correctly when 
###visiting your server’s IP address in a browser. Make sure that your .
###git directory is not publicly accessible via a browser!

Run app.py main project file

python3 app.py

Check If you are getting this message when you're running the file
Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
Configure Apache Server by editing 000-default.conf file
sudo nano /etc/apache2/sites-enabled/000-default.conf

Add the following to 000-default.conf file





<VirtualHost *:80>
            ServerName 172.26.15.54
            WSGIScriptAlias / /var/www/CatalogItemProject/CatalogItemProject.wsgi
            <Directory /var/www/CatalogItemProject/>
                  Order allow,deny
                  Allow from all
                  Options -Indexes
            </Directory>
            Alias /static /var/www/CatalogItemProject/static
            <Directory /var/www/CatalogItemProject/static/>
                  Order allow,deny
                  Allow from all
                  Options -Indexes
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
Reload Apache Server

sudo service apahce2 reload

Activate env virtual enviroment
source env/bin/activate

Run database_setup.py
python3 database_setup.py

Deactivate env virtual enviroment
deactivate

Restart Apache Server
sudo service apache2 restart
