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

<img src="images/screen1.png" width="600px">

4. Choose a instance plan.

<img src="images/screen2.png" width="600px">

5. You can name your instance or just leave the default name provided by AWS.

<img src="images/screen3.png" width="600px">

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

9. Check which ports are open: `sudo ufw status`. You should see like this:

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
then the **Networking** tab, and then change the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed). Make sure to deny the default port 22.

<img src="images/screen4.png" width="600px">
<img src="images/screen5.png" width="600px">

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
grader  ALL=(ALL:ALL) ALL
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

<img src="images/screen6.png" width="600px">

3. Install the mod_wsgi package: `sudo apt-get install libapache2-mod-wsgi python-dev`.<br>
If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server: `sudo apt-get install libapache2-mod-wsgi-py3`.

4. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`.

### 11. Install and configure PostgreSQL

#### Install PostgreSQL

1. Install PostgreSQL by running `sudo apt-get install postgresql`.

2. Make sure PostgreSQL is not allowing remote connections. 
To check that, open the `/etc/postgresql/9.5/main/pg_hba.conf` file and make sure it looks like this, if you removed the comments:

```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

#### Create a new PostgreSQL user named `catalog` with limited permissions

3. PostgreSQL creates a Linux user with the name postgres during installation; switch to this user by running `sudo su - postgres` (for security reasons, it is important to only use the postgres user for accessing the PostgreSQL software).

4. Open PostgreSQL interactive terminal by running: `psql`.

5. Create the catalog user with a password by running:<br>
`postgres=# CREATE ROLE catalog WITH PASSWORD 'catalog';`

6. Give to our new user the ability to create databases:<br>
`postgres=# ALTER ROLE catalog CREATEDB;`

7. List the existing roles:<br>
`postgres=# \du`

Your output should look like the following:
```
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

8. Exit psql by running `\q`.

9. Switch back to the `grader` user by running `exit`.

#### Create a Linux user called `catalog`

10. Create a new Linux user called catalog by running: `sudo adduser catalog`. Enter password and fill out information.

11. Give to catalog user the permission to sudo. Run `sudo visudo`.

12. Search for the lines that looks like this:
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```

13. Below this line, add a new line to give sudo privileges to `catalog` user like this:
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
catalog  ALL=(ALL:ALL) ALL
```

14. Save and close the file by hitting **Ctrl-X**, followed by **Y**, and then hit **Enter** to confirm.

15. Verify that `catalog` has sudo permissions. Run `su - catalog`, enter the password, run `sudo -l` and enter the password again. You should see like this:

```
Matching Defaults entries for catalog on ip-172-26-13-170.us-east-2.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User catalog may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
    (ALL : ALL) ALL
```

#### Create a database called `catalog`

16. While logged in as `catalog`, create a database called catalog by running `createdb catalog`.

17. Run `psql` and then run `\l` to see that the new database has been created.

Your output should look like the following:
```
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
```

18. Exit psql by running `\q`.

19. Switch back to the `grader` user by running `exit`.

### 12. Install git

Install `git` by running the following commans: `sudo apt-get install git`.

## Deploy the Item Catalog project

### 13. Clone and setup the Item Catalog project from the Github repository 

#### Clone the Item Catalog project

1. Create a directory called `catalog` in the `/var/www/` directory by running: `sudo mkdir catalog`.

2. Change to the `/var/www/catalog` directory, and clone the catalog project:<br>
`sudo git clone https://github.com/boisalai/udacity-catalog-app.git catalog`.

#### Setup the Item Catalog Project

3. From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` by running:<br>
`sudo chown -R grader:grader catalog/`.

4. Change to the `/var/www/catalog/catalog` directory.

5. Rename the `application.py` file to `__init__.py` by running `mv application.py __init__.py`.

6. In `__init__.py`, replace line 27<br>
`app.run(host="0.0.0.0", port=8000, debug=True)` by<br>
`app.run()`.

7. In `database.py`, replace line 9<br>
`engine = create_engine("sqlite:///catalog.db")` by<br>
`engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`. 

<!--
$ sudo chown -R www-data:www-data /var/www/catalogApp/
Concernant la précédente instruction, voir ... faudra chercher davantage pour savoir pourquoi.
-->

#### Authenticate login through Google

Create a new project on the Google API Console.

8. Go to https://console.cloud.google.com/ of Google Cloud Plateform.

9. Click `APIs & services` on left menu.

10. Click `Credentials`.

11. Create an OAuth Client ID (under the Credentials tab), and make sure to add http://XX.XX.XX.XX and 
http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com as authorized JavaScript 
origins.

12. Add http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/login, 
http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/gconnect, 
and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/oauth2callback 
as authorized redirect URIs.

<!--
OAuth client
Here is your client ID:
534192602257-j2ednqkkgkc373ivi0e95pj9msbq9207.apps.googleusercontent.com
Here is your client secret:
ipqoZMk97Wh5VGbZkIY6gPGl
-->

13. Download the corresponding JSON file, open it et copy the contents.

14. Open `/var/www/catalog/catalog/client_secret.json` and paste the previous contents into the this file.

15. Replace the client ID to line 25 of the `templates/login.html` file in the project directory.

#### Authenticate login through Facebook

Create a new app at Facebook for Developers

16. Go to https://developers.facebook.com/.

17. Click `My Apps` and click `Add a New App`. 

18. Enter as `Display Name` then name `catalog`, enter your email and click 
`Create App ID`.

19. Click `Set Up` button of the `Facebook Login` card.

20. Choose Web Plateform.

21. Enter `http://XX.XX.XX.XX/` as site URL and ckick `Save` button.

