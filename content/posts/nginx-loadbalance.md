---
title: Nginx 配置负载均衡
date: 2019-08-26 14:16:58+08:00
categories:
- 运维
tags:
- nginx
summary: 负载均衡建立在现有网络结构之上，它提供了一种廉价有效透明的方法扩展网络设备和服务器的带宽、增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性。接下来以两个
  Tomcat 为例简单说一下 Nginx 如何配置负载均衡。
---

## 配置 nginx.conf

```text
upstream tomcat{
    # 服务器性能较好，权重设置为 2
    server  192.168.0.100:8080 weight=2;
    # 服务器性能较弱，权重设置为 1
    server  192.168.0.101:8080 weight=1;
}

server {
    listen       80;
    server_name  localhost;

    #access_log  logs/host.access.log  main;

    location / {
        #root   html;
        #index  index.html index.htm;
        proxy_pass http://tomcat;
    }
}
```

## 效果

每三个请求，会有两个访问 8080 服务器，一个访问 8081 服务器