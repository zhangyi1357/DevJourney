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


## 部署指南

使用 [Github Pages 部署](https://www.mkdocs.org/user-guide/deploying-your-docs/) mkdocs 文档相当简单，只需要将项目作为 GitHub 项目存储，然后在命令行运行如下命令：

```bash
mkdocs gh-deploy
```

接下来就会自动完成 GitHub Pages 的部署工作。

部署完成的网站链接为：[Yi's DevJourney](https://zhangyi1357.github.io/DevJourney/)

不过需要注意的 GitHub Pages 有[用量限制](https://docs.github.com/zh/pages/getting-started-with-github-pages/about-github-pages#usage-limits)，摘录如下：

- GitHub Pages 源仓库的建议限制为 1 GB。有关详细信息，请参阅 [关于 GitHub 上的大文件](https://docs.github.com/zh/repositories/working-with-files/managing-large-files/about-large-files-on-github#file-and-repository-size-limitations)
- 发布的 GitHub Pages 站点不得超过 1 GB。
- 如果花费的时间超过 10 分钟，GitHub Pages 部署将超时。
- GitHub Pages 站点的软带宽限制为每月 100 GB。
- GitHub Pages 站点的软限制为每小时 10 次生成。 如果使用自定义 GitHub Actions 工作流生成和发布站点，则此限制不适用。
- 为了为所有 GitHub Pages 站点提供一致的服务质量，可能会实施速率限制。 这些速率限制无意干扰 GitHub Pages 的合法使用。 如果你的请求触发了速率限制，你将收到相应响应，其中包含 HTTP 状态代码 `429` 以及信息性 HTML 正文。

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

以上问题可以通过配置文件解决[^1]，在 `mkdocs.yml` 文件中添加如下配置即可：

```yaml
markdown_extensions:
  - toc:
      slugify: !!python/name:pymdownx.slugs.uslugify
```

使用以上配置需要安装 `pymdown-extensions`，命令行如下：

```bash
pip install pymdown-extensions
```

[^1]:[https://squidfunk.github.io/mkdocs-material/setup/extensions/python-markdown/#+toc.slugify](https://squidfunk.github.io/mkdocs-material/setup/extensions/python-markdown/#+toc.slugify)

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