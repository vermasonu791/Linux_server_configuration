### udacity-linux-server-configuration

### Project Description

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host one of your web applications, to include installing updates, securing it from a number of attack vectors..

### For completing this task I use Amazon Liightsail and SmarTTY

AWS Lightsail:Amazon Lightsail is an Amazon cloud service that offers bundles of cloud compute power and memory for new or less experienced cloud users.

SmarTTY:SmarTTY is a free multi-tabbed SSH client that supports copying files and directories with SCP on-the-fly and editing files in-place.
using smarTTy we can access our cloud virtual machine on windows using public ip and ssh-key 

- IP address: 13.127.168.72

- Accessible SSH port: 2200

- Application URL: http://13.127.168.72/

### Steps to perform this task

1. Create new user named grader and give it the permission to sudo
  - `sudo adduser grader`
  - `sudo usermod -a -G sudo grader`
  - `sudo access sudo nano /etc/sudoers.d/grader`
  - `add following line  grader ALL=(ALL:ALL) ALL`
  -  create key using command `ssh-keygen` on local terminal and save in location /.ssh/linux
 
2. Set-up SSH keys for user grader
  - `mkdir /home/grader/.ssh`
  - `chown grader:grader /home/grader/.ssh`
  - `chmod 700 /home/grader/.ssh`
  - `copy linux.pub key in folder .ssh/authorized_keys`
  - `chown grader:grader /home/grader/.ssh/authorized_keys`
  - `chmod 644 /home/grader/.ssh/authorized_keys`

3. Update all currently installed packages
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

4. Add one more SSH port from 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Confirm by running `ssh -i ~/.ssh/linux -p 2200 root@13.127.168.72`
  
5. Configure the local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC
 
6. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - check the firewall status using `sudo ufw status`.
  - block all incoming connections on all ports using `sudo ufw default deny incoming`.
  - allow outgoing connections on all ports using `sudo ufw default allow outgoing`.
  - allow incoming connection for SSH(port 2200) using `sudo ufw allow 2200/tcp`.
  - allow incoming connection for HTTP(port 80) using `sudo ufw allow 80/tcp`.
  - allow incoming connection for NTP(port 123) using `sudo ufw allow 123/udp`.
  - check the added rules using `sudo ufw show added`.
  - enable the firewall using `sudo ufw enable`.
  - check whether firewall is enable or not using `sudo ufw status`.

7. Disable ssh login for root user
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh -i ~/.ssh/linux -p 2200 grader@13.127.168.72`
 
8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

10.Install Flask and other dependencies
  - `sudo apt-get install python3-psycopg2 python3-flask`
  - `sudo apt-get install python3-sqlalchemy python3-pip`
  - `sudo pip3 install --upgrade pip`
  - `sudo pip3 install oauth2client`
  - `sudo pip3 install requests`
  - `sudo pip3 install httplib2`
  - `sudo pip3 install flask-seasurf`
11. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir FlaskApps`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader FlaskApps`
  - `cd /FlaskApps`
  - Clone your project from github `git clone https://github.com/vermasonu791/catalog.git`
  - Create a FlaskApps.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/FlaskApps/")
  
  from home import app as application
  application.secret_key = 'supersecretkey'
  ```
  - Rename `mv project.py home.py`

  
12. Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - Change create engine line in your `home.py` and `database_setup.py` to: 
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - `python /var/www/catalog/catalog/database_setup.py`
  ```  

13. Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  <VirtualHost *:80>
      ServerName 13.127.168.72
      ServerAdmin vermasonu791@gmail.com
      WSGIScriptAlias / /var/www/FlaskApps/FlaskApps.wsgi
    <Directory /var/www/FlaskApps/catalog/>
	  Order allow,deny
	  Allow from all
    </Directory>
    Alias /static /var/www/FlaskApps/catalog/static
    <Directory /var/www/FlaskApps/catalog/static/>
	    Order allow,deny
	    Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite catalog`


14. Restart Apache 
  - `sudo service apache2 restart`
 
15. Visit site at http://13.127.168.72

16. refrence `http://amunategui.github.io/idea-to-pitch/`