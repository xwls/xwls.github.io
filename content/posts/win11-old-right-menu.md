---
title: Windows11 恢复右键菜单
date: 2022-05-09 13:45:13+08:00
categories:
- 小技巧
tags:
- windows
summary: 刚刚安装的 win11，对新的右键菜单用不习惯（后来慢慢习惯了），此操作可以恢复 windows11 的右键菜单为原始样式。
---

## 打开注册表

使用`WIN + R`快捷键打开运行，输入`regedit`打开注册表编辑器

## 找到路径

直接在路径中输入`HKEY_CURRENT_USER\Software\Classes\CLSID`。

## 新建项

1. 右键点击`CLSID`项，新建一个项，名为`{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}`

2. 右键点击新创建的项，再新建一个项，名为`InprocServer32`

3. 选择新创建的项，双击右侧的默认条目，直接按下回车键

4. 保存注册表后，重启电脑

![202210111348645](https://xwls-oss.netlify.app/images/202210111348645.webp)