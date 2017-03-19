
# FSND Linux Server Configuration

Installation of an Ubuntu server instance to host a Flask application.
The application hosted on this intance is from the Udacity FSND Item Catalog project,
and is a fulfillment of the larger FSND program.

* Public IP: http://34.205.145.118
* SSH Port: 2200
* ServerAlias: http://ec2-34-205-145-118.us-west-2.compute.amazonaws.com

## Software installed
* finger: To display information about server users
* ntp: To synchronize time between servers
* apache2: Linux Web Server
* mod-wsgi: apache module for hosting python applications
* python-setuptools: For easily downloading, updating, developing python tools
* pip: Python package manager
* Flask: Python Web Development framework
* SQLAlchemy: SQL Object Relational mapper for python
* Postgresql: Object relational DBMS
* httplib2: HTTP client library for python
* requests: Another python HTTP library
* oauth2client: Authentication API for secure Facebook and Google Plus login.
* psychopg2: postgresql adapter for python

## Configuration summary
1. Setup virtual server instance with AWS Lightsail, connect via SSH
2. Added user ```grader``` with password 'Gr@der1', and granted sudo priveleges
3. Updated and upgradered current software packages 
4. Changed SSH port from 22 to 2200
5. Cofigured Firewall in AWS Lightsail
6. Generated SSH key pair on local machine(```udacity_key_rsa```), and saved in ```grader```(```.ssh/authorized_keys```)
7. Configured ```ufw``` Firewall for user ```grader```
8. Configured timezone to UTC and installed ```ntp``` for better synchronization
9. Installed ```apache2``` and ```mod-wsgi```
10. Installed ```git``` and setup global configuration in preparation for cloning Item Catalog repo
11. Setup ```virtualenv``` for deploying Flask app
12. Cloned repository in ```virtualenv``` and installed necessary dependencies
13. Installed ```postgresql```, setup database, and updated Item Catalog application accordingly
14. Reconfigured ```oauth2``` login credentials for Facebook and Google Plus 

## Server Instance setup using AWS Lightsail

