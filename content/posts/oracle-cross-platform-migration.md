---
title: Oracle 数据库跨平台迁移全记录：从 Windows 到 Linux
date: 2026-01-12 15:52:00+08:00
categories:
- 数据库
tags:
- oracle
- linux
- windows
- 数据迁移
summary: 记录一次 Oracle 数据库从 Windows 到 Linux 的跨平台迁移实战，包括 Data Pump 导出导入的完整流程及踩坑解决方案。
---

## 迁移背景

* **源端**：Windows Server (IP: 192.168.10.54), Oracle 11g
* **目标端**：Linux Server (IP: 192.168.10.33), Oracle 11g
* **需求**：将源端用户 `RZGWL_WLLH` 下的所有对象（表、视图、序列、函数等）全量迁移到目标端。
* **工具**：Oracle Data Pump (`expdp` / `impdp`)

## 核心步骤梳理

### 第一步：源端导出 (Windows)

在 Windows 服务器上使用 `expdp` 进行逻辑导出。

**1. 创建导出目录** (SQLPlus / System用户):

```sql
CREATE OR REPLACE DIRECTORY win_dump_dir AS 'D:\backup';
GRANT READ, WRITE ON DIRECTORY win_dump_dir TO RZGWL_WLLH;
```

**2. 执行导出** (CMD命令行):

使用 `schemas` 参数指定按用户导出，开启 `compression` 压缩节省空间。

```bash
expdp RZGWL_WLLH/密码@orcl directory=win_dump_dir dumpfile=rzgwl_full.dmp logfile=exp_rzgwl.log schemas=RZGWL_WLLH compression=all
```

### 第二步：文件传输

使用 WinSCP 或 SCP 命令将生成的 `rzgwl_full.dmp` 文件上传至 Linux 服务器目录：

* **目标路径**：`/u01/app/oracle/oradata/`

### 第三步：目标端环境准备 (Linux) —— 关键

为了避免导入过程中的各种权限和空间报错，推荐先执行**清理与初始化脚本**。

**1. 操作系统层权限设置** (Root 用户):

Oracle 进程需要对存放 DMP 文件的目录有读写权限，否则会报 `ORA-29283`。

```bash
mkdir -p /u01/app/oracle/oradata
chown -R oracle:oinstall /u01/app/oracle/oradata
chmod -R 775 /u01/app/oracle/oradata
```

**2. 数据库初始化脚本** (SQLPlus / System用户):

> **重点**：`RESOURCE` 角色不包含 `CREATE VIEW` 权限，必须单独授予，否则视图导入会失败。

```sql
-- 1. 清理旧环境 (如果已存在)
DROP USER RZGWL_WLLH CASCADE;
DROP TABLESPACE RZGWL_WLLH INCLUDING CONTENTS AND DATAFILES;

-- 2. 创建大容量表空间 (避免 ORA-01658)
CREATE TABLESPACE RZGWL_WLLH 
DATAFILE '/u01/app/oracle/oradata/orcl/rzgwl_wllh.dbf' 
SIZE 10G AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;

-- 3. 创建临时表空间
CREATE TEMPORARY TABLESPACE RZGWL_WLLH_TMP 
TEMPFILE '/u01/app/oracle/oradata/orcl/rzgwl_wllh_tmp.dbf' 
SIZE 100M AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED;

-- 4. 创建用户
CREATE USER RZGWL_WLLH 
IDENTIFIED BY 密码
DEFAULT TABLESPACE RZGWL_WLLH 
TEMPORARY TABLESPACE RZGWL_WLLH_TMP;

-- 5. 完整授权 (避免 ORA-31685 / ORA-01031)
GRANT CONNECT, RESOURCE, CREATE VIEW TO RZGWL_WLLH;
GRANT UNLIMITED TABLESPACE TO RZGWL_WLLH;

-- 6. 注册目录
CREATE OR REPLACE DIRECTORY linux_dump_dir AS '/u01/app/oracle/oradata';
GRANT READ, WRITE ON DIRECTORY linux_dump_dir TO RZGWL_WLLH;
```

### 第四步：目标端导入 (Linux)

执行 `impdp` 导入。

> **技巧**：使用 `transform=segment_attributes:n` 忽略源端的表空间设置，强制所有数据进入 Linux 端用户默认的表空间，解决跨平台路径不一致问题。

```bash
impdp RZGWL_WLLH/密码@orcl \
directory=linux_dump_dir \
dumpfile=rzgwl_full.dmp \
logfile=imp_rzgwl.log \
table_exists_action=replace \
transform=segment_attributes:n
```

## 踩坑记录与解决方案 (Troubleshooting)

在本次迁移过程中，遇到了几个非常经典的 Oracle 错误，记录如下：

### 问题一：表空间不足

* **报错**：`ORA-01658: unable to create INITIAL extent for segment in tablespace ...`
* **原因**：表空间初始大小设置太小（如 10M），且 `MAXSIZE` 设置了上限。
* **解决**：重建表空间时直接给足 `SIZE 10G`，并设置 `MAXSIZE UNLIMITED`。

### 问题二：游标超限

* **报错**：`ORA-01000: maximum open cursors exceeded`
* **场景**：执行大量初始化 SQL 脚本时发生。
* **解决**：调整系统参数。

```sql
ALTER SYSTEM SET open_cursors = 2000 SCOPE=BOTH;
```

### 问题三：Linux 目录权限拒绝

* **报错**：

```
ORA-39070: Unable to open the log file.
ORA-29283: invalid file operation
```

* **原因**：Linux 系统上，存放 DMP 文件的目录归属者是 `root`，而 Oracle 数据库进程用户是 `oracle`，无权写入。
* **解决**：

```bash
chown -R oracle:oinstall /u01/app/oracle/oradata
```

### 问题四：视图创建失败（权限不足）

* **报错**：

```
ORA-31685: Object type VIEW failed due to insufficient privileges
ORA-39126: Worker unexpected fatal error
```

* **原因**：这是一个经典误区。Oracle 的 `RESOURCE` 角色包含了 `CREATE TABLE`、`CREATE SEQUENCE` 等权限，但**不包含** `CREATE VIEW` 权限。
* **解决**：显式授予视图权限。

```sql
GRANT CREATE VIEW TO RZGWL_WLLH;
```

## 总结

Oracle Data Pump (`expdp`/`impdp`) 是跨平台迁移的神器。但要顺利完成迁移，关键不在于命令本身，而在于**环境的准备**：

1. **OS 层面**：确保 `oracle` 用户对目录有读写权限。
2. **DB 层面**：表空间要够大，权限给的要够足（特别是 `CREATE VIEW`）。
3. **参数层面**：善用 `transform=segment_attributes:n` 来屏蔽平台差异。