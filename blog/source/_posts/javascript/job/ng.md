---
title: nginx配置
date: 2018-09-01
categories: work
tags: nginx
---

最近在项目中用node新建了一个web服务，需要修改原有的nginx配置以沿用之前的对外域名，这里mark下配置项*.conf文件

```bash
upstream xx_com_001 {
  server <ip>:<port>;
}
upstream xx_com_002 {
  server <ip>:<port>;
}

server {
    listen <ip>:80;
    listen <ip>:443
    server_name www.example.com;

    gzip on;
    gzip_http_version 1.1;
    gzip_buffers 256 64k;
    gzip_comp_level 5;
    gzip_min_length 1000;
    gzip_types text/javascript application/x-javascript text/css text/plain;
    gzip_disable "MSIE 6";
 
    if ( $request_method !~ ^(GET|POST|HEAD)$ ) {
        return 403;
    }

    # 可以引入其它配置
    include xx.conf;
    
    proxy_buffers 64 4k;
    
    # 直接转发请求, 转发整个路径
    location /app/ {
        proxy_pass http://xx_com_001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    location /node/ {
        proxy_pass http://xx_com_002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # 重写请求
    location ^~/apis/ {
        # 这里重写了请求，将正则匹配中的第一个()中$1的path，拼接到真正的请求后面，并用break停止后续匹配
        rewrite ^/apis/(.*)$ /$1 break;
        proxy_pass http://www.xx.com/;
    }
}

server {
    listen <本机ip>:443;
    server_name www.example.com;

    gzip on;
    gzip_http_version 1.1;
    gzip_buffers 256 64k;
    gzip_comp_level 5;
    gzip_min_length 1000;
    gzip_types text/javascript application/x-javascript text/css text/plain;
    gzip_disable "MSIE 6";

    ssl_certificate      <.pem文件路径>;
    ssl_certificate_key  <.key文件路径>;
    ssl_session_timeout  5m;
    ssl_protocols  SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers <ssl_ciphers>
    ssl_prefer_server_ciphers   on;

    if ( $request_method !~ ^(GET|POST|HEAD)$ ) {
        return 403;
    }

    include xx.conf;
    proxy_buffers 64 4k;
    
    location /app/ {
        proxy_pass http://xx_com_001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    location /node/ {
        proxy_pass http://xx_com_002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }

```




