# Description
A goal is to user a baseline installation of Linux server to host web applications. The process involves securing the server of a number of attach vectors, install and configure a database server, and deploy using [existing application](https://github.com/nvictor7/project-item-catalog).   
Server Information:
- Public IP address: 18.219.158.15
- SSH port: 2200

# Setup
## Prerequisites

### SSH to server
- Download private SSH Keys from Amazon account page
- Move private keys from `Download` directory to `~/.ssh`
- run `CHMOD 600 ~/.sss/udacity_project_key.pem` 
- `ssh -i ~/.ssh/udacity_project_key.pem ubuntu@18.219.158.15` log into the server

### Update
- Update currently installed packages  

- `apt-get update` gets status of packages that require updates   

- `apt-get upgrade` installs all necessary updates  

- `*** System restart required ***` shown. `sudo reboot` to restart 

### Add User 'Grader' 
Add a new user named **grader**   
Password: **grader**

- `sudo adduser grader` adds a user named 'grader'    

- `sudo nano /etc/sudoers.d/grader` opens the file for editting

- Add `grader ALL=(ALL:ALL) NOPASSWD:ALL` inside the file
 
### SSH Login as user 'grader'
 
- `ssh-keygen` generates key pair locally, saving to `~/.ssh` local machine directory  

- Back on the server, `su - grader` switchs to grader user

- `mkdir .ssh` creates a key-related directory

- `touch .ssh/authorized_keys` creates a file for storing public keys

- `cat .ssh/[your_key_filename]` opens the private key file, then copy keys

- `nano .ssh/authorized_project_keys` opens the file, then place copy of keys from previous step

- `chmod 700 .ssh`

- `chmod 644 .ssh/authorized_keys` 

- Now login as grader using `ssh -i ~/.ssh/udacity_project_keys grader@18.219.158.15`

### Disable Root login

- `sudo nano /etc/ssh/sshd_config` opens configuration file

- Find line `PermitRootLogin prohibit-password`, change this line to `PermitRootLogin no`

- `sudo service ssh restart`

### Firewall Configurations

- `sudo ufw default deny incoming` blocks all incoming connection

- `sudo ufw default allow outgoing` allows all connections to the internet

- `sudo ufw allow 2200/tcp` allows incoming connection for SSH on port 2200

- `sudo ufw allow www` allows HTTP incoming connections on port 80

- `sudo ufw allow ntp` allows NTP incoming connections on port 123

- `sudo ufw enable` enables firewall rules

### Configure timezone to UTC

- `sudo dpkg-reconfigure tzdata' opens dialog, then select `None of above`, lastly select `UTC`

### Apache web server & WSGI module installations

- `sudo apt-get install apache2`

- `sudo apt-get install libapache2-mod-wsgi`

### Install & Configure PSQL

- Install PSQL `sudo apt-get install postgresql`

- `sudo nano /etc/postgresql/9.5/main/pg_hba.conf` checks remote connection to PSQL. IPv4 set to 127.0.0.1 and IPv6 set to ::1

- `sudo su - postgres` logs into 'postgres' user

- `psql` enters into PSQL mode

- `CREATE DATABASE catalog;` creates database named 'catalog'

- `CREATE USER catalog;` creates user named 'catalog'

- `ALTER ROLE catalog WITH PASSWORD 'password';` sets password for user 'catalog'

- `\q` exits PSQL mode

- Exit out user 'postgres'

### Installations of Git, Framework, Module, Dependencies, Libraries

- `sudo apt-get install git`

- `sudo apt-get install python-pip`

- `sudo pip install Flask`

- `sudo pip install requests`

- `sudo pip install sqlalchemy oauth2client httplib2 psycopg2`

- `sudo apt-get -qqy install postgresql python-psycopg2`

### Clone & Setup 'catalogApp' application

- `cd /var/www`

- `sudo mkdir catalogApp`

- `cd catalogApp`

- `sudo chown -R grader:grader /var/www/catalogApp` 

- `git clone https://github.com/nvictor7/project-item-catalog.git`

- `mv ./project-item-catalog ./catalogApp`

- `cd catalogApp`

- `mv app.py __init__.py`

- Inside `__init__.py`, replace `CLIENT_ID = json.loads(open('client_secrets.json', 'r')` with `CLIENT_ID = json.loads(open('/var/www/catalogApp/catalogApp/client_secrets.json', 'r')`

- Replace this line `engine = create_engine('sqlite:///printingmachines.db')` with `engine = create_engine('postgresql://catalog:password@localhost/catalog')` in files `database_setup.py`, `initial_data.py`, and `__init__.py`

- `python database_setup.py`

- `python initial_data.py`

- `sudo nano /etc/apache2/sites-available/catalogApp.conf` opens the file, then add the following: 
  ```
     <VirtualHost *:80>
         ServerName 18.219.158.15
         ServerAdmin sna_07@yahoo.com
         WSGIScriptAlias / /var/www/catalogApp/catalogApp.wsgi
         <Directory /var/www/catalogApp/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalogApp/catalogApp/static
        <Directory /var/www/catalogApp/catalogApp/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
    
- `sudo nano /etc/apache2/sites-available/catalogApp.conf` enables virtual host

- Inside `/var/www/catalogApp` directory, create a file `catalogApp.wsgi`

- `nano catalogApp.wsgi` create a file, then adds the following
  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalogApp/")

  from catalogApp import app as application
  application.secret_key = 'super_secret_key'
  ```

### Edit Google Loggin Configuration

- Enter the file `nano client_secrets.json`, then replace existing line to `"javascript_origins":["http://18.219.158.15/"]`

- Also update inside Google developer console `"javascript_origins":["http://18.219.158.15/"]`

### Load Application

- `sudo service apache2 restart` loads application
