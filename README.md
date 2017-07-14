---
# Linux Server Configuration

---

## Introduction
This project aims to set up a secure Linux server aro support [Item Catalog App](https://github.com/MomokoXu/Project-Item-Catalog) running on it.

---
## Server Info
* IP Address: 52.11.8.252
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
        ![Image of webpage](https://github.com/MomokoXu/Project-Linux-Server-Configuration/blob/master/images/lightsail_8.png)
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
    * Create a itemcatalog.wsgi file to serve the application
    `$ sudo nano itemcatalog.wsgi`
    Paste the following into the wsgi file
        ```
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0, "/var/www/itemcatalog/Project-Item-Catalog/project/catalog/")
        from project import app as application
        ```
    * Move into itemcatalog directory and clone the app repository:
    `$ cd /var/www/itemcatalog`
    `$ git clone https://github.com/MomokoXu/Project-Item-Catalog`
    *
4. Install the virtual environment
---

## Author
[Yingtao Xu](https://github.com/MomokoXu)

---
## Copyright
This is a project for practicing skills in databses and backend courses not for any business use. Some templates and file description are used from [Udacity FSND program](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). Please contact me if you think it violates your rights.
