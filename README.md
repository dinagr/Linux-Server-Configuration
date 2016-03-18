## Linux server configuration

### General

You can connect to the server usong this IP - 52.35.54.206 and port 2200

Here you can visit my app http://ec2-52-35-54-206.us-west-2.compute.amazonaws.com/

#### A summary of software installed and configuration changes made


###### 1.	Launch your Virtual Machine -

    		Follow the instruction in the https://www.udacity.com/account#!/development_environment

###### 2.	Follow the instructions provided to SSH into your server - 

			'mv /home/.ssh/udacity_key.rsa ~/.ssh/udacity_key.rsa'

###### 3.	Create a new user named grader - 

			'adduser grader'

###### 4.	Give the grader the permission to sudo - 

			create a file named 'grader' in 'etc/sudoers.d' 

			enter the following line to the file 'grader ALL=(ALL) NOPASSWD:ALL'

###### 5.	Update all currently installed packages - 

			'sudo apt-get update'
			
			'sudo apt-get upgrade'

			Generating Key Pairs - 

			1.  run 'ssh-keygen' on my computer (not on the virtual machine)
				I entered where to save the file and created a passphrase

			2.  On the server - 

				1.	Run 'mkdir .ssh'
				2.	Enter to the directory 'cd .ssh'
				3.	Create the authorized_keys file - 'touch authorized_keys'
				4.	Open the file for editing -  'nano authorized_keys'
				5.	I copied the content of the file that was created on my coputer in step a (the file with the '.pub' ending) 
					to the authorized_keys file and saved the changes
				6.  I gave authorozations to the .ssh file 'chmod 700 .ssh' and to the authorized_keys file 'chmod 644 .ssh/authorized_	keys'

			Now I can log in to the server using 'ssh grader@52.35.54.206 -p2200' and than enter the passphrase to log in

###### 6.	Change the SSH port from 22 to 2200 -

			Go to 'sudo nano /etc/ssh/sshd_config' and changed the port to 2200

###### 7.	Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) - 

			Check if the firewall is active 'sudo ufw status'

			Deny all incoming comunication  'sudo ufw default deny incoming'

			Allow all outgoing communication 'sudo ufw default allow outgoing'

			Configure the incoming communication (At this point I am connected to the srver in 2 different session to make sure that I will not be locked out of the server)

			'sudo ufw allow ssh'
			'sudo ufw deny 2222/tcp'
			'sudo ufw allow 22/tcp'
			'sudo ufw allow 2200/tcp'
			'sudo ufw allow www'
			'sudo ufw allow 80/tcp'
			'sudo ufw allow http'
			'sudo ufw allow 123'
			'sudo ufw enable' - activate the firewall
			
			Then I entered the following commaned 'service sshd restart' and checked if I can stil log in to the server

###### 8.	Configure the local timezone to UTC - 

			I entered 'date' coomand and saw that tme zone is allready UTC

###### 9.	Install and configure Apache to serve a Python mod_wsgi application - 

			'sudo apt-get install apache2'
			'sudo apt-get install python-setuptools libapache2-mod-wsgi'
			'sudo apt-get install libapache2-mod-wsgi python-dev'
			'sudo a2enmod wsgi'
			'sudo service apache2 restart' - after running that I received an error

			So I went to the '/etc/apache2/apache2.conf' and added this line to the end of the file 'ServerName localhost'
			
			After restarting apache again the error did not appear

###### 10.	Install and configure PostgreSQL - 

			Install postgreSQL 'sudo apt-get install postgresql'

			Log in to postgreSQL 'sudo -i -u postgres'

			Open postgreSQL enviroment using 'psql'
			
			This creates the user with a password  'CREATE ROLE catalog PASSWORD 'catalogLinuxWebServer';'
			
			This gives him the authorozation to create and change DB 'ALTER USER catalog CREATEDB;'
			
			Do not allow remote connections - By default this is configured
											  This can be checked in the file '/etc/postgresql/9.3/main/pg_hba.conf'	
