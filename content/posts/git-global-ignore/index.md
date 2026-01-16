---
title: Git 全局忽略 .DS_Store 文件
date: 2022-08-29 16:29:36+08:00
categories:
- 小技巧
tags:
- git
summary: Git 全局忽略指定文件，以 MacOS 的 .DS_Store 文件为例。
---

## .DS_Store 文件

DS_Store，英文全称是 Desktop Services Store（桌面服务存储），开头的 DS 是 Desktop Services（桌面服务） 的缩写。它是一种由 macOS 系统自动创建的隐藏文件，存在于每一个用「访达」打开过的文件夹下面。

虽然不能在「访达」中直接看到它，但是通过「终端」App，可以输入`ls -la`命令列出。同时，通过`file`命令，可以显示出其文件类型，即"Desktop Services Store"。

DS_Store 文件的主要作用，是存储当前文件夹在桌面显示相关方面的一些自定义属性，包括文件图标的位置、文件夹上次打开时窗口的大小、展现形式和位置等。这有助于保留为特定文件夹配置的设置，例如，将桌面文件夹设置为查看按名称排序的图标，同时将下载文件夹配置为将文件显示为列表并按日期排序，最近修改的先显示。

## Git 忽略 .DS_Store 文件

作为一名使用 Mac 的开发者，在日常开发过程中，经常会使用 Git 来对代码文件夹进行版本控制。而在默认情况下，Git 会把 .DS_Store 文件带入版本控制的范围内。所以，可以手动将其踏入 Git 的版本管理忽略列表。

* 将 `.DS_Store` 加入全局的 `.gitignore` 文件，执行命令：

```bash
echo .DS_Store >> ~/.gitignore_global
```

* 将这个全局的 `.gitignore` 文件加入 Git 的全局 `config` 文件中，执行命令：

```bash
git config --global core.excludesfile ~/.gitignore_global