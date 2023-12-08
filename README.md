## Hello 👋! There you will learn how to deploy the project with nginx

### Firstly we will create folders where our projects will be stored
```
mkdir var
```

```
cd /var
```

```
mkdir www
```

```
cd www
```

#### Get update

```
apt-get update -y
```

#### Install Required Packages
```
apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx -y
```

#### Configure PostgreSQL
```
sudo -u postgres psql
```

#### And Next, setting the default encoding to UTF-8, setting the default transaction isolation scheme to "read committed" and setting the timezone to UTC with the following command:
```
ALTER ROLE testuser SET client_encoding TO 'utf8';
ALTER ROLE testuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE testuser SET timezone TO 'UTC';
```

#### Next, grant all the privileges to your database and exit from the PostgreSQL session with the following command: 

```
postgres=# GRANT ALL PRIVILEGES ON DATABASE project_name TO testuser;
postgres=#  \q
```

#### Next, create a PostgreSQL database and user for your Django project using the following command:
```
CREATE DATABASE project_name;
```
#### Create user for postgres
```
CREATE USER user WITH PASSWORD 'password'; 
```

### And now we will clone our project to server

#### Download git

```
apt install git
```

#### Clone the project

```
git clone git@github.com:username/project_name.git
```

#### Enter to the project

```
cd project_name
```

### And let's install packages

#### Install virtual environment to project
```
pip install virtualenv
```

#### Create virtual environment
```
python3 -m venv .env
```

#### Activate it
```
. .env/bin/activate
```
#### Or like that
```
source env/bin/activate
```

#### Install all packages from requirements.txt

```
pip install -r requirements.txt
```

#### Make migrations of your project
```
py manage.py makemigrations
```

```
py manage.py migrate
```

#### Create Super User

```
py manage.py createsuperuser
```

<pre>
Username (leave blank to use 'root'): <br>
Email address:<br> 
Password: <br> 
Password (again): <br>
Superuser created successfully.
</pre>

#### Collect Static files

```
py manage.py collectstatic
```

### After collecting static files we can start deploying our website 👌

#### Create a Systemd Service file for Gunicorn
```
nano /etc/systemd/system/gunicorn.service
```
#### Add the following lines:
<pre>
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/var/www/project_name
ExecStart=/var/www/project_name/.env/bin/gunicorn --access-logfile - --workers 3 --bind unix:/var/www/project_name/project.sock config.wsgi:application

[Install]
WantedBy=multi-user.target
</pre>

#### Start gunicorn and enable it
```
systemctl start gunicorn
systemctl enable gunicorn
```

#### You can check the status of Gunicorn with the following command:
```
systemctl status gunicorn
```

#### Output:
<pre>
 gunicorn.service - gunicorn daemon
 Loaded: loaded (/etc/systemd/system/gunicorn.service; enabled; vendor preset: enabled)
 Active: active (running) since Mon 2018-09-10 20:58:14 IST; 10s ago
 Main PID: 2377 (gunicorn)
   CGroup: /system.slice/gunicorn.service
           ├─2377 /root/testproject/.env/bin/python3 /root/project_name/.env/bin/gunicorn --access-logfile - --workers 3 --b
           ├─2384 /root/testproject/.env/bin/python3 /root/project_name/.env/bin/gunicorn --access-logfile - --workers 3 --b
           ├─2385 /root/testproject/.env/bin/python3 /root/project_name/.env/bin/gunicorn --access-logfile - --workers 3 --b
           └─2386 /root/testproject/.env/bin/python3 /root/project_name/.env/bin/gunicorn --access-logfile - --workers 3 --b

Sep 10 20:58:14 mail.example.com systemd[1]: Started gunicorn daemon.
Sep 10 20:58:14 mail.example.com gunicorn[2377]: [2018-09-10 20:58:14 +0530] [2377] [INFO] Starting gunicorn 19.9.0
Sep 10 20:58:14 mail.example.com gunicorn[2377]: [2018-09-10 20:58:14 +0530] [2377] [INFO] Listening at: unix:/root/project_name/project_name.soc
Sep 10 20:58:14 mail.example.com gunicorn[2377]: [2018-09-10 20:58:14 +0530] [2377] [INFO] Using worker: sync
Sep 10 20:58:14 mail.example.com gunicorn[2377]: [2018-09-10 20:58:14 +0530] [2384] [INFO] Booting worker with pid: 2384
Sep 10 20:58:15 mail.example.com gunicorn[2377]: [2018-09-10 20:58:15 +0530] [2385] [INFO] Booting worker with pid: 2385
Sep 10 20:58:15 mail.example.com gunicorn[2377]: [2018-09-10 20:58:15 +0530] [2386] [INFO] Booting worker with pid: 2386
</pre>

### Configure Nginx to Proxy Pass to Gunicorn
```
nano /etc/nginx/sites-available/gunicorn
```
<pre>
server {
    listen 80;
    server_name example.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /var/www/project_name;
    }

    location /media/ {
        root /var/www/project_name;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/project_name/project.sock;
    }
}
</pre>

#### Save and close the file. Then, enable the Nginx virtual host by creating symlink:
```
ln -s /etc/nginx/sites-available/gunicorn /etc/nginx/sites-enabled/
```

#### Check is it works correctly
```
nginx -t
```
#### Output:
<pre>
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
</pre>

#### Finally, restart Nginx and gunicorn by running the following command:
```
systemctl daemon-reload
systemctl restart gunicorn
systemctl restart nginx
```

<hr>
<br>

<h3>This was an example of how you can deploy your Django project, there are many more ways but this is the best one that can be, good luck in development 🍀</h3>
