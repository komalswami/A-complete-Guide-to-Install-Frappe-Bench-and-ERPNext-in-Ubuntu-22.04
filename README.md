# Installing ERPNext Frappe on Ubuntu 22.04

## Software Requirements
- Updated Ubuntu 22.04
- A user with sudo privileges

## Hardware Requirements
- 4GB RAM
- 40GB Hard Disk

## Pre-requisites
  - Python 3.10
  - Node.js 14+
  - Redis 5                                       (caching and real time updates)
  - MariaDB 10.3.x / Postgres 9.5.x               (to run database driven apps)
  - yarn 1.12+                                    (js dependency manager)
  - pip 20+                                       (py dependency manager)
  - wkhtmltopdf (version 0.12.5 with patched qt)  (for pdf generation)
  - NGINX                                         (proxying multitenant sites in production)

### Install git
```
sudo apt update
sudo apt -y upgrade
sudo apt-get install git
```

### Install Python Tools & wkhtmltopdf
```
sudo apt-get install python3-dev
sudo apt-get install python3-setuptools python3-pip
sudo apt-get install xvfb libfontconfig wkhtmltopdf
sudo apt-get install libmysqlclient-dev
```
###  Install virtualenv
```
sudo apt-get install virtualenv
sudo apt install python3.10-venv
```

### Install Curl, Redis and Node.js
```
sudo apt install curl 
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile
nvm install 14.15.0  
```

###  Install Yarn
```
sudo apt-get install npm
sudo npm install -g yarn
```

### Install Redis
```
sudo apt-get install redis-server
```


### Install MariaDB
```
sudo apt-get install software-properties-common
sudo apt install mariadb-server
sudo mysql_secure_installation
```

When you run this command, the server will show the following prompts. Please follow the steps as shown below to complete the setup correctly.

- Enter current password for root: (Enter your SSH root user password)
- Switch to unix_socket authentication [Y/n]: Y
- Change the root password? [Y/n]: Y
- It will ask you to set new MySQL root password at this step. This can be different from the SSH root user password.
- Remove anonymous users? [Y/n] Y
- Disallow root login remotely? [Y/n]: N
- This is set as N because we might want to access the database from a remote server for using business analytics software like Metabase / PowerBI / Tableau, etc.
- Remove test database and access to it? [Y/n]: Y
- Reload privilege tables now? [Y/n]: Y


### MySQL database development files
```
sudo apt-get install libmysqlclient-dev
```

###  Edit the mariadb configuration ( unicode character encoding )
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
add this to the 50-server.cnf file
```
 [server]
 user = mysql
 pid-file = /run/mysqld/mysqld.pid
 socket = /run/mysqld/mysqld.sock
 basedir = /usr
 datadir = /var/lib/mysql
 tmpdir = /tmp
 lc-messages-dir = /usr/share/mysql
 bind-address = 127.0.0.1
 query_cache_size = 16M
 log_error = /var/log/mysql/error.log

 [mysqld]
 innodb-file-format=barracuda
 innodb-file-per-table=1
 innodb-large-prefix=1
 character-set-client-handshake = FALSE
 character-set-server = utf8mb4
 collation-server = utf8mb4_unicode_ci 
```
Now press (Ctrl-X) to exit
```
sudo service mysql restart
```
```
sudo nano /etc/mysql/my.cnf
```

Add the following block of code exactly as is:
```
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

Now press (Ctrl-X) to exit

```
sudo service mysql restart
```

### Install frappe-bench
Initialize Frappe Bench

```
bench init --frappe-branch version-14 frappe-bench
```

Switch directories into the Frappe Bench directory
```
cd frappe-bench
```

### Change user directory permissions
This will give the bench user execution permission to the home directory.
```
chmod -R o+rx /home/[frappe-user]
```

## Create a New Site
A site is a requirement in ERPNext, Frappe and all the other apps we will be needing to install. We will create the site in this step.

```
bench new-site [site-name]
```

## Install ERPNext and other Apps
```
bench get-app payments
bench get-app --branch version-14 erpnext
```

## Install all the apps on our site
```
bench --site [site-name] install-app erpnext
bench start
```

If you didn’t have any other ERPNext instance running on the same server, ERPNext will get started on port 8000. If you visit [YOUR SERVER IP:8000], you should be able to see ERPNext version 14 running.

# Setting ERPNext for Production
Enable Scheduler
```
bench --site [site-name] enable-scheduler
```
Disable maintenance mode
```
bench --site [site-name] set-maintenance-mode off
```
Setup production config
```
sudo bench setup production [frappe-user]
```
Setup NGINX to apply the changes
```
bench setup nginx
```
Restart Supervisor and Launch Production Mode
```
sudo supervisorctl restart all
sudo bench setup production [frappe-user]
```

If you are prompted to save the new/existing config file, respond with a Y.

When this completes doing the settings, your instance is now on production mode and can be accessed using your IP, without needing to use the port.

This also will mean that your instance will start automatically even in the event you restart the server.

Default User is Administrator and use password you entered while creating new site.

## To setup multitenancy check out this link
- https://frappeframework.com/docs/v13/user/en/bench/guides/setup-multitenancy


