+++
title = 'Nginx 配置 http 自动跳转到 https'
date = 2019-10-22T13:58:42+08:00
categories = ['运维']
tags = ['运维', 'https', 'http']
summary = '前几天在阿里云申请了一个免费的 SSL 证书，给网站配置了 https 访问，顺便配置了 http 自动跳转到 https，在这里记录下配置文件。'
+++

![image](https://i.loli.net/2019/09/05/ocQLh8iOMuxWAFN.png)

* nginx.config

```text
upstream halo_server{
 server 172.31.10.131:8090;
}

server {
 listen 80;
 server_name www.xwls.me;
 return 301 https://$server_name$request_uri;
}

# Settings for a TLS enabled server.
#
server {
 listen 443;
 server_name www.xwls.me;
 ssl on;
 root html;
 index index.html index.htm;
 ssl_certificate   /home/ec2-user/2728032_xwls.me.pem;
 ssl_certificate_key  /home/ec2-user/2728032_xwls.me.key;
 ssl_session_timeout 5m;
 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
 ssl_prefer_server_ciphers on;
 location / {
 #     root html;
 #     index index.html index.htm;
  proxy_pass http://halo_server;
 }
}