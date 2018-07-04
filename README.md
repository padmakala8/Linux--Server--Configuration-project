# Udacity-Linux-Server-Configuration
By padmakala pampana
About
This is the Udacity project 6 about the Configuring the Linux the server.

Server Details
Server IP Address 13.127.49.86

Hosted site Url http://13.127.49.86.xip.io/

Configuring Linux Server
Updating all packages
sudo apt-get update
sudo apt-get upgrade
Creating grader User:
sudo adduser grader
This will add new user give a password. (Mine is) : kala

sudo nano /etc/sudoers
Below the Root user append the following line

grader  ALL=(ALL:ALL) ALL
This will grant sudo permission to grader

Creating a ssh key pair for grader
On your local machine in terminal/command prompt

ssh-keygen
This will generate public and private ssh keys which is saved to .ssh folder

Then in your virtual machine change to newly created user

su - grader
Create a new directory .ssh and new file authorized_keys in that directory

mkdir .ssh
sudo nano .ssh/authorized_keys
Copy the public key with .pub extension to authorized_keys and save the file

chmod 700 .ssh
chmod 644 .ssh/authorized_keys
700 will give read write and execute permission to user.
644 prevent other user from writting in to file. Then restart ssh server
service ssh restart
Now from your log in to grader with private key generated

ssh -i .ssh/id_rsa grader@13.127.49.86
Changing the ssh port to 2200
sudo nano /etc/ssh/sshd_config
Change port 22 to port 2200

Restart the ssh server

service ssh restart
Note: Before Logging using ssh add custom TCP port 2200 under lightsaail firewall in networking tab in lightsail instance console

Now Login using command like this

ssh -i .ssh/id_rsa -p 2200 grader@13.127.49.86
Disabling ssh login as root
sudo nano /etc/ssh/sshd_config

make change PermitRootLogin no

Configurating Ufw firewall
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
This will allow all required ports and enables the ufw

After that

sudo ufw status
It will display all allowed ports

Installing Apache2
In terminal

sudo apt-get install apache2

Now mod_wsgi

sudo apt-get install python-setuptools libapache2-mod-wsgi

Enable mod_wsgi

sudo a2enmod wsgi

Setting up your flask application to work with apache2
Creating a flask app

In /var/www directory create a new folder sudo mkdir FlaskApp

Install git

sudo apt-get install git

move to the FlaskApp cd FlaskApp

In that direcory clone your github repository

sudo git clone 'https://github.com/PravallikaChodisetty/catalogpr.git'

Rename the repository to FlaskApp

Then rename your project file to __init__.py

Make Following changes in init.py

PROJECT_ROOT = os.path.realpath(os.path.dirname(__file__))
json_url = os.path.join(PROJECT_ROOT, 'client_secrets.json')
CLIENT_ID = json.load(open(json_url))['web']['client_id']
Use json_url instead client_secrets.json in script

Install and configuring postgresql for project
Install Postgres sudo apt-get install postgresql

login to postgres sudo su - postgres

postgres shell psql

create user CREATE USER catalog WITH PASSWORD 'password';

permit user to createdb ALTER USER catalog CREATEDB;

Create a db name catalog with user catalog CREATE DATABASE catalog WITH OWNER catalog;

connect to db \c catalog

revoke all permission to public REVOKE ALL ON SCHEMA public FROM public;

Give schema permission to user catalog GRANT ALL ON SCHEMA public TO catalog;

exit from db and postgres \q and exit

Change the database connection in both db_setup.py and __init__.py as engine = create_engine('postgresql://catalog:password@localhost/catalog')

Now you are ready with your applicatiom

Configure and Enable a New Virtual Host
sudo nano /etc/apache2/sites-available/FlaskApp.conf

In this add the following code

<VirtualHost *:80>
 	ServerName 13.127.162.197
 	ServerAdmin admin@mywebsite.com
 	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
 	<Directory /var/www/FlaskApp/FlaskApp/>
 		Order allow,deny
 		Allow from all
 	</Directory>
 	Alias /static /var/www/FlaskApp/FlaskApp/static
 	<Directory /var/www/FlaskApp/FlaskApp/static/>
 		Order allow,deny
 		Allow from all
 	</Directory>
 	ErrorLog ${APACHE_LOG_DIR}/error.log
 	LogLevel warn
 	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
Enable the virtual host sudo a2ensite FlaskApp

Disabling the default apache2 page sudo a2dissite 000-default.conf

Create the .wsgi File
```
cd /var/www/FlaskApp
sudo nano flaskapp.wsgi 
```
Add the following code

