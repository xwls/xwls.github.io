---
title: CentOS 安装 MySQL5.7
date: 2019-10-22 14:04:37+08:00
categories:
- 运维
tags:
- centos
- linux
- mysql
summary: 记录一下在 CentOS7 中安装 MySQL5.7 版本遇到的一个小问题，顺便记录下安装过程。
---

## 安装 MySQL5.7 的安装源

```bash
yum localinstall http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```

## 安装 MySQL5.7

```bash
yum install mysql-community-server
```

## 启动并设置开机自启

```bash
systemctl start mysqld
systemctl enable mysqld
```

## 查看 root 密码

```bash
grep 'temporary password' /var/log/mysqld.log
```

![image](https://i.loli.net/2019/09/03/IX73RthTMQ4xk6A.png)

可以看到我这边的密码为：:/XaahH&%5Qe，是的，包含开始的冒号，我一开始以为冒号后边才是，试了几次发现不对，冒号也是！！！

注意这个密码只能够让你登录到 MySQL，登录进去之后必须马上修改密码，否则无法进行其他任何的操作。

## 修改 root 密码（本篇重点）

修改密码的命令：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
```

我希望给 root 设置一个简单的密码，比如 123456，但是发现并不可以

![image](https://i.loli.net/2019/09/03/vSbtzAZpwXCTo2P.png)

意思是说我的密码太过简单不符合他的密码策略，但是我想查询或者修改密码策略必须先设置密码才可以，无奈之下，先设置一个复杂的：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'OracleOAEC$2019';
```

![image](https://i.loli.net/2019/09/03/XvWKmZbgRcj5yuY.png)

密码设置好了，接下来看下密码策略：

```sql
SHOW VARIABLES LIKE 'validate_password%';
```

这里挑了几个说明下：

![image](https://i.loli.net/2019/09/03/Eg2HZK9sF1Ayc6d.png)

我想设置的密码是简单密码 123456，需要将 validate_password_policy 设置为 LOW，设置为 LOW 之后就只验证长度，同时还需要设置 validate_password_length 为 6：

```sql
set global validate_password_policy=LOW;
set global validate_password_length=6;
```

![image](https://i.loli.net/2019/09/03/7eCtQv4fwEpN2Fr.png)

![image](https://i.loli.net/2019/09/03/GcPgxKjtWwQoaH2.png)

然后再尝试修改密码为 123456：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

![image](https://i.loli.net/2019/09/03/acgvAtPFnN2bHrS.png)

可以发现，密码已经成功修改为 123456

## 允许 root 远程登录

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY '123456' WITH GRANT OPTION;
FLUSH PRIVILEGES;