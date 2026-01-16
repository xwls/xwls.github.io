---
title: 修改 Oracle 数据库的字符集
date: 2022-08-25 16:09:14+08:00
categories:
- 数据库
tags:
- oracle
summary: 修改 Oracle 数据库的字符集，项目中遇到的问题，客户的 Oracle 字符集是 ZHS16GBK，我们是 UTF8，需要将 UTF8 修改为
  ZHS16GBK。
---

库 A 的编码为 gbk，库 B 的编码为 utf-8，两种编码格式下，汉字占的字节数不一样，gbk 为 2 个字节，utf-8 为 3 个字节。所有针对某些情况，需要修改字符集编码。

修改 Oracle 数据库的字符集（UTF8→ZHS16GBK）步骤如下：

## 1. 登录

以 sysdba 的身份登录上去

```text
conn sys/root as sysdba
```

## 2. 关闭数据库

```text
shutdown immediate;
```

## 3. 以 mount 打来数据库

```text
startup mount;
```

## 4. 设置 session

```sql
ALTER SYSTEM ENABLE RESTRICTED SESSION;
ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
ALTER SYSTEM SET AQ_TM_PROCESSES=0;
```

## 5. 启动数据库

```sql
alter database open;
```

## 6. 修改字符集

```sql
ALTER DATABASE character set INTERNAL_USE ZHS16GBK;
```

## 7. 关闭，重新启动

```sql
shutdown immediate;
startup;
```

## 8. 查看修改后的字符集

```sql
SELECT * FROM V$NLS_PARAMETERS WHERE PARAMETER = 'NLS_CHARACTERSET' ;
select userenv('language') from dual;
```

> 当然字符集最好不要轻易修改，因为这会对数据库的数据有直接的影响，
> 如果是正式环境的话，可能会造成不可估计的损失。
> 修改完字符集后，数据库中已有的汉字将会出现乱码。请修改编码集前备份数据，修改完编码集后重新导入数据。