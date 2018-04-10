# Configure Linux server project #
This project demonstrate the ability to configure a Linux server.
List of requirements can be found [here](https://review.udacity.com/#!/rubrics/7/view). This project is running on [34.214.183.242](34.214.183.242).
## Tool used ##
1. [Amazon light snail](https://aws.amazon.com/lightsail/)
2. ssh
3. git
4. ufw
5. Apache
6. dpkg-reconfigure
7. pip
8. git

## Setup Environment ##
1. Create a Light snail instance.
2. Once the instance is created, a public ip address
can be found. There is a ssh button on the amazon
light snail web client. Another way to ssh into the
Light snail instance. Download key under account. Use
```ssh -i path/to/key/ ubuntu@yourIpAddress``` to remotely
access light snail instance.
3. Update package list with ```apt-get update```. Update
new version of software with ```apt-get upgrade```.
4. Configure ufw.
    1. Enable port 2222/tcp with ```ufw allow 2222/tcp``` for
    ssh.
    2. Enable port 80/tcp with ```ufw allow 80/tcp``` for http.
    3. Enable port 123/udp with ```ufw allow 123/tcp```.
    4. Enable ufw with ```ufw enable```
    5. Check ufw rule with ```ufw status```.
4. Change ssh port from 22 to 2200.
    1. Run ```vi /etc/ssh/sshd_config```.
    2. Locate the line that says Port 22. Make sure
    read comment that you changed correct location.
    3. Change 22 to 2200. And save file.
    4. Add a custom rule under networking tab on Light Snail instance.
    5. Run ```service sshd restart```.
    6. Add ```-p 2200``` when remotely access Light snail instance.
5. Configure new user grader.
    1. Add new user with ```adduser grader```.
    2. Add user to sudo group with ```usermod -aG sudo grader```.
    3. Create SSH key pair using ssh-keygen.
        1. On the remote server side create a .ssh/ folder under /home/grader with ```mkdir .ssh```.
        2. Make a file named authorized_keys with ```touch authorized_keys``` under .ssh file.
        3. Use proper permission. ```chmod 600 authorized_keys``` and ```chmod 700 .ssh```.
        3. Switch to client side(Local machine). Generate rsa key with ```ssh-keygen -t rsa```. Make sure file is stored under .ssh folder as well.
        4. Copy public key to the server with ```ssh-copy-id ubuntu@YourIpAddress -p 2200```.
        5. Run ```ssh-add``` to add public key in the local machine.
        6. Try to remotely access server with the public key.
    4. Configure the local time zone to UTC with ```dpkg-reconfigure
    7. pip
    8. git tzdata```.
    5. Install git with ```apt-get install git-core```. Use ```git --version```
    to check if git is installed correctly.

## Deploy project ##
1. Install apache with ```apt-get install apache2```
2. Install wsgi ```apt-get install libapache2-mod-wsgi python-dev```
3. Clone project to /var/www/ ```cd /var/www & git clone https://github.com/yanlinl/item_catelog```
4. Change catalog folder owner to grader ```chown -R grader:grader catalog```.
5. Create a .wsgi file with ```nano catalog.wsgi``` and add following code.
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from items_catalogs import app as application
application.secret_key = 'supersecretkey'
```
6. Install and start virtual machine.
    1. ```pip install virtualenv```
    2. ```sudo virtualenv venv```
    3. ```source venv/bin/active```
    4. ```chmod -R 777 venv```
7. Install required packages to run items_catalog program.
    1. ```pip install Flask```
    2. ```pip install httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests```
8. Check your project. Update all the file path.
9. Configure virtual host. Run ```nano /etc/apache2/sites-available/catalog.conf``` and past following code to the file.
```
<VirtualHost *:80>
    ServerName [YOUR PUBLIC IP ADDRESS]
    ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME]
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
10. Run program on the background with ```python run_project.py&```
11. Restart apach server ```service apache2 restart```.