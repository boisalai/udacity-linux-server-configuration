# Linux Server Configuration

This is the final project for "Full Stack Web Developer Nanodegree" on Udacity. 

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

You can visit http://35.164.53.24/ for the website deployed.

## Get a server

### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail 

The following steps will create a LightSail Instance for Ubuntu.

1. Login into [AWS Light Sail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account

2. Once you are login into the site, click **Create instance**. 

3. Choose **Linux/Unix** platform, **OS Only** and  **Ubuntu 16.04 LTS**.

![alt text](screen1.png "screen1")

<img src="screen1.png" width="600px">

4. Choose a instance plan.

![alt text](screen2.png "screen2")

5. You can name your instance or just leave the default name provided by AWS.

![alt text](screen3.png "screen3")

6. Click **Create**.

7. Wait for the instance to start up.

### 2. SSH into the server

The following steps outline how to connect to the instance via the Terminal program on Mac OS machines.

1. Download Default Private Key from the **SSH keys** section in the **Account** section on Amazon Lightsail.

2. Move this private key file into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.

3. In your terminal, type in `chmod 600 ~/.ssh/lightsail_key.rsa`.

4. In your terminal, type in `ssh -i ~/.ssh/lightsail_key.rsa ubunut@XX.XX.XX.XX`, where `XX.XX.XX.XX` is the public IP address of the instance.

## Secure the server

### 3. Update all currently installed packages

1. Notify the system of what package updates are available by 
running `sudo apt-get update`.

2. Download available package updates by running `sudo apt-get upgrade`.

### 4. Change the SSH port from 22 to 2200

1. Edit the file `/etc/ssh/sshd_config` with the command `sudo nano /etc/ssh/sshd_config`.

2. Change the port number on line 5 from 22 to 2200, save and quit.

3. Restart SSH by running `sudo service ssh restart`.

### 5. Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. The UFW should be inactive. Check it by running `sudo ufw status`.

2. By default, block all incoming connections on all ports: `sudo ufw default deny incoming`.

3. Allow outgoing connection on all ports: `sudo ufw default allow outgoing`.

4. Allow incoming connection for SSH on port 2200: `sudo ufw allow 2200/tcp`.

5. Allow incoming connections for HTTP on port 80: `sudo ufw allow www`.

6. Allow incoming connection for NTP on port 123: `sudo ufw allow 123/udp`.

7. Since the virtual machine has now been configured so that SSH uses port 2200, port 22 is not being used for anything. So, deny port 22: `sudo ufw deny 22`.

8. enable the ufw firewall: `sudo ufw enable`.

9. Check which ports are open: `sudo ufw status`. You should see like this:

```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```

10. Now, click on the **Manage** option of the Amazon Lightsail Instance, 
then the **Networking** tab, and then change the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP)should be allowed; make sure to deny the default port 22).

![alt text](screen4.png "screen4")
![alt text](screen5.png "screen5")

## Give grader access

### 6. Create a new user account named grader

1. Add user: `sudo adduser grader`. 

2. Enter a password (twice) and fill out information for this new user.

### 7. Give grader the permission to sudo

1. Edits the sudoers file: `sudo visudo`.

2. Search for the line that looks like this:
```
root    ALL=(ALL:ALL) ALL
```

3. Below this line, add a new line to give sudo privileges to **grader** user like this:
```
root    ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL
```

4. Save and close the file by hitting **Ctrl-X**, followed by **Y**, and then hit **Enter** to confirm.

5. Verify that **grader** has sudo permissions. Run `su - grader`, enter the password, run `sudo -l` and enter the password again. You should see like this:

```
Matching Defaults entries for grader on ip-172-26-13-170.us-east-2.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
    (ALL : ALL) ALL
```

### 8. Create an SSH key pair for grader using the `ssh-keygen` tool

1. Run `ssh-keygen` on the local machine.

2. Enter file in which to save the key (for this example, I give the name `grader_key`) in the local directory `~/.ssh`.

