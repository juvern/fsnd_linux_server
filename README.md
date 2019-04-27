This is the final project for the Udacity Nanodegree Full Stack Web Developer to deploy an application on a Linux server.

The URL is <http://35.158.243.112.xip.io>

Here are the instructions for the Udacity grader to access the server.

##### IP address and SSH port to access the server for user 'grader'
`ssh grader@35.158.243.112 -p 2200 -i {{PRIVATE_KEY}}` 

##### IP address and SSH port to access the server for user 'udacity' with keyfile saved locally
`ssh -i LightsailDefaultKey-eu-central-1.pem ubuntu@35.158.243.112 -p 2200`

Below is a summary of software installed and configuration changes made

# Configuration 


#### Creating a new user and giving sudo rights to the new user

**Create a new user called 'grader'**

1. `sudo adduser grader`
3. `sudo touch /etc/sudoers.d/grader`
4. `sudo nano /etc/sudoers.d/grader`, type in grader `grader ALL=(ALL) NOPASSWD:ALL`, save and quit
5. To login as the new user `sudo su - grader`

**Generate a key pair**

1. Generate a key pair on local machine and save in the default directory.
2. While logged in as the new user, create a folder for all key related tasks `mkdir .ssh`
3. Create a file to store all public keys `touch .ssh/authorized_keys`
4. Edit the file `nano .ssh/authorized_keys` by pasting the contents of the public key generated
5. `chmod 700 .ssh` so the owner can read, write and open the directory
6. `chmod 644 .ssh/authorized_keys`


#### Changing SSH port from 22 to 2200
1. `sudo nano /etc/ssh/sshd_config` to edit the configuration file to update SSH port from 22 to 2200
2. `sudo service ssh restart` to restart the server

#### Setting up firewall
1. `sudo ufw allow ssh` to allow incoming SSH connections
2. `sudo ufw allow 2200/tcp` to open the port 2200 for SSH connections
3. `sudo ufw allow http` to open up port 80 for http connections
4. `sudo ufw allow 123/tcp` to open up port 123 for NTP

Finally, run `sudo ufw enable`to enable UFW firewall. To confirm that all rules have been set up correctly, run `sudo ufw status` 


#### Serving an application

**Upgrading packages**

1. `sudo apt-get update` to update available package lists
2. `sudo apt-get upgrade` to upgrade all currently installed packages


**Set up the server to respond to HTTP requests**

1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

**Copying files to the server**

1. Change the owner and group of `/var/www/html/index.html`

**Configure and enable the virtual host**

1. Edit the file `sudo nano /etc/apache2/sites-available/FlaskApp.conf`

```
<VirtualHost *:80>
                ServerName 35.158.243.112.xip.io
                ServerAdmin admin@mywebsite.com
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

**Create the WSGI file for Apache to serve the app**

1. Edit the file `sudo nano flaskapp.wsgi`

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = "Add your secret key"
```

**Create a database 'catalog' with a new user called 'catalog'**

1. `sudo apt-get install postgresql`
2. Access postgres `sudo -u postgres psql`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`

```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

6. Exit postgres `\q`
 
7. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
8. Create database schema `sudo python database_setup.py`

9. Connect to Postgres Shell `psql` as postgres user
10. Connect as the 'catalog' user `\c catalog`
11. Confirm database created by listing `\dt`



# Third-party resources
<https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>
<https://github.com/jungleBadger/-nanodegree-linux-server/blob/master/README.md>

# Dependencies
* Amazon Lightsail
* Apache HTTP Server `sudo apt-get install apache2`
* Python 2.7
* mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
* Git `sudo apt-get install git-core`
* Pip `sudo apt install python-pip`
* SQLAlchemy `sudo easy_install SQLAlchemy`
* Flask `pip install Flask`
* Postgres
* mod_wsgi - interface between web servers and web apps for Python
* OAuth2 - `pip install --upgrade oauth2client`

# Useful Commands
View error log
`tail -f /var/log/apache2/error.log`

Restart server
`sudo service apache2 restart`