1. Set up an Ubuntu server instance in AWS Lightsail as per Udacity [instructions](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e#).

## Add new user with sudo privileges

1. Once you have connected to your Lightsail instance via SSH, type the following into the command line interface to create
a new user named 'grader'. Pssword for this instance is 'Gr@der1'.

```
$ sudo adduser grader
```

2. Grant the user 'grader' sudo privileges using the following command:

```
$ sudo visudo
```
This will open up the sudo configuration file. Add the following line below 'root ALL=(ALL:ALL) ALL':
```
grader ALL=(ALL:ALL) ALL
```

## Update software packages on server instance

1. Enter the following lines into the command line interface:
```
$ sudo apt-get update

$ sudo apt-get upgrade
```

## Change SSH port to 2200 and configure access

1. Enter the following command to access the server's SSH configuration file:
```
$ sudo nano /etc/ssh/sshd_config
```

2. Change the SSH port from 22 to 2200
3. Change ```PermitRootLogin without-password``` to ```PermitRootLogin no```.
4. Change ```PasswordAuthentication``` from ```yes``` to ```no``` (this is only temporary).
5. Add the following to the end of the file:
```  
UseDNS no
AllowUsers grader
```

## Configure Firewall in AWS Lightsail
1. In your AWS Lightsail instance, click on the Netwroking tab.
2. Add the following rules to the Firewall configuration:
Custom  TCP  2200
Custom  UDP  123

## Generate SSH keys on local machine
1. On your local machine enter the following command using terminal (Mac), bash (PC), or the like:
```
$ ssh-keygen
```

This will generate a public/private key pair that you will use to login to your server securely via SSH.
2. Switch back to your Lightsail console interface and enter the following command:
```
$ sudo su - grader
grader@PUBLIC_IP:~$ sudo mkdir .ssh
grader@PUBLIC_IP:~$ sudo nano .ssh/authorized_keys
```

3. Switch to your local machine and enter the folowing command to display the contents of your SSH key:
```
$ sudo cat ~/.ssh/udacity_key_rsa.pub
```

4. Switch back to Lightsail console and paste the contents of the SSH key into authorized_keys file that you created.
5. Enter the following commands to set access privileges to these directories:
```
grader@PUBLIC_IP:~$ sudo chmod 700 .ssh
grader@PUBLIC_IP:~$ sudo chmod 644 .ssh/authorized_keys
```

6. Now switch back to your local machine and login as grader:
```
$ ssh grader@PUBLIC_IP -p 2200 -` ~/.ssh/udacity_key_rsa
```

7. Reconfigure settings in SSHD config file:
```
grader@PUBLIC_IP:~$ sudo nano /etc/ssh/sshd_config
```
Change ```PasswordAuthentication``` from ```yes``` to ```no```.

## Configure UFW, Uncomplicated Firewall
1. Enter the following commands:
```
 grader@PUBLIC_IP:~$ sudo ufw default deny incoming

 grader@PUBLIC_IP:~$ sudo ufw default allow outgoing

 grader@PUBLIC_IP:~$ sudo ufw allow 80/tcp

 grader@PUBLIC_IP:~$ sudo ufw allow 2200/tcp

 grader@PUBLIC_IP:~$ sudo ufw allow 123/udp

 grader@PUBLIC_IP:~$ sudo ufw enable
 ```
 You can check the Firewall status with ```sudo ufw status```.

 ## Configure Timezone
 1. Enter the following command:
 ```
 grader@PUBLIC_IP:~$ sudo dkpg-reconfigure tzdata
 ```
 This will bring up a UI to set the current timezone configuration. Click enter twice, which will
 set the timezone to UTC.

 2. Enter the following to install ntp, which will facilitate time synchronization between servers:
 ```
 grader@PUBLIC_IP:~$ sudo apt-get install ntp
 ```

 ## Configure Apache/mod-wsgi
 1. Install Apache by entering the following into the console:
 ```
 grader@PUBLIC_IP:~$ sudo apt-get install apache2
 ```

 2. Install mod_wsgi for Python environment with the following command:
 ```
 grader@PUBLIC_IP:~$ sudo apt-get install python-setuptools libapache2-mod-wsgi
 ```

 3. Restart the Apache server:
 ```
 grader@PUBLIC_IP:~$ sudo service apache2 restart
 ```

 ## Install git and clone Item Catalog repository
 1. Install and configure git:
 ```
 grader@PUBLIC_IP:~$ sudo apt-get install git

 grader@PUBLIC_IP:~$ git config --global user.name 'YOUR NAME'
 grader@PUBLIC_IP:~$ git config --global user.email 'YOUR EMAIL'
 ```

 2. Install the following package in preparation for deploying your Flask application:
 ```
 grader@PUBLIC_IP:~$ sudo apt-get install libapache2-mod-wsgi python-dev
 ```

 3. cd into the following directory:
 ```
 grader@PUBLIC_IP:~$ cd /var/www
 ```

 4. Ceate a directory called catalog and cd into it:
 ```
 grader@PUBLIC_IP:/var/www$ sudo mkdir catalog   
 grader@PUBLIC_IP:/var/www$ cd catalog
 ```

 5. Create another directory called catalog in this directory:
 ```
 grader@PUBLIC_IP:/var/catalog$ sudo mkdir catalog
 ```

6. Create the following directories within this directory:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo mkdir static templates
```

7. Create a file to store the initial Flask logic:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo nano __init__.py
```
```python
from flask import Flask

app = Flask(__name__)
@app.route("/")
def hello():
    return "Greetings Dawgs!"
if __name__ == "__main__":
  app.run()
```

8. Install Pip, Flask, and virtualenv:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo apt-get install python-pip

grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install virtualenv
```

Rename virtualenv 'venv':

```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo virtualenv venv

```

Grant all privileges to venv:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo chmod -R 777 venv
```

Activate virtual environment:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ source venv/bin/activate
```

Your console should now look like this:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$
```

Now install Flask:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ pip install Flask
```

Deactivate the virtual environment:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ deactivate
```

9. Configure Apache mod-wsgi module:

```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo nano /etc/apache2/sites-available/catalog.conf
```
Paste the following into the newly made catalog.conf file:
```
VirtualHost *:80>
    ServerName PUBLIC_IP
    ServerAdmin admin@PUBLIC_IP
    ServerAlias ec2-PUBLIC_IP.us-west-2.compute.amazonaws.com
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

Now create the mod-wsgi file:
```
grader@PUBLIC_IP:/var/www/catalog/$ sudo nano catalog.wsgi
```

Paste the following code into this newly created file:

```python
#/user/bin/python
import sys
import logging
logging.basicConfig(stream=stderr)
sys.path.insert(0,"/var/www/catalog/catalog/")

from __init__ import app as application
application.secret_key == 'Your application secret key'
```
(You will be renaming you app __init__.py, which is why I have entered 'import __init__
as app').

## Cloning the Item Catalog project

1. Make sure that you are in the catalog/catalog directory and enter the following:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo git clone https://github.com/sharkwhistle/Item_catalog.git
```

2. Create a .htaccess file from the catalog/ directory:
```
grader@PUBLIC_IP:/var/www/catalog$ sudo nano .htaccess
```

Paste the following into the newly created file:
```
RedirectMatch 404 /\.git
```
## Install necessary dependencies in the virtual environment:

1. Activate the virtual environment:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ source venv/bin/activate
```

```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo apt-get install python-setuptools
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install Flask
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install requests
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install Flask-SQLAlchemy
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install httplib2
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo pip install oauth2client
```

2. Restart Apache:
```
grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo service apache2ctl restart
```
## Install Postrgresql and set up database

1. Restart virtual environment and install postrgresql:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo apt-get install postrgresql postgresql-contrib
```

2. SU into user postgres and startup psql:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo su - postgres
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ psql
```

3. Create user 'catalog' and set password for (Password for this instance: c@talog)
```
postgres=# CREATE USER catalog WITH PASSWORD 'c@talog';
```

4. Allow this user to create databases:
```
postgres=# ALTER USER catalog CREATEDB;
```

5. Create 'catalog' database:
```
postgres=# CREATE DATABASE catalog WITH USER catalog;
```

6. Connect to database:
```
postgres=# \c catalog
```

7. Set sole credentials for user 'catalog':
```
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
```

8. Quit psql and exit:
```
catalog=# \q
postgres@PUBLIC_IP~$ exit
```

9. Update 'create_engine =' line in __init__.py and database_setup.py files:
Line should look like this --
```python
engine = create_engine('postgresql://catalog:c@talog@localhost/catalog/')
```
Note: This line will need to be edited in any other files that use the 'create_engine' function. In my case, this also included the lotsofitems.py file.

10. Run database_setup file to populate catalog database:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ python database_setup.py
```

11. Restart Apache:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo service apache2 restart
```

## Update FB and Google Plus oauth credentials

Oauth credentials will need to be updated in FB and Google for signin functionality to work properly.

1. For FB, go to https://developers.facebook.com and change the site url for your app to reflect your Lightsail instance.
```
Site URL

http://34.205.104.118/
```

2. Fo Google Plus, go to https://console.developers.google.com and change the oauth credentials in you app console. You will need to reset the authorized Javascript origins to reflect your instance IP address and Alias:
```
Authorized Javascript origins:

http://34.205.104.118
http://ec2-34-205-104-118.us-west-2.compute.amazonaws.com

https://34.205.104.118
https://ec2-34-205-104-118.us-west-2.compute.amazonaws.com
```

You will also need to reset the Authorized Redirect URI:
```
http://ec2-34-205-104-118.us-west-2.compute.amazonaws.com/oauth2callback
https://ec2-34-205-104-118.us-west-2.compute.amazonaws.com/oauth2callback
```

3. After resetting these credentials you will need to update your fb_client secrets.json and client_secrets.json file accordingly to reflect your new client secret and App ID information. You will also need to update the following in your __init__.py file:
```python
app_id = json.loads(open(r'/var/www/catalog/catalog/fb_client_secrets.json', 'r').read())[
        'web']['app_id']
app_secret = json.loads(
    open(r'/var/www/catalog/catalog/fb_client_secrets.json', 'r').read())['web']['app_secret']
```

You will need to update the full application path as shown above.
You will also need to update this information for your Google Plus login:
```python
oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')
oauth_flow.redirect_uri = 'postmessage'
credentials = oauth_flow.step2_exchange(code)
```

4. Restart Apache server:
```
(venv) grader@PUBLIC_IP:/var/www/catalog/catalog$ sudo service apache2 restart
```

## Resources

1. DigitalOcean article on deploying a [Flask](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) app on Ubuntu.
2. Granting [sudo](https://www.digitalocean.com/community/tutorials/how-to-add-delete-and-grant-sudo-privileges-to-users-on-a-debian-vps) privileges.
3. [Idroot](http://idroot.net/tutorials/how-to-change-ssh-port-in-ubuntu/) article on ssh port configuration.
4. Digital Ocean article on installing and using [postgresql](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04).
5. [Ubuntu](https://help.ubuntu.com/community/PostgreSQL) postgresql documentation.
6. Fantastic walkthrough from [elnobun](https://github.com/elnobun/Linux-Server-Configuration).