22. Click `Settings` under `Facebook Login`, and put `http://XX.XX.XX.XX/` and 
`http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/` as the Valid OAuth redirect URIs, and click `Save Changes` button.

<!--
52.14.148.42
http://52.14.148.42/
http://ec2-52-14-148-42.compute-1.amazonaws.com/
-->

23. Click `Dashboard` on left menu. You should see `API Version` and `App ID` for the `catalog` application.

23. Replace the `appId` and `version`, respectively on lines 74 and 78 of the `templates/login.html`, with the correspoding `App ID` and `API Version`.

#### Set up a vitual environment 

24. Install pip with the following command: `sudo apt-get install python-pip`.

25. Install the virtual environment by running: `sudo apt-get install python-virtualenv`.

26. Change to the `/var/www/catalog/catalog/` directory.

27. Create the virtual environment by running: `virtualenv venv`.

28. Activate the new environment by running `. venv/bin/activate`.

#### Install dependencies

29. With the virtual environment active, install the following dependenies (note: with the exception of the libpq-dev package, make sure to not use sudo for any of the package installations as this will cause the packages to be installed globally rather than within the virtualenv):

`pip install httplib2`

`pip install requests`

`pip install --upgrade oauth2client`

`pip install sqlalchemy`

`pip install flask`

`sudo apt-get install libpq-dev` (Note: this will install to the global evironment)

`pip install psycopg2`

30. In order to make sure everything was installed correctly, run `python __init__.py`. The following (among other things) should be returned:

```
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

31. Deactivate the virtual environment by running `deactivate`.

#### Set up and enable a virtual host

32. Create a file in `/etc/apache2/sites-available/` called `catalog.conf`.

33. Add the following into the file:

```
<VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin ay.boisvert@gmail.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

<!--
52.14.148.42
-->

Note: the `Options -Indexes` lines ensure that listings for these directories in the browser is disabled.

34. Run `sudo a2ensite catalog` to enable the virtual host. The following prompt will be returned:
```
Enabling site catalog.
To activate the new configuration, you need to run:
  service apache2 reload
```

35. Reload Apache by running: `sudo service apache2 reload`.

#### Write the catalog.wsgi file 

Apache serves Flask applications by using a .wsgi file.

36. Create a file called `catalog.wsgi` in `/var/www/catalog` directory.

37. Add the following to the file:

```
activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from nuevoMexico import app as application
application.secret_key = "12345"
```

38. Restart Apache by running: `sudo service apache2 restart`.

#### Disable the default Apache site

39. At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run `sudo a2dissite 000-default.conf`. The following prompt will be returned:

```
Site 000-default disabled.
To activate the new configuration, you need to run:
  service apache2 reload
```

40. Reload Apache by running: `sudo service apache2 reload`.

#### Change the ownership of the project directories

41. Change the ownership of the project directories and files to the `www-data` user (this is done because Apache runs as the `www-data` user); while in the `/var/www` directory, run: `sudo chown -R www-data:www-data catalog/`.

#### Set up the database schema and populate the database

42. While in the `/var/www/catalog/catalog/` directory, activate the virtual environment by running `. venv/bin/activate`.

43. Then run `python data.py`.

#### Correct problems

We get the following error message:

```
sqlalchemy.exc.DataError: (psycopg2.DataError) value too long for type character varying(250)
```

To correct this problem, you need to modify the `data.py` file slightly:
- replace `lig.random_para(250)` by `lig.random_para(240)` on lines 86, 143, 191, 234 and 280.
- add the following instructions after line 16:<br>
```
# Delete all rows.
session.query(Item).delete()
session.query(Category).delete()
session.query(User).delete()
```

https://stackoverflow.com/questions/6357361/alternative-to-execfile-in-python-3


44. Then run again `python data.py`.

45. Deactivate the virtualenv (run `deactivate`).

46. Resart Apache again: `sudo service apache2 restart`.

47. Now open up a browser and check to make sure the app is working by going to http://XX.XX.XX.XX or http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com

<!--
http://52.14.148.42/
http://ec2-52-14-148-42.compute-1.amazonaws.com/
-->

<!--
J'obtiens un erreur "500 Internal Server Error"
sudo tail /var/log/apache2/error.log
-->




## Sources

This is a list of sources I used to complete this project.

Digital Ocean tutorial: [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

GitHub Repository: [bencam/linux-server-configuration](https://github.com/bencam/linux-server-configuration)

GitHub Repository: [adityamehra/udacity-linux-server-configuration](https://github.com/adityamehra/udacity-linux-server-configuration)

http://sageelliott.com/post/post2-AWS-Flask_setup/
