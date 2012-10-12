---
layout: post
title: "使用Ngins+SSL代理chef-server"
date: 2012-10-12 21:09
comments: true
categories:
---

## 说明

本文介绍使用Nginx做为chef-server的前端代理服务器,并使用SSL加密传输数据

这里使用的系统是centos,应该也适用于其它的发行版

这里假设你已经配置好了chef-server,在整个过程中都不需要对chef-server做修改(你可能会想到让chef-server只监听在127.0.0.1上)

## 安装Nginx

    yum -y install nginx

## 生成自签名证书

    mkdir -p /etc/ssl/nginx
    cd /etc/ssl/nginx
    openssl genrsa -out chef-server.pem 2048
    openssl req -new -x509 -key chef-server.pem -out chef-server.crt -days 1095 # 回车后按提示填一堆乱七八糟的东西

## 修改nginx配置文件

这里有一个例子(注意key的路径以及server_name直接写IP地址)

{% codeblock vim /etc/nginx/nginx.conf lang:bash %}
#######################################################################
#
# This is the main Nginx configuration file.
#
# More information about the configuration options is available on
#   * the English wiki - http://wiki.nginx.org/Main
#   * the Russian documentation - http://sysoev.ru/nginx/
#
#######################################################################

#----------------------------------------------------------------------
# Main Module - directives that cover basic functionality
#
#   http://wiki.nginx.org/NginxHttpMainModule
#
#----------------------------------------------------------------------

user              nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log;
#error_log  /var/log/nginx/error.log  notice;
#error_log  /var/log/nginx/error.log  info;

pid        /var/run/nginx.pid;


#----------------------------------------------------------------------
# Events Module
#
#   http://wiki.nginx.org/NginxHttpEventsModule
#
#----------------------------------------------------------------------

events {
    worker_connections  1024;
}


#----------------------------------------------------------------------
# HTTP Core Module
#
#   http://wiki.nginx.org/NginxHttpCoreModule
#
#----------------------------------------------------------------------

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    # Load config files from the /etc/nginx/conf.d directory
    # The default server is in conf.d/default.conf
    include /etc/nginx/conf.d/*.conf;
    upstream chef-server {
      server 127.0.0.1:4000;
    }

    server {
        listen 443 ssl;

        server_name 192.168.122.44;
        root /var/lib/chef/rack/api/public;

        ssl_certificate     /etc/ssl/nginx/chef-server.crt;
        ssl_certificate_key /etc/ssl/nginx/chef-server.key;

        location / {
            try_files $uri @backend;
        }

        location @backend {
            proxy_set_header X-Forwarded-Proto 'https';
            proxy_set_header Host $server_name;
            proxy_pass http://chef-server;
        }
    }

}
{% endcodeblock %}

## 检查配置文件并启动服务

    nginx -t
    /etc/init.d/nginx start

## 修改client使用新的地址

{% codeblock vim /etc/chef/client.rb lang:ruby %}
...
chef_server_url  'https://ip-of-chef-server'
...
{% endcodeblock %}

{% codeblock vim ~/.chef/knife.rb lang:ruby %}
...
chef_server_url  'https://ip-of-chef-server'
...
{% endcodeblock %}
