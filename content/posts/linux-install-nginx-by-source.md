+++
title = 'Linux 源码安装 Nginx'
date = 2019-08-26T14:21:23+08:00
categories = ['环境搭建']
tags = ['linux', 'nginx']
summary = '在 CentOS7 下以软件源码包的方式安装 Nginx，主要分为：下载、解压缩、配置、编译、安装几个步骤，其他源码包软件的安装方式类似。'
+++

## 安装 nginx 需要的依赖

```bash
# 安装 gcc
yum install -y gcc
# 安装 gcc-c++
yum install -y gcc-c++
# 安装 pcre pcre-devel
yum install -y pcre pcre-devel
# 安装 zlib zlib-devel
yum install -y zlib zlib-devel
# 安装 openssl openssl-devel
yum install -y openssl openssl-devel
```

## 下载

```bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

## 解压缩

```bash
tar -zxvf nginx-1.16.1.tar.gz
```

## 安装

```bash
# 进入解压得到的目录
cd nginx-1.16.1
# 配置
./configure
# 编译
make
# 安装
make install
```

## 启动、停止

```bash
# 进入/usr/local/nginx/sbin
cd /usr/local/nginx/sbin
# 启动
./nginx
# 停止
./nginx -s stop
# 重启
./nginx -s reload