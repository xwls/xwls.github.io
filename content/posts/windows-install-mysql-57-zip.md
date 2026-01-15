---
title: Windows 下安装 MySQL5.7.28 压缩版
date: 2019-10-24 13:50:15+08:00
categories:
- 环境搭建
tags:
- windows
- mysql
summary: 关于 MySQL，之前大部分都是在 Linux 下安装使用，在 Windows 下也用过，但用的都是安装版，这里记录一下 Windows 下 MySQL
  压缩版的安装和配置过程。
---

## 下载

浏览器访问 [https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/) 下载需要的版本，我这里下载的是 Windows64 位的压缩版。

![图片](https://i.loli.net/2019/10/24/8br2uSX3jQkTNMs.png)

## 解压安装

将下载的的压缩文件 mysql-5.7.28-winx64.zip 解压缩到你准备安装的目录，并在压缩目录中创建文件 my.ini，文件内容如下：

* 其中 basedir 为 MySQL 的解压缩目录

```text
[mysql]
# 设置 mysql 客户端默认字符集
default-character-set=utf8
[mysqld]
#设置 3306 端口
port = 3306
# 设置 mysql 的安装目录
basedir=D:\mysql-5.7.28-winx64
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为 8 比特编码的 latin1 字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

* 将 MySQL 安装目录下的 bin 目录添加到环境变量 Path

![图片](https://i.loli.net/2019/10/24/45MsqrRmVzgjpQF.png)

* 以管理员身份打开 CMD，进入 MySQL 的 bin 目录，执行以下命令安装服务

```text
mysqld -install
# or：mysqld -install MySQL57，其中 MySQL57 表示服务名
```

![图片](https://i.loli.net/2019/10/24/Pu5o1eQ6kfxKIUn.png)

* 安装完成之后，打开服务应该可以看到刚刚安装的服务

![图片](https://i.loli.net/2019/10/24/bAugD7o1OIqyC5w.png)

## 初始化

* 开始初始化，初始化完成后会输出随机生成的 root 密码

```text
mysqld --initialize --console
```

* 红框中的是生成的 root 密码

![图片](https://i.loli.net/2019/10/24/tuzFXHvkwacofVP.png)

## 设置 root 密码

* 启动 MySQL 的服务，然后使用刚刚生成的 root 密码登录到 MySQL

```text
mysql -uroot -p
```

![图片](https://i.loli.net/2019/10/24/SWFBOIiAJ2c8XHj.png)

* 将 root 密码修改为 123456

```text
set password = password('123456');
```

![图片](https://i.loli.net/2019/10/24/Az9hDiY2Xaltj5F.png)