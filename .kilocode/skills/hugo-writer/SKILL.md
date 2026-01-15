---
name: hugo-writer
description: Create and manage Hugo blog posts with proper front matter and formatting.
---

# Hugo 博客文章编写技能

## When to use this skill

当用户需要为 Hugo 静态博客网站创建技术文章时使用此技能，适用场景包括：

- 用户提供了文章内容，需要转换为 Hugo 博客格式
- 用户需要创建新的技术博客文章
- 用户需要编辑或更新现有的博客文章
- 用户需要整理技术笔记成博客格式

## 项目结构

本项目是一个 Hugo 博客，使用 PaperMod 主题：

```
hugo-xwls/
├── content/
│   ├── posts/           # 所有博客文章存放目录
│   │   ├── example.md
│   │   └── ...
│   ├── about/           # 关于页面
│   └── archives.md      # 归档页面
├── config/_default/     # 配置文件目录
└── ...
```

## 文章 Front Matter 格式

所有文章必须使用 **YAML 格式**（三个短横线 `---`）的 front matter：

```yaml
---
title: '文章标题'
date: 2025-01-14T11:16:00+08:00
categories:
- 分类名称
tags:
- 标签1
- 标签2
- 标签3
summary: '文章摘要，简短描述文章的主要内容，会显示在文章列表中'
---
```

### 字段说明

| 字段 | 必需 | 说明 |
|------|------|------|
| title | ✅ | 文章标题，使用单引号包裹 |
| date | ✅ | 发布日期，格式 `YYYY-MM-DDTHH:MM:SS+08:00` |
| categories | ✅ | 分类，通常只有一个，使用数组格式 |
| tags | ✅ | 标签，可以有多个，使用数组格式 |
| summary | ✅ | 摘要，简短描述文章内容 |

## 文件命名规范

1. 使用**小写英文字母**
2. 单词之间用**短横线**（`-`）连接
3. 文件扩展名为 `.md`
4. 文件名应能反映文章主题

**示例：**
- `java-generate-clientid-clientsecret.md`
- `docker-redis-cluster.md`
- `k8s-ingress-canary.md`
- `macos-install-hadoop-334.md`

## 常用分类参考

根据现有文章，常用分类包括：

- `Java` - Java 相关技术
- `Linux` - Linux 系统、Shell 脚本
- `Docker` - Docker 容器技术
- `Kubernetes` - K8s 相关
- `Database` - 数据库（MySQL、Oracle 等）
- `Python` - Python 编程
- `Golang` - Go 语言
- `Git` - 版本控制
- `Nginx` - Nginx 服务器
- `Windows` - Windows 系统
- `macOS` - macOS 系统
- `Util` - 工具方法、通用技巧

## 写作规范

### 1. 代码块格式

所有代码块必须指定语言类型：

````markdown
```java
public class Example {
    // Java 代码
}
```

```bash
#!/bin/bash
echo "Shell 脚本"
```

```python
def example():
    # Python 代码
    pass
```
````

### 2. 文章结构

推荐的文章结构：

```markdown
+++
title = '...'
...
+++

简短的引言或背景介绍（1-2段）

## 背景/问题描述

描述遇到的问题或需求背景

## 解决方案/实现步骤

### 步骤一：xxx

详细说明...

### 步骤二：xxx

详细说明...

## 代码示例

完整的代码...

## 总结

总结要点...
```

### 3. 格式要求

- 使用**简体中文**
- 中英文之间添加空格
- 适当使用**粗体**强调重点
- 使用有序/无序列表组织内容
- 代码中的注释也使用中文

## 创建文章步骤

1. **确定文件名**：根据文章主题生成英文短横线分隔的文件名
2. **设置日期**：使用当前时间，格式 `YYYY-MM-DDTHH:MM:SS+08:00`
3. **选择分类**：从常用分类中选择最合适的一个
4. **确定标签**：3-5 个相关标签
5. **编写摘要**：1-2 句话概括文章内容
6. **格式化内容**：确保代码块、标题层级正确
7. **保存文件**：保存到 `content/posts/` 目录

## 示例文章

参考 `content/posts/` 目录下的现有文章：

- [`java-generate-clientid-clientsecret.md`](content/posts/java-generate-clientid-clientsecret.md) - Java 技术文章示例
- [`batch-decompile-jar-with-shell.md`](content/posts/batch-decompile-jar-with-shell.md) - Shell 脚本实战示例
- [`docker-redis-cluster.md`](content/posts/docker-redis-cluster.md) - Docker 部署教程示例