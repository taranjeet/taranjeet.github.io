---
layout: post
title: Configuring uwsgi, nginx for django project
date: 2016-03-05 18:00
tag:
- django
- nginx
- uwsgi
category: blog
author: taranjeet
description: This post is about configuring nginx as a web server and uwsgi as app server for a django project.
---

This post is about setting up Nginx and uwsgi for a django project.

Some conventions that I have sticked to while doing so are written down here. First of all I will be creating a user named `ubuntu` and a group named `ubuntu`. This `ubuntu` user will be responsible for running nginx as well as uwsgi.  This is necessary so that the socket created between nginx and uwsgi does not get trapped in Permission denied error. Also, I am assuming that the server is a fresh install of Ubuntu 14.04. Let's begin with the steps:

* Create a user named `ubuntu`.

```
adduser ubuntu
gpasswd -a ubuntu sudo
```

* Install the following packages

```
sudo apt-get install python-dev
sudo pip install uwsgi
sudo apt-get -f install nginx-full
```

* Clone the repository in the home directory of `ubuntu` user. Assuming that the name of the project is explore-webserver.

```
git clone https://github.com/staranjeet/explore-webserver.git
```

* Configure uwsgi now. Create two folders

```
mkdir -p /etc/uwsgi/apps-available /etc/uwsgi/apps-enabled
```

* Create a config `ini` file for your project in `sites-available` folder. Idea is to create to config files in `available` folder and then create symlink for these in `enabled` folder. The file `explore-webserver.ini` should look something like this

```
[uwsgi]
project = explore-webserver
base = /home/ubuntu
uid = ubuntu
gid = ubuntu

chdir = %(base)/%(project)
home = %(base)/%(project)/pyenv
module = %(project).wsgi:application
env = DJANGO_SETTINGS_MODULE=settings.live

master = true
processes = 3
socket = %(base)/%(project)/%(project).sock
chmod-socket = 664
vacuum = true

```

* Create a upstart script for uwsgi in `/etc/init` folder. This script is created to automatically start the service at boot plus we can manually start/stop or restart/reload the service. Let's name this script as `uwsgi.conf`.

```
description "uWSGI application server in Emperor mode"

start on runlevel [2345]
stop on runlevel [!2345]

setuid ubuntu
setgid ubuntu

exec /usr/local/bin/uwsgi --emperor /etc/uwsgi/apps-enabled --daemonize /var/log/uwsgi/app/explorewebserver.log
```

* Configure nginx now. Nginx also follows the `sites-available` and `sites-enabled` convention. In `/etc/nginx/sites-available` folder, create a file named `explore-webserver`.

```
server{
        listen 80;
        server_name server_ip_address_or_domain_name;

        location / {
                include uwsgi_params;
                uwsgi_pass unix:/home/ubuntu/explore-webserver/explore-webserver.sock;
        }

        location /static {
                root /home/ubuntu/explore-webserver;
        }

        location /media {
                root /home/ubuntu/explore-webserver/explorewebserver;
        }
}
```

* Check whether this config is correct or not by

```
sudo nginx -t
```

* Create a soft link to this file in `sites-enabled` directory by

```
sudo ln -s /etc/nginx/sites-available/explore-webserver /etc/nginx/sites-enabled/
```

* Now change the user in `/etc/nginx/nginx.conf` from ~~`www-data`~~ to `ubuntu`

```
user ubuntu;
```

* Now start uwsgi by

```
sudo service uwsgi start
```

* Restart nginx by
```
sudo service nginx restart
```

Note: Make sure that all directories(socket and logs) are owned by the same user(here ubuntu).

