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
| `params.yaml` | Congo 主题参数：主页布局、文章显示、页头页脚、作者信息等 |
| `menus.yaml` | 导航菜单配置 |
| `languages.yaml` | 多语言支持（当前仅 zh-Hans） |
| `module.yaml` | Hugo Modules 配置，引入 Congo 主题 |
| `markup.yaml` | Markdown 渲染配置：Goldmark、代码高亮、目录层级 |

## 关键依赖与配置

### 主题依赖

```yaml
# module.yaml
imports:
- path: github.com/jpanther/congo/v2
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
defaultAppearance: light       # 默认外观
autoSwitchAppearance: true     # 自动切换
homepage:
  layout: page                 # 主页布局
  showRecent: true             # 显示最新文章
article:
  showReadingTime: true        # 阅读时间
  showTableOfContents: true    # 目录导航
  showBreadcrumbs: true        # 面包屑导航
enableSearch: true             # 全局搜索
```

## 数据模型

无数据库，配置为静态 YAML 文件。

## 测试与质量

配置正确性通过 `hugo server` 或 `hugo` 命令验证，构建失败即表示配置错误。

## 常见问题 (FAQ)

**Q: 如何修改站点标题/描述？**
A: 编辑 `hugo.yaml` 的 `title` 和 `languages.yaml` 的 `description`。

**Q: 如何添加/修改导航菜单？**
A: 编辑 `menus.yaml`，添加条目并设置 `weight` 控制顺序。

**Q: 如何切换主题模式？**
A: 在 `params.yaml` 设置 `defaultAppearance` 为 `light` 或 `dark`，设置 `autoSwitchAppearance` 控制自动切换。

**Q: 如何升级主题版本？**
A: 运行 `hugo mod get -u github.com/jpanther/congo/v2`。

**Q: 如何修改主页布局？**
A: 在 `params.yaml` 设置 `homepage.layout`，可选值：`page`、`profile`、`hero`、`card`、`background`、`custom`。

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
| 2026-01-16 | 更新文档：主题从 PaperMod 切换至 Congo，更新配置说明 |
| 2026-01-16 | 初始化模块文档 |
