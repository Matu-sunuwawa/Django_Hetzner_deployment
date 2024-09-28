# Django_Hetzner_deployment

<p>In this article, we will learn how to deploy a Django application with Nginx, Gunicorn, PostgreSQL</p>

## Production Stack Architecture

* OS - Ubuntu
* WSGI Server - Gunicorn
* Web Server - Nginx
* Database - PostgreSQL

## The following diagram illustrates how Django works in the production environment

<p align="center">
    <a href="#" target="_blank">
        <img src="https://djangocentral.com/media/uploads/django_nginx_gunicorn.png"/>
    </a>
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
<p>Go back to http://server_domain_or_IP:8000 you should see the application running but without static assets such as CSS and images.</p>
<p>Once you finishing testing the app press ctrl + c to stop the process and deactivate the virtual environment.</p>

```
deactivate
```

<p>Gunicorn uses its .sock files to talks to other parts of the process. Sock files are Unix domain sockets that processes use to communicate. I personally prefer keeping the sock file in /var/log/gunicorn so it can be easily accessible by Nginx. Therefore we need to make a directory there. <mark>Reminder! continue with the directory that is you are working on(project folder name)</mark></p>

```
mkdir /var/log/gunicorn
```
<p>With that now create and open a systemd service file for Gunicorn with sudo privileges and add the below configuration. <mark>Reminder! unix_user_name, /path/to/the/project/directory, /path/to/virtual-env/bin/gunicorn, project_name.sock project_name.wsgi:application</mark></p>

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


















