---
title: SpringBoot中Jackson的简单使用
date: 2019-08-12 13:38:42+08:00
categories:
- Java
tags:
- java
- spring
- springboot
- json
summary: SpringBoot默认使用Jackson作为对象的序列化工具，在进行对象的序列化时，日期时间类型和null值往往需要单独处理。
---

## 统一处理

在SpringBoot的配置文件application.yml中添加以下配置：

```java
spring:
  jackson:
    #时区
    time-zone: GMT+8
    #日期格式
    date-format: yyyy-MM-dd HH:mm:ss
    #只对非空属性进行序列化
    default-property-inclusion: non_null
```

## 单独处理

* 处理日期格式

```java
//格式化日期属性
@JsonFormat(pattern = "yyyy年MM月dd日")
private Date birthday;
```

* 序列化时忽略属性

```java
//不对密码进行序列化
@JsonIgnore
private String password;
```

* 为属性重命名

```java
//属性email会在序列化时重命名为mail
@JsonProperty("mail")
private String email;