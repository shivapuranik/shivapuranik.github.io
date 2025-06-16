---
layout: single
type: docs
permalink: /docs/installation/providers/enterprise/centos8s-apache-ssl/
redirect_from:
  - /theme-setup/
last_modified_at: 2020-06-09
toc: true
---

# Install LetsEncrypt SSL for Faveo on CentOS-8-Stream Running Apache Web Server <!-- omit in toc -->


<img alt="CentOS-8-Stream OS Logo" src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/Centos-logo-light.svg/300px-Centos-logo-light.svg.png"  width="200"  />


## Introduction
This document will list on how to install Let’s Encrypt SSL on CentOS-8-Stream Running Apache Web Server

PS : Please replace example.com with your valid domain name which is mapped with your server

We will install following dependencies in order to make Let’s Encrypt SSL work:

- epel-release
- mod_ssl
- python-certbot-apache

## Installing dependent modules

```sh
yum install epel-release mod_ssl
```


## Downloading the LetsEncrypt for CentOS-8-Stream

```sh
yum install python3-certbot-apache
```

## Setting up the SSL certificate

Certbot will handle the SSL certificate management quite easily, it will generate a new certificate for provided domain as a parameter.

In this case, example.com will be used as the domain for which the certificate will be issued:

```sh
certbot --apache -d example.com
```

If you want to generate SSL for multiple domains or subdomains, please run this command:

```sh
certbot --apache -d example.com -d www.example.com
```

**PS :** IMPORTANT! The first domain should be your base domain, in this sample it’s example.com

## Setting up auto renewal of the certificate

Create new cron job for automatic renewal of SSL

This job can be safely scheduled to run every Monday at midnight:

Create a new `/etc/cron.d/faveo-ssl` file with:

```sh
echo "45 2 * * 6 /etc/letsencrypt/ && ./certbot renew && /bin/systemctl restart httpd.service" | sudo tee /etc/cron.d/faveo-ssl
```