# linuxServer

Another project for the [Udacity Full
Stack Nanodegree](https://classroom.udacity.com/vimdegrees/nd004/); proving
acces to a database via PostgreSQL and Flask, hosted via a Ubuntu Linux-based Amazon AWS Lightsail instance.

## Submission

IP address: [52.34.52.72](http://52.34.52.72)

Hostname: [ec2-54-218-251-215.us-west-2.compute.amazonaws.com](ec2-54-218-251-215.us-west-2.compute.amazonaws.com)

SSH port: 2200

Private Key: udacity

Grader: `ssh -i udacity -p 2200 grader@54.218.251.215`

## Dependencies
* Apache 2
* Python 2
* Flask
* PostgreSQL 

## Building

### Register on Amazon

![](img/lightsail.jpeg)

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
3. Create application directory: `sudo mkdir catalog`
4. Move inside this directory: `cd catalog`
5. Clone the Catalog App: `sudo git clone <catalog repo> catalog`
6. Move to the inner directory: `cd catalog`
8. Rename `application.py` to `__init__.py` with `sudo mv application.py __init__.py`
9. Edit `database_setup.py` and `fill_catalog.py` changing `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Install the virtual environment `sudo pip install virtualenv`
12. Create a new virtual environment with `sudo virtualenv venv`
13. Activate the virutal environment source `venv/bin/activate`
14. Change permissions `sudo chmod -R 777 venv`
15. Use pip to install dependencies `pip install requirements.txt`
16. Create database schema `sudo python database_setup.py`

## Configure and Enable New Virtual Host
1. Create FlaskApp.conf to edit: `sudo vim /etc/apache2/sites-available/catalog.conf`
2. Add the following lines of code:

	```
        <VirtualHost *:80>
    ServerName 54.218.251.215
    ServerAlias ec2-54-218-251-215.us-west-2.compute.amazonaws.com
    ServerAdmin admin@54.218.251.215
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
3. Enable the virtual host: `sudo a2ensite FlaskApp`

## Define [WSGI](http://wsgi.readthedocs.io/en/latest/what.html) 
1. Create the `.wsgi` under /var/www/FlaskApp:

	```
	cd /var/www/catalog
	sudo vim catalog.wsgi
	```
2. Add the following lines of code:

	```
        import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")
    from catalog import app as application
    application.secret_key = 'supersecretkey'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `


## References
* [How To Set Up a Firewall with UFW on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04)
* [@iliketomatoes's example](https://github.com/iliketomatoes/linux_server_configuration)
* [Adding and Managing Users on PostgreSQL](https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e)
