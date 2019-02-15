# README

Udacity Full Stack Web Developer Nanodegree Program

Project 5: Linux Server Configuration

Author: Aleksandr Zonis

## Description

In this project, I configured an Ubuntu Linux server instance on AWS and deployed a Flask app to the Apache server([Flask Catalog Application](https://github.com/AlikZi/udacity-fullstack-catalog-project4)). The app is available at [http://35.175.70.121.xip.io/](http://35.175.70.121.xip.io/).

The purpose of this project is to deploy a Flask (python) application on the Amazon Web Service(AWS) Lightsail platform, take a baseline installation of a Linux server and prepare it to host web applications. Secure server from a number of attack vectors, install and configure a database server, and deploy "Catalog Application" onto it.

Below you can find a computational narrative.

## Step 1. Create a New Instance on AWS Lightsail

1. First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.

2. Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon data center.

3. For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu 16.04 as the operating system.

4. Choose your instance plan. (I picked the cheapest one, which was $3.50 at the moment)

5. Give your instance a hostname.

6. Click the 'Create' button.

7. Once the instance is running. Find 'Networking' and create Static IP address


## Step 2. Log Into Your Instance

1. To log into your instance you will need to download SSH key. Once you are on the instance page click on the 'Account' button. Find 'SSH Keys' and download 'Default key' with `.pem` extension to your local machine. 

2. Once the key is downloaded place it in `.ssh` file in your home directory, name it as you like. 

3. Change permissions to the key
 
 `chmod 400 ~/.ssh/{ default key }.pem`

4. Now you are ready to log in from your terminal

`ssh -i ~/.ssh/{ default key }.pem ubuntu@{ ip address }`


## Step 3. Create a New User and Set Up Key Based Login

1. Once you are logged in and in the Ubuntu terminal, update and upgrade Ubuntu

`sudo apt-get update`
`sudo apt-get upgrade`

2. Create a new user

`sudo adduser { username }`

3. Give sudo access to user:
    
    - `sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/{ username }`

    - `sudo nano /etc/sudoers.d/{ username }`

    - Change Ubuntu to { username } and save the file.

4. Go to username directory

`cd /home/{ username }`

5. Create `.ssh` directory inside username

`sudo mkdir .ssh`

6. Change user and group ownership of the .ssh folder

`sudo chown { username }:{ username } .ssh`

7. Create 'authorized_keys' file

`sudo touch .ssh/authorized_keys`

8. Change user and group ownership of 'authorized_keys' file

`sudo chown { username }:{ username }`


Once steps above are completed switch to your local machine terminal. We will generate a key pair for the new user.

9. Generate a key

`ssh-keygen`

Save in the suggested directory, such as '/Users/< username >/.ssh/< keyname >'. You can give whatever name to the key you prefer. It will generate to two keys: <keyname> and <keyname>.pub

10. Copy and paste the pub key:

    - `cat .ssh/<keyname>.pub`

    - Copy the content of the public key

    - In the Ubuntu terminal open {username}/.ssh/authorized_keys file

    `sudo nano {username}/.ssh/authorized_keys`

    - Paste the content of the public key and save the file

11. Set permissions to `.ssh` and `.ssh/authorized_keys`:

    - `sudo chmod 700 .ssh`

    - `sudo chmod 644 .ssh/authorized_keys`

12. Allow to log in only with the key pair, not password:

    - `sudo nano /etc/ssh/sshd_config`

    - Find `PasswordAuthentication` change to `no`

13. You are ready to log in. On your local machine run:

    - `ssh -i <keyname> { username }@{ IP address}`


## Step 4. Secure your server. Configure Firewall

1. Add port 2200

	- On Ubuntu terminal open `sshd_config` file:

	`sudo nano /etc/ssh/ssh_config`

	- Find `Port 22`, don't delete it yet to avoid getting locked out. On the next line add `Port 2200`

	- Go to the Lightsail AWS instance. Find 'Networking'. Add  port to the 'Firewall' table.
	Set Application to 'Custom', protocol to 'TCP', port to 2200

2. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

	- `sudo ufw default deny incoming` 

	- `sudo ufw default allow outgoing`

	- `sudo ufw allow ssh`

	- `sudo ufw allow 2200`

	- `sudo ufw allow http`

	- `sudo ufw allow ntp`

	- `sudo ufw enable`

	- Check if all configurations are set.

	`sudo ufw status`

3.  Change the SSH port from 22 to 2200.

	- On your local machine try to log in using port 2200

	`ssh -i <keyname> { username }@{ IP address } -p 2200`

	- If you logged in successfully. Then follow these steps:

		- In `sshd_config` file delete `port 22` line and save the changes

		`sudo nano /etc/ssh/sshd_config`

		- Go to the Lightsail AWS instance. Find 'Networking'. Delete Port 22 connection.

		- `sudo ufw deny 22`

		- Check is configurations are changed

		`sudo ufw status`


## Step 5. Installing Apache and mod_wsgi

1. Install Apache using your package manager with the following command,
confirm that Apache is working by visiting your Public IP: 

` sudo apt-get install apache2`

2. Install mod_wsgi: 

`sudo apt-get install libapache2-mod-wsgi`

3. Delete the directory /var/www/html

`sudo rm -rf /var/www/html`

4. Create the new directory

`sudo mkdir /var/www/catalog`

5. Create a test file inside 'catalog' directory with the **fig-a** text, save the file:

`sudo nano /var/www/catalog/catalog.wsgi`

**fig-a**
```shell
def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
```

6. Go to Apache configuration files:

`cd /etc/apache2/sites-available`

7. Copy default file

`sudo cp 000-default.conf catalog.conf`

8. Make changes to file. 
	- `sudo nano catalog.conf`
	- Change from `DocumentRoot /var/www/html` to `DocumentRoot /var/www`
	- Before closing tag </VirtualHost> insert `WSGIScriptAlias / /var/www/catalog/catalog.wsgi`
	- Close and Save the file

9. Disable 000-default and enable catalog.conf, reload Apache after. Read about [a2dissite/a2ensite](http://manpages.ubuntu.com/manpages/xenial/man8/a2ensite.8.html) commands.

	- `sudo a2dissite 000-default.conf`
	- `sudo a2ensite catalog.conf`
	- `sudo service apache2 reload`

10. Go to Public IP address, it should display 'Hello World' message.


## Step 6. Setting up the Application. [DigitalOcean. "How To Deploy a Flask Application on an Ubuntu VPS"](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) was a very helpful resource. I recommend trying to deploy a simple Flask application from this tutorial if you are doing it for the first time. 

1. Install Git.

`sudo apt-get install git`

2. Clone your application from the github:

	- `cd /var/www/catalog` 
	
	- Clone your application inside new 'catalog' directory:

	`sudo git clone https://github.com/AlikZi/udacity-fullstack-catalog-project4.git catalog`

3. Replace with the following lines of code to the 'catalog.wsgi' file:

```shell
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```

4. Inside /var/www/catalog/catalog rename 'project.py' to '__init__.py' and change the code in the bottom of  the '__init__.py' file to match:

```shell
if __name__ == "__main__":
    app.run()
 ```

5. Create a virtual environment for our flask application inside /var/www/catalog/catalog:

	- Install *python* and *virtualenv*

	`sudo apt-get install python-pip`
	`sudo pip install virtualenv`

	- Run the following command, *venv* is the name for a temporary environment:

	`sudo virtualenv venv`

	- Activate the virtual environment:

	`source venv/bin/activate`

	- Install neccessary modules:

	```shell
	sudo pip install Flask
	sudo pip install httplib2
	sudo pip install sqlalchemy
	sudo pip install oauth2client
	sudo pip install psycopg2
	```

	- Deactivate the virtual environment:

	`deactivate`
 
 6. Now your directory structure should look like this:

 ```shell
|--------catalog
|----------------catalog
|-----------------------static
|-----------------------templates
|-----------------------addproducts.py
|-----------------------database_setup.py
|-----------------------README.md
|-----------------------venv
|-----------------------__init__.py
|----------------catalog.wsgi
```

7. Add (or replace) the following lines of code from **fig-b** to the file to configure the virtual host:

`sudo nano /etc/apache2/sites-available/catalog.conf`

**fig-b**
```shell
<VirtualHost *:80>
		ServerName http://35.175.70.121.xip.io/
		ServerAdmin zonis7@gmail.com
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

8. It should be already enabled, but just in case run:

`sudo a2ensite catalog.conf`

9. For troubleshooting, check logs for errors:

```shell
sudo cat /var/log/apache2/error.log
sudo cat /var/log/apache2/access.log
```


## Step 7. Configure PostgreSQL Database to serve data.

1. Create a new database user named catalog that has permissions to the Catalog Application database.

```shell
sudo apt-get install postgresql
sudo passwd postgres
su postgres

postgres~: psql

>> CREATE USER catalog;
CREATE ROLE
>> ALTER USER catalog WITH PASSWORD 'catalog';
ALTER ROLE;
>> CREATE DATABASE catalog WITH OWNER catalog;
CREATE DB
>> REVOKE ALL ON SCHEMA public FROM public;
>>GRANT ALL ON SCHEMA public TO catalog;
```

2. Quit psql.

3. As a super user make configurations for PostgreSQL instead of SQLite in files '__init__.py', 'database_setup.py' and 'addproducts.py':

`engine = create_engine("postgresql://{ DB user }:{ DB user password }@{ IP address }/{ DB name }")`

or 

`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

4. Reload Apache:

`sudo service apache2 reload`


## Step 8. Make Final Adjustments to the Application.

1. Change path to 'client_secrests.json' in the '__init__.py' file to the relative path:

`/var/www/catalog/catalog/client_secrets.json`

2. To make sure Google Sign In is working you need to create new OAuth Client ID credentials.
	
	- Go to credentials section at [https://console.cloud.google.com/apis/api](https://console.cloud.google.com/apis/api)

	- Create new Credential

	- Enter Authorized JavaScript origins. I entered my IP address with *xip.io* extension

	`http://35.175.70.121.xip.io/`

	- Enter Redirect URIs:

	```shell
	http://35.175.70.121.xip.io/
	http://35.175.70.121.xip.io/login
	http://35.175.70.121.xip.io/gconnect
	http://35.175.70.121.xip.io/gdisconnect
	```

	- Click 'Create Credential'

	- Download JSON

	- Copy content of the downloaded JSON client secrets and paste it into 'client_secrets.json' in the */var/www/catalog/catalog*

3. To view your application, open your browser and navigate to the domain name or IP address that you entered in your virtual host configuration.


## Reference

1. [DigitalOcean. How To Install the Apache Web Server on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04)

2. [DigitalOcean. How To Configure the Apache Web Server on an Ubuntu or Debian VPS](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)

3. [DigitalOcean. How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

4. [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)

5. Apache [a2dissite/a2ensite](http://manpages.ubuntu.com/manpages/xenial/man8/a2ensite.8.html) commands


## License

The contents of this repository are covered under the [MIT license](License.md).



