[根目录](../CLAUDE.md) > **content**

# content - 博客内容模块

## 模块职责

存放博客的所有内容，包括文章、关于页面等。所有内容使用 Markdown 格式编写，通过 Front Matter 定义元数据。文章采用 Page Bundle 格式组织。

## 入口与启动

Hugo 自动扫描 `content/` 目录，按目录结构生成页面：
- `posts/` -> `/posts/` (文章列表)
- `about/` -> `/about/` (关于页)
- 根目录 `_index.md` -> 首页

## 目录结构

```
content/
├── posts/                    # 博客文章 (40篇)
│   ├── article-name/         # Page Bundle 格式
│   │   └── index.md          # 文章内容
│   └── ...
├── about/
│   └── _index.md             # 关于页面
├── tags/
│   └── c#/_index.md          # 标签自定义
└── _index.md                 # 首页内容
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

### Page Bundle 格式

Congo 主题推荐使用 Page Bundle 格式：

```
content/posts/my-article/
├── index.md          # 文章内容
├── featured.jpg      # 特色图片（可选）
└── images/           # 文章图片目录（可选）
    └── screenshot.png
```

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
A: 创建目录 `content/posts/my-article/`，在其中创建 `index.md` 文件。

**Q: 如何设置文章为草稿？**
A: 在 Front Matter 添加 `draft: true`。

**Q: 如何添加文章特色图片？**
A: 在文章目录中放置 `featured.jpg` 或 `featured.png`，Congo 会自动使用。

**Q: 如何在文章中引用图片？**
A: 将图片放在文章目录中，使用相对路径引用：`![描述](image.png)`

**Q: 如何创建新分类/标签？**
A: 直接在文章 Front Matter 中使用，Hugo 会自动创建。

**Q: 为什么使用 Page Bundle 格式？**
A: Page Bundle 将文章及其资源（图片等）组织在同一目录，便于管理和迁移。

## 相关文件清单

- `posts/*/index.md` - 博客文章 (40 篇)
- `about/_index.md` - 关于页面
- `tags/c#/_index.md` - 标签自定义页面
- `_index.md` - 首页内容

---

## 变更记录 (Changelog)

| 日期 | 变更内容 |
|------|----------|
| 2026-01-16 | 更新文档：文章结构改为 Page Bundle 格式，移除 archives.md 和 search.md |
| 2026-01-16 | 初始化模块文档 |
