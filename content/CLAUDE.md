[根目录](../CLAUDE.md) > **content**

# content - 博客内容模块

## 模块职责

存放博客的所有内容，包括文章、关于页面、归档页、搜索页等。所有内容使用 Markdown 格式编写，通过 Front Matter 定义元数据。

## 入口与启动

Hugo 自动扫描 `content/` 目录，按目录结构生成页面：
- `posts/` -> `/posts/` (文章列表)
- `about/` -> `/about/` (关于页)
- 根目录 `.md` -> 对应 URL

## 目录结构

```
content/
├── posts/              # 博客文章 (42篇)
├── about/
│   └── _index.md       # 关于页面
├── tags/
│   └── c#/_index.md    # 标签自定义
├── archives.md         # 归档页
└── search.md           # 搜索页
```

## 对外接口

无 API 接口。内容通过 Hugo 构建为静态 HTML。

## 关键依赖与配置

### Front Matter 规范

所有文章必须包含以下 Front Matter：

```yaml
---
title: "文章标题"
date: 2024-01-01T00:00:00+08:00
categories:
- 分类名称
tags:
- 标签1
summary: 文章简介
---
```

### 特殊页面布局

| 页面 | layout |
|------|--------|
| 归档 | `archives` |
| 搜索 | `search` |
| 关于 | `simple` |

## 数据模型

### 文章分类 (Categories)

当前存在的分类：
- Golang, Python, Java
- Oracle, 数据库
- 云原生, 中间件, 大数据
- 运维, 环境搭建
- 技术教程, 小技巧, AI, Util, 数据科学

### 常用标签 (Tags)

go, python, java, docker, k8s, mysql, oracle, nginx, redis, git 等

## 测试与质量

- 使用 `hugo server -D` 本地预览（包含草稿）
- 构建时 Hugo 会检查 Markdown 语法和链接
- 文章发布前建议本地预览确认格式

## 常见问题 (FAQ)

**Q: 如何新建文章？**
A: 运行 `hugo new posts/my-article.md` 或手动创建文件。

**Q: 如何设置文章为草稿？**
A: 在 Front Matter 添加 `draft: true`。

**Q: 如何添加文章封面图？**
A: 在 Front Matter 添加：
```yaml
cover:
  image: "/images/cover.jpg"
  alt: "封面描述"
```

**Q: 如何创建新分类/标签？**
A: 直接在文章 Front Matter 中使用，Hugo 会自动创建。

## 相关文件清单

- `posts/*.md` - 博客文章 (42 篇)
- `about/_index.md` - 关于页面
- `archives.md` - 归档页配置
- `search.md` - 搜索页配置

---

## 变更记录 (Changelog)

| 日期 | 变更内容 |
|------|----------|
| 2026-01-16 | 初始化模块文档 |
