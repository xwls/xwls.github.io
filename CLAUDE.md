# CLAUDE.md - xwls.github.io

## 项目愿景

Hugo 静态博客站点，基于 [Congo](https://github.com/jpanther/congo) 主题，用于记录技术学习与分享。博客内容涵盖后端开发、云原生、数据库、大数据等多个技术领域。

## 架构总览

```
xwls.github.io/
├── config/_default/    # Hugo 配置文件（YAML 格式）
├── content/            # 博客内容（Markdown）
│   ├── posts/          # 文章目录（Page Bundle 格式）
│   ├── about/          # 关于页面
│   └── tags/           # 标签自定义
├── layouts/            # 自定义布局模板（当前为空）
├── static/             # 静态资源
├── i18n/               # 国际化翻译
├── .claude/            # Claude AI 配置
├── public/             # 构建输出（已忽略）
└── .github/workflows/  # CI/CD 配置
```

## 模块结构图

```mermaid
graph TD
    A["xwls.github.io (根)"] --> B["config/_default"]
    A --> C["content"]
    A --> D["layouts"]
    A --> E["static"]
    A --> F["i18n"]
    A --> G[".github/workflows"]
    A --> H[".claude"]

    C --> C1["posts (40篇文章)"]
    C --> C2["about"]
    C --> C3["tags"]

    B --> B1["hugo.yaml"]
    B --> B2["params.yaml"]
    B --> B3["menus.yaml"]
    B --> B4["module.yaml"]
    B --> B5["languages.yaml"]
    B --> B6["markup.yaml"]

    click B "./config/_default/CLAUDE.md" "查看配置模块文档"
    click C "./content/CLAUDE.md" "查看内容模块文档"
```

## 模块索引

| 模块路径 | 职责 | 入口文件 | 配置存在 |
|----------|------|----------|----------|
| `config/_default/` | Hugo 站点配置 | `hugo.yaml` | 是 |
| `content/` | 博客内容管理 | `posts/` | - |
| `layouts/` | 自定义模板覆盖（当前为空） | - | - |
| `static/` | 静态资源 | `favicon.svg` | - |
| `i18n/` | 国际化翻译 | `zh-Hans.yaml` | - |
| `.github/workflows/` | CI/CD 自动部署 | `hugo.yml` | - |
| `.claude/` | Claude AI 配置 | `index.json` | - |

## 运行与开发

### 环境要求

- Hugo Extended >= 0.154.5
- Go >= 1.25 (用于 Hugo Modules)
- Git

### 本地开发

```bash
# 克隆仓库
git clone https://github.com/xwls/xwls.github.io.git
cd xwls.github.io

# 启动本地开发服务器
hugo server -D

# 构建生产版本
hugo --gc --minify
```

### 新建文章

使用 Page Bundle 格式创建文章：

```bash
# 创建文章目录和 index.md
mkdir -p content/posts/my-new-post
touch content/posts/my-new-post/index.md
```

文章 Front Matter 模板：

```yaml
---
title: "文章标题"
date: 2024-01-01T00:00:00+08:00
categories:
- 分类名称
tags:
- 标签1
- 标签2
summary: 文章摘要
---
```

## 部署配置

项目支持多平台自动部署：

| 平台 | 配置文件 | 部署 URL |
|------|----------|----------|
| GitHub Pages | `.github/workflows/hugo.yml` | https://xwls.github.io |
| Netlify | `netlify.toml` | https://xwls.netlify.app |
| Vercel | `vercel.json` + `build.sh` | https://xwls.vercel.app |

## 主题配置

使用 Hugo Modules 引入 Congo 主题：

```yaml
# config/_default/module.yaml
imports:
- path: github.com/jpanther/congo/v2
```

主要特性配置 (`config/_default/params.yaml`)：
- 自动主题切换 (light/dark/auto)
- 阅读时间显示
- 字数统计
- 目录导航 (TOC)
- 面包屑导航
- 全局搜索
- 编辑链接

## 测试策略

本项目为纯静态站点，无单元测试。质量保证依赖：
- Hugo 构建时的错误检查
- CI/CD 流水线构建验证
- 本地 `hugo server` 预览

## 编码规范

### 内容规范

1. 文章使用 Markdown 格式
2. 文章采用 Page Bundle 格式：`posts/article-name/index.md`
3. 文件名使用英文小写，单词间用 `-` 连接
4. 必须包含 `title`、`date`、`categories`、`tags` 等 Front Matter
5. 代码块必须指定语言类型

### 配置规范

1. 配置文件使用 YAML 格式
2. 配置按功能分离到独立文件

## AI 使用指引

### 常见任务

1. **新建文章**: 在 `content/posts/` 创建目录，内含 `index.md` 文件
2. **修改菜单**: 编辑 `config/_default/menus.yaml`
3. **修改主题参数**: 编辑 `config/_default/params.yaml`
4. **添加自定义样式/脚本**: 在 `layouts/` 目录添加覆盖模板

### 关键文件

- 主配置: `config/_default/hugo.yaml`
- 主题参数: `config/_default/params.yaml`
- 菜单配置: `config/_default/menus.yaml`
- 模块配置: `config/_default/module.yaml`
- 构建脚本: `build.sh` (Vercel)

### 注意事项

- `themes/` 目录已被 `.gitignore` 忽略，主题通过 Hugo Modules 管理
- `public/` 为构建输出，不要手动修改
- 部署配置需保持 Hugo 版本一致 (当前: 0.154.5)
- 主题已从 PaperMod 切换到 Congo，配置格式有所不同

---

## 变更记录 (Changelog)

| 日期 | 变更内容 |
|------|----------|
| 2026-01-16 | 更新文档：主题切换至 Congo，文章结构改为 Page Bundle 格式 |
| 2026-01-16 | 初始化 CLAUDE.md 文档 |