3. Enter in a passphrase twice. Two files will be generated (for this example,  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`).

4. Log in to the grader's virtual machine.

5. Create a new directory called `~/.ssh` (`mkdir .ssh` from grader's home directory).

6. Run `touch .ssh/authorized_keys`.

7. On the local machine, run `cat ~/.ssh/grader_key.pub` and copy 
the contents of the file.

8. On the grader's virtual machine, run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and close.

9. On the grader's virtual machine, run `chmod 700 .ssh`. 

10. On the grader's virtual machine, run `chmod 644 .ssh/authorized_keys`. 

11. On the grader's virtual machine, make sure key-based authentication is forced. Check in `/etc/ssh/sshd_config` file if **PasswordAuthentication** is set to **no** like this:

```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```

12. On the grader's virtual machine, run `sudo service ssh restart`.

13. On the local machine, log in as the grader using the following command: `ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`.

<!--
ssh -i ~/.ssh/grader_key -p 2200 grader@52.14.148.42
le paraphrase est grader
-->

## Prepare to deploy the project

### 9. Configure the local timezone to UTC

Configure the time zone: `sudo dpkg-reconfigure tzdata`. You should see something like that:

```
Current default time zone: 'America/Montreal'
Local time is now:      Thu Oct 19 21:55:16 EDT 2017.
Universal Time is now:  Fri Oct 20 01:55:16 UTC 2017.
```

### 10. Install and configure Apache to serve a Python mod_wsgi application

1. Run `sudo apt-get install apache2` to install Apache.

2. Check to make sure Apache is working by using the public IP of the Amazon Lightsail instance as a URL in a browser. If yes, a page with the title **Apache2 Ubuntu Default Page** should load.

![alt text](screen6.png "screen6")

3. Install the mod_wsgi package: `sudo apt-get install libapache2-mod-wsgi python-dev`.

4. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`.

### 11. Install and configure PostgreSQL

Do not allow remote connections
Create a new database user named catalog that has limited permissions to your catalog application database.

(14) 
Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
Install PostgreSQL by running sudo apt-get install postgresql
$ sudo apt-get install postgresql
Open the /etc/postgresql/9.5/main/pg_hba.conf file
$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
Make sure it looks like this (comments have been removed here for easier reading):
See https://github.com/boisalai/linux-server-configuration
See also https://www.postgresql.org/docs/9.1/static/auth-pg-hba-conf.html

(16) 
# Create a new PostgreSQL user named catalog with limited permissions
$ sudo su - postgres
# Connect to psql (the terminal for interacting with PostgreSQL) by running psql
$ psql
# Create the catalog user by running CREATE ROLE catalog WITH LOGIN;
CREATE ROLE catalog WITH LOGIN;
# Next, give the catalog user the ability to create databases: ALTER ROLE catalog CREATEDB;
ALTER ROLE catalog CREATEDB;
# Finally, give the catalog user a password by running \password catalog

\password catalog
catalog [Enter]
catalog [Enter]

\du
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 
\q
exit

(17)
Create a Linux user called catalog and a new PostgreSQL database
$ sudo adduser catalog
[sudo] password for alan: 
Adding user `catalog' ...
Adding new group `catalog' (1002) ...
Adding new user `catalog' (1002) with group `catalog' ...
Creating home directory `/home/catalog' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for catalog
Enter the new value, or press ENTER for the default
	Full Name []: Catalog User
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] 

Give the catalog user sudo permissions
$ sudo visudo
add the following line below this one: catalog ALL=(ALL:ALL) ALL

verify that catalog has sudo permissions
$ sudo su - catalog
$ sudo -l
[sudo] password for catalog: [catalog|
Matching Defaults entries for catalog on ip-172-26-1-254.us-east-2.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User catalog may run the following commands on ip-172-26-1-254.us-east-2.compute.internal:
    (ALL : ALL) ALL

(18)
While logged in as catalog, create a database called catalog by running createdb catalog

$ createdb catalog
catalog@ip-172-26-1-254:~$ psql
psql (9.5.9)
Type "help" for help.

catalog=> \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

Switch back to the ubuntu user by running exit
\q
exit



### 12. Install git

Install git and clone the catalog project
$ sudo apt-get install git

Pas nécessaire, mais bon...
$ git config --global user.name "YOURNAME"
$ git config --global user.email "YOU@DOMAIN.com"




## Deploy the Item Catalog project

### 13. Clone and setup the Item Catalog project from the Github repository 

(20)
--> Puis-je recommencer avec seulement 'catalog' au lieu de 'catalogApp'...
Create a directory called 'catalogApp/catalogApp' in the /var/www/ directory
Create flask app taken from digitalocean
See https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
$ cd /var
$ sudo mkdir www
$ cd www
$ sudo mkdir catalogApp
$ cd catalogApp
$ sudo mkdir catalogApp
$ sudo chown -R www-data:www-data /var/www/catalogApp/
Concernant la précédente instruction, voir ... faudra chercher davantage pour savoir pourquoi.
En plus, si je fais cela ici, ne pas faire plus bas...
https://github.com/boisalai/fullstack-nanodegree-linux-server-config
https://github.com/boisalai/linux-server-configuration
$ sudo git clone https://github.com/boisalai/udacity-catalog-app.git catalogApp

Change the ownership of the 'catalogApp' directory to ubuntu by running (while in /var/www):
$ sudo chown -R ubuntu:ubuntu catalogApp/

Change to the /var/www/catalogApp directory

Change the name of the application.py file to __init__.py by running mv application.py __init__.py
$ mv application.py __init__.py
$ sudo nano __init__.py

In __init__.py, find line x:
app.run(host='0.0.0.0', port=8000)
Change this line to:
app.run()

Switch the database in the application from SQLite to PostgreSQL
Replace line 9 in database.py with the following:
$ sudo nano database.py

engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

Add client_secrets.json and fb_client_secrets.json files

.....
2017-10-17
5. Deploying to Linux Servers
Project: Linux Server Configuration

Je continue...

(21)
Create a new project on the Google API Console
Goto
https://console.cloud.google.com/home/dashboard?project=api-project-534192602257
Click on APIs & services, puis sur Credentials

Create an OAuth Client ID (under the Credentials tab), and make sure to add http://13.59.171.132 and 
http://ec2-13.59.171.132.compute-1.amazonaws.com as authorized JavaScript origins

Add http://ec2-13.59.171.132.compute-1.amazonaws.com/login, 
http://ec2-13.59.171.132.compute-1.amazonaws.com/gconnect, 
and http://ec2-13.59.171.132.compute-1.amazonaws.com/oauth2callback 
as authorized redirect URIs

J'obtiens...

OAuth client
Here is your client ID:
534192602257-bgho9e8qcasnt2inktff0apocg6lga1c.apps.googleusercontent.com
Here is your client secret:
clOZt6W_WjVk3812EmHZ_sNt

Télécharger le fichier client_secret*.json
L'ouvrir et copier le contenu.

Ouvrir le fichier "client_secret.json"
ubuntu@ip-172-26-1-254:/var/www/catalogApp$ sudo nano client_secret.json
Copier le contenu à l'intérieur.

(22)
Create a new app at Facebook for Developers
Goto
https://developers.facebook.com/
Cliquer sur "My Apps" puis sur "Add a New App"
Cliquer sur "Add the 'Facebook Login'"

Add the 'Facebook Login' product, and put http://13.59.171.132/ and 
http://ec2-13-59-171-132.compute-1.amazonaws.com/ as the Valid OAuth redirect URIs

Facebook me retourne du code...

```
<script>
  window.fbAsyncInit = function() {
    FB.init({
      appId      : '1979089279036389',
      xfbml      : true,
      version    : 'v2.10'
    });
    FB.AppEvents.logPageView();
  };

  (function(d, s, id){
     var js, fjs = d.getElementsByTagName(s)[0];
     if (d.getElementById(id)) {return;}
     js = d.createElement(s); js.id = id;
     js.src = "//connect.facebook.net/en_US/sdk.js";
     fjs.parentNode.insertBefore(js, fjs);
   }(document, 'script', 'facebook-jssdk'));
</script>
```

Ne pas oublier mon mot de passe dans Facebook Developer qui est alpha111
Secret key is... f61e0c9b48ab62f09392d20419cc312b
Mais où dois-je mettre ma clé secrète?

Disons que je vais seulement changer appId et version

Ouvrir le fichier login.html
ubuntu@ip-172-26-1-254:/var/www/catalogApp/catalogApp/templates$ sudo nano login.html
Remplacer le champ "data-clientid=" par la clé "Client ID" de Google.
Remplacer le champ "appId=" de Facebook par '1979089279036389'
Remplacer le champ "version=" de Facebook par 'v2.10'

(23)
Set up a vitual environment and install dependencies
$ sudo apt-get install python-pip

/*
Finalement, je n'ai pas utilisé d'environnement virtuel.
Donc ne pas faire...
Cependant, voir ceci http://sageelliott.com/post/post2-AWS-Flask_setup/

$ sudo apt-get install python-virtualenv

Exécuter la commande 'virtualenv venv' dans le bon répertoire...
ubuntu@ip-172-26-1-254:/var/www/catalogApp/catalogApp$ virtualenv venv
Running virtualenv with interpreter /usr/bin/python2
New python executable in /var/www/catalogApp/catalogApp/venv/bin/python2
Also creating executable in /var/www/catalogApp/catalogApp/venv/bin/python
Installing setuptools, pkg_resources, pip, wheel...done.
*/

$ sudo pip install httplib2
$ sudo pip install requests
$ sudo pip install --upgrade oauth2client
$ sudo pip install flask
$ sudo pip install sqlalchemy
$ sudo pip install psycopg2

Je ne sais pas s'il m'en manque.
sudo apt-get install libpq-dev (Note: this will install to the global evironment)
pip install psycopg2 ??

In order to make sure everything was installed correctly, run python __init__.py; the following (among other things) 
should be returned:
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)


(24)
Set up and enable a virtual host
Create a file in /etc/apache2/sites-available/ called catalogApp.conf 
Je serai obligé de créer le répertoire $ mkdir sites-available
ubuntu@ip-172-26-1-254:/etc/apache2/sites-available$ sudo nano catalogApp.conf 

Add the following into the file:

<VirtualHost *:80>
		ServerName 13.59.171.132
		ServerAdmin ay.boisvert@gmail.com
		WSGIScriptAlias / /var/www/catalogApp/catalogApp.wsgi
		<Directory /var/www/catalogApp/catalogApp/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalogApp/catalogApp/static
		<Directory /var/www/catalogApp/catalogApp/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

Run sudo a2ensite catalogApp to enable the virtual host
$ sudo a2ensite catalogApp

The following prompt will be returned:
Enabling site catalogApp.
To activate the new configuration, you need to run:
  service apache2 reload
  
Reloader
$ sudo service apache2 reload

(25)
Write a .wsgi file

Regarder ce qui est écrit ici...
http://sageelliott.com/post/post2-AWS-Flask_setup/

Create the wsgi file
cd /var/www/catalogApp
sudo nano catalogApp.wsgi

activate_this = '/var/www/catalogApp/catalogApp/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalogApp/")

from nuevoMexico import app as application
application.secret_key = '12345'
```
Resart Apache: 
$ sudo service apache2 restart




J'obtiens un message d'erreur.
Je remplace "description=lig.random_para(250)," par "description=lig.random_para(240)," à 4 endroits.

Je suis obligé d'ajouter des lignes à data.py pour détruire les occurances déjà inséré dans la table.
 https://stackoverflow.com/questions/16573802/flask-sqlalchemy-how-to-delete-all-rows-in-a-single-table

Voici les lignes que j'ajoute...
$ sudo nano data.py

Create database.
create_db()

Delete all rows.
session.query(Item).delete()
session.query(Category).delete()
session.query(User).delete()



### 14. Set it up in the server so that it functions correctly when visiting your server’s IP address in a browser. 

Make sure that your .git directory is not publicly accessible via a browser!

## Sources

This is a list of sources I used to complete this project.

Digital Ocean tutorial: [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

GitHub Repository: [bencam/linux-server-configuration](https://github.com/bencam/linux-server-configuration)

GitHub Repository: [adityamehra/udacity-linux-server-configuration](https://github.com/adityamehra/udacity-linux-server-configuration)
