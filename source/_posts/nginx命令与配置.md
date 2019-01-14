---
title: nginx命令与配置
date: 2018-11-29 10:44:33
tags:
---

* nginx 启动命令

``` bash
nginx -c /usr/local/nginx/conf/nginx.conf 
```

<!-- more -->
* 查找nginx配置文件位置

``` bash
nginx -t
```
* nginx 重启命令

``` bash
nginx -s reload
```

* 停止nginx 

``` bash
nginx -s stop
nginx -s quit
```

* nginx 配置

``` bash
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;
    client_max_body_size 500M;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;


server {
    listen       80;
    server_name   k701-third.baifendian.com;
    index index.html index.htm;
    proxy_set_header Host $host;
    root /opt/www/webapp/dist_index/v1_index;

#  location ^~ / {
   #      root /opt/www;
#          try_files $uri webapp/dist_index/v1_index/index.html;
#         index index.html;
#   }
#图片服务器
   location ^~ /group1/M00 {
#       ngx_fastdfs_module;
       proxy_pass http://*.*.*.10/group1/M00/;
    }
#指标
    location ^~ /api/indicators/ {
     proxy_pass http://*.*.*.10:8101/indicators/;
    }
#模型管理工具
  #  location ^~ /models {
   #      root /opt/www;
  #        try_files $uri models/index.html;
 #        index index.html;
#   }
#身份证
    location ^~ /id-cards/ {
        proxy_pass http://*.*.*.167:83/;
    }
#身份证后端
    location ^~ /api/identity_web/ {
        proxy_pass http://*.*.*.167:7001/identity_web/;
    }


     location /bdos-base  {
        proxy_pass http://*.*.*.167:8080;

     }
     location /bdos-luban  {
        proxy_pass http://*.*.*.167:18990;

     }

     location /bdos-runner-web  {
        proxy_pass http://*.*.*.167:8181;
     }
    location ~ /assets/*\.(json|gif|jpg|jpeg|bmp|png|ico|txt|js|css)$ {
        try_files $uri $uri/ /assets/;
    }


    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~ /.*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css)$ {
        expires 7d;
    }

}
}





```
