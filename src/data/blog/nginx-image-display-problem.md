---
author: Denney Yang
pubDatetime: 2021-11-30T21:38:18+08:00
title: nginx反向代理后图片不能正常显示的问题
featured: False
draft: False
tags:
  - notes
description: 问题描述  自己在搭建halo博客的时候，当做到使用nginx做反向代理这一步时，按照[官方文档](https://docs.halo.run/getting-started/install/other/oneinstack)中“与 OneinStack 配合使用”做方向代理的方法，我根据文档的步骤...
---
##  问题描述

自己在搭建halo博客的时候，当做到使用nginx做反向代理这一步时，按照[官方文档](https://docs.halo.run/getting-started/install/other/oneinstack)中“与 OneinStack 配合使用”做方向代理的方法，我根据文档的步骤一步一步做下来，一切都感觉良好，博客的确成功使用nginx做了反向代理，但是当我打开管理后台的时候，发现主题的图片、自己上传到本地服务器的图片都不能正常显示出来，然后就开始疯狂google，结果网上的回答比较零碎，但发现应该就是nginx配置文件的问题，最后更改了配置文件的内容，成功解决问题。

## 解决过程

原来的nginx.conf文件配置内容:

```shell
upstream halo {
  server 127.0.0.1:8090;
}
server {
  listen 80;
  listen [::]:80;
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  ssl_certificate /usr/local/nginx/conf/ssl/domainName.com.crt;
  ssl_certificate_key /usr/local/nginx/conf/ssl/domainName.com.key;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers on;
  ssl_session_timeout 10m;
  ssl_session_cache builtin:1000 shared:SSL:10m;
  ssl_buffer_size 1400;
  add_header Strict-Transport-Security max-age=15768000;
  ssl_stapling on;
  ssl_stapling_verify on;
  server_name domainName.com www.domainName.com;
  access_log /data/wwwlogs/domainName.com_nginx.log combined;
  index index.html index.htm index.php;
  root /data/wwwroot/domainName.com;
  if ($ssl_protocol = "") { return 301 https://$host$request_uri; }
  if ($host != domainName.com) {  return 301 $scheme://domainName.com$request_uri;  }
  include /usr/local/nginx/conf/rewrite/none.conf;
  #error_page 404 /404.html;
  #error_page 502 /502.html;
  location ~ .*\.(wma|wmv|asf|mp3|mmf|zip|rar|jpg|gif|png|swf|flv|mp4)$ {
    valid_referers none blocked *.domainName.com domainName.com www.domainName.com;
    if ($invalid_referer) {
        return 403;
    }
  }
  location ~ /(\.user\.ini|\.ht|\.git|\.svn|\.project|LICENSE|README\.md) {
    deny all;
  }
  location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
	  proxy_pass http://halo;
	  expires 30d;
	  access_log off;
  }
  location ~ .*\.(js|css)?$ {
	  proxy_pass http://halo;
	  expires 7d;
	  access_log off;
  }
  location / {
    proxy_pass http://halo;
    proxy_set_header HOST $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
  location ^~ /.well-known/acme-challenge/ {
    default_type "text/plain";
    allow all;
    root /data/wwwroot/domainName.com/;
  }
}
```

修改后nginx.conf文件配置内容：

```shell
# HTTP server
server {
    listen 80;
    server_name domainName.com www.domainName.com;				# 这里填自己的域名
    rewrite ^(.*)$ https://$server_name$1 permanent;			# 将所有http请求通过rewrite重定向到https
    client_max_body_size 1024m;
}
  
# HTTPS server
server {
	listen 443 ssl;   								# SSL协议访问端口号为443，如果这里没有ssl，可能无法启动nginx
	server_name domainName.com www.domainName.com;  # 这里填自己的域名
	root html;
	index index.html index.htm;
	ssl_certificate /usr/local/nginx/conf/ssl/domainName.com.crt;   	# 这里填证书的文件名及其路径
	ssl_certificate_key /usr/local/nginx/conf/ssl/domainName.com.key;   # 这里填证书密钥的文件名及其路径
	ssl_session_timeout 5m;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  # 使用的加密套件
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;								# 使用这些协议进行配置
	ssl_prefer_server_ciphers on;

	location / {
	  add_header Content-Security-Policy upgrade-insecure-requests;	# 这里就是https代理http时静态资源处理的关键
	  proxy_set_header HOST $host;
	  proxy_set_header X-Forwarded-Proto $scheme;
	  proxy_set_header X-Real-IP $remote_addr;
	  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	  proxy_pass http://127.0.0.1:8090/;                			# 这里填要代理的端口
	}
}
```

## 参考文档

https://blog.csdn.net/weixin_43040726/article/details/108104987
