---
# Linux Server Configuration

---

## Introduction
This project aims to set up a secure Linux server and support [Item Catalog App](https://github.com/MomokoXu/Project-Item-Catalog) running on it.

---
## Server Info
* IP Address: http://52.11.8.252
* URL: http://52.11.8.252
* SSH port number: 2200
---
## Setup Steps
### Step1: Get Your Server
1. Register an account on [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp) and create an instance:
![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_1.png)
![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_2.png)
![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_3.png)
![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_4.png)
![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_5.png)
2. SSH to your server
    * You can connect directly using your browser
    ![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_6.png)
    * You can also SSH from your local machine
        1. Download the private key file from your Lightsail account page
        ![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_7.png)
        2. Move the file into the folder ~./ssh:
        ```$ mv ~Downloads/LightsailDefaultPrivateKey-us-west-2.pem ~/.ssh/```
        3. Change file permission level:
        ```$ chmod 600 ~/.ssh/LightsailDefaultPrivateKey-us-west-2.pem```
        4. SSH into the server:
        ```$ ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-west-2.pem ubuntu@52.11.8.252```
### Step2: Secure Your Server
1. Change the SSH port from 22 to 2200:
    * Firstly configure your firewall to allow incoming tcp connections from port 2200 and udp connections from port 123.
        ![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_8%20.png)
    * Then change the **Port** from 22 to 2200, set **PermitRootLogin no**, and **PasswordAuthentication no** by editing your ssh config file:
    `$ sudo nano /etc/ssh/sshd_config`
    * Restart ssh:
    `$ sudo service ssh restart`
    * Logout and login agian from port 2200:
    `$ ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-west-2.pem ubuntu@52.11.8.252 -p 2200`
2. Update all installed packages
    * `$ sudo apt-get update`
    * `$ sudo apt-get upgrade`
3. Configure the local timezone UTC:
    * `$ sudo dpkg-reconfigure tzdata`
    * first choose **none of the above** and then choose **UTC**
### Step 3: User Management
1. Create user "grader"
    * `$ sudo adduser grader`
2. Give "grader" sudo permission
    * `$ sudo nano /etc/sudoers.d/grader`
    * Paste **grader ALL=(ALL:ALL) ALL** into the file, save and exit.
3. Configure the key-based authentication for grader user
    * **In your local machine**
        * Generate encryption key: `$ ssh-keygen ~/.ssh/graderkey`
        * Get content and copy it: `$ cat ~/.ssh/graderkey.pub`
    * **In your instance server**
        * Make directory fo ryour grader ssh: `$ sudo mkdir /home/grader/.ssh`
        * Paste the content you've copied from your local machine into authorized_keys: `$ touch /home/grader/.ssh/authorized_keys`
        * Change the permission and ownership of the key pair:
        `$ sudo chmod 700 /home/grader/.ssh`
        `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
        `$ sudo chown -R grader:grader /home/grader/.ssh`
    * **Now login the server as user grader** with grader's key in your local machine
    `$ ssh -i ~/.ssh/graderkey grader@52.11.8.252 -p 2200`
### Step 4: Configure the Uncomplicated Firewall
1. `$ sudo ufw allow 2200/tcp`
2. `$ sudo ufw allow 80/tcp`
3. `$ sudo ufw allow 123/udp`
4. `$ sudo ufw enable`

### Step 5: Prepare to deploy Item Catalog Project
1. Install Apache and mod_wsgi
    * `$ sudo apt-get install apache2`
    * `$ sudo apt-get install libapache2-mod-wsgi`
    * `$ sudo service apache2 restart`
2. Install Python and Git
    * `$ sudo apt-get install git`
    * `$ sudo apt-get install python`
3. Clone Item Catalog App from Github
    * `$ sudo mkdir /var/www/itemcatalog`
    * Change owner for the itemcatalog folder `sudo chown -R grader:grader itemcatalog`
    * Move into itemcatalog directory and clone the app repository:
        * `$ cd /var/www/itemcatalog`
        * `$ git clone https://github.com/MomokoXu/Project-Item-Catalog`
4. Install and configure the virtual environment
    * Install pip: `$ sudo apt-get install python-pip`
    * Install virtualenv: `$ sudo pip install virtualenv`
    * Rename virtualenv: `$ sudo virtualenv venv`
    * Change premission: `$ sudo chmod -R 777 venv`
    * Activate virtual environment: `$ source venv/bin/activate`
    * Install all modules for this project, for example Flask, `$ pip install Flask`
    * Restart apache: `$ sudo service apache2 restart`
5. Install PostgreSQL and setup database
    * Install PostgreSQL: `sudo apt-get install postrgresql postgresql-contrib`
    * Change user to postgres and start psql:
        * `$ sudo su - postgres`
        * `$ psql`
    * Create user itemcatalog with passwrod: `postgres=# CREATE USER itemcatalog WITH PASSWORD 'anypassword';`
    * Allow user itemcatalog to create database, created database and connect to database:
        * `postgres=# ALTER USER itemcatalog CREATEDB;`
        * `postgres=# CREATE DATABASE itemcatalog WITH USER itemcatalog;`
        * `postgres=# \c itemcatalog`
    * Set credentials for user itemcatalog:
        * `$ REVOKE ALL ON SCHEMA public FROM public;`
        * `$ GRANT ALL ON SCHEMA public TO itemcatalog;`
    * Quit psql and exit:
        * `\q`
        * `exit`
    * Change data_setup.py and project.py to connect current database:
        **Change** `engine = create_engine('sqlite:///item_catalog.db')`
        **to** `engine = create_engine('postgresql://itemcatalog:anypassword@localhost/itemcatalog')`
    * Setup database: `$ python database_setup.py`
6. Configure and Enable a new virtual host
    * Change directory: `$ cd /var/www/itemcatalog`
    * Create a itemcatalog.wsgi file to serve the application
    `$ sudo nano itemcatalog.wsgi`
    * Paste the following into the wsgi file
        ```
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0, "/var/www/itemcatalog/Project-Item-Catalog/project/catalog/")
        from project import app as application
        ```
    * Configure the virtual host to our domain
    `$ sudo nano /etc/apache2/sites-available/itemcatalog.conf`
    ```
        <VirtualHost *:80>
            ServerName 52.11.8.252
            ServerAlias ec2-52-11-8-252.us-west-2.compute.amazonaws.com
            ServerAdmin admin@52.11.8.252
            WSGIDaemonProcess itemcatalog python-path=/var/www/itemcatalog:/var/www/itemcatalog/venv/lib/python2.7/site-packages
            WSGIProcessGroup itemcatalog
            WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi
            <Directory /var/www/itemcatalog/Project-Item-Catalog/project/catalog/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/itemcatalog/Project-Item-Catalog/project/catalog/static
            <Directory /var/www/itemcatalog/Project-Item-Catalog/project/catalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
    ```
    * Enable this new virtual host: `$ sudo a2ensite itemcatalog`
7. Fix the Oauth for Google and Facebook Login
    * First change the oauth credentials in your app console by adding `ec2-52-11-8-252.us-west-2.compute.amazonaws.com` into **Authorized Javascript origins** and `ec2-52-11-8-252.us-west-2.compute.amazonaws.com/oauth2callback` into **Authorized Redirect URL**
    * Second, change the site url for the app in FB developer console: `http://52.11.8.252`
    * Third, update `fb_client secrets.json` and `client_secrets.json` file for the new app information
    * Finally, update all paths in project.py that use `fb_client secrets.json` and `client_secrets.json`, for example,
    `CLIENT_ID = json.loads(
    open('/var/www/itemcatalog/Project-Item-Catalog/project/catalog/client_secrets.json', 'r').read())['web']['client_id']`
8. Restart Apache server
    * `$ sudo service apache2 restart`
### Final Step: Open Link: http://52.11.8.252
![image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/app_sample.png)

---

## Author
[Yingtao Xu](https://github.com/MomokoXu)

---
## References and Resources
* [SSH Essentials: Working with SSH Servers, Clients, and Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)
* [README.md from sturken](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)
* [README.md from sharkwhistle ](https://github.com/sharkwhistle/Udacity-FSND-Linux-Server-Configuration-)
* [askubuntu](https://askubuntu.com/questions)
* [Stackoverflow](https://stackoverflow.com/)
* [Udacity Forum](https://discussions.udacity.com/)
## Copyright
This is a project for practicing skills in databses and backend courses not for any business use. Some templates and file description are used from [Udacity FSND program](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). Please contact me if you think it violates your rights.