#!/usr/bin/python
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0,"/var/www/FlaskApp/")

 from FlaskApp import app as application
 application.secret_key = 'Add your secret key'
save and exit

Deploying flask app with apache2 is referred from Digital ocean

Installing require modules
You can either install all modules on your machine or create a virtual environment for the project and install the modules pip install flask sqlalchemy psycopg2 requests oauth2client

Setting up your Google Oauth2
Go to Google Cloud Plateform.

Click APIs & services on left menu.

Click Credentials.

Create an OAuth Client ID (under the Credentials tab), and add http://13.127.162.197.xip.io as authorized JavaScript origins.

Add the below three links as authorized javascript origins

http://13.127.162.197.xip.io/login

http://13.127.162.197.xip.io/gconnect

http://13.127.162.197.xip.io/callback

Download the corresponding JSON file, open it et copy the contents.

Open /var/www/FlaskApp/FlaskApp/client_secret.json and paste the previous contents into the this file.

Replace the client ID to line 25 of the templates/login.html file in the project directory.

Final Step
Restart your apache2 server

sudo service apache2 restart

Login to the grader
First open puttygen and then load the test1.pem to genarate private key

It is saved as privatekey1.ppk

Now open putty give the static ip:13.127.162.197 port:2200

now go to ssh and in ssh goto to auth load the privatekey1

login as : ubuntu

su - grader

password for grader:prav123

Making .ssh folder
mkdir .ssh

creates .ssh directory

touch .ssh/authorized_keys

create authorized_keys in .ssh folder

nano .ssh/authorized_keys

In git bash ssh-keygen

.ssh file will be created with id_rsa files

now copy the contents of id_rsa.pub file (microsoft office file) and paste it in the grader.

Test1.pem file
  -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAt+mtvz9ZghPkwYtGzYkf+YmxulNfFfxg3fQZY7WcQO8BjNN3 VvsbAWVE7+QmpZmaRjvnCoBSG0EIK1AgdK6H1yXXcex0xZxeuKyhUuypQkfaIKgv zXoVCacpa85gweUSgq+QYQ15fq0N1wCORa1rmOYkltFv0brCHlu04MYvLknkMs8l HU+5LCFkOVEy2ANxy2BMvU8Q9AbF+Gty6my77faoMvfSnmISlyUmc7cyfjHmkPCo LoAofECx/IazbytozfNAc/7jSrXU7xzE2i7KIWgwbV85JjFxadp/zbp3PaTwGq4F CfBH5IWzHEgLWaZ5MJvGZjO5F8Z/khzhGKuLzwIDAQABAoIBAFUI6NsKkXpBdH3A xgX2pyAb+F8seUSTIr69RJgDurGTUOYqSH2hMQVeK5e3p97dvKVIwTTrzArp8LsG G1uX7xsdVhZIvF06RdmhiB3tav1Id6St3xxknCGQduhvzfEY14wxXNJjBo/5t/J3 QVEaNCvIDZbmU4tnjKW4xVNAj0QZAGILSMXXt3oSCHPoSOPveGXRFRbWKB4Omo4f FWiVubO9/UDWmWj2IeEDOEHo9sRFpI0n8KKtftrbqpOzGUvl6dQomBGLxOrmf2kj H1pxsL9L+qEcPrsoKQ32CuIRpEWhFTZZo91rSqqp6zp30SuuMIhAUOoASkwS/O/w MRFeMEECgYEA8dteD/Igws7zfpj7Z6DWrOxxdn7LuEkV/06RXfKAfzm78Jr5xcYq YFvAOtuYoy/1ieB//VIaquEVp3JPcoNGdNG5pmysu8jUb4V66MJB/YDExugQ+4yU +SWeT9FNvves41RuwtcrxVAwmVkgKADW1V+j0nHH7h/MMmORj/hVThECgYEAwqrh PDlbBHlzjqBYhjmWiey5lo/KvkPZMzwYQ+jxXwrYBA+FHon9g6NzcMwZjXy7UON/ Vd2t8nW4jgR7cJQJ8SVhqVpEf3tuT5or7zUHaxNCMJ7p169ZfW+VG2Ne/zYXdBoM Amvz53xz3sXrA0TPmrSO6futKcWzFz14oz9U298CgYA8bDaquy4OHU/d3/BnKlqX pxaNqQ3SQ4gYWZOdqfkKT+0xJjaif2iU3DdBPR18H34zbP/s1LdO257iT3+jt0JB 6yd7eYkJ/Rl9pxZW0jlUUPhYTR/5CF0rhYdwn3TR8eSigrSNPt5zlB4gIZEUDWme sx8lc0GkrxL/v7pdAoilUQKBgQC5XUfuNdtSbme35z2ESm/rU+wAz1lKRYccP1wH xleYndXGQBUNWG57m/e/78lhLeWcB5Tn6ZfKaYhcSy5Tq9OvuV2+ikLxdVI8IF03 gTJYJlV/wMKA6+r2A3tjQgNiV1qL5oWLBMqSobIf7ixzx2E8OjRf35QrU6LOPW2T XSnr1wKBgQC+FNJD/jfuDC4ObFyyCvkkKoT9bgo2J6DRwvXsKilgX6l7rv5hKFgD R/0aNdD9bTVtYL8X8uUGTHxLfqUtqIySS996Mx593sUQKMpPx+mKEchQOyZYDSLP AE7s0V5C5P0CeETDTjxI/5JrRNmrsJob9B+ilH9uF8DLo0a+K7/IhA== -----END RSA PRIVATE KEY-----

