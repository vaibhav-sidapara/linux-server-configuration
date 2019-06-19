## Project: Item Catalog

#### APP URL: http://52.221.231.64.xip.io/ OR http://52.221.231.64/ (OAuth won't work).

#### Description
This is the fifth project in Udacity's Full Stack Web Developer Nanodegree Program.

This is the final project for "Full Stack Web Developer Nanodegree" on Udacity. In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

Deployed URL: http://52.221.231.64.xip.io/

#### SSH Key
Use the attached udacityAWS to ssh into the hosting machine.

You have to use port 2200 for ssh, as the default ssh port 22 is denied using ufw. Also make sure you give correct permission to the key file before doing ssh.
Such as ``` chmod 400 udacityAWS ```.

You have to connect via the user **grader** and the public ip is **52.221.231.64**. The ssh command is as follows.
```
ssh -i udacityAWS grader@52.221.231.64 -p 2200
```

---
#### How Did I Do The Setup
##### Referrals
1. [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
2. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

##### Things Needed
1. Cloud Hosting - [DigitalOcean](https://www.digitalocean.com/), [AWS LightSails](https://aws.amazon.com/lightsail/)

##### System Setup
----
1. Download Private Key from the __SSH keys__ section in the __Account__ section on Amazon Lightsail.
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory).
	```mv ~/Downloads/Lightsail-key.pem ~/.ssh/```
3. Open your terminal and type in
	```chmod 400 ~/.ssh/Lightsail-key.pem```
4. In your terminal, type in
	```ssh -i ~/.ssh/Lightsail-key.pem ubunut@YOUR-SERVER-PUBLIC-IP```

###### Adding New User ```grader```

1. ```sudo adduser grader```
2. ```sudo touch /etc/sudoers.d/grader```
2. ```sudo nano /etc/sudoers.d/grader```, add the following text ```grader ALL=(ALL:ALL) NOPASSWD:ALL``` and save it.

###### Setting SSH Key

1. Generate keys on local machine using ```ssh-keygen```
2. Deploy public key on hosting server:
  * ```su - grader```
	* ```mkdir .ssh```
	* ```touch .ssh/authorized_keys```
	* ```nano .ssh/authorized_keys```
  * Copy the content of the key generate on the local machine ```cat YOUR_KEY.pub``` and paste into the the above file we created
  * ```chmod 700 .ssh```
	* ```chmod 644 .ssh/authorized_keys```
3. Reload SSH using ```service ssh restart```
4. Use the new key to ssh ```ssh -i privateKeyFilename] grader@YOUR-SERVER-PUBLIC-IP```

##### Disable Root Login
```sudo nano /etc/ssh/sshd_config``` and modify ```PermitRootLogin``` line to  ```PermitRootLogin no```

###### Update Your APT-GET
```
sudo apt-get update
sudo apt-get upgrade
# if there are some packages left back, execute the following command
sudo apt-get --with-new-pkgs upgrade
```

###### Change the SSH port from 22 to 2200

1. Use ```sudo nano /etc/ssh/sshd_config``` and then change Port 22 to Port 2200.
2. Reload SSH using ```sudo service ssh restart```

__Note:__ Remember to add and save port 2200 with _Application __as__ Custom and Protocol __as__ TCP_ in the Networking section of your instance on Amazon Lightsail. 

###### Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
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
 
###### Configure the local timezone to UTC

1. Configure the time zone ```sudo dpkg-reconfigure tzdata```

###### Installing Apache

1. Install Apache ```sudo apt-get install apache2```
2. Install mod_wsgi ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
3. Restart Apache ```sudo service apache2 restart```

###### Installing PostgreSQL

1. Install PostgreSQL ```sudo apt-get install postgresql```
2. Login as user "postgres" ```sudo su - postgres```
3. Get into postgreSQL shell ```psql```
4. Create a new database named catalog and create a new user named catalog in postgreSQL shell
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


###### Installing GIT & Clone
1. Install Git using ```sudo apt-get install git```
2. Use ```cd /var/www``` to move to the /var/www directory 
3. Create the application directory ```sudo mkdir YOUR_APP```
4. Move inside this directory using ```cd YOUR_APP```
	* ```sudo pip install sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests```
	* ```sudo pip install flask packaging oauth2client redis passlib flask-httpauth```
  * If you get an error about "unsupported locale setting"
```
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
sudo dpkg-reconfigure locales
```
13. Install psycopg2 ```sudo apt-get -qqy install postgresql python-psycopg2```
14. Create database schema ```sudo python database_setup.py```


###### Configure and Enable a New Virtual Host

1. Create YOUR_APP.conf to edit: ```sudo nano /etc/apache2/sites-available/YOUR_APP.conf```
2. Add the following lines of code to the file to configure the virtual host. 
```
<VirtualHost *:80>
    ServerName SERVER_IP
    ServerAdmin ADMIN_EMAIL
    WSGIScriptAlias / /var/www/YOUR_APP/YOUR_APP.wsgi
    <Directory /var/www/YOUR_APP/YOUR_APP/>
    Order allow,deny
    Allow from all
    </Directory>
    Alias /static /var/www/YOUR_APP/YOUR_APP/static
    <Directory /var/www/YOUR_APP/YOUR_APP/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable the virtual host with the following command: ```sudo a2ensite YOUR_APP.conf```

###### Create the .wsgi File
1. Create the .wsgi File under /var/www/YOUR_APP: 
```
cd /var/www/YOUR_APP
sudo nano YOUR_APP.wsgi 
```
2. Add the following lines of code to the flaskapp.wsgi file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/YOUR_APP/")

from YOUR_APP import app as application
application.secret_key = 'super_secret_key'
```

###### Update Code
* Rename your main application file to ```__init__.py``` (For example: application.py to ```__init__.py```)
* Change the DB code, replace the code with your existing code for DB connection ```postgresql://catalog:password@localhost/catalog'```

###### Restart Apache
Restart Apache ```sudo service apache2 restart```

##### It's running; let's use it!

Once your instance has started up, you can access it via a browser via the public ip of your instance

For example my public ip is http://52.221.231.64

When you set up OAuth for your application, you will need a DNS name that refers to your instance's IP address. You can use the xip.io service to get one; this is a public service offered for free by Basecamp. For instance, the DNS name YOUR_PUBLIC_IP.xip.io refers to the server above.

So my URL will http://52.221.231.64.xip.io/. You just need to add the xip.io URL part in the end incase if you don't want to understand xip.io's magic domain name that provides wildcard DNS for any IP address.
