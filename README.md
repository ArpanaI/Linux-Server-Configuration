Udacity-linux-server-configuration

Project Description
Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address: 18.218.6.78

Accessible SSH port: 2200

Application URL: http://ec2-18-218-6-78.us-east-2.compute.amazonaws.com/

Third-Party Components
Flask
PostgreSQL
SQLAlchemy
oauth2client

Basic Configuration
Created user “grader1”.
Gave user “grader1” sudo privileges by adding grader1 file to /etc/sudoers.d with the following line: grader1 ALL=(ALL) NOPASSWD:ALL
Enabled SSH login for “grader1” user:
Copied /root/.ssh/authorized_keys to /home/grader1/.ssh/authorized_keys.
Changed permissions for /home/grader1/.ssh to 700.
Changed permissions for /home/grader1/.ssh/authorized_keys to 644.
Change owner and group for /home/grader1/.ssh and /home/grader1/.ssh/authorized_keys to grader1 and grader1.
Disabled remote root login by editing /etc/ssh/sshd_config: PermitRootLogin no.
Verified that password authentication is disabled by checking /etc/ssh/sshd_config for PasswordAuthentication no.
Restart ssh with sudo service ssh restart
Updated package source lists: sudo apt-get update
Updated installed packages: sudo apt-get upgrade
Set timezone to UTC using sudo dpkg-reconfigure tzdata.

Security Configuration
Changed ssh default port to 2200 by editing /etc/ssh/sshd_config.
Configured the Uncomplicated Firewall for ssh, http, and ntp:
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200
sudo ufw allow http
sudo ufw allow 123
sudo ufw enable

Install Apache
sudo apt-get install apache2
Install mod_wsgi
Run sudo apt-get install libapache2-mod-wsgi python-dev
Enable mod_wsgi with sudo a2enmod wsgi
Start the web server with sudo service apache2 start
Clone the Catalog app from Github
Install git using: sudo apt-get install git
cd /var/www
sudo mkdir catalog
Change owner of the newly created catalog folder sudo chown -R grader1:grader1 catalog
cd catalog
Clone your project from github git clone https://github.com/ArpanaI/Item-Catalog-Clothing-.git catalog
Create a catalog.wsgi file, then add this inside:

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, “/var/www/catalog/”)
from catalog import app as application
application.secret_key = ‘My Secret Key’

Rename application.py to init.py mv application.py init.py

Install virtual environment
Install the virtual environment sudo pip install virtualenv
Create a new virtual environment with sudo virtualenv venv
Activate the virutal environment source venv/bin/activate
Change permissions sudo chmod -R 777 venv
Install Flask and other dependencies
Install pip with sudo apt-get install python-pip
Install Flask pip install Flask
Install other project dependencies sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

Update path of client_secrets.json file
nano init.py
Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json
Configure and enable a new virtual host
Run this: sudo nano /etc/apache2/sites-available/catalog.conf
Paste this code:
<VirtualHost *:80>
ServerName 18.218.6.78
ServerAlias http://ec2-18-218-7-78.us-east-2.compute.amazonaws.com/
ServerAdmin admin@18.218.6.78
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

Install and configure PostgreSQL
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog1 WITH PASSWORD ‘password’;
ALTER USER catalog1 CREATEDB;
CREATE DATABASE clothes_catalog WITH OWNER catalog1;
\c catalog1
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog1;
\q
exit

Change create engine line in your init.py and database_setup.py to: engine = create_engine(‘postgresql://catalog1:password@localhost:5432/clothes_catalog’)
python /var/www/catalog/catalog/database_setup.py
Make sure no remote connections to the database are allowed. Check if the contents of this file sudo nano /etc/postgresql/9.5/main/pg_hba.conf looks like this:

Database administrative login by Unix domain socket
local all postgres peer

TYPE DATABASE USER ADDRESS METHOD
local all catalog1 password

“local” is for Unix domain socket connections only
local all all peer

IPv4 local connections:
host all all 127.0.0.1/32 md5

IPv6 local connections:
host all all ::1/128 md5

Restart Apache
sudo service apache2 restart
Visit site at http://18.218.6.78