Privatekey1.ppk file
PuTTY-User-Key-File-2: ssh-rsa Encryption: none Comment: imported-openssh-key Public-Lines: 6 AAAAB3NzaC1yc2EAAAADAQABAAABAQC36a2/P1mCE+TBi0bNiR/5ibG6U18V/GDd 9BljtZxA7wGM03dW+xsBZUTv5CalmZpGO+cKgFIbQQgrUCB0rofXJddx7HTFnF64 rKFS7KlCR9ogqC/NehUJpylrzmDB5RKCr5BhDXl+rQ3XAI5FrWuY5iSW0W/RusIe W7Tgxi8uSeQyzyUdT7ksIWQ5UTLYA3HLYEy9TxD0BsX4a3LqbLvt9qgy99KeYhKX JSZztzJ+MeaQ8KgugCh8QLH8hrNvK2jN80Bz/uNKtdTvHMTaLsohaDBtXzkmMXFp 2n/Nunc9pPAargUJ8EfkhbMcSAtZpnkwm8ZmM7kXxn+SHOEYq4vP Private-Lines: 14 AAABAFUI6NsKkXpBdH3AxgX2pyAb+F8seUSTIr69RJgDurGTUOYqSH2hMQVeK5e3 p97dvKVIwTTrzArp8LsGG1uX7xsdVhZIvF06RdmhiB3tav1Id6St3xxknCGQduhv zfEY14wxXNJjBo/5t/J3QVEaNCvIDZbmU4tnjKW4xVNAj0QZAGILSMXXt3oSCHPo SOPveGXRFRbWKB4Omo4fFWiVubO9/UDWmWj2IeEDOEHo9sRFpI0n8KKtftrbqpOz GUvl6dQomBGLxOrmf2kjH1pxsL9L+qEcPrsoKQ32CuIRpEWhFTZZo91rSqqp6zp3 0SuuMIhAUOoASkwS/O/wMRFeMEEAAACBAPHbXg/yIMLO836Y+2eg1qzscXZ+y7hJ Ff9OkV3ygH85u/Ca+cXGKmBbwDrbmKMv9Yngf/1SGqrhFadyT3KDRnTRuaZsrLvI 1G+FeujCQf2AxMboEPuMlPklnk/RTb73rONUbsLXK8VQMJlZICgA1tVfo9Jxx+4f zDJjkY/4VU4RAAAAgQDCquE8OVsEeXOOoFiGOZaJ7LmWj8q+Q9kzPBhD6PFfCtgE D4Ueif2Do3NwzBmNfLtQ439V3a3ydbiOBHtwlAnxJWGpWkR/e25PmivvNQdrE0Iw nunXr1l9b5UbY17/Nhd0GgwCa/PnfHPexesDRM+atI7p+60pxbMXPXijP1Tb3wAA AIEAvhTSQ/437gwuDmxcsgr5JCqE/W4KNieg0cL17CopYF+pe67+YShYA0f9GjXQ /W01bWC/F/LlBkx8S36lLaiMkkvfejMefd7FECjKT8fpihHIUDsmWA0izwBO7NFe QuT9AnhEw048SP+Sa0TZq7CaG/QfopR/bhfAy6NGviu/yIQ= Private-MAC: 21649b6f28663d9b50ab593ff2bbf85c3cb49ecd

