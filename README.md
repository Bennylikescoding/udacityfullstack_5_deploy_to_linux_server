# Linux Server Configuration
- This project is related to the udacity fullstack nanodegree, which is the last project. You will develop an application that provides a list of items within a variety of categories as well as provide a user registration and authentication system. Registered users will have the ability to post, edit and delete their own items.

## Server Details:
- URL: 18.222.182.39
- SSH port: 2200
- User name and password for Udacity reviewer: `grader`, `grader`
- Login with: `ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.222.182.39`

## Laptop on which I deploy my webapplication:
- Macbook (MacOS Sierra 10.12.6)

# Steps to complete this project

## Create Amazon Lightsail instance (remote server)

1. Sign in to Amazon Lightsail

2. Search for Lightsail, or move to (https://lightsail.aws.amazon.com/ls/webapp/home/instances)

3. Create an instance according to (https://lightsail.aws.amazon.com/ls/docs/en/articles/how-to-create-amazon-lightsail-instance-virtual-private-server-vps)

## Configure connection using my local machine:
1. Create ssh keypair according to (https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-set-up-ssh)

2. Choose Download, get the `us-east-backup.pem` keypair (name based on your own choice)

3. Enter into the Downloads folder using `cd Downloads/`

4. Open the terminal on Mac, type in:
`chmod 600 us-east-backup.pem`

5.  now you can login using:
`ssh -i us-east-backup.pem ubuntu@18.222.182.39`

- Now you have successfully logged in to the remote instance using your own computer.

## 1. Create a new user named grader:
First, login to the remote server using `ssh -i us-east-backup.pem ubuntu@18.222.182.39`

1.1. Run `sudo adduser grader`, set both password and account name to "grader"

1.2. Give access to all files for the newly added grader:
`sudo nano /etc/sudoers.d/grader`

In the editor opened, add the following text, then save and exit: 
`grader ALL=(ALL:ALL) ALL`



## 2. Update and Upgrade existing packages
`sudo apt-get update`

`sudo apt-get upgrade`

## 3. Configure the ssh
`sudo nano /etc/ssh/sshd_config, change port 22 to 2200`

`sudo service ssh restart`

- Open the lightsail instance page, go to Account page, move to the "Networking" tab, under the "Firewall" configuration part, choose "Edit rules" at the right bottom, then choose "Custom", "TCP", then type in "2200"  to the port range.

- now you can log in using the new port number:
`ssh -i us-east-backup.pem -p 2200 ubuntu@18.222.182.39`

## For grader:
1. On local machine, open the terminal, type in `ssh-keygen -f ~/.ssh/udacity_key.rsa`, this will create two keypairs under the folder `/Users/wangxiansheng/.ssh/`, one is `udacity_key.rsa`, the other is `udacity_key.rsa.pub`

2. Log into the remote machine as *root* user through ssh:

- Open a new terminal, type in 
`cd Downloads/`

`ssh -i us-east-backup.pem -p 2200 ubuntu@18.222.182.39`

- and create the following file:
`sudo vim /home/grader/.ssh/authorized_keys`

3. Copy the content of the `udacity_key.rsa.pub` file from your local machine to the `/home/grader/.ssh/authorized_keys` file you just created on the remote machine. Then change some permissions on remote machine: (Be very careful that the copy method of nano is different from the vim, if incomplete .pub content is copied, it will result in incomplete authorized_keys on the remote machine, which will take hours to debug.)

`sudo chmod 700 /home/grader/.ssh`

`sudo chmod 644 /home/grader/.ssh/authorized_keys`

4. Finally change the owner from root to grader: 

`sudo chown -R grader:grader /home/grader/.ssh`

*Now you can open a new terminal window on the local machine*:

`ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.222.182.39`

- Note: you can't use `ssh -i ~/.ssh/udacity_key.rsa -p 2200 ubuntu@18.222.182.39` to login anymore because the owner has been changed from root(ubuntu) to grader. However, you can still login to the root user using `cd Downloads/`, and then `ssh -i us-east-backup.pem -p 2200 ubuntu@18.222.182.39`.

5. Force key-based authentication for grader:

- `sudo nano /etc/ssh/sshd_config`, change PasswordAuthentication to `no`

- `sudo service ssh restart`

## 4. Configure the firewall
- login to the remote server, type in:

`sudo ufw allow 2200/tcp`

`sudo ufw allow 80/tcp`

`sudo ufw allow 123/udp`

`sudo ufw enable `

## 5. Configure the local timezone to UTC

Run `sudo dpkg-reconfigure tzdata`, press "none of the above" -> "UTC"

## 6. Install and configure Apache to serve a Python mod_wsgi application

6.1 Install Apache:
`sudo apt-get install apache2`

6.2 Install mod_wsgi:
`sudo apt-get install python-setuptools libapache2-mod-wsgi`

6.3 Restart Apache:
`sudo service apache2 restart`

## 7. Install and configure PostgreSQL

7.1 Install postgreSQL: `sudo apt-get install postgresql`

7.2 Login as user "postgres": `sudo su - postgres`

7.3 Get into postgreSQL shell: `psql`

7.4 Create a new database named catalog and create a new user named catalog in postgreSQL shell:

`CREATE DATABASE catalog;`

`CREATE USER catalog;`

7.5 Set a password for user catalog: (password is catalog)

`ALTER ROLE catalog WITH PASSWORD 'catalog';`

7.6 Give user "catalog" permission to "catalog" application database:
`GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

7.7 Quit postgreSQL:
`\q`

7.8 Exit from user postgres:
`exit`

## 8. Install git, clone and setup my Catalog App project:

8.1 Install Git:
`sudo apt-get install git`

8.2 move to the /var/www:
`cd /var/www`

8.3 Create the application directory；
`sudo mkdir MyApp`

8.4 Move inside the MyApp directory:
`cd MyApp`

8.5 Clone the Catalog App to the virtual machine:

`sudo git clone https://github.com/Bennylikescoding/udacityfullstack_itemcatalog.git`

8.6 Move into the application folder:
`cd udacityfullstack_itemcatalog/`

8.7 Rename project.py to __init__.py :
`sudo mv project.py __init__.py`

8.8 Edit database_setup.py, __init__.py, listsofanalysis.py, change engine = create_engine('sqlite:///analysislists_withusers.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog'):

`sudo nano database_setup.py`

`sudo nano __init__.py`

`sudo nano listsofanalysis.py`

（reference: https://github.com/blurdylan/linux-server-configuration）

8.9 Install pip:
`sudo apt-get install python-pip`

8.10 Use pip to install dependencies:
`sudo pip install -r requirements.txt`

8.11 Install psycopg2:
`sudo apt-get -qqy install postgresql python-psycopg2`

8.12 Create database:
`sudo python database_setup.py`

8.13 Update path of client_secrets.json file
- `sudo nano __init__.py`

- Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json:
move to: open('client_secrets.json', 'r'), change to: open('/var/www/MyApp/udacityfullstack_itemcatalog/client_secrets.json','r')

## 9. Configure and Enable a New Virtual Host
9.1. Create MyApp.conf to edit:
`sudo nano /etc/apache2/sites-available/MyApp.conf`

9.2. Add the following lines of code to the file to configure the virtual host.
```
<VirtualHost *:80>
	ServerName 18.222.182.39
	ServerAdmin ubuntu@18.222.182.39
	WSGIScriptAlias / /var/www/MyApp/MyApp.wsgi
	<Directory /var/www/MyApp/udacityfullstack_itemcatalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/MyApp/udacityfullstack_itemcatalog/static
	<Directory /var/www/MyApp/udacityfullstack_itemcatalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

9.3. Enable the virtual host:

`sudo a2ensite MyApp`

`sudo systemctl reload apache2`

## 10. Create the .wsgi File
10.1. Move to /var/www/MyApp:

`cd /var/www/MyApp`

`sudo nano MyApp.wsgi`

10.2. Add the following content:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/MyApp")

from udacityfullstack_itemcatalog import app as application
application.secret_key = 'Add your secret key here'
```


10.3. Restart apache service:
`sudo service apache2 restart`


10.4 Visit site at 18.222.182.39


- Note: If wrong, type sudo tail /var/log/apache2/error.log to see what happened, and also, disable and restart the apache site with the following steps
1. Run `sudo a2dissite 000-default.conf`
2. Restart the server with `sudo service apache2 reload`
