# MkDocs 项目构建、配置说明

## 构建指南

本项目使用 mkdocs 生成文档，使用 mkdocs-material 主题。

下面提供逐步构建文档的指南：

1. 安装依赖

   ```bash
   pip install mkdocs
   pip install mkdocs-material
   pip install pymdown-extensions
   pip install mkdocs-git-revision-date-localized-plugin
   pip install mkdocs-git-authors-plugin
   ```

2. 本地构建预览

   ```bash
   mkdocs serve
   ```

   然后访问 [http://127.0.0.1:8000](http://127.0.0.1:8000) 即可预览文档。


## 配置项

### 中文符号引用问题

在 Markdown 文档中引用标题链接时，需要使用 `#` 符号，如下：

```markdown
[Material 主题配置](#Material-主题配置)
```

然而这样的格式在网站中访问会出现问题，由于默认的标题链接规则仅支持 ASCII 字符，实际的标题链接规则如下编写：

```markdown
[Material 主题配置](#Material)
```

这样在 Markdown 文件中就无法正常跳转，本地预览会出问题。另外这样的编写方式也不符合逻辑上的需要，试想两个标题分别为 “Material 主题配置” 和 “Material 笔记指南”，在网页端两个标题的链接分别为 `#Material` 和 `#Material`，显然这样的重复链接会导致无法正确找到正确的引用。

以上问题可以通过[配置文件](https://squidfunk.github.io/mkdocs-material/setup/extensions/python-markdown/#+toc.slugify)解决，在 `mkdocs.yml` 文件中添加如下配置即可：

```yaml
markdown_extensions:
  - toc:
      slugify: !!python/name:pymdownx.slugs.uslugify
```

使用以上配置需要安装 `pymdown-extensions`，命令行如下：

```bash
pip install pymdown-extensions
```

### Material 主题配置

Material 主题有很多配置选项，具体可以参考 [MkDocs Material 文档](https://squidfunk.github.io/mkdocs-material/setup/)。

当前项目配置直接从 MkDocs Material 文档自身构建的官方 GitHub 仓库的[配置文件](https://github.com/squidfunk/mkdocs-material/blob/master/mkdocs.yml)中复制，具体配置如下：

```yaml
theme:
  name: material
  language: zh
  features:
    - announce.dismiss
    - content.action.edit
    - content.action.view
    - content.code.annotate
    - content.code.copy
    - content.tooltips
    - navigation.footer
    - navigation.indexes
    - navigation.sections
    - navigation.tabs
    - navigation.top
    - navigation.tracking
    - search.highlight
    - search.share
    - search.suggest
    - toc.follow
  palette:
    - media: "(prefers-color-scheme)"
      toggle:
        icon: material/link
        name: Switch to light mode
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/toggle-switch
        name: Switch to dark mode
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: black
      accent: indigo
      toggle:
        icon: material/toggle-switch-off
        name: Switch to system preference
  font:
    text: Roboto
    code: Roboto Mono
```

## 参考资料

- [MkDocs 文档](https://www.mkdocs.org/user-guide/writing-your-docs/)
- [MkDocs Material 文档](https://squidfunk.github.io/mkdocs-material/getting-started/)