# Linux Server Configuration Project
This project is part of Udacity's Full Stack Web Developer Nanodegree Program.
The objective was to take a baseline installation of a Linux distribution on a virtual machine and complete the following:

1. Install package updates and secure it from a number of attack vectors
2. Install and configure web and database servers
3. Host the [Item Catalog Web Application](https://github.com/ncapps/ud-item-catalog) built in a previous project

[Amazon Lightsail](https://lightsail.aws.amazon.com/) was used to host the Linux server instance.



## Getting Started
- Static IP Address : 35.164.196.169
- Domain Name : http://ec2-35-164-196-169.us-west-2.compute.amazonaws.com
- Connect to server with SSH ```ssh -i LightsailDefaultPrivateKey-us-west-2.pem ubuntu@35.164.196.169 -p 2200```
- Update installed packages
```
sudo apt-get update
sudo apt-get upgrade
```

## Setup security
- Change SSH port to 2200 ```sudo nano /etc/ssh/sshd_config```
- Restart the SSH service ```sudo service ssh restart```
- Add firewall rule to Lightsail instance: App = Custom, Protocol = TCP, Port Range = 2200
- Configure and enable Uncomplicated Firewall (UFW) to all connections on the following ports
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
sudo ufw enable
```

## Configure a new user
- Create user **grader** ```sudo adduser grader```
- Give **grader** permission to **sudo**
```
sudo nano /etc/sudoers.d/grader
```
- Add ```grader ALL=(ALL) NOPASSWD:ALL``` to file
- On my local machine, generate a public key ```ssh-keygen``` and name it **grader**
- Copy key to server tmp directory ```scp -P 2200 -i LightsailDefaultPrivateKey-us-west-2.pem grader.pub ubuntu@35.164.196.169:/tmp/```
- Install public key for **grader** user
```
su grader
cd ~
mkdir .ssh
cat /tmp/grader.pub > .ssh/authorized_keys
chmod 644 .ssh/*
chmod 700 .ssh/
```
Confirm ssh connectivty with **grader** user ```ssh -i grader grader@35.164.196.169 -p 2200```


## Install/configure web and database servers
- Install Apache
```
sudo apt-get update
sudo apt-get install apache2
```
- Install and Enable mod_wsgi (Web Server Gateway Interface)
```
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
```
- Install PostgreSQL ```sudo apt-get install postgresql postgresql-contrib```
- Create **catalog** user with no database creation rights, no add user rights, and prompt for a password
```sudo -u postgres createuser -D -A -P catalog
```
- Create **movies** database with **catalog** as the owner ```sudo -u postgres createdb -O catalog movies```
- Restart PostgreSQL server ```sudo /etc/init.d/postgresql restart```

## Deploy application
- Install git ```sudo apt-get install git```
- Clone item catalog repo ```git clone https://github.com/ncapps/ud-item-catalog.git```
- Install Python libraries
```
sudo apt-get install python-pip
sudo pip install virtualenv
sudo pip install sqlalchemy
sudo pip install psycopg2
sudo pip install httplib2
sudo pip install requests
sudo pip install oauth2client
sudo pip install jsonify
```
- Update application.py to work with PostgreSQL rather than sqlite
- Follow instructions on How to Deploy a Flask Application on Ubuntu below
- Restart Apache server ```sudo service apache2 restart```

## License
The contents of this repository are covered under the [MIT License](https://opensource.org/licenses/MIT).

## References
- [Add new sudo user to AWS Ubuntu instance](http://brianflove.com/2013/06/18/add-new-sudo-user-to-ec2-ubuntu/)
-[How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
-[Ubuntu PostgreSQL Server Setup](https://help.ubuntu.com/community/PostgreSQL#Basic_Server_Setup)
