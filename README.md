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

#### Luckily Django comes with a utility to run checks for a production-ready application run the following command in your terminal.
```
python manage.py check --deploy
```
 - You will see an output with no errors but several warnings. This means the check was successful, but you should go through the warnings to see if there is anything more you can do to make your project safe for production.



















