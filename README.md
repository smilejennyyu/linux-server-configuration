# Udacity Linux Configuration Project

### Project Description

This project is to deploy the item catalog web applications onto Linux servers! This includes securing your server from a number of attack vectors, install and configure a database server and using Amazon Lightsail.

- IP address: 54.198.69.144

- Accessible SSH port: 2200

- Application URL: http://54.198.69.144.xip.io

### Instructions
#### Step 1: Amazon Lightsail
1. Create an instance on Amazon Lightsail
2. Under the Networking tab, add two custom port in Firewall. They are port **2200** and port **123**.
3. Download the default private key from Lightsail Account page
#### Step 2: Create a New User and Setup the port
1. Put the private key we just downloaded into `.ssh`, and renamed it as `udacity_key.pem`
2. Change the file permission: `sudo chmod 600 ~/.ssh/continue2_key.pem`
3. Connect with SSH: `sudo ssh -i ~/.ssh/continue2_key.pem ubunutu@54.198.69.144`
4. Login as a root user: `sudo su -` to add a new user by doing `sudo adduser grader`
5. Create a new file under the sudoers.d directory: `sudo nano /etc/sudoers.d/grader`. Then fill the file with `grader ALL=(ALL:ALL) ALL` and save the file.
6. Edit the hosts: `sudo nano /etc/hosts`, add `127.0.0.1 ip-172-26-11-28` at the end of the file
7. Go to `/home/grader`, and create a `.ssh` directory. Then inside the `.ssh`directory, create a file called `authorized_keys`
8. Open a new terminal and logout as root user, then generate a ssh key: `sudo ssh-keygen -f ~/.ssh/udacity_key.rsa`
9. Copy the public key in `udacity_key.rsa.pub` and go back to the previous terminal. Then past the public key to the `authorized keys` file in `/home/grader/.ssh/` directory
10. Change the file and folder permision: `sudo chmod 700 /home/grader/.ssh` and `sudo chmod 644 /home/grader/.ssh/authorized_keys`. Then change the owner from root to grader user: `sudo chown -R grader:grader /home/grader/.ssh`
11. The new user has been created. Restart the ssh: `sudo service ssh restart`
12. Logout root user and then login as grader: `sudo ssh -i ~/.ssh/udacity_key.rsa grader@54.198.69.144`
13. Edit the `ssdh_config` in `/etc/ssh/sshd_config`: 
- On the PasswordAuthentication line, change the text after to no.
- Change the port from **22** to **2200**
14. Restart the ssh: `sudo service ssh restart`. Then logout.
15. Login as grader through port 2200: `sudo ssh -i ~/.ssh/continue2_key.rsa -p 2200 grader@54.198.69.144`
16. Disable ssh login for root user: `sudo nano /etc/ssh/sshd_config`, then find the PermitRootLogin line and change to no. Restart ssh.
17. configure UFW:
- `sudo ufw allow 2200/tcp`
- `sudo ufw allow 123/udp`
- `sudo ufw allow 80/tcp`
- `sudo ufw enable`
####Step 3: Deploy Catalog Project
1. Install required packages
`sudo apt-get install apache2`
`sudo apt-get install libapache2-mod-wsgi python-dev`
`sudo apt-get install git`
2. Start Apache
- Enable mod_wsgi : `sudo a2enmod wsgi` 
- Start the web server :`sudo service apache2 start` 
3. Make a new direcory called `catalog` in `/var/www`, and cd into the new directory.
4. Change the owner: `sudo chown -R grader:grader catalog`
5.  Clone the project: `git clone [project link] catalog`
6.  Create a wsgi file in `/var/www/catalog folder`, add the following:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
7. Rename `project.py` to `__init__.py`
8. Install and start the virtual machine
`sudo apt-get install python-pip`
`sudo pip install virtualenv`
`sudo virtualenv venv`
`source venv/bin/activate`
`sudo chmod -R 777 venv`
9. Install python packages
`pip install Flask httplib2 sqlalchemy oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests`
`sudo pip install --upgrader pip`
10. In `__init__.py`, change `client_secrets.json` to `/var/www/catalog/catalog/client_secrets.json`
11. Enable VirtualHost:
- `sudo nano /etc/apache2/sites-available/catalog.conf`
- Paste the following into the file
```
<VirtualHost *:80>
    ServerName 54.198.69.144
    ServerAdmin grader@54.198.69.144
    ServerAlias 54.198.69.144.xip.io
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
12. Setup database
- Install packages: `sudo apt-get install libpq-dev python-dev`, then `sudo apt-get install postgresql postgresql-contrib`
- `sudo su - postgres`
- `psql`
- `CREATE USER catalog WITH PASSWORD [password];`
- `ALTER USER catalog CREATEDB;`
- `CREATE DATABASE catalog WITH OWNER catalog;`
- `\c catalog`
- `REVOKE ALL ON SCHEMA public FROM public;`
- `GRANT ALL ON SCHEMA public TO catalog;`
- Quit the postgrel command line: `\q` and then `exit`
13. Change all engine to `engine = create_engine('postgresql://catalog:[password]@localhost/catalog)` in catalog project
14. Initiate the database by running ` python database_setup.py` and `python lotsofitems.py`
15. `sudo a2ensite catalog` and then try `sudo service apache2 restart`.
16.  Enter your application address http://54.198.69.144.xip.io into the browser. You should see your application on the browser!
#### Reference
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
https://blog.codeasite.com/how-do-i-find-apache-http-server-log-files/
https://httpd.apache.org/docs/2.4/vhosts/examples.html
