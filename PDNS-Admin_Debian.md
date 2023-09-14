# PowerDNS-Admin installation guide:

Author: Daan Selen.

Thanks to this [HowToForge Tutorial](https://www.howtoforge.com/how-to-install-powerdns-admin-on-ubuntu-20-04/).

<br/>

---

In this tutorial I will show you how to install [PowerDNS-Admin](https://github.com/PowerDNS-Admin/PowerDNS-Admin).  
In the case of this tutorial I will use a freshly installed and updated Debian GNU/LINUX 11.

## **Prerequisites:**

* An installed instance of PowerDNS running and access to a corresponding REST API key.
* Installed and configured the Sudo command.
* Installed the curl command.
* Configured internet acces the way you need, and prefferably already set a hostname. <br/>

## **Installation:**

### **0\. Database server and Database configuration.**

If you have already set up a Database(-server), you can skip this step.  
In my case I am going to use MariaDB as my SQL database server.

First we make sure the system is up to date:

```
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

<br/>

After that we install MariaDB and configure the application as needed:

```
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```

After the previous command you get a couple questions fill them in to the circumstances you desire.

<br/>

Now we can check if the installation of MariaDB has been succesful with a couple commands:

```
sudo systemctl status mariadb
```

or

```
sudo service mariadb status
```

<br/>

You should get an output like this:

```
root@PDNSAdmin:~$ sudo systemctl status mariadb 
mariadb.service - MariaDB 10.5.15 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-11-14 06:25:18 EST; 3min 45s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 1548 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysqld (code=exited, status=0/SUCCESS)
    Process: 1552 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 1556 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`cd /usr/bin/..; /usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl se>
    Process: 1619 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 1621 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
   Main PID: 1604 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 9 (limit: 4678)
     Memory: 73.0M
        CPU: 463ms
     CGroup: /system.slice/mariadb.service
             \u2514\u25001604 /usr/sbin/mariadbd

Nov 14 06:25:18 PDNSAdmin mariadbd[1604]: Version: '10.5.15-MariaDB-0+deb11u1'  socket: '/run/mysqld/mysqld.sock'  port: 3306  Debian 11
Nov 14 06:25:18 PDNSAdmin systemd[1]: Started MariaDB 10.5.15 database server.
```

<br/>

Now you should be able to access the MariaDB Server with:

```
mysql -u root -p
```

And then fill in the password.

<br/>

You should get an output like this if succesful:

```
root@PDNSAdmin:~$ mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 40
Server version: 10.5.15-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

<br/>

Next we need to make a database and a user for the PowerDNS-Admin,  
in my case I made a database with the name 'pda' and a user called 'pdaadmin' with password: 'password' (Not recommended). using the following commands:

```
MariaDB [(none)]> create database pda;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT ALL ON pda.* to 'pdaadmin'@'localhost' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> exit;
Bye
```

<br/>

### **1\. PowerDNS-Admin installation**

Now with the Database server application running and configured (enough) we will continue to the installation of PowerDNS-Admin itself. I am going to use the root account for the entire process.
First we install all the dependencies using:

```
sudo -i
apt install nginx python3-dev libsasl2-dev libldap2-dev libssl-dev libxml2-dev libxslt1-dev libxmlsec1-dev libffi-dev pkg-config apt-transport-https virtualenv build-essential libmariadb-dev git python3-flask -y
```

<br/>

Once all those packages are installed we can add the Node.js repository with the following bash script, be mindful of the version you are installing:

```
curl -sL https://deb.nodesource.com/setup_16.x | bash -
```

<br/>

Then install Node.js itself with the following command:

```
apt install nodejs -y
```

<br/>

The next repository we need to add manually with the following commands:

```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
apt update
```

<br/>

After that we can install yarn itself through this command:

```
apt install yarn -y
```

<br/>

Now all dependencies are installed, we can continue to the actual PowerDNS-Admin application itself.  
First we need to download the repository from GitHub and we are going to download it to the /var/www/html/pda directory.

```
git clone https://github.com/ngoduykhanh/PowerDNS-Admin.git /var/www/html/pda
```

<br/>

Next we can change our current working directory to the location where we just installed the PowerDNS-Admin repository and create a virutal environment:

```
cd /var/www/html/pda
virtualenv -p python3 flask
```

<br/>

Next we activate the said environment and install the python dependencies, after that we deactivate the virtual environment:

```
source ./flask/bin/activate
pip3 install -r requirements.txt
deactivate
```

<br/>

Next we are going to configure the database connection between PowerDNS-Admin and the SQL database.

```
nano /var/www/html/pda/powerdnsadmin/default_config.py
```

And change the following lines to your liking in my case I have this (Be sure to randomize your SALT and SECRET_KEY or use a strong key, I didn't do it in this case):

```
### BASIC APP CONFIG
SALT = '$2b$12$yLUMTIfl21FKJQpTkRQXCu'
SECRET_KEY = 'e951e5a1f4b94151b360f47edf596dd2'
BIND_ADDRESS = '0.0.0.0'
PORT = 9191
HSTS_ENABLED = False
OFFLINE_MODE = False
FILESYSTEM_SESSIONS_ENABLED = False
SESSION_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_HTTPONLY = True

### DATABASE CONFIG
SQLA_DB_USER = 'pdaadmin'
SQLA_DB_PASSWORD = 'password'
SQLA_DB_HOST = '127.0.0.1'
SQLA_DB_NAME = 'pda'
SQLALCHEMY_TRACK_MODIFICATIONS = True
```

<br/>

After you save and close that file succesfully we go back to the pda directory and activate the virtual environment again this time I will login as root:

```
cd /var/www/html/pda
source ./flask/bin/activate
```

<br/>

Now we need to update the database with the following commands and after that I will logout of the root user and deactivate the virtual environment:

```
export FLASK_APP=powerdnsadmin/__init__.py
flask db upgrade
yarn install --pure-lockfile
flask assets build
deactivate
```

<br/>

### **2\. Configure NGINX**

Now we are going to configure NGINX to use your PowerDNS-Admin interface application.
First we add our PowerDNS-Admin configuration to NGINX using:

```
nano /etc/nginx/conf.d/pdns-admin.conf
```

And add this:

```
server {
  listen	*:80;

  index                     index.html index.htm index.php;
  root                      /var/www/html/pda;
  access_log                /var/log/nginx/pdnsadmin_access.log combined;
  error_log                 /var/log/nginx/pdnsadmin_error.log;

  client_max_body_size              10m;
  client_body_buffer_size           128k;
  proxy_redirect                    off;
  proxy_connect_timeout             90;
  proxy_send_timeout                90;
  proxy_read_timeout                90;
  proxy_buffers                     32 4k;
  proxy_buffer_size                 8k;
  proxy_set_header                  Host $host;
  proxy_set_header                  X-Real-IP $remote_addr;
  proxy_set_header                  X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_headers_hash_bucket_size    64;

  location ~ ^/static/  {
    include  /etc/nginx/mime.types;
    root /var/www/html/pda/powerdnsadmin;

    location ~*  \.(jpg|jpeg|png|gif)$ {
      expires 365d;
    }

    location ~* ^.+.(css|js)$ {
      expires 7d;
    }
  }

  location / {
    proxy_pass            http://unix:/run/pdnsadmin/socket;
    proxy_read_timeout    120;
    proxy_connect_timeout 120;
    proxy_redirect        off;
  }

}
```

Save this configuration

<br/>

Now edit the file at /etc/nginx/sites-enabled/default and comment/remove EVERYTHING.  
Test the configuration we just filled in with this command and if successful you will get this response:

```
root@PDNSAdmin:/var/www/html/pda/powerdnsadmin$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

<br/>

Now change the ownership of the directory to www-data and after that restart nginx:

```
chown -R www-data:www-data /var/www/html/pda
```

```
systemctl restart nginx
```

<br/>

Now create the following Systemd services files for PowerDNS-Admin to run as a service, but first we have to create a user called pdns on the system.

```
useradd pdns
nano /etc/systemd/system/pdnsadmin.service
```

```
[Unit]
Description=PowerDNS-Admin
Requires=pdnsadmin.socket
After=network.target

[Service]
PIDFile=/run/pdnsadmin/pid
User=pdns
Group=pdns
WorkingDirectory=/var/www/html/pda
ExecStart=/var/www/html/pda/flask/bin/gunicorn --pid /run/pdnsadmin/pid --bind unix:/run/pdnsadmin/socket 'powerdnsadmin:create_app()'
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Save and close.

<br/>

Next create the socket file:

```
nano /etc/systemd/system/pdnsadmin.socket
```

```
[Unit]
Description=PowerDNS-Admin socket

[Socket]
ListenStream=/run/pdnsadmin/socket

[Install]
WantedBy=sockets.target
```

Save and close again.

<br/>

Then create the files and directories with the following commands again I quickly login as root:

```
echo "d /run/pdnsadmin 0755 pdns pdns -" >> /etc/tmpfiles.d/pdnsadmin.conf
mkdir /run/pdnsadmin/
chown -R pdns: /run/pdnsadmin/
chown -R pdns: /var/www/html/pda/powerdnsadmin/
```

<br/>

After that we do a systemctl reload and enable the newly created services while immediatly checking their status.

```
systemctl daemon-reload
systemctl enable --now pdnsadmin.service pdnsadmin.socket; systemctl status pdnsadmin.service pdnsadmin.socket
```

<br/>

After this you can use your preffered browser and navigate to the IP-address of the server and register as a user, the first user will always be an admin.
Once you login you will be prompted to fill in the URL of the PowerDNS API and it's corresponding key,  
after this the installation is finished and everything should be ready.
