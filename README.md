# Linux Configuration
Current Site:

## Step 1
### Create Lightsail instance
* download private key from Lightsail
* change permissions of private key: chmod 600 LightsailDefaultPrivateKey-us-east-1.pem
* ssh into Lightsail server: sudo ssh -i LightsailDefaultPrivatekey-us-east-1.pem unbuntu@35.172.119.117

### Add ports to Lightsail
* UDP: 123
* TCP: 2200

### Update Server
* sudo apt-get update
* sudo apt-get upgrade

### Change ssh port from 22 to 2200
* Open the sshd_config file: $ sudo nano /etc/ssh/ssdh_config
* Change the ssh port from 22 to 2200
* sudo sevice ssh restart

### Update UFW
* allow 2200/TCP
* allow 80/TCP
* allow 123/UDP
* turn on ufw: sudo ufw enable

### Configure local timezone to UTC
* sudo dpkg-reconfigure tzdata

## Step 2
### Add Grader User
* get to root: sudo su-
* sudo adduser grader
* give sudo status: sudo nano /etc/sudoers.d/grader, and type grader ALL=(ALL:ALL) ALL

### Create key pair
* Create a file with the name authorized_keys inside the .ssh directory: sudo touch /.ssh/authorized_keys
* Change the permission for the .ssh folder: sudo chmod 700 .ssh
* Change the permissions for the authorized_keys file: sudo chmod 644 /.ssh/authorized_keys
* Open a new terminal window and create a key pair: ssh-keygen
* Copy key pair to /.ssh/authorized_keys file

### Login as grader user
* sudo service ssh restart
* ssh grader@35.172.119.117 -p 2200 -i ~/.ssh/authorized_keys

## Step 3
### Install Apache2
* sudo apt-get install apache2

### Add mod-wsgi
* sudo apt-get install libapache2-mod-wsgi python-dev

## Step 4
### Clone catalog app to Apache2
* create new directory FlaskApp
* clone catalog app from github as FlaskApp in the FlaskApp directory
* rename project.py to __init__.py

## Change path of all client_secrets.json in __init__.py file
* change path to: /var/www/FlaskApp/FlaskApp/client_secrets.json

## Setup database
* Install PostgreSQL
* Turn remote connections off
* Login as user "postgres": $ sudo su - postgres
* postgreSQL shell: psql
* Create a new database named "catalog" and create a new user named "catalog" in postgreSQL shell
* Set a password for user catalog with the password as 'password'
* Give user "catalog" permission to "catalog" application database: GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog
* Change the path to the database in the __init__.py and the database_setup.py files to : create_engine('postgresql://catalog:password@localhost/catalog')
* Install psycopg2: sudo apt-get -qqy install postgresql python-psycopg2
* Create database schema: sudo python database_setup.py
* Add items to database: sudo python lotsofitems.py

## Step 5
### Launch app
* Create a new virtual host in the /etc/apache2/sites-available/FlaskApp.conf
* `<VirtualHost *:80>  
      ServerName 35.172.119.117
      ServerAdmin emjohn612@gmail.com
      ServerAlias HOSTNAME, e.g. http://ec2-35-172-119-117.compute-1.amazonaws.com  
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
</VirtualHost> `
*
