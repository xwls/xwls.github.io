+++
title = 'Oracle 表空间'
date = 2022-08-25T16:06:59+08:00
categories = ['数据库']
tags = ['oracle']
summary = 'Oracle 数据库的表空间操作，包括：创建、修改、删除、查询。'
+++

## 创建

```sql
-- 表空间类型及名称，默认不指定类型（永久）
create [temporary | undo] tablespace "TBS" 
-- 数据文件的位置及大小
datafile 'D:\Oracle\TBS.dbf' size 10m
-- 是否自动扩展，默认 'off'
[autoextend off] | [autoextend on next n maxsize m]  
-- 是否产生日志，默认 'loggin'
[loggin | nologgin]
-- 段空间自动管理，默认 'auto' 推荐
[segment space management auto]        
-- 表空间管理方式，dictionary | local(默认，推荐)              
[extent management local [uniform size n]]
```

### 例1：创建一个永久表空间 "TBS01"，其大小为 10MB

```sql
create tablespace "TBS01"
datafile 'D:\Oracle\TBS01.dbf' size 10m;
```

1.路径必须存在，否则报错！
2.表空间名称默认大写，除非用引号注明，如 "tbs" 则为小写

### 例2：创建一个自增表空间 "TBS02"，其大小为 10MB，每次扩展 1MB，最大扩展到 20MB

```sql
create tablespace "TBS02"
datafile 'D:\Oracle\TBS02.dbf' size 10m
autoextend on next 1m maxsize 20m;
```

查询上述表空间的情况：1M = 1024KB，1KB = 1024 Byte

```sql
select t.tablespace_name, -- 表空间
       t.file_name, -- 文件名
       t.autoextensible, -- 是否自增
       t.bytes / 1024 / 1024 "SIZE(M)", -- 初始值
       t.increment_by * 8 / 1024 "NEXT(M)", -- 步长 1blok = 8KB 
       t.maxbytes / 1024 / 1024 "MAXSIZE(M)"  -- 最大值
  from dba_data_files t
 where t.tablespace_name IN ('TBS01','TBS02');
```

## 修改

### 例1：修改数据文件的大小为 20M

```sql
alter database datafile 'D:\Oracle\TBS01.dbf' resize 20m;
```

### 例2：修改数据文件为自动扩展，最大值为 1G

```sql
alter database datafile 'D:\Oracle\TBS01.dbf' autoextend on next 20m maxsize 1g;
```

### 例3：新增数据文件

```sql
alter tablespace "TBS01" add datafile 'D:\Oracle\TBS01_1.dbf' size 200m;
```

## 删除

### 例1：脱机（表空间为空）

```sql
drop tablespace "TBS";
```

### 例2：脱机（表空间里有数据）

```sql
drop tablespace "TBS" including contents;
```

### 例3：完全删除（表空间 + 数据文件）

```sql
drop tablespace "TBS" including contents and datafiles;
```

### 例4：若存在约束，则追加下列子句即可

```sql
cascade constraints;
```

## 查询

```sql
-- 数据文件
select * from dba_data_files;

-- 表空间
select * from dba_tablespaces;
select * from dba_free_space;

-- 权限
select distinct t.privilege
  from dba_sys_privs t
 where t.privilege like '%TABLESPACE%';
```

### 例1：查询表空间清单

```sql
select ddf.tablespace_name 表空间名,
       ddf.file_name 数据文件名,
       ddf.file_id 数据文件id,
       ddf.autoextensible 是否自动扩展,
       ddf.bytes / 1024 / 1024 "数据文件大小（M）",
       ddf.increment_by * 8 / 1024 "自增步长（M）",
       round(ddf.maxbytes / 1021 / 1021) "数据文件最大值（M）",
       dt.contents 表空间类型,
       dt.logging 是否生成日志,
       dt.extent_management 管理模式,
       dt.allocation_type 分配类型,
       dt.segment_space_management 段管理模式
  from dba_data_files  ddf, -- tablespace_name
       dba_tablespaces dt -- tablespace_name
 where dt.tablespace_name = ddf.tablespace_name
 order by ddf.file_id;

```

### 例2：表空间使用情况

```sql
with temp_data_files as
 (select ddf.tablespace_name, sum(bytes) total
    from dba_data_files ddf
   group by ddf.tablespace_name),
temp_free_space as
 (select dfs.tablespace_name, sum(bytes) free
    from dba_free_space dfs
   group by dfs.tablespace_name)
select dt.tablespace_name 表空间名称,
       dt.contents 类型,
       (tdf.total / 1024 / 1024) "总大小(M)",
       (tfs.free / 1024 / 1024) "空闲(M)",
       round((tdf.total - tfs.free) / 1024 / 1024, 2) "已使用(M)",
       round((tdf.total - tfs.free) / tdf.total * 100, 2) "占比(%)"
  from dba_tablespaces dt, -- tablespace_name
       temp_data_files tdf, -- tablespace_name
       temp_free_space tfs -- tablespace_name
 where tdf.tablespace_name = dt.tablespace_name
   and tfs.tablespace_name = dt.tablespace_name;

```

### 例3：创建用户并指定表空间

```sql
-- 创建表空间
CREATE TABLESPACE WZW DATAFILE 'C:\APP\ORACLE\ORADATA\ORCL\WZW.DBF' SIZE 10m autoextend ON NEXT 10m maxsize 100m;
-- 创建临时表空间
CREATE TEMPORARY TABLESPACE WZW_TMP TEMPFILE 'C:\APP\ORACLE\ORADATA\ORCL\WZW_TMP.DBF' SIZE 10m autoextend ON NEXT 10m maxsize 100M;
-- 创建用户并指定表空间
CREATE USER WZW IDENTIFIED BY WZW DEFAULT tablespace WZW temporary tablespace WZW_TMP;
-- 授权
GRANT CONNECT TO WZW;
GRANT RESOURCE TO WZW;