---
title: MyBatis 调用数据库存储过程
date: 2022-10-17 14:19:13+08:00
categories:
- Java
tags:
- java
- 数据库
- 存储过程
- mybatis
summary: 使用 MyBatis 调用数据库的存储过程。
---

项目中有个需求：调用数据库中已有的存储过程实现用户登录业务，此存储过程接收 3 个参数，两个入参：username/password，一个出参：result，登录成功 result 为 1，失败为 0。

## 创建存储过程

这里简单创建一个存储过程，模拟登录，当用户名不为空且密码为 123456，则认为登录成功，否则认为登录失败。

```sql
create or replace procedure login_wzw(username in varchar2,password in varchar2, result out int)
as
begin
    if username is null or username = '' then
        select 0 into result from dual;
        return ;
    end if;
    if password = '123456' then
        select 1 into result from dual;
    else
        select 0 into result from dual;
    end if;
end;
```

## 测试存储过程

* 测试用户名为空，输出：result=0

```sql
declare
    username varchar2(100);
    password varchar2(100);
    result int;
begin
    username := '';
    password := '123456';
    login_wzw(username, password, result);
    dbms_output.put_line('result='||result);
end;
```

* 测试密码错误，输出：result=0

```sql
declare
    username varchar2(100);
    password varchar2(100);
    result int;
begin
    username := 'zhangsan';
    password := '1234567';
    login_wzw(username, password, result);
    dbms_output.put_line('result='||result);
end;
```

* 测试密码正确，输出：result=1

```sql
declare
    username varchar2(100);
    password varchar2(100);
    result int;
begin
    username := 'zhangsan';
    password := '123456';
    login_wzw(username, password, result);
    dbms_output.put_line('result='||result);
end;
```

## MyBatis 调用

使用 MyBatis 调用存储过程。

### 参数为实体类

* UserLogin.java

```java
public class UserLogin {

    private String username;
    private String password;
    private Integer result;

    //省略 setter/getter
}
```

* Mapper.java

```java
void loginWzw(UserLogin userLogin);
```

* Mapper.xml

```xml
<select id="loginWzw" statementType="CALLABLE" useCache="false">
  {call LOGIN_WZW(
      #{username,jdbcType=VARCHAR,mode=IN},
      #{password,jdbcType=VARCHAR,mode=IN},
      #{result,jdbcType=INTEGER,mode=OUT}
    )}
</select>
```

* 执行结果：

```text
ProcedureLoginVo{username='admin', password='123456', result=null}
13:54:37.307 [main] DEBUG c.r.p.s.m.S.loginWzw - [debug,137] - ==>  Preparing: {call LOGIN_WZW( ?, ?, ? )}
13:54:37.318 [main] DEBUG c.r.p.s.m.S.loginWzw - [debug,137] - ==> Parameters: admin(String), 123456(String)
ProcedureLoginVo{username='admin', password='123456', result=1}
```

### 参数为 Map

* Mapper.java

```java
void loginWzw2(Map<String,Object> param);
```

* Mapper.xml

```xml
<select id="loginWzw2" statementType="CALLABLE" parameterType="map" useCache="false">
  {call LOGIN_WZW(
      #{username,jdbcType=VARCHAR,mode=IN},
      #{password,jdbcType=VARCHAR,mode=IN},
      #{result,jdbcType=INTEGER,mode=OUT}
    )}
</select>
```

* 执行结果

```text
{password=1234567, username=admin}
13:54:37.335 [main] DEBUG c.r.p.s.m.S.loginWzw2 - [debug,137] - ==>  Preparing: {call LOGIN_WZW( ?, ?, ? )}
13:54:37.335 [main] DEBUG c.r.p.s.m.S.loginWzw2 - [debug,137] - ==> Parameters: admin(String), 1234567(String)
{result=0, password=1234567, username=admin}