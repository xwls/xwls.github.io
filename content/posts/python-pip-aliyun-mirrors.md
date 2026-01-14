+++
title = 'Python 中 pip 设置为阿里镜像源'
date = 2022-07-01T15:47:42+08:00
categories = ['Python']
tags = ['mirror', 'python']
summary = 'Python 中的 pip 默认源，在国内下载很慢，修改为阿里云的 pip 源提升下载速度。'
+++

## 配置文件

a. 找到下列文件

Linux 在`~/.pip/pip.conf`,Windows 在`~/AppData/Roaming/pip/pip.ini`

b. 在上述文件中添加或修改：

```text
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

## 使用命令设置

```text
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/