# Linux Server Configuration Project

### About the project
> Configuring and securing a Linux Server to host web site and a database


* IP Address: 18.130.149.246
* SSH Port: 2200
* URL using DNS: [http://ec2-18-130-149-246.eu-west-2.compute.amazonaws.com](http://ec2-18-130-149-246.eu-west-2.compute.amazonaws.com)


### Project steps
#### 1. Get your server


* Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://aws.amazon.com/).
* SSH to your new Server.

#### 2. Secure your server
 Update all currently installed packages.
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
Enable automatic security updates.
```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```
#### 3. Change the SSH port from 22 to 2200.Enable Lightsail firewall to allow port 2200.
```
sudo nano /etc/ssh/sshd_config
```
Then change the following:
* Find the Port line and edit it to 2200.
* Save the file and run `sudo service ssh restart`

#### 4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 8000/tcp  `serve another app on the server`
sudo ufw enable
```
#### 5. Create a new user grader and Give him `sudo` permission
```
sudo adduser grader
sudo nano /etc/sudoers.d/grader
```
Then add the following text `grader ALL=(ALL) ALL`

#### 6. Create an SSH key pair for grader using the ssh-keygen tool
* On local machine
`ssh-keygen`.

Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa): (/your path/filename)

It will create two files, public and private key
* On remote machine (server) change to user grader
```
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys

```
local machine
```sudo cat /public_key_path/filename```.

Copy the contents and  paste it to authorized_keys.

server ```nano .ssh/authorized_keys```

**Note** You can log in to your server using ssh from your local machine by using.

```
sudo ssh grader@18.130.149.246 -p 2200 -i ~/.ssh/authorized_keys
```
#### 7. Configure the local timezone to UTC.
```
sudo timedatectl set-timezone UTC
```



#### 8. Install and configure Apache to serve a Python mod_wsgi application
```
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
```
**Note**: For Python2 replace `libapache2-mod-wsgi-py3` with `libapache2-mod-wsgi`

#### 9. Install and configure PostgreSQL
```
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```
Then
```
CREATE USER catalog WITH PASSWORD '12345';
CREATE DATABASE computershop WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
**Note:** In your catalog project you should change database engine to
```
engine = create_engine('postgresql://catalog:12345@localhost/computershop')
"12345" is the password
```

#### 10. Clone the Catalog app from GitHub and Configure it
```
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone <your_repo_url> catalog
cd catalog
nano catalog.wsgi
```
Then add the following in `catalog.wsgi` file
```python
#!/usr/bin/python3
import sys
sys.stdout = sys.stderr

sys.path.insert(0,"/var/www/catalog")

from project import app as application
application.secret_key = 'super_secret_key'
```
Setup virtual environment and Install app dependencies
```
sudo apt-get install python3-pip
sudo -H pip3 install virtualenv
virtualenv env
source env/bin/activate
pip3 install -r requirements.txt
```
- If you don't have `requirements.txt` file, you can use
```
pip3 install flask packaging oauth2client redis passlib flask-httpauth
pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
```


#### 12. Configure apache server
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
Then add the following content:
```
# serve catalog app
<VirtualHost *:80>
  ServerName <IP_Address or Domain>
  ServerAlias <DNS>
  ServerAdmin <Email>
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

```

#### 14. Reload & Restart Apache Server
```
sudo service apache2 reload
sudo service apache2 restart
```
***Notes to Reviewer***
Passphrase for private key is: ***12345***
grader password is: ***Gr@der!u$er***

