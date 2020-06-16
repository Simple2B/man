# Deploy flask app using gunicorn
This instruction for deploy Flask application.

**Prerequisites:**
* OS: Ubuntu 18.*
* Web Server: Nginx
* Python3
* Flask
* Gunicorn

## Update your local package index and then install the packages, nginx enabling.

```
# update your local packages
1.sudo apt-get update
# install dependencies
2. sudo apt-get install python3-pip python3-dev nginx virtualenv
```
After it’s installed, we can enable Nginx to auto start when Ubuntu is booted by running the following command.
```
sudo systemctl enable nginx
```
Then start Nginx with this command:
```
sudo systemctl start nginx
```
Now check out its status.
```
systemctl status nginx
```
Output:

```
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-05-17 02:20:05 UTC; 2min 56s ago
     Docs: man:nginx(8)
 Main PID: 19851 (nginx)
    Tasks: 2 (limit: 2059)
   CGroup: /system.slice/nginx.service
           ├─19851 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─19853 nginx: worker process
```



## Prepare environment

Now, create python virtual environment:
```
cd ~/myproject
virtualenv -p python3 .venv
source .venv/bin/activate
pip install -r requirements.txt
```


## Configure WSGI
We use gunicorn service from python virtual environment.

### Create Linux Service

```
sudo nano /etc/systemd/system/app.service
```

```ini
[Unit]
#  specifies metadata and dependencies
Description=Gunicorn instance to serve myproject
After=network.target
# tells the init system to only start this after the networking target has been reached
# We will give our regular user account ownership of the process since it owns all of the relevant files

[Service]
# Service specify the user and group under which our process will run.
User=username
# give group ownership to the www-data group so that Nginx can communicate easily with the Gunicorn processes.
Group=www-data
# We'll then map out the working directory and set the PATH environmental variable so that the init system knows where our the executables for the process are located (within our virtual environment).
WorkingDirectory=/home/tasnuva/work/deployment/src
Environment="PATH=/home/username/workspace/myproject/myprojectvenv/bin"
# We'll then specify the commanded to start the service
ExecStart=/home/username/workspace/myproject/myprojectvenv/bin/gunicorn --workers 3 --bind unix:app.sock -m 007 wsgi:app
# This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:

[Install]
WantedBy=multi-user.target
```

Note: In the last line of [Service] We tell it to start 3 worker processes. We will also tell it to create and bind to a Unix socket file within our project directory called app.sock. We’ll set a umask value of 007 so that the socket file is created giving access to the owner and group, while restricting other access. Finally, we need to pass in the WSGI entry point file name and the Python callable within.
We can now start the Gunicorn service we created and enable it so that it starts at boot:

Run service:

```
sudo systemctl start app

sudo systemctl enable app
```
A new file app.sock will be created in the project directory automatically.


#### Attention! This file content for:
* User: `username`
* Project folder `home/username/workspace/myproject`
You need fix it if you have another user name or/and project folder



##  Configuring Nginx

create a new server block configuration file in Nginx’s sites-available directory named app

```
sudo nano /etc/nginx/sites-available/app

```
```
server {
    listen 80;
    server_name server_domain_or_IP;

location / {
  include proxy_params;
  proxy_pass http://unix:/home/tasnuva/work/deployment/src/app.sock;
    }
}
```

Activate this Nginx configuration:
```
sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/
```
Check if our configuration file is OK:
```
sudo nginx -t
```
If this does not indicate any issues, we can restart the Nginx process to read our new config:
```
sudo systemctl restart nginx
```
The last thing we need to do is adjust our firewall to allow access to the Nginx server:
```
sudo ufw allow 'Nginx Full'
```
You should now be able to go to your server’s domain name or IP address in your web browser and see your app running.
```
http://server_domain_or_IP
```

# Install MariaDB Database Server

Install packages:
```
sudo apt install mariadb-server mariadb-client
```
Check status:
```
systemctl status mariadb
```
Output:

```
● mariadb.service - MariaDB database server
   Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-05-17 02:39:57 UTC; 49s ago
 Main PID: 21595 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 27 (limit: 2059)
   CGroup: /system.slice/mariadb.service
           └─21595 /usr/sbin/mysqld
```
If it’s not running, start it with this command:
```
sudo systemctl start mariadb
```
To enable MariaDB to automatically start at boot time, run
```
sudo systemctl enable mariadb
```
Now run the post installation security script.
```
sudo mysql_secure_installation
```
Run:
```
sudo mariadb -u root
```
To exit, run
```
exit;
```

# Https certification using certbot

Add Certbot PPA
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```
Install Certbot
```
sudo apt-get install certbot python3-certbot-nginx
```
Run:
```
sudo certbot --nginx
```


## Top links

### Deploy flask app
https://medium.com/faun/deploy-flask-app-with-nginx-using-gunicorn-7fda4f50066a

### Certbot certification
https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx

### Mariadb installation
https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-ubuntu-18-04-ru