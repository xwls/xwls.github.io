---
title: 'ORA-01502: 索引 ''xxx'' 或这类索引的分区处于不可用状态'
date: 2022-08-25 16:13:07+08:00
categories:
- 数据库
tags:
- oracle
summary: 'ORA-01502: 索引 ''xxx'' 或这类索引的分区处于不可用状态，移动表空间后，导致索引失效，处于不可用状态。'
---

```text
### The error occurred while setting parameters
### SQL: insert into USER_LBL_INF( LBL_ID,LBL_NM,USERID,CRT_TM)       values(       ?,       ? ,       ? ,       ?       )
### Cause: java.sql.SQLException: ORA-01502: 索引 'BASE_PD.PK_USER_LBL_INF' 或这类索引的分区处于不可用状态
 
; uncategorized SQLException; SQL state [72000]; error code [1502]; ORA-01502: 索引 'BASE_PD.PK_USER_LBL_INF' 或这类索引的分区处于不可用状态
; nested exception is java.sql.SQLException: ORA-01502: 索引 'BASE_PD.PK_USER_LBL_INF' 或这类索引的分区处于不可用状态
] with root cause
 
java.sql.SQLException: ORA-01502: 索引 'BASE_PD.PK_USER_LBL_INF' 或这类索引的分区处于不可用状态
```

## 查询不可用的索引

```sql
SELECT INDEX_NAME, STATUS FROM DBA_INDEXES WHERE OWNER = 'RZGWL_WLLH' AND STATUS != 'VALID';
```

## 重建索引

```sql
ALTER INDEX PK_BT_RY_SYS_USER_ROLE REBUILD;