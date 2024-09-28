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



















