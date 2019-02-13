# README

Udacity Full Stack Web Developer Nanodegree Program

Project 5: Linux Server Configuration

Author: Aleksandr Zonis

## Description

Take a baseline installation of a Linux server and prepare it to host web applications. Secure server from a number of attack vectors, install and configure a database server, and deploy "Catalog Appliction" onto it.


## **Step 1.** Create a New Instance on AWS Lightsail

1. First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.

2. Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.

3. For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu 16.04 as the operating system.

4. Choose your instance plan. (I picked th cheapest one, which was $3.50 at the moment)

5. Give your instance a hostname.

6. Click 'Create' button.

7. Once instance is running. Find 'Networking' and create Static IP address


## **Step 2** Log Into Your Instance

1. To log into your instance you will need to download SSH key. Once you are on the instance page click on the 'Account' button. Find 'SSH Keys' and download 'Default key' with `.pem` extension to your local machine. 

2. Once the key is downloaded place it in `.ssh` file in your home directory, name it as you like. 

3. Change permissions to the key
 
 `chmod 400 ~/.ssh/{ default key }.pem`

4. Now you are ready to log in from your terminal

`ssh -i ~/.ssh/{ default key }.pem ubuntu@{ ip address }`


## **Step 3** Create a New User and Set Up Key Based Login

1. Once you are logged in and in the Ubuntu terminal, update and upgrade Ubuntu

`sudo apt-get update`
`sudo apt-get upgrade`

2. Create a new user

`sudo adduser { username }`

3. Give sudo access to user:
	
	- `sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/{ username }`

	- `sudo nano /etc/sudoers.d/{ username }`

	- Change ubuntu to { username } and save the file.

4. Go to username directory

`cd /home/{ username }`

5. Create `.ssh` directory inside username

`sudo mkdir .ssh`

6. Change user and group ownerhsip of .ssh folder

`sudo chown { username }:{ username } .ssh`

7. Create 'authorized_keys' file

`sudo touch .ssh/authorized_keys`

8. Change user and group ownerhsip of 'authorized_keys' file

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


## **Step 4** Secure your server. Configure Firewall

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


## **Step 5** Installing Apache and mod_wsgi

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

8. Make chages to to file. 
	- `sudo nano catalog.conf`
	- Change from `DocumentRoot /var/www/html` to `DocumentRoot /var/www`
	- Before closing tag </VirtualHost> insert `WSGIScriptAlias / /var/www/catalog/catalog.wsgi`
	- Close and Save the file

9. Disable 000-default and enable catalog.conf, reload Apache after:

	- `a2dissite 000-default`
	- `sudo a2ensite catalog`
	- `sudo service apache2 reload`

10. Go to Public IP address, it should display 'Hello World' message.


## Reference
1. [DigitalOcean. How To Install the Apache Web Server on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04)
2. [DigitalOcean. How To Configure the Apache Web Server on an Ubuntu or Debian VPS](How To Configure the Apache Web Server on an Ubuntu or Debian VPS)
3. [DigitalOcean. How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)





