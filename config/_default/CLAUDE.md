[根目录](../../CLAUDE.md) > **config/_default**

# config/_default - Hugo 站点配置模块

## 模块职责

管理 Hugo 站点的全局配置，包括站点元信息、主题参数、菜单导航、语言设置、Markdown 渲染规则及主题模块引入。

## 入口与启动

Hugo 自动加载 `config/_default/` 目录下的所有 YAML 配置文件。主入口为 `hugo.yaml`。

## 配置文件说明

| 文件 | 职责 |
|------|------|
| `hugo.yaml` | 核心配置：baseURL、标题、语言、分页、隐私设置、分类法、输出格式 |
| `params.yaml` | PaperMod 主题参数：主题切换、社交链接、搜索、封面图、编辑链接等 |
| `menus.yaml` | 导航菜单配置 |
| `languages.yaml` | 多语言支持（当前仅 zh-Hans） |
| `module.yaml` | Hugo Modules 配置，引入 PaperMod 主题 |
| `markup.yaml` | Markdown 渲染配置：Goldmark、代码高亮、目录层级 |

## 关键依赖与配置

### 主题依赖

```yaml
# module.yaml
imports:
- path: github.com/adityatelange/hugo-PaperMod
```

### 核心站点配置

```yaml
# hugo.yaml (摘要)
baseURL: https://xwls.github.io/
title: Wang's Blog
languageCode: zh-Hans
timeZone: Asia/Shanghai
enableRobotsTXT: true
```

### 主题关键参数

```yaml
# params.yaml (关键项)
defaultTheme: auto          # 主题切换
ShowReadingTime: true       # 阅读时间
ShowCodeCopyButtons: true   # 代码复制
showtoc: true               # 目录导航
```

## 数据模型

无数据库，配置为静态 YAML 文件。

## 测试与质量

配置正确性通过 `hugo server` 或 `hugo` 命令验证，构建失败即表示配置错误。

## 常见问题 (FAQ)

**Q: 如何修改站点标题/描述？**
A: 编辑 `hugo.yaml` 的 `title` 和 `params.yaml` 的 `description`。

**Q: 如何添加/修改导航菜单？**
A: 编辑 `menus.yaml`，添加条目并设置 `weight` 控制顺序。

**Q: 如何切换主题模式？**
A: 在 `params.yaml` 设置 `defaultTheme` 为 `light`、`dark` 或 `auto`。

**Q: 如何升级主题版本？**
A: 运行 `hugo mod get -u github.com/adityatelange/hugo-PaperMod`。

## 相关文件清单

- `hugo.yaml` - 核心配置
- `params.yaml` - 主题参数
- `menus.yaml` - 菜单配置
- `languages.yaml` - 语言配置
- `module.yaml` - 模块配置
- `markup.yaml` - Markdown 渲染配置

---

## 变更记录 (Changelog)

| 日期 | 变更内容 |
|------|----------|
| 2026-01-16 | 初始化模块文档 |
