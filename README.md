# Linux Server Configuration

This is the final project for Udacity's Full Stack Web Developer Nanodegree.

A baseline installation of a Linux server is prepared to host web applications. the server is secured from a number of attack vectors, install and configure a database server, and deploy an existing web application onto it.

- The Linux distribution is [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS.
- Server instance provisioned from [Amazon Lightsail](https://lightsail.aws.amazon.com/).
- The web application is [World Cuisines](https://github.com/joshvarun/WorldCuisines) created earlier in this Nanodegree program.
- The database server is [PostgreSQL](https://www.postgresql.org/).

Link to Application: ~~http://ec2-13-233-237-95.ap-south-1.compute.amazonaws.com/~~ (Shut down instance as I graduated from the ND)

## Setting up your instance

### Creating an account on Amazon Lightsail & provisioning an instance

1. Create an account on [Amazon Lightsail](https://lightsail.aws.amazon.com/)
2. Click on create an instance.
3. Choose the following options: ```Linux/Unix```, ```OS-Only```, ```Ubuntu 16.04 LTS```
4. Select a plan as per your requirement
5. Give a unique name to your instance and click create
6. Take a break while your instance boots up.

### Connecting to the server via SSH
There are two ways by which you can connect to the server
1. Via Amazon Lightsail dashboard - Hit the Connect via ssh button.
2. Download the key from Amazon Lightsail and login via your terminal.

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `key.rsa`.
- In your terminal, type: `chmod 600 ~/.ssh/key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/key.rsa ubuntu@<YOUR INSTANCE IP address>`

## Securing the server

### Step 1: Check for new updates & install them

```
sudo apt-get update
sudo apt-get upgrade
```
### Step 2: Change default SSH Port to 2200

- In order to change the post, you need to edit `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the SSH port number from `22` to `2200`.
- Save and exit nano
- Restart the SSH Service: `sudo service ssh restart`

### Step 3: Configure the Firewall

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.=
  ```

- Enable the firewall: `sudo ufw enable`.

- Check the status of the firewall `sudo ufw status`.

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)  
  ```
- NOTE: You will be required to edit firewall settings to match the above on the Amazon Lightsail Instance page as well. Use type custom for SSH port.

- From your terminal, run: `ssh -i ~/.ssh/key.rsa -p 2200 ubuntu@<YOUR INSTANCE IP ADDRESS>` to login

## Configuring `grader` account

### Step 1: Create a new user called grader

- Add a new user by typing `sudo adduser grader`. 
- Password is `grader`

### Step 2: Granting sudo permissions
- Create a file named `grader` in `/etc/sudoers.d`
- Add the following line to it

```grader  ALL=(ALL:ALL) ALL```

### Step 3: Use the key-gen to generate a key pair for grader
- On your computer:
  - Run `ssh-keygen`
  - Enter file in which to save the key in the local directory `~/.ssh`
  - Enter in a password. Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - Run `cat ~/.ssh/grader_key.pub` and copy the key
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - To restrict logging in via password, set the `PasswordAuthentication` to no in `/etc/ssh/sshd_config` 
  - Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@<YOUR INSTANCE IP address>`.

## Web Application Deployment

### Step 1: Configure local timezone to UTC

Use this command ```sudo dpkg-reconfigure tzdata``` to set the UTC Time

### Step 2: Install Apache2 & WSGI
- Make sure you are logged in as `grader`
- Install Apache 2 ```sudo apt-get install apache2```
- To make sure it was installed correctly, visit your instance's IP address in the browser.
- You should see the Apache2 Default Page

- Install `mod_wsgi` -> ```sudo apt-get install libapache2-mod-wsgi```

- Enable wsgi -> `sudo a2enmod wsgi`
### Step 3: Install & configure PostgreSQL

- Make sure you are logged in as `grader`
- Install PostgreSQL ```sudo apt-get install postgresql```

- Switch to the `postgres` user: `sudo su - postgres`.
- Open PostgreSQL terminal with `psql`.
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;```

### Step 4: Install git

- Make sure you are logged in as `grader`
- Install `git`: `sudo apt-get install git`.

## Deploy your application
### Step 1: Clone the project repository from GitHub.

1. Login as `grader`, navigate to `/var/www`->
```mkdir catalog```
2. CD to catalog and clone the catalog project
```git clone https://www.github.com/WorldCuisines```
3. CD to `/var/www/catalog/catalog`
4. Rename `application.py` to `__init__.py`
5. In `__init__.py` replace the `app.run(host="0.0.0.0", port=8000)` line with `app.run()`
6. Change the create_engine line to -> 

`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')` 

### Step 2: Update your Google OAuth Credentials
1. Go to [Google Developers Console](https://console.developers.google.com)
2. Select your project. I recommend creating new credentials for this.
3. Create an OAuth Client ID (under the Credentials tab), and add your instance host name
http://ec2-xx-xxx-xxx-xxx.<REGION>.compute.amazonaws.com as authorized JavaScript 
origins.
4. http://ec2-xx-xxx-xxx-xxx.<REGION>.compute.amazonaws.com/cuisines
as authorized redirect URI.
5. Download the new JSON file, and replace the one in ```/var/www/catalog/WorldCuisines```
6. Replace the new client ID in the ```templates/login.html```

### Step 3: Installing Dependencies
- While logged in as `grader`, install pip: `sudo apt-get install python-pip`.
- Install the virtual environment: `sudo apt-get install python-virtualenv`
- Change to the `/var/www/catalog/WorldCuisines/` directory.
- Create the virtual environment: `sudo virtualenv -p python venv`.
- Change the ownership to `grader` with: `sudo chown -R grader:grader venv/`.
- Install the following dependencies:
  ```
  sudo -H pip install httplib2
  sudo -H pip install requests
  sudo -H pip install --upgrade oauth2client
  sudo -H pip install sqlalchemy
  sudo -H pip install flask
  sudo -H sudo apt-get install libpq-dev
  sudo -H pip install psycopg2
  ```

### Step 4: Setup a virtual host

-Create `/etc/apache2/sites-available/catalog.conf` and add the 
following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
	  ServerName <INSTANCE IP address>
      ServerAlias ec2-xx-xxx-xxx-xxx.<REGION>.compute.amazonaws.com
	  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	  <Directory /var/www/catalog/WorldCuisines/>
	  	Order allow,deny
		  Allow from all
	  </Directory>
	  Alias /static /var/www/catalog/WorldCuisines/static
	  <Directory /var/www/catalog/WorldCuisines/static/>
		  Order allow,deny
		  Allow from all
	  </Directory>
	  ErrorLog ${APACHE_LOG_DIR}/error.log
	  LogLevel warn
	  CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- Enable virtual host: `sudo a2ensite catalog`.
- Reload Apache: `sudo service apache2 reload`.
- Go to `/etc/apache2/sites-enabled/catalog.conf`. If empty, copy paste the above contents in this file also.

- Disable the default Apache site: `sudo a2dissite 000-default.conf`. 

- Reload Apache: `sudo service apache2 reload`.

### Step 5: Set up the Flask application

- Create `/var/www/catalog/catalog.wsgi` file add the following lines:

  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from WorldCuisines import app as application
  application.secret_key = 'super_secret_key'
  ```
- Restart Apache: `sudo service apache2 restart`.

### Step 6: Database changes
- Open up the database_setup.py & replace the create_engine command with ```engine = create_engine('postgresql://catalog:catalog@localhost/catalog'```

- Run the script with ```python database_setup.py```

## Launch the Web Application

- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to <INSTANCE IP address> or http://ec2-xx-xxx-xxx-xxx.<REGION>.compute.amazonaws.com.

## References

- DigitalOcean [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- Stack Overflow [Unsupported Locale Settings](https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting/14548156)
- [Apache Error Logs](https://unix.stackexchange.com/questions/38978/where-are-apache-file-access-logs-stored)
- Shout out to [iliketomatoes](https://github.com/iliketomatoes/linux_server_configuration) & [boisalai](https://github.com/boisalai/udacity-linux-server-configuration) for writing a great README.md to refer
