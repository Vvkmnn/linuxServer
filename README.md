# linuxServer

![](img/1.jpeg)

Another project for the [Udacity Full
Stack Vimdegree](https://classroom.udacity.com/vimdegrees/nd004/); proving
acces to a database via PostgreSQL and Flask, hosted via a Ubuntu Linux-based Amazon AWS Lightsail instance.

## Result

IP address: [http://18.218.229.119/](http://18.218.229.119/)
SSH port: 2200

## Building

### Register on Amazon

Create an [Amazon AWS Lightsail account](https://portal.aws.amazon.com/), and
create a [Ubuntu
Linux](https://lightsail.aws.amazon.com/ls/docs/getting-started/article/getting-started-with-amazon-lightsail)
server instance.

## SSH into the instance

1. Download Private Key in the __Account__ section on Amazon Lightsail.
2. Move the private key into the directory `~/.ssh` (~ is your home directory).
3. In your terminal, type in: `chmod 400 ~/.ssh/Lightsail-key.pem`.
4. After, type in: `ssh -i ~/.ssh/Lightsail-key.pem ubuntu@18.218.229.119`.

## Create a new user named grader

1. `sudo adduser grader` (*password: `password`*)
2. `sudo touch /etc/sudoers.d/grader`
3. `sudo vim /etc/sudoers.d/grader`, paste `grader ALL=(ALL:ALL) NOPASSWD:ALL`, save & quit

## SSH login using keys

1. Generate keys on local machine with the command: `ssh-keygen`, then save the private key in `~/.ssh` on local machine;

	On you virtual machine:
	```
	$ sudo su - grader
	$ mkdir .ssh
	$ sudo vim .ssh/authorized_keys
	```
	Copy the public key (_with the extension .pub_) generated on your local machine to this file and save

	```
	$ sudo chmod 700 .ssh
	$ sudo chmod 644 .ssh/authorized_keys
	```

3. reload SSH with the command: `service ssh restart`
4. Login with grader:

	`ssh -i ~/.ssh/udacity_key.rsa grader@18.218.229.119`

## Update installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change SSH port from 22 to 2200

1. Use `sudo vim /etc/ssh/sshd_config` and then change Port 22 to Port 2200, save & quit.
2. Reload SSH: `sudo service ssh restart`

__Note:__ Remember to add and save port 2200 with __Application__ as __Custom__ and __Protocol__ as __TCP__ in the Networking section of your instance on Amazon Lightsail:

![](img/lightsail.jpeg)

## Configure Uncomplicated Firewall (UFW)

Only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

```
sudo ufw allow ssh
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
sudo ufw status
```

![](img/firewall.png)

## Configure local timezone

1. Configure the time zone `sudo dpkg-reconfigure tzdata`

## Install and configure Apache to serve a Python mod_wsgi app

1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL

1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and  in postgreSQL shell
    ```
    postgres=# CREATE DATABASE catalog;
    ```
6. Create a new user named catalog
    ```
	postgres=# CREATE USER catalog;
	```
7. Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
8. Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
9. Quit postgreSQL: `postgres=# \q`
10. Exit from user "postgres": `exit`

## Git checkout the Catalog App 
1. Install Git: `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create application directory: `sudo mkdir FlaskApp`
4. Move inside this directory: `cd FlaskApp`
5. Clone the Catalog App: `sudo git clone https://github.com/vvkmnn/itemCatalog.git catalog`
6. Rename the project: `sudo mv ./catalog ./FlaskApp`
7. Move to the inner FlaskApp directory: `cd FlaskApp`
8. Rename `server.py` to `__init__.py` with `sudo mv website.py __init__.py`
9. Edit `database_setup.py` and `fill_catalog.py` changing `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies -
	* `sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests`
	* `sudo pip install flask packaging oauth2client redis passlib flask-httpauth`
13. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
14. Create database schema `sudo python database_setup.py`
15. Fill database `sudo pip install fill_catalog.py`


## Configure and Enable New Virtual Host
1. Create FlaskApp.conf to edit: `sudo vim /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code:

	```
	<VirtualHost *:80>
		ServerName lotsofmenus.py
		ServerAdmin vvkmnn@gmail.com
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
3. Enable the virtual host: `sudo a2ensite FlaskApp`

## Define [WSGI](http://wsgi.readthedocs.io/en/latest/what.html) 
1. Create the `.wsgi` under /var/www/FlaskApp:

	```
	cd /var/www/FlaskApp
	sudo vim flaskapp.wsgi
	```
2. Add the following lines of code:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'super_secret_key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `
