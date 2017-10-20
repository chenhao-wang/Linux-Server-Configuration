# Project 5 of the Udacity Full Stack Nanodegree programme
_Configuring a Linux server to host a web app securely._

# Server details
IP address: `13.58.70.243`

SSH port: `2200`

URL: `http://ec2-13-58-70-243.us-east-2.compute.amazonaws.com/`

# Tasks
####  Get your server.
1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.
2. Follow the instructions provided to SSH into your server.
#### Secure your server.
3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
#### Give grader access.
6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.
#### Prepare to deploy your project.
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.

- If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server: sudo apt-get install libapache2-mod-wsgi-py3.
11. Install and configure PostgreSQL:
- Do not allow remote connections
- Create a new database user named catalog that has limited permissions to your catalog application database.
12. Install git.
####  Deploy the Item Catalog project.
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!


# Configuration changes

## SSH to AWS Lightsail Instance
1. down private key from website account page -> SSH keys
2. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory).
3. connect via ssh
```
    ssh -i ~/.ssh/localkey.pem your_user_name@13.58.70.243
```

## Add a new user named grader (sudo group)
1. `sudo adduser grader`
2. `nano /etc/sudoers`
3. `touch /etc/sudoers.d/grader`
4. `nano /etc/sudoers.d/grader`, add `ALL=(ALL:ALL) ALL`, then ctrl + x => yes and enter to quit and save

## Set ssh login using key-pair (private-public)
1. generate keys at local by `ssh-keygen ~/.ssh/authorized_keys`, then you'll get two keys authorized_keys and authorized_keys.pub
2. deploy public on AWS virtual machine, ssh in as former steps, then
`su - grader`
`mkdir .ssh`
`touch .ssh/authorized_keys`
`nano .ssh/authorized_keys`
copy the authorized_keys.pub content and paste in it, quit & save
`chmod 700 .ssh`
`chmod 644 .ssh/authorized_keys`

Now you can login as grader
`ssh -i ~/.ssh/authorized_keys grader@13.58.70.243`
 
## Update all currently installed packages

`apt-get update` - to update the package indexes

`apt-get upgrade` - to actually upgrade the installed packages

## Disable root login
Change the following line in the file `/etc/ssh/sshd_config`:

From `PermitRootLogin without-password` to `PermitRootLogin no`.

## Change timezone to UTC
Check the timezone with the `date` command. This will display the current timezone after the time.
If it's not UTC change it like this:

`sudo timedatectl set-timezone UTC`

## Change SSH port from 22 to 2200
1. Edit the file `/etc/ssh/sshd_config` and change the line `Port 22` to:

`Port 2200`
#### Note:
after failed several times, you'd better modify the settings in AWS Lightsail website from your-instance => Networking => Firewall => Custom TCP 2200
  
2. Then close current ssh connection and now you will need to use the following command to login to the server:

`ssh -i ~/.ssh/authorized_keys grader@13.58.70.243 -p 2200`

## Configuration Uncomplicated Firewall (UFW)
By default, block all incoming connections on all ports:

`sudo ufw default deny incoming`

Allow outgoing connection on all ports:

`sudo ufw default allow outgoing`

Allow incoming connection for SSH on port 2200:

`sudo ufw allow 2200/tcp`

Allow incoming connections for HTTP on port 80:

`sudo ufw allow www`

Allow incoming connection for NTP on port 123:

`sudo ufw allow ntp`

To check the rules that have been added before enabling the firewall use:

`sudo ufw show added`

To enable the firewall, use:

`sudo ufw enable`

To check the status of the firewall, use:

`sudo ufw status`

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.5/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
 
## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/chenhao-wang/Item-Catalog.git`
6. Rename the project's name `sudo mv ./Item_Catalog_UDACITY ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `website.py` to `__init__.py` using `sudo mv website.py __init__.py`
9. Edit `database_setup.py`, `website.py` and `functions_helper.py` and change engine to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Install pip `sudo apt-get install python-pip`
11. Use pip to install dependencies `sudo pip install -r requirements.txt`
12. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
13. Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
		ServerName 13.58.70.243
		ServerAdmin your_account_email@gmail.com
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
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'super_secret_key'
	```
## Update Google OAuth Path
1. change relative path to absolute path
in __init__.py, there are two places need to notice
 ```
path = '/var/www/FlaskApp/FlaskApp'
CLIENT_ID = json.loads(open(path + '/client_secrets.json', 'r').read())\['web']\['client_id']
```
in gconnect()
```
oauth_flow = flow_from_clientsecrets(path + '/client_secrets.json', scope='')
```
2. update client_secrets
go to google API console
locate to Credentials
- add `http://ec2-13-58-70-243.us-east-2.compute.amazonaws.com` and `http://13.58.70.243` to Authorized JavaScript origins
- add `http://ec2-13-58-70-243.us-east-2.compute.amazonaws.com/oauth2callback` to Authorized redirect URIs

then download json file and open it, copy and paste that to your client_secrets file in remote service.

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## Reference
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config/blob/master/README.md
- https://discussions.udacity.com/t/app-not-running-after-all-this-time/362885