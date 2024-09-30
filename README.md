# 🚀Django_Hetzner_deployment

<p>In this article, we will learn how to deploy a Django application with Nginx, Gunicorn, PostgreSQL</p>

## Production Stack Architecture

* OS - Ubuntu
* WSGI Server - Gunicorn
* Web Server - Nginx
* Database - PostgreSQL

## The following diagram illustrates how Django works in the production environment

<p align="center">
    <img src="https://djangocentral.com/media/uploads/django_nginx_gunicorn.png"/>
</p>

## Log in to the server
```
ssh root@IP_Address
```
```
sudo apt update
sudo apt upgrade
```

## Create a system user
```
adduser master
```
```
usermod -aG sudo master
```
<p>We will also add user master to our www-data group(Optional)</p>

```
usermod -aG www-data master
```
<p>Now, we can log in as the new user ‘master’</p>

```
su - master
```

## Install Packages
```
sudo apt-get install postgresql postgresql-contrib libpq-dev python3-dev
```

## Setting up PostgreSQL
```
sudo -u postgres psql
```
```
CREATE USER database_user WITH ENCRYPTED PASSWORD 'some_password';
CREATE DATABASE database_name OWNER database_user;
ALTER ROLE database_user SET client_encoding TO 'utf8';
ALTER ROLE database_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE database_user SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE database_name TO database_user;
\q
```

## Creating a Virtual Environment
```
sudo apt install python3-venv
```
```
mkdir dir_name
 
cd dir_name
```
```
python3 -m venv venv_name
 
source venv_name/bin/activate
```

## Settings up the project
```
git clone git_url
```
```
pip install gunicorn psycopg2-binary
```
```
pip install -r requirements.txt
```

### changes in <mark>settings.py</mark> file to make it deployment-ready
```
ALLOWED_HOSTS = ['Ip_Adress', 'domain_name.com', 'www.domain_name.com']
```
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djangodb',
        'USER': 'djangouser',
        'PASSWORD': 'm0d1fyth15',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

<p>Luckily Django comes with a utility to run checks for a production-ready application run the following command in your terminal.</p>

```
python manage.py check --deploy
```

- You will see an output with no errors but several warnings. This means the check was successful, but you should go through the warnings to see if there is anything more you can do to make your project safe for production.
  
```
python manage.py makemigrations
 
python manage.py migrate
```

<p>We can test the app by running the local development server but first, we need to create an exception at port 8000.</p>

```
sudo ufw allow 8000
 
python manage.py runserver 0.0.0.0:8000
```

## In your web browser navigate to <mark>http://server_domain_or_IP:8000</mark> and you should see the app.

## Setting up Gunicorn Server
<p>Django's primary deployment platform is WSGI. WSGI stands for Web Server Gateway Interface and it is the standard for serving Python applications on the web.

When you generate a new project using the start project command, Django creates a wsgi.py file inside your project directory. This file contains a WSGI application callable, which is an access point to your application. WSGI is used for both running your project with the Django development server, and deploying your application with the server of your choice in a production environment.

You can test if Gunicorn is able to serve the project as follows.</p>

```
gunicorn --bind 0.0.0.0:8000 project_name.wsgi
```
<p>🎉Go back to http://server_domain_or_IP:8000 you should see the application running but without static assets such as CSS and images.</p>
<p>Once you finishing testing the app press ctrl + c to stop the process and deactivate the virtual environment.</p>

```
deactivate
```

<p>Gunicorn uses its .sock files to talks to other parts of the process. Sock files are Unix domain sockets that processes use to communicate. I personally prefer keeping the sock file in /var/log/gunicorn so it can be easily accessible by Nginx. Therefore we need to make a directory there. <mark>Reminder! continue with the directory that is you are working on(project folder name)</mark></p>

```
mkdir /var/log/gunicorn
```
<p>With that now create and open a systemd service file for Gunicorn with sudo privileges and add the below configuration. <mark>Reminder! unix_user_name, /path/to/the/project/directory, /path/to/virtual-env/bin/gunicorn, project_name.sock project_name.wsgi:application</mark></p>

- <mark>unix_user_name</mark>- By default it's <mark>root</mark>

```
sudo nano /etc/systemd/system/gunicorn.service
```
```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=unix_user_name
Group=www-data
WorkingDirectory=/path/to/the/project/directory
ExecStart=/path/to/virtual-env/bin/gunicorn --access-logfile - --workers 3 --bind unix:/var/log/gunicorn/project_name.sock project_name.wsgi:application

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl start gunicorn
 
sudo systemctl enable gunicorn
```
```
sudo systemctl status gunicorn
```

<p>If the output indicates an error has occurred you must have misconfigured something so check the logs to find it out. To see the Gunicorn logs run the following command.</p>