###### 11.	Install git - 
			'sudo apt-get install git'
			'git config --global user.name "YOUR NAME"'
			'git config --global user.email "YOUR EMAIL ADDRESS"'

###### 12.	clone and setup your Catalog App project

			Go to '/var/www' and 'mkdir catalogApp'

			Enter to catalogApp and create the same folder again 'mkdir catalogApp'
			
			Create a file here '/var/www/catalogApp' named 'catalogApp.wsgi'
			
			Enter to the file the following 
			
			'#!/usr/bin/python
			import sys
			import logging
			logging.basicConfig(stream=sys.stderr)
			sys.path.insert(0,"/var/www/catalogApp/")


			from catalogApp import app as application
			application.secret_key = 'Add your secret key' '
			
			Go to 'cd '/var/www/catalogApp/catalogApp' and clone the gir repository to the server
			
			'git clone https://github.com/dinagr/CatalogApp'

			I arranged the file from the repository in the following structure - 

			var
				www
					catalogApp
						CatalogApp.wsgi
						CatalogApp
							__init__.py
							database_setup.py
							data.py
							client_secrets.json
							fb_client_secrets.json
							static
								All the css files for the project
							templates
								All the html files for the project

			I changed the following lines in the __init__.py/database_setup.py, data.py file -
			
			All the line with the refered to sqlite were changed to refer to  postgreSQL as following:
			
			'engine = create_engine('postgresql://catalog:catalogLinuxWebServer@localhost/categorieitems')'

			I changed the folling line in the __init__.py file - 

			'CLIENT_ID = json.loads(
		    open('client_secrets.json', 'r').read())['web']['client_id']''

		    Was changed to - 

		    'full_path_to_secrets_file = os.path.join('/var/www/catalogApp/catalogApp/', 'client_secrets.json')
			CLIENT_ID = json.loads(
		    open(full_path_to_secrets_file, 'r').read())['web']['client_id']'

		    I did the same chnage for every place in the code where I am refering to 'client_secrets.json' or 'fb_client_secrets.json'

		    Install pip 'sudo apt-get install python-pip'
			Install Flask 'sudo pip install Flask'
			Install virtualenv 'sudo pip install virtualenv'
			Create a virtualenv 'sudo virtualenv venv'
			Activate it 'source venv/bin/activate'
			Install Flask inside the virtualenv 'sudo pip install Flask'

			Create the file 'catalogApp.conf' in this path '/etc/apache2/sites-available/'

			Enter the following inside the file -

			'<VirtualHost *:80>
		                ServerName ec2-52-35-54-206.us-west-2.compute.amazonaws.com
		                ServerAdmin admin@52.35.54.206
		                WSGIScriptAlias / /var/www/catalogApp/catalogApp.wsgi
		                <Directory /var/www/catalogApp/catalogApp/>
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
			</VirtualHost>'

			Open the '000-default.conf' file in the same path and copy the same inside this file

			And the I enabled the virtual host using 'sudo a2ensite catalogApp' 

			I restarted apache 'sudo service apache2 restart'

			Inside the virualenv venv I installed the following -

			'pip install httplib2'
			'pip install requests'
			'pip install oauth2client'
			'pip install sqlalchemy'
			'sudo apt-get install python-psycopg2'

			During the installation I had some problems so I changed the authorozations to this directory

			'/var/www/catalogApp/catalogApp/venv/lib/python2.7/site-packages' using 'chmod 777 site-packages'

			I restarted apache 'sudo service apache2 restart'

			I checked if there are no errors 'sudo tail -20 /var/log/apache2/error.log'

###### 13.	Configurate google and facebook login to work with the new IP

			Google -
				a. In the google console developer I changed all the configuration from 'loclhost' to 'ec2-52-35-54-206.us-west-2.compute.amazonaws.com' 

				b. In the 'client_secrets.json' I did the same changes

			Facebook - 
				a. In the facebook developer enviroment I changed the configuration from 'localhost' to 'ec2-52-35-54-206.us-west-2.compute.amazonaws.com' 

			I restarted the app again 'sudo service apache2 restart'

