# Linux Server Configuration
Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers. 


## Server Details:

- **URL:**  [http://35.158.220.139.xip.io/](http://35.158.220.139.xip.io/) 
- **IP Address :** 35.158.220.139 
- **SSH port:** 2200 
- **Username:** grader 
- [keys](https://drive.google.com/drive/folders/1k5ky7WsJ8Nie3oYAgkV8dGW09ntB9vSJ?usp=sharing)

## Configuration

#### Start a new Ubuntu Linux server instance on Amazon Lightsail 

1. Login into Amazon Lightsail using an Amazon Web Services account (aws)
2. Click on Create instance.
		choose Instance location , Select a platform Linux/Unix , Select a blueprint OS Only Ubuntu 16.04 LTS, Choose your instance plan , can rename your instance from Identify your instance or can keep the default name
3. Click the Create button to create the instance.

#### Connection to the instance from a local machine

1. Click your instance select Connect from menu , below the page click on Account page , from SSH key pairs download the Default Private Key
2. Move this private key file in local folder ```~/.ssh``` and rename it to lightsail_key.rsa
3. open terminal  
	```chmod 600 ~/.ssh/lightsail_key.rsa```
4. To connect to the instance via the terminal
	```ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.158.220.139 ```

#### Update and upgrade installed packages

- ``` sudo apt-get update```
- ```sudo apt-get upgrade```

#### Change the SSH port from 22 to 2200
- ```sudo nano /etc/ssh/sshd_config``` 
 change the port number on line 5 from 22 to 2200 
- *Restart SSH* ```sudo service ssh restart```

#### Configuring the UFW (Uncomplicated Firewall)

1. ``` sudo ufw default deny incoming``` <br />
```sudo ufw default allow outgoing```<br />
```sudo ufw allow 80/tcp ```<br />
```sudo ufw allow 2200/tcp ```<br />
```sudo ufw allow 123/udp ```<br />
```sudo ufw deny 22```<br />
```sudo ufw enable```<br />
```sudo ufw status ```<br />
2. go to Amazon Lightsail Instance Click your instance select Networking from menu click Edit rules 
Allow ports 80(TCP), 123(UDP), and 2200(TCP), and remove the  port 22 .

3. open Terminal 
```ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@35.158.220.139 ```

#### Create a new user grader
- ```sudo adduser grader``` *password:* [ 12345 ]
- run ```su - grader ``` to switch to the grader user
- exit 

#### Give grader the permission to sudo
1. ```sudo visudo``` add  grader  ALL=(ALL:ALL) ALL after root  ALL=(ALL:ALL) ALL.
2. To verify that grader has sudo permissions ```su - grader```
3. ```sudo -l```

#### Allow grader to log in to the virtual machine
*On the local machine:*
1.  ```cd ~/.ssh```<br />
2. ``` ssh-keygen```<br />
Enter file in which to save the key:  grader_key 
Two files will be generated: grader_key and grader_key.pub

3. ```sudo nano grader_key.pub``` copy the contents of the file. 
4. Login to the grader's virtual machine ```su - grader ```
5. Create a new directory called ~/.ssh  ```mkdir .ssh```
6. ```sudo nano ~/.ssh/authorized_keys ``` then paste the content grader_key.pub into this file, and save it
7. Give the permissions: 
```chmod 700 .ssh ```
``` chmod 644 .ssh/authorized_keys```
8. ```sudo nano /etc/ssh/sshd_config```  PasswordAuthentication is set to no
9. Restart SSH ```sudo service ssh restart```
10. On the local machine, run: ```ssh -i ~/.ssh/grader_key -p 2200 grader@35.158.220.139 ```

#### Configure the local timezone to UTC
login as grader: 
```sudo dpkg-reconfigure tzdata```<br />
select none of the above then select UTC

#### Install Apache
loginas grader: <br />
 ```sudo apt-get install apache2```<br />
Enter IP address in  browser ( 35.158.220.139 ) Apache is working

#### Install mod_wsgi
```sudo apt-get install libapache2-mod-wsgi python-dev```<br />
```sudo a2enmod wsgi```

#### Install and configure PostgreSQL
login as grader: <br />

- ```sudo apt-get install postgresql```
- Switch to the postgres user:  ```sudo su - postgres```
- Connect to psql ```psql```
- ```CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';```
- ```ALTER ROLE catalog CREATEDB;```
- ```\du``` to list existing roles
- ``` \q``` to exit 
- ```exit``` to back grader

#### Create a new catalog
- ```sudo adduser catalog``` *password:* [ 12345 ]
- run ```su - catalog ``` to switch to the catalog user
- exit 

#### Give catalog the permission to sudo
1. ```sudo visudo```
2. Add  catalog  ALL=(ALL:ALL) ALL after root  ALL=(ALL:ALL) ALL and grader ALL=(ALL:ALL) ALL 
3. To verify that catalog has sudo permissions ```su - catalog```
4,```sudo -l```

#### create a database
login as catalog: 
- ```createdb catalog```
- ```psql```
- ```\l``` to see the new database has been created
- ```\q``` to exit 
- ```exit``` to back grader user

#### Install git
login as grader:
 - ```sudo apt-get install git```

#### clone the itemCatalog project
login as grader:
1. ```cd  /var/www/ ```
2. Create a directory called itemCatalog ```mkdir itemCatalog ```
3. ```sudo git clone https://github.com/reemas93/ItemCatalog.git itemCatalog```
4. ```cd /var/www/ ```
5. ```sudo chown -R grader:grader itemCatalog/```
6. ```cd /var/www/itemCatalog/itemCatalog```
7. Rename the application.py file to __init__.py 
``` mv application.py __init__.py```
8. ```sudo nano __init__.py```
9. replace 
```app.run(host="0.0.0.0", port=8000, debug=True)``` to 
```app.run()```
10. in __init__.py , database_setup.py and items.py replace 
```engine = create_engine("sqlite:///catalog.db")``` to 
```engine = create_engine('postgresql://catalog:catalog@localhost/catalog')```

#### Authenticate login through Google
1. go to [Google Cloud Plateform ](https://console.cloud.google.com/home/dashboard?project=apt-weft-226307) click APIs & services then click Credentials
2. go to OAuth Client ID <br>

Authorized JavaScript origins:
- add http://35.158.220.139
- add http://35.158.220.139.xip.io <br>

Authorized redirect URIs: 
- http://ec2-35-158-220-139.eu-central-1.compute.amazonaws.com
- http://35.158.220.139.xip.io/login
- http://35.158.220.139.xip.io/gconnect
3. Download JSON file, copy the contents to client_secrets.json

#### Install the virtual environment

1. virtualenv ```sudo apt-get install python-virtualenv ```
2. ```cd /var/www/itemCatalog/itemCatalog``` 
3. ```virtualenv venv```
4. to activate the environment ``` . venv/bin/activate```

#### Install the required libraries
With the virtual environment active, install the following: 

```sudo apt-get install python-pip``` <br>
```pip install psycopg2``` <br>
```pip install flask``` <br>
```pip install flask-seasurf``` <br>
```pip install sqlalchemy``` <br>
```pip install httplib2``` <br>
```pip install requests```<br>
```pip install --upgrade oauth2client```<br>
```sudo apt-get install libpq-dev``` <br>

- run ```python __init__.py```
- ```deactivate```  to deactivate the virtual env

#### Setup apache service
1. Create a configuration file itemCatalog.conf ```sudo nano /etc/apache2/sites-available/itemCatalog.conf```
```
<VirtualHost *:80>
          ServerName 35.158.220.139
          ServerAdmin youremail@domain.com
          ServerAlias ec2-35-158-220-139.eu-central-1.compute.amazonaws.com
          WSGIScriptAlias / /var/www/itemCatalog/itemCatalog.wsgi
          <Directory /var/www/itemCatalog/itemCatalog/>
                Order allow,deny
                Allow from all
                Options -Indexes
          </Directory>
            Alias /static /var/www/itemCatalog/itemCatalog/static
          <Directory /var/www/itemCatalog/itemCatalog/static/>
                Order allow,deny
                Allow from all
                Options -Indexes
          </Directory>
          ErrorLog ${APACHE_LOG_DIR}/error.log
          LogLevel warn
          CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost> 
```
2. ```sudo a2ensite itemCatalog```
3. ```sudo service apache2 reload```

#### Create a wsgi file

1. ```cd /var/www/itemCatalog/```
2. sudo nano itemCatalog.wsgi
```
activate_this = '/var/www/itemCatalog/itemCatalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, '/var/www/itemCatalog')

  from Itemcatalog import app as application
  application.secret_key = 'Google_Client_ID'
  ```
3. ```sudo service apache2 restart```

#### Disable the apache site
1. ```sudo a2dissite 000-default.conf```
2. ```sudo service apache2 reload```

#### Setup the schema and populate the DB
1. ```cd /var/www/itemCatalog/itemCatalog```
2. ``` . venv/bin/activate```
3. ```python items.py ```to populate the database
4. ```sudo service apache2 restart```
5. Open the browser through the public ip address  35.158.220.139 

#### Restart Apache
```sudo service apache2 restart```

#### log messages from Apache server
```sudo tail /var/log/apache2/error.log``` 

#### Sources
- https://www.udacity.com/course/configuring-linux-web-servers--ud299
- http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
- https://github.com/blurdylan/linux-server-configuration
