
# Linux Server Configuration Project

### About the project
> A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

 Ip address : 52.47.153.61

 SSH port : 2200
 Web app url : http://52.47.153.61/
 Grader sudo password : 123

### Steps Followed to Configure the server

#### 1. Update all packages
```
sudo apt-get update
sudo apt-get upgrade
```
Enable automatic security updates
```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

#### 2. Change timezone to UTC and Fix language issues 
```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

#### 3. Create a new user grader and Give him `sudo` access
```
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
```
Add the following text `grader ALL=(ALL) ALL`

#### 4. Setup SSH keys for grader
* On the local machine type `ssh-keygen` Then choose the path for storing public and private keys.
* Type the following commands in the remote machine home (as user grader):
```
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys 
nano .ssh/authorized_keys 
```
* Copy the contents of the public key created on the local machine form the .pub file (in the directory where you saved it) and paste it in  the .ssh/authorized_keys file on the remote machine.

#### 5. Change the SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user
```
sudo nano /etc/ssh/sshd_config
```
Then change the following:
* To change the SSH port from 22 to 2200 find the Port line and edit it to 2200.
* To Enforce key-based authentication change the PasswordAuthentication line and edit it to no.
* To disable login for the root user find the PermitRootLogin line and edit it to no.
* Save the file and run `sudo service ssh restart`

#### 6. Configure the Uncomplicated Firewall (UFW)
* To configure the firewall:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 8000/tcp  `serve another app on the server`
```
* As you have the UFW been configured you should now enable it as it is disabled by default. So type `sudo ufw enable` to do so.

#### 8. Install Apache2 and mod-wsgi and Git
```
sudo apt-get install apache2 libapache2-mod-wsgi git
```

#### 9. Install and configure PostgreSQL
```
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql
sudo su - postgres
psql
```
Then
```
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
**Note:** In your catalog project you should change database engine to
```
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```

#### 10. Clone the Catalog app from GitHub and Configure it
```
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone https://github.com/BakrFrag/Udacity_Flask_Project.git catalog
cd catalog
git checkout production
nano catalog.wsgi
```
Then add the following in `catalog.wsgi` file
```python
import sys
sys.stdout = sys.stderr

activate_this = '/var/www/catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

sys.path.insert(0,"/var/www/catalog")

from app import app as application
```
Setup virtual environment and Install app dependencies 
```
sudo apt-get install python-pip
sudo -H pip install virtualenv
virtualenv env
source env/bin/activate
pip install -r requirements.txt
```
Edit Authorized JavaScript origins

#### 12. Configure apache server
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
Then add the following content:
```
# serve catalog app
<VirtualHost *:80>
  ServerName 52.47.153.61
  ServerAlias ec2-52-47-153-61.us-west-2.compute.amazonaws.com
  ServerAdmin es-moamen.ahmed1415@alexu.edu.eg
  DocumentRoot /var/www/catalog
  WSGIDaemonProcess catalog user=grader group=grader
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi

  <Directory /var/www/catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# Serve another project on the server (different port)
LISTEN 8000
<VirtualHost *:8000>
  ServerName 52.47.153.61
  ServerAlias ec2-52-47-153-61.us-west-2.compute.amazonaws.com
  DocumentRoot /var/www/map
  ServerAdmin es-moamen.ahmed1415@alexu.edu.eg

 <Directory /var/www/map/.git>
    Require all denied
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### 14. Restart Apache Server
```
sudo service apache2 restart
```


### References:
* [github.com/BakrFrag](https://github.com/BakrFrag/)
* [Stack Overflow](https://stackoverflow.com/)