```
sudo journalctl -u gunicorn
```
<p>In case you had to make changes in the service file reload the daemon to reread the new service definition and restart the Gunicorn process by these commands.</p>

```
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

## Setting up Nginx

```
sudo apt-get install nginx
```
<p>Since Nginx is going to serve our static assets it's recommended to set the <mark>STATIC_ROOT</mark> to <mark>/var/www/static/</mark> so that it can be easily accessible to Nginx.

So in your project's <mark>settings.py</mark> modify the <mark>STATIC_ROOT</mark> as shown below.</p>

<h3>Notice That!!!</h3>

```
  288  source env/bin/activate
  289  cd Django_Hetzner_deployment/
  290  sudo mkdir /var/www/static
  292  sudo chown -R www-data:www-data /var/www/static
  293  python manage.py collectstatic
```
```
location /media/ {
    root /var/www;
}
location  /static/ {
    root /var/www;
}
```
```
STATIC_URL = 'static/'
STATIC_ROOT = '/var/www/static'

MEDIA_URL = 'media/'
MEDIA_ROOT = '/var/www/media'
```
<p align="center">--//--</p>

```
STATIC_ROOT = '/var/www/static'
```
```
python manage.py collectstatic
```

```
sudo nano /etc/nginx/sites-available/project_name
```

- <mark>server_domain_or_IP</mark> -Type <mark>domain name</mark> or <mark>server IP</mark>.
- <mark>/path/to/staticfiles</mark>: If you are following the guide then it should be <mark>/var/www</mark>

```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location  /static/ {
        root /path/to/staticfiles;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/log/gunicorn/project_name.sock;
    }
}
```
```
sudo ln -s /etc/nginx/sites-available/project_name /etc/nginx/sites-enabled
```

## 🧪Test your Nginx configuration for syntax errors by the following command
```
sudo nginx -t
```
<p>👌If the test was was successful go forward and restart the Nginx server so that changes can take place.</p>

```
sudo systemctl restart nginx
```

<p>Finally, we need to open up our firewall to normal traffic on port 80. Since we no longer need access to the development server, we can remove the rule to open port 8000 as well.</p>

```
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
```

## Now in your browser navigate to <mark>http://domain_name_or_server_IP</mark> the application should be running here.

## Troubleshooting Nginx
<p>In case you don't see your application running that means there must be some misconfiguration in the server block so go through the Nginx logs and solve the issue.

Run the following command to access Nginx logs.
</p>

```
sudo tail -F /var/log/nginx/error.log
```


## Securing the application with SSL
<p>Let's Encrypt is a free Certificate Authority (CA) that issues SSL certificates. You can use these SSL certificates to secure traffic on your Django application. Lets Encrypt has an automated installer called Certbot with Certbot you can very easily add a certificate to your site in just a couple of minutes</p>

```
sudo apt-get update
sudo apt-get install python3-certbot-nginx
```

## Configuring Nginx for Certbot
<p>Certbot can automatically configure SSL for Nginx, but it needs to be able to find the correct server block in your config. It does this by looking for a server_name directive that matches the domain you're requesting a certificate for so make sure you have set the correct domain in the /etc/nginx/sites-available/project_namefile.</p>

```
. . .
server_name example.com www.example.com;
. . .
```

In case you made changes to this server block reload Nginx.
```
sudo systemctl reload nginx
```

Next, we need to configure ufw firewall to allow HTTPS traffic.So first enable ufw firewall if it's not already.
```
sudo ufw allow ssh
 
sudo ufw enable 
```
- Enter y for confirmation next add following rules to the firewall.
```
sudo ufw allow 'Nginx Full'
 
sudo ufw delete allow 'Nginx HTTP'
```
```
sudo ufw status
```
### output:
```
Status: active

To                         Action      From
--                         ------      ----
Nginx Full                 ALLOW       Anywhere
22/tcp                     ALLOW       Anywhere
Nginx Full (v6)            ALLOW       Anywhere (v6)
22/tcp (v6)                ALLOW       Anywhere (v6)
```

## Now we can obtain the <code>SSL</code> certificate with the following command
```
sudo certbot --nginx -d example.com -d www.example.com
```
But, "Certbot not found":
```
sudo apt update

sudo apt install snapd
```
```
sudo snap install core
```
```
sudo snap install --classic certbot

sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
Now, try again <code>sudo certbot --nginx -d example.com -d www.example.com</code>

You will be asked a series of questions for the setup:
```
ANSWER Y,Y,Y....for YES,YES,YES
```

## Renewing SSL certificates
```
sudo certbot renew --dry-run
```
- If you see no errors, you’re all set. When necessary, Certbot will renew your certificates and reload Nginx to pick up the changes.




















