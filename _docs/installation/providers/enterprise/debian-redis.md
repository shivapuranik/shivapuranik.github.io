---
layout: single
type: docs
permalink: /docs/installation/providers/enterprise/debian-redis/
redirect_from:
  - /theme-setup/
last_modified_at: 2020-06-09
toc: true
title: Install and configure Redis, Supervisor and Worker for Faveo on Debian
---

<img alt="debian" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4a/Debian-OpenLogo.svg/109px-Debian-OpenLogo.svg.png" width="120" height="120" />

## Introduction
This document will list steps on how to install Redis, Supervisor and Worker for faveo.

We will install following dependencies in order to make Redis work:

- Redis
- PHP extension for Redis
- supervisor

Switch to root user or run the following commands as sudoers.

```sh
sudo su
```

##  Install Redis and PHP redis extension.
```
apt-get install redis-server
```

## Start, Enable and restart the Redis-service
```
systemctl start redis-server

systemctl enable redis-server
```

## Install and Configure Supervisor
```
apt-get install supervisor
```
## Copy paste the below configuration.( Change the directories according to your configuration)

```
nano /etc/supervisor/conf.d/faveo-worker.conf
```
```
[program:faveo-Horizon]
process_name=%(program_name)s
command=php /var/www/faveo/artisan horizon
autostart=true
autorestart=true
user=www-data
redirect_stderr=true
stdout_logfile=/var/www/faveo/storage/logs/horizon-worker.log

```
## Restart the Supervisor to reread configuration

```sh
systemctl restart supervisor
```


To check the Status of workers use the below command
```sh
supervisorctl
```

## Enable Redis in Faveo
After Redis installation is complete, follow these [instructions](/docs/helper/enable-redis) to configure Redis with Faveo. 
