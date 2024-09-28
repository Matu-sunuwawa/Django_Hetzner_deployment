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





















