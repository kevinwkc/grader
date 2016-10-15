project
------------------
ssh port:2200 ip:54.190.14.214

webapp: http://54.190.14.214

passwd grader G4a1e4
passwd root U1aci1y

Reference
--------------   
https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587
https://discussions.udacity.com/t/p5-how-i-got-through-it/15342
   
Create a new user named grader
-----------------------------------------
https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
sudo adduser grader
passwd grader 
>G4a1e4
https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user
Give the grader the permission to sudo
sudo visudo

Update all currently installed packages
--------------------------------------------------
$ sudo apt-get update
$ sudo apt-get dist-upgrade

Change the SSH port from 22 to 2200
-------------------------------------------
/etc/ssh/sshd_config 
PermitRootLogin no
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2-ssh-configuration-keypairs.html
PasswordAuthentication no
/etc/init.d/ssh restart
echo y | ufw enable

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
-------------------------------------------------------------
https://help.ubuntu.com/community/UFW
sudo ufw status verbose
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
sudo ufw enable
sudo ufw status

Configure the local timezone to UTC
------------------------------------------
https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29
sudo dpkg-reconfigure tzdata

Install and configure Apache to serve a Python mod_wsgi application
----------------------------------------------------------------------------------
sudo apt-get install apache2
https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4340119836/concepts/48189486140923
sudo apt-get install python-setuptools libapache2-mod-wsgi
catalog.conf:
```
<VirtualHost *>
    ServerName example.com

    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    WSGIDaemonProcess hello
    <Directory /var/www/catalog/*>
       WSGIProcessGroup catalog
       WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
    </Directory>
		ErrorLog /var/log/apache2/error.log
		LogLevel warn
		CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```
catalog.wsgi
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.append('/var/www/catalog')
from main import app as application
application.secret_key="tomomi"
```

sudo a2dissite 000-default.conf
sudo a2ensite catalog.conf
sudo service apache2 reload

/etc/apache2/sites-enabled/000-default.conf
For now, add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: WSGIScriptAlias / /var/www/html/myapp.wsgi
sudo apache2ctl restart

http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html

tail -f /var/log/apache2/error.log

Install and configure PostgreSQL:
------------------------
sudo apt-get install postgresql

Do not allow remote connections
---------------------------
/etc/postgresql/current/main/postgresql.conf
listen_addresses = '' #set empty

https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
sudo nano /etc/postgresql/9.1/main/pg_hba.conf
uncomment:
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
sudo /etc/init.d/postgresql restart

Create a new user named catalog that has limited permissions to your catalog application database
--------------------------------------------------------------------------------------------------
https://www.postgresql.org/docs/9.1/static/app-createuser.html
http://killtheyak.com/use-postgresql-with-django-flask/
sudo -i -u postgres
psql
create user catalog;
create database catalog owner = catalog;
ALTER USER myuser PASSWORD 'Ca1a5o8'
postgres=# BEGIN;
BEGIN
postgres=# REVOKE connect ON DATABASE  fred FROM public; 
REVOKE
postgres=# GRANT connect ON DATABASE fred TO fred;
GRANT
postgres=# COMMIT;
COMMIT
postgres=#  
psql user_name  -h 127.0.0.1 -d db_name

sudo -u postgres createuser --interactive catalog
createdb -U postgres --locale=en_US.utf-8 -E utf-8 -O catalog catalog -T template0

install python package
---------------------------
apt-get install python-psycopg2
apt-get install python-flask python-sqlalchemy
apt-get install python-pip
pip install bleach
pip install oauth2client
pip install requests
pip install httplib2
pip install redis
pip install passlib
pip install itsdangerous
pip install flask-httpauth


Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. 
----------------------
sudo apt-get install git-all
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR EMAIL ADDRESS"
git clone 

Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
cd /var/www
git clone https://github.com/kevinwkc/catalog
cd catalog
PARAMETERIZATION:
postgresql://catalog:secret@localhost/catalog IN main.py models.py lotsofCat.py
/var/www/ItemCatalog/client_secrets.json IN main.py
https://discussions.udacity.com/t/lesson-2-cant-not-login-error-500-internal-server-error/40052/9
https://discussions.udacity.com/t/login-error-while-deployoing-app/34125/7
https://discussions.udacity.com/t/configuring-mod-wsgi-to-use-specific-python-version/162392/3

https://docs.google.com/document/d/1J0gpbuSlcFa2IQScrTIqI6o3dice-9T7v8EDNjJDfUI/pub?embedded=true