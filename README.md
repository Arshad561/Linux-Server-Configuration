# Linux-Server-Configuration
Part of Udacity's Full Stack Web Developer Course

# Server Configuration

- **Public IP:** 13.127.198.27
- **Port:** 2200
### you can access this application at below links ###
- http://13.127.198.27/
- http://ec2-13-127-198-27.ap-south-1.compute.amazonaws.com/

## Get Started
To complete this project, you'll need a Linux server instance. We recommend using Amazon Lightsail for this. If you don't already have an Amazon Web Services account, you'll need to set one up. Once you've done that, here are the steps to complete this project.

## Step 1: Get your server
- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources)
- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Click the `Create` button to create the instance.
- Wait for the instance to start up.

## Step 2: SSH into your Server
- Download private key from the **SSH keys** section in the **Account** section on Amazon Lightsail.
- Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your local machine
- Copy and paste content from downloaded private key file to **lightsail_key.rsa**
- Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`
- SSH into the instance: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.127.198.27`

## Step 3: Secure your server
### Update all currently installed packages
- Run `sudo apt-get update` to update packages
- Run `sudo apt-get upgrade` to install new versions of packages
- check for future updates: `sudo apt-get dist-upgrade`
### Change the SSH port from 22 to 2200
- Run `sudo nano /etc/ssh/sshd_config` to edit the mentioned file
- Change the port number from `22` to `2200`.
- Restart SSH: `sudo service ssh restart`.
### Configure the Uncomplicated Firewall
- Run `$ sudo ufw status` to check firewall status
- Run `$ sudo ufw default deny incoming` to set default firewall to deny all incomings
- Run `$ sudo ufw default allow outgoing` to set default firewall to allow all outgoings
- Run `$ sudo ufw allow 2200/tcp` to allow incoming TCP packets on port 2200 
- Run `$ sudo ufw allow www` to allow incoming TCP packets on port 80 
- Run `$ sudo ufw allow 123/udp` to allow incoming UDP packets on port 123
- Run`$ sudo ufw deny 22` to close port 22
- Run `$ sudo ufw enable` to enable firewall
- Run `$ sudo ufw status` to check current firewall status
- Update the firewall configuration on Amazon Lightsail website under **Networking**. Delete default SSH port 22 and add **port 80, 123, 2200**
- Open a new terminal and you can now ssh in via the new port 2200: <br>
`$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.127.198.27 -p 2200`

## Step 4: Give grader access
### Create a new user account named
- login as `ubuntu`, add user: `sudo adduser grader`. 
### Give grader the permission to sudo
- Edits the sudoers file: `sudo visudo`.
- add below line after 'root    ALL=(ALL:ALL) ALL'
  ```
  grader  ALL=(ALL:ALL) ALL
  ```
- save the file and exit
### Create an SSH key pair for grader using the ssh-keygen tool
- Run `ssh-keygen` on the local machine:
- Enter file in which to save the key in the local directory `~/.ssh`.Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
- Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
- Log in to the grader's virtual machine
- Create a new directory called `~/.ssh` (`mkdir .ssh`) on the grader's virtual machine
- Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
- Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
- Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`
- Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@13.127.198.27`.

## Step 5: Prepare to deploy your project
### Configure the local timezone to UTC
- Run `$ sudo dpkg-reconfigure tzdata`
### Install and configure Apache to serve a Python mod_wsgi application
- Install **Apache**: `$ sudo apt-get install apache2`
- Go to http://13.127.198.27/, if Apache is working correctly, a **Apache2 Ubuntu Default Page** will show up
- Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
- Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
- Restart **Apache**: `$ sudo service apache2 restart`
###  Install and configure PostgreSQL
- login as `grader`, Run `sudo apt-get install postgresql` to install postgresql
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```
- run `sudo su - postgres`
- Open PostgreSQL interactive terminal with `psql`
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```
- Exit psql using `\q`.
- Switch back to the `grader` user: `exit`.
- login as grader and create a new Linux user called `catalog`: `sudo adduser catalog`
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
- add below line under `root    ALL=(ALL:ALL) ALL grader  ALL=(ALL:ALL) ALL` to give sudo previliges to catalog user
  ```
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- While logged in as `catalog`, create a database: `createdb catalog`.
- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.
### Install git
- Run `$ sudo apt-get install git`
- Create dictionary: `$ mkdir /var/www/catalog`
- CD to this directory: `$ cd /var/www/catalog`
- Clone the catalog app: `$ sudo git clone 'URL OF YOUR REPO' catalog`
- Change the ownership: `$ sudo chown -R ubuntu:ubuntu catalog/`
- CD to `/var/www/catalog/catalog`
- Change file **project.py** to **__init__.py**: `$ mv project.py __init__.py`
- Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in **__init__.py** file
- Create a new project on Google API Console and download `client_scretes.json` file
- Copy and paste contents of downloaded `client_scretes.json` to the file with same name under directory `/var/www/catalog/catalog/client_secrets.json`

## Step 6: Final setup
### Installing the required packages
- Install pip: `$ sudo apt-get install python-pip`
- Install the following packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
   ```
### Setup and enble a virtual host
- Create file: `$ sudo touch /etc/apache2/sites-available/catalog.conf`
- Add the following to the file:
```
   <VirtualHost *:80>
		ServerName 13.127.198.27
		ServerAdmin admin@13.127.198.27
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
- Run `$ sudo a2ensite catalog` to enable the virtual host
- Restart **Apache**: `$ sudo service apache2 reload`

### Configure .wsgi file
- Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`
- Add content below to this file and save:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/catalog/")
   sys.path.insert(1,"/var/www/catalog/catalog")

   from catalog import app as application
   application.secret_key = 'super_secret_key'
```
- Restart **Apache**: `$ sudo service apache2 reload`

### Edit the database path
- Replace lines in `__init__.py`, `database_setup.py`, and `data.py` with `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`

### Disable defualt Apache page
- `$ sudo a2dissite 000-defualt.conf`
- Restart **Apache**: `$ sudo service apache2 reload`

### Set up database schema
- Run `$ sudo python database_setup.py`
- Run `$ sudo python lotsofitems.py`
- Restart **Apache**: `$ sudo service apache2 reload`
- Now follow the link to http://13.127.198.27/  the application should be runing online

#### Use xip.io for setting DNS name to your IP Address in order to run your application smoothly. 