id_rsa file
-----BEGIN RSA PRIVATE KEY----- MIIEpAIBAAKCAQEA3Y3Jgm0b7Meb6jUqcXxSchgkC4SgxQFL9BnCU377zQRWL52E IGFj/mQtYx51NXpF1453oxWmQcuhYm/lrTmX6lXTKlCY8Rop/t49ohw1LTIzO02a BNN66b6S5cnSDDQkJPz76ZY9rCHehAwU4qrQ+2x3maCDm9XdTUzDhkDCN/gIkhR3 m7SNgxZsoh9SclcBbRxL2DCEWP1XlS7qkaOuEEc3aHYbOxLUrDcRT2QNws95A5FT bal0OoifcozceABzH7bwFIp9R/8diPdWgb7X1gtAaxx+08SsxHGGbOwHQQvGya4c t+vfvKD4TwBH3Wz0JN6OOt8nxKZhBRJUgzCyowIDAQABAoIBAQCCYtRJnr7FbX+z qRBOc3Tt0ok8AhS/fBDgGlzD2MKFmFzEbvX4PHilmP0hDSa7jzcVMbH012Pd6uIS VXocpP1jmpjImWA43EYEyXQKOIGPFLET4IqY9oCqmU5IT6r/JZxz7DKw+ItZByG2 IooUD6UnomEMl2t8xbMUr+2N0AS8/HHn0ZXaTguyKYfocI23vFqnvB/vd7DH/+1m AQsgzk9vzKDaixqk87rVrhVRBrlYJxs84Wsrpe9wh85BXLF7UJl8PdQlYdAvYFo7 GJ9ApXJVIq9cZ/oX9dI8knIcXx0hwyFcUwqu2/Zgm5Dl7ZTijYZzfUpOQS7ZGXFT yPR+nc/JAoGBAP2egRba7o1Fh9UhrdFj4nqWL1iLflvvSQH0we78U2Z6+ZP3/XEz jd7VVwbqzDSRqlLnSFBiqIONieiDfayUfkQp79ic7X8PIg76f+Z3LanySGtXqNqX Komflhj7vwgozeSbKAHlVLgL1oWeUwToEzW1CROl/2abACzFO9aGrkt9AoGBAN+i OUo56g/uFdKl9jt1+PKvMEhtexXSzcejgKfA0bcLlsP8//RE2NNyru4adz4mAtgK c+BQvG8z1U/ClrYvLMs9cLIr8J1VfL5M0QOyQfzoddPyb03RRmeMBnQe3q2DuhiV /8TvQo1urqCIIc2gkVAJNRDxXR2PknK9T3iemBCfAoGAOl2Bw6q6f1RjAA9yGt/r vN5XIv9nIu0cVhDnmIIFB26BhSXq82Qz3/YkwvjB6c8j7VRAdg9eTHgEJrgr8c1y iWlFQwpKNZVb9yp6yKTgo12YgOt5dmSXa3iIaFN3EP+i8lsqh9Uo+/ntcm6PGG/4 C8IPxq/K6Of+0E7LiIESF1kCgYAEx9oDrTNEvek3hJa79Kyqu3YDxltgPw5pGySv 8V5KVHQXonvyQ9qCwqvcqTItYPCkn6yUegVdNzVzLAy+vklt2tVi6a+OHXq6Mrgx JQlmF64VJtScTEH+Cq9EojhOa0d6KRK7qgW5MkmHYwGd/LW+iUPaJd7AuAYAb+2X /YPwrwKBgQCY4OBd+gFYNpoLbbilwbsVIutYYRagSouBpE0iOZ1DQhAcMouc2Wa5 Q1jLRsvDz+Xn5+zmu6K3zCapntaNns+kHN49UVc77lo2RgI7WysMJRu1QuttTWTw IJEq6Z3fkHIxa9xg11UQmeTB1+AvIiF1lIs+IkdfMbEHdnyNFpxqbw== -----END RSA PRIVATE KEY-----

id_rsa.pub file
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDdjcmCbRvsx5vqNSpxfFJyGCQLhKDFAUv0GcJTfvvNBFYvnYQgYWP+ZC1jHnU1ekXXjnejFaZBy6Fib+WtOZfqVdMqUJjxGin+3j2iHDUtMjM7TZoE03rpvpLlydIMNCQk/Pvplj2sId6EDBTiqtD7bHeZoIOb1d1NTMOGQMI3+AiSFHebtI2DFmyiH1JyVwFtHEvYMIRY/VeVLuqRo64QRzdodhs7EtSsNxFPZA3Cz3kDkVNtqXQ6iJ9yjNx4AHMftvAUin1H/x2I91aBvtfWC0BrHH7TxKzEcYZs7AdBC8bJrhy369+8oPhPAEfdbPQk3o463yfEpmEFElSDMLKj admin@admin-PC

References
https://github.com/rrjoson/udacity-linux-server-configuration

https://github.com/boisalai/udacity-linux-server-configuration

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

stackoverflow website
