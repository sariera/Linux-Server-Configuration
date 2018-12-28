# Linux Server Configuration:

Installation of a Linux server and prepare it to host web applications. securing server from a number of attack vectors, install and configure a database server, and deploy a catalog web application onto it.

* Application URL : `http://68.183.69.27/catalog`
* IP address: `http://68.183.69.27/`
* SSH port- `2200`

* Login with: `ssh -i ~/.ssh/project -p 2200 grader@68.183.69.27`


## Configuration Steps:
### Step 1 : Create new user named grader and give it the permission to sudo:
* Run `sudo adduser grader` to create a new user named grader
* `sudo visudo` (edit the sudoers file . it is save to use sudo visudo to edit the sudoers file otherwise file will not be saved)
* add the below line of code after root ALL=(ALL:ALL) ALL
	`grader ALL=(ALL:ALL) ALL` and save it (ctrl-X , then Y and Enter)
* Your new user(grader) is able to execute commands with administrative privileges. ( for example - sudo anycommand)
* You can check the grader entry by below command:
	`sudo cat /etc/sudoers`

**Resources -** [Add User and give permissions](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

### Step-2 : Update all currently installed packages
* `sudo apt-get update` - command will  update list of packages and their versions on your machine.
* `sudo apt-get upgrade` - command will install the packages

**Resources -** [update and upgrade the packages](https://wiki.ubuntu.com/Security/Upgrades)

### Step-3 : Change the SSH port from 22 to 2200
* Run `nano /etc/ssh/sshd_config`
    * change port from `22` to `2200`
    * add `AllowUsers grader` at end of the file so that we will login through grader.
* restart the SSH service: `sudo service ssh restart`

### Step-4 : SSH- Keys
* generate key-pair with `ssh-keygen`
* Save keygen file into (/home/user/.ssh/project) and fill the password, 2 keys will be generated,  public key (project.pub) and identification key(project).
* Login into grader account using `ssh -v grader@68.183.69.27 -p 2200`,  type the password that you have fill during user creation.
* make a directory in grader account : `mkdir .ssh`
* make a authorized_keys file using `touch .ssh/authorized_keys`
* from your local machine,copy the contents of public key(project.pub).
* paste that contents on authorized_keys of grader account using `nano authorized_keys` and save it .
* give the permissions : `chmod 700 .ssh`    and `chmod 644 .ssh/authorized_keys`.
*  do `nano /etc/ssh/sshd_config` , change `PasswordAuthentication` to  no .
* Restart ssh service: `sudo service ssh restart`.
*  `ssh grader@68.183.69.27 -p 2200 -i ~/.ssh/project` in new terminal .A pop-up window will open for authentication. just fill the password that you have fill during ssh-keygen creation.

**Resources -** [initial server setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04), [udacity course videos](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4331066009/concepts/48010894750923#)

### Step-5:Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
* check the firewall status using `sudo ufw status`.
* block all incoming connections on all ports using `sudo ufw default deny incoming`.
* allow outgoing connections on all ports using `sudo ufw default allow outgoing`.
* allow incoming connection for SSH(port 2200) using `sudo ufw allow 2200/tcp`.
* allow incoming connection for HTTP(port 80) using `sudo ufw allow 80/tcp`.
* allow incoming connection for NTP(port 123) using `sudo ufw allow 123/udp`.
* check the added rules using `sudo ufw show added`.
* enable the firewall using `sudo ufw enable`.
* check whether firewall is enable or not using `sudo ufw status`.

**Resources -** [UFW](https://help.ubuntu.com/community/UFW)

### Step-6: Configure the local timezone to UTC
* configure timezone using `sudo dpkg-reconfigure tzdata` ( select none of the above and then set timezone to UTC)

**Resources -** [timezone to UTC](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)

### Step 7: Install and configure Apache to serve a Python mod_wsgi application:
* install apache using `sudo apt-get install apache2`
* Install mod_wsgi using `sudo apt-get install libapache2-mod-wsgi`
* You then need to configure Apache to handle requests using the WSGI module. You’ll do this by editing the `/etc/apache2/sites-enabled/000-default.conf` file. This file tells Apache how to respond to requests, where to find the files for a particular site and much more.
* add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: 
`WSGIScriptAlias / /var/www/html/myapp.wsgi`
* restart Apache with the `sudo service apache2 restart` command.

### Step 8: Install Git
*  Install git using `sudo apt-get install git`
* set up git using :
```
        git config --global user.name "username"
        git config --global user.email "email@domain.com"
```
* check the configurations items using `git config --list`

**Resources -** [install git](https://help.github.com/articles/set-up-git/) , [install git on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)

### Step 9: Deploy Flask Application:
* `sudo apt-get install python-dev`
* To enable mod_wsgi, run `sudo a2enmod wsgi`.
* move to the `/var/www` directory:
* Create the application directory structure using mkdir
    `sudo mkdir catalog`
* Move inside this directory : `cd catalog`
* Create another directory : `sudo mkdir catalog`
* move inside this directory and create two subdirectories named static and templates:
    `cd catalog`
    `sudo mkdir static templates`
* Now , we will create a virtual environment for our flask application.  use pip to install virtualenv and Flask. Install pip :
    `sudo apt-get install python-pip`
* Install virtualenv:   `sudo pip install virtualenv`
* Set enviornment name using : `sudo virtualenv venv`
* Install Flask in that environment by activating the virtual environment using : `source venv/bin/activate`
* Install Flask using : `sudo pip install Flask`
* Run the following command to test if the installation is successful and the app is running:  `sudo python __init__.py`
* It should display "Running on `http://127.0.0.1:5000/"`. If you see this message, you have successfully configured the app.
* To deactivate the environment : `deactivate`
* Create the wsgi file using:  `cd /var/www/catalog`
* `sudo nano catalog.wsgi`  and add the code :
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```
* Restart Apache :  `sudo service apache2 restart`

**Resources -** [Install flask, Virtual Env](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

### Step-10 : Clone the Github Repository
* `sudo mv Item-Catalog_ND-Project /var/www/catalog/catalog/`
* move the Item-Catalog_ND-Project directory to `/var/www/catalog/catalog`.

### Step-11: Install required Packages
* `sudo apt-get install python-pip`
* `source venv/bin/activate`
* `sudo pip install httplib2`
* `sudo pip install requests`
* `sudo pip install --upgrade oauth2client`
* `sudo pip install sqlalchemy`
* `sudo pip install Flask-SQLAlchemy`
* `sudo pip install flask-seasurf`

### Step 12:  Install Postgresql
*  Install the Python PostgreSQL adapter psycopg:
        `sudo apt-get install python-psycopg2`
* Install PostgreSQL:
    `sudo apt-get install postgresql postgresql-contrib`
* open database_setup.py using :
    `sudo nano database_setup.py`
* update the create_engine line:
	`python engine = create_engine('postgresql://catalog:catalog-pw@localhost/catalog')`
* Update the create_engine line in project.py and lotsofitems.py too.
* move the project.py file to __init__.py file :
    mv application.py __init__.py
* Change to default user postgres:
    `sudo su - postgre`
* Connect to the system:
    `psql`
* Create user catalog:
    `CREATE USER catalog WITH PASSWORD 'catalog-pw';`
* check lists of roles using` \du`
* Allow the user to create database :
    `ALTER USER catalog CREATEDB;`
  and check the roles and attributes using \du.
* Create database using :
   ` CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to database using :
       ` \c catalog`
* Revoke all the rights :
    `REVOKE ALL ON SCHEMA public FROM public;`
* Grant the access to catalog:
    `GRANT ALL ON SCHEMA public TO catalog;`
* exit from Postgresql : `\q `then `exit `from postgresql user.
* restart postgresql: `sudo service postgresql restart`

**Resources -** [Install postgresql](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps) , [engine configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql)

### Step 13: Run the application:
* Create the datbase schema:
    `python database_setup.py`
    `python lotsofmenus.py`
* Restart Apache : `sudo service apache2 restart`
* in /var/www/catalog/catalog directory : execute -  `python __init__.py`
* type  public IPaddress (`http://35.165.147.241/`) on URL and you will see Catalog Application Webpage.

* related to client_secrets.json and fb_client_secrets.json files. You need to give absolute path to these files .
change the ```CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']``` to
    ```CLIENT_ID = json.loads(open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']```
    Similarly for `fb_client_secrets.json` file.

**Resources -** [Udacity Discussion Forum](https://discussions.udacity.com/t/client-secret-json-not-found-error/34070) , [forum post]( https://discussions.udacity.com/t/how-to-login-to-my-aws-virtual-server-as-new-user-grader/201164/4).