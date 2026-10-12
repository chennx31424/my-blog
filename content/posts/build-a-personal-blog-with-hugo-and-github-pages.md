---
title: "从零搭建个人博客：Hugo + GitHub Pages 完整新手教程"
date: 2026-10-12
draft: false
description: "在 Windows 上使用 Hugo、GitHub 和 GitHub Actions，从零搭建并免费发布一个个人静态博客。"
tags: ["Hugo", "GitHub Pages", "Git", "个人博客"]
categories: ["网站搭建"]
---

这篇文章记录如何在 Windows 上从零搭建一个个人博客，并免费发布到互联网。整套方案使用 Hugo、GitHub、GitHub Actions 和 GitHub Pages，不需要购买服务器或数据库，适合记录技术问题、学习笔记和日常生活。

本文面向第一次尝试搭建网站的读者。除了给出命令，还会简单解释每一步为什么要这样做。

## 一、最终使用的方案

整个博客由下面几部分组成：

- **Markdown**：用来编写文章。
- **Hugo**：把 Markdown、HTML 模板和 CSS 转换成静态网页。
- **Git**：记录文件的修改历史。
- **GitHub**：在线保存博客源文件。
- **GitHub Actions**：每次推送代码后，自动运行 Hugo 构建网站。
- **GitHub Pages**：免费托管构建完成的网页。

完整流程如下：

```text
本地编写 Markdown 文章
        ↓
Hugo 本地预览
        ↓
Git 提交并推送到 GitHub
        ↓
GitHub Actions 自动构建
        ↓
GitHub Pages 发布网站
```

这是一种“静态博客”方案。访问者打开网页时，服务器只需要返回已经生成好的 HTML、CSS 等文件，不需要实时查询数据库。因此它的成本低、速度快，安全维护也相对简单。

## 二、开始前需要准备什么

需要准备：

1. 一台 Windows 电脑。
2. 一个 GitHub 账号。
3. 可以正常使用的 PowerShell。
4. 基本的文件和文件夹操作能力。

本文示例中的博客目录是：

```text
D:\Blog
```

GitHub 用户名和仓库名使用下面的占位值：

```text
你的用户名
my-blog
```

实际操作时，要把“你的用户名”替换成自己的 GitHub 用户名。

## 三、安装 Git 和 Hugo

### 1. 安装 Git

Git 用来管理博客源文件的版本，并把文件推送到 GitHub。

打开 PowerShell，执行：

```powershell
winget install Git.Git
```

安装完成后，关闭 PowerShell，再重新打开一个 PowerShell 窗口。这样系统能够重新加载安装程序修改过的环境变量。

检查 Git 是否安装成功：

```powershell
git --version
```

如果能看到类似下面的输出，就说明安装成功：

```text
git version 2.56.0.windows.1
```

具体版本号可能不同，不需要完全一致。

### 2. 安装 Hugo Extended

Hugo 是静态网站生成器。这里安装 Extended 版本，因为它兼容更多主题和样式处理功能。

执行：

```powershell
winget install Hugo.Hugo.Extended
```

同样建议安装后重新打开 PowerShell，然后检查版本：

```powershell
hugo version
```

输出中应该包含 `extended`，例如：

```text
hugo v0.167.0+extended windows/amd64
```

如果提示 `hugo` 不是命令，通常是当前 PowerShell 还没有刷新环境变量。先关闭终端并重新打开，再执行一次。

## 四、创建 Hugo 博客项目

### 1. 创建并进入博客目录

执行：

```powershell
New-Item -ItemType Directory -Path D:\Blog -Force
Set-Location D:\Blog
```

如果 `D:\Blog` 已存在并且里面有重要文件，先确认文件内容，避免把不相关的项目混在一起。

### 2. 初始化 Hugo 站点

在 `D:\Blog` 中执行：

```powershell
hugo new site . --format yaml
```

命令中的 `.` 表示直接在当前目录创建站点，`--format yaml` 表示使用 YAML 格式保存 Hugo 配置。

创建后，可以执行下面的命令查看目录：

```powershell
Get-ChildItem
```

通常会看到：

```text
archetypes
assets
content
data
i18n
layouts
static
themes
hugo.yaml
```

这些目录的作用如下：

- `content`：保存 Markdown 文章。
- `layouts`：保存 HTML 页面模板。
- `static`：保存 CSS、JavaScript、图片等静态文件。
- `themes`：保存第三方 Hugo 主题。
- `archetypes`：保存新文章的默认模板。
- `public`：Hugo 构建后生成的网站文件。第一次构建后才会出现。
- `hugo.yaml`：整个网站的主要配置文件。

### 3. 初始化 Git 仓库

仍然在 `D:\Blog` 中执行：

```powershell
git init
```

这会创建一个隐藏的 `.git` 目录。Git 会通过这个目录记录项目历史，所以不要手动修改里面的文件。

第一次使用 Git 时，还需要配置提交者姓名和邮箱：

```powershell
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

这里的邮箱可以使用 GitHub 账号邮箱，也可以使用 GitHub 提供的隐私邮箱。

## 五、配置博客基本信息

打开项目根目录中的 `hugo.yaml`，写入：

```yaml
baseURL: http://localhost:1313/
locale: zh-CN
title: 我的个人博客

params:
  description: 记录技术问题、解决方案与日常生活

menu:
  main:
    - name: 首页
      pageRef: /
      weight: 10
    - name: 文章
      pageRef: /posts
      weight: 20
```

几个字段的含义：

- `baseURL`：网站的基础地址。本地开发阶段可以先写本地地址。
- `locale`：网站语言和地区，这里使用简体中文。
- `title`：网站名称。
- `params.description`：自定义的网站简介，稍后会在模板中读取。
- `menu.main`：定义顶部导航菜单。
- `weight`：菜单顺序，数值越小越靠前。

YAML 对缩进很敏感，请使用空格缩进，不要随意删除层级前的空格。

## 六、创建第一篇文章

执行：

```powershell
hugo new content posts/hello-world.md
```

Hugo 会在下面的位置创建文章：

```text
D:\Blog\content\posts\hello-world.md
```

打开文件，将内容改成：

```markdown
---
title: "Hello World"
date: 2026-10-12
draft: false
---

这是我的第一篇博客文章。
```

顶部两个 `---` 之间的内容叫作 Front Matter，用来保存文章元数据：

- `title`：文章标题。
- `date`：发布日期。
- `draft`：是否为草稿。

当 `draft: true` 时，正式构建默认不会发布这篇文章。准备发布时要改成 `false`。

## 七、为网站创建最小可用模板

Hugo 负责生成网站，但它并不知道页面应该长什么样。如果既没有安装主题，也没有在 `layouts` 中添加模板，启动后可能会看到 `Page Not Found`。

为了减少外部依赖，下面直接创建一套最小模板。理解这部分也有助于认识 Hugo 的工作原理。

### 1. 创建基础模板

创建文件：

```text
D:\Blog\layouts\_default\baseof.html
```

写入：

```html
<!doctype html>
<html lang="{{ site.Language.Locale | default "zh-CN" }}">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="{{ site.Params.description }}">
  <title>{{ if .IsHome }}{{ site.Title }}{{ else }}{{ .Title }} | {{ site.Title }}{{ end }}</title>
  <link rel="stylesheet" href="{{ "css/site.css" | relURL }}">
</head>
<body>
  <header class="site-header">
    <nav class="shell" aria-label="主导航">
      <a href="{{ site.Home.RelPermalink }}">{{ site.Title }}</a>
      <div>
        {{ range site.Menus.main }}
          <a href="{{ .URL }}">{{ .Name }}</a>
        {{ end }}
      </div>
    </nav>
  </header>

  <main class="shell">
    {{ block "main" . }}{{ end }}
  </main>

  <footer class="site-footer shell">
    <span>&copy; {{ now.Year }} {{ site.Title }}</span>
    <span>由 Hugo 构建</span>
  </footer>
</body>
</html>
```

`baseof.html` 是所有页面共享的外壳，包含 HTML 基础结构、导航、正文插槽和页脚。

模板中双大括号包围的内容是 Hugo 模板语法。例如：

- `{{ site.Title }}` 读取网站标题。
- `{{ range site.Menus.main }}` 遍历导航菜单。
- `{{ block "main" . }}` 为不同页面预留正文区域。

### 2. 创建首页模板

创建：

```text
D:\Blog\layouts\index.html
```

写入：

```html
{{ define "main" }}
  <section class="intro">
    <h1>{{ site.Title }}</h1>
    <p>{{ site.Params.description }}</p>
  </section>

  <section>
    <div class="section-heading">
      <h2>最近文章</h2>
      <a href="{{ "posts/" | relURL }}">查看全部</a>
    </div>

    <div class="post-list">
      {{ range first 10 (where site.RegularPages "Section" "posts") }}
        <article class="post-row">
          <div>
            <h3><a href="{{ .RelPermalink }}">{{ .Title }}</a></h3>
            {{ with .Summary }}<p>{{ . | plainify | truncate 120 }}</p>{{ end }}
          </div>
          <time datetime="{{ .Date.Format "2006-01-02" }}">
            {{ .Date.Format "2006-01-02" }}
          </time>
        </article>
      {{ else }}
        <p>还没有文章。</p>
      {{ end }}
    </div>
  </section>
{{ end }}
```

首页模板会取出 `posts` 栏目中最近的 10 篇文章。Hugo 构建时会自动把文章标题、摘要、日期和链接填入 HTML。

### 3. 创建文章列表模板

创建：

```text
D:\Blog\layouts\_default\list.html
```

写入：

```html
{{ define "main" }}
  <header class="page-heading">
    <h1>{{ .Title }}</h1>
  </header>

  <div class="post-list">
    {{ range .Pages.ByDate.Reverse }}
      <article class="post-row">
        <div>
          <h2><a href="{{ .RelPermalink }}">{{ .Title }}</a></h2>
          {{ with .Summary }}<p>{{ . | plainify | truncate 120 }}</p>{{ end }}
        </div>
        <time datetime="{{ .Date.Format "2006-01-02" }}">
          {{ .Date.Format "2006-01-02" }}
        </time>
      </article>
    {{ end }}
  </div>
{{ end }}
```

这个模板负责 `/posts/` 等列表页面，并按照日期倒序显示文章。

### 4. 创建文章详情模板

创建：

```text
D:\Blog\layouts\_default\single.html
```

写入：

```html
{{ define "main" }}
  <article class="article">
    <header class="article-header">
      <a href="{{ "posts/" | relURL }}">返回文章列表</a>
      <h1>{{ .Title }}</h1>
      <time datetime="{{ .Date.Format "2006-01-02" }}">
        {{ .Date.Format "2006年01月02日" }}
      </time>
    </header>

    <div class="content">
      {{ .Content }}
    </div>
  </article>
{{ end }}
```

`single.html` 负责单篇内容页面。`{{ .Content }}` 会被替换为 Markdown 转换后的 HTML。

### 5. 添加基础 CSS

创建目录和文件：

```text
D:\Blog\static\css\site.css
```

写入一份简单样式：

```css
:root {
  --paper: #f7f7f3;
  --surface: #ffffff;
  --ink: #20211f;
  --muted: #686b65;
  --line: #dedfd9;
  --accent: #087f5b;
  --max-width: 820px;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  color: var(--ink);
  background: var(--paper);
  font-family: "Segoe UI", "Microsoft YaHei", sans-serif;
  line-height: 1.7;
}

a {
  color: inherit;
  text-decoration: none;
}

a:hover {
  color: var(--accent);
}

.shell {
  width: min(calc(100% - 40px), var(--max-width));
  margin-inline: auto;
}

.site-header {
  border-bottom: 1px solid var(--line);
}

.site-header nav {
  min-height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.site-header nav div {
  display: flex;
  gap: 24px;
}

.intro {
  padding: 72px 0 52px;
  border-bottom: 1px solid var(--line);
}

.intro h1 {
  margin: 0 0 12px;
  font-size: 3rem;
}

.intro p,
.post-row p,
.article-header time {
  color: var(--muted);
}

.section-heading {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  padding-top: 40px;
}

.post-list {
  margin: 18px 0 64px;
  border-top: 1px solid var(--line);
}

.post-row {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 24px;
  padding: 22px 0;
  border-bottom: 1px solid var(--line);
}

.post-row h2,
.post-row h3,
.post-row p {
  margin: 0;
}

.page-heading,
.article {
  padding-top: 56px;
}

.article-header {
  padding-bottom: 24px;
  border-bottom: 1px solid var(--line);
}

.content {
  padding: 28px 0 64px;
  font-size: 1.05rem;
}

.content pre {
  overflow-x: auto;
  padding: 18px;
  border: 1px solid var(--line);
  background: var(--surface);
}

.site-footer {
  min-height: 88px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-top: 1px solid var(--line);
  color: var(--muted);
  font-size: 0.86rem;
}

@media (max-width: 600px) {
  .post-row {
    grid-template-columns: 1fr;
    gap: 6px;
  }
}
```

放在 `static` 目录中的文件会在构建时原样复制到 `public`。因此 `static/css/site.css` 最终会变成 `public/css/site.css`。

## 八、在本地预览博客

执行：

```powershell
Set-Location D:\Blog
hugo server
```

看到类似下面的信息时，说明预览服务器已启动：

```text
Web Server is available at http://localhost:1313/
```

在浏览器中打开：

```text
http://localhost:1313/
```

只要 `hugo server` 仍在运行，修改 Markdown、HTML 或 CSS 后，浏览器通常会自动刷新。

停止服务器时，在 PowerShell 窗口中按：

```text
Ctrl + C
```

### 如果看到 Page Not Found

常见原因有：

1. `layouts` 中没有页面模板，同时也没有安装主题。
2. 模板文件路径或名称写错。
3. 文章仍然是 `draft: true`。
4. Hugo 启动时不在博客根目录。

可以先执行一次正式构建来查看错误：

```powershell
hugo --cleanDestinationDir
```

如果构建成功，项目中会出现 `public` 目录，并生成 `public/index.html`。

## 九、设置 Git 忽略文件

Hugo 的 `public` 是自动生成的文件，不需要和源代码一起提交，因为 GitHub Actions 会重新生成它。

在项目根目录创建 `.gitignore`：

```gitignore
# Hugo 构建产物和本地预览文件
/public/
/.hugo_build.lock
/.hugo-server.log
/.hugo-server-error.log

# 操作系统和编辑器文件
.DS_Store
Thumbs.db
.vscode/
```

这样执行 `git add .` 时，上述文件不会进入 Git 仓库。

## 十、在 GitHub 创建空仓库

登录 GitHub，打开：

```text
https://github.com/new
```

按下面的方式创建：

1. `Repository name` 填写 `my-blog`。
2. 选择 `Public`。
3. 不要勾选 `Add a README file`。
4. 不要添加 `.gitignore`。
5. 不要选择 License。
6. 点击 `Create repository`。

之所以创建空仓库，是因为本地已经有完整项目。如果 GitHub 端提前创建了 README，本地和远程就会各自拥有不同的初始提交，新手在第一次推送时更容易遇到历史冲突。

## 十一、把生产地址写入 Hugo 配置

项目仓库形式的 GitHub Pages 地址通常是：

```text
https://你的用户名.github.io/my-blog/
```

将 `hugo.yaml` 中的 `baseURL` 修改为自己的实际地址：

```yaml
baseURL: https://你的用户名.github.io/my-blog/
```

末尾的 `/` 建议保留。由于站点发布在 `/my-blog/` 子路径下，正确的 `baseURL` 能帮助 Hugo 生成正确的资源和页面链接。

## 十二、配置 GitHub Actions 自动部署

创建目录和文件：

```text
D:\Blog\.github\workflows\hugo.yaml
```

写入：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "0.167.0"
          extended: true

      - name: Configure Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build site
        run: hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

这个文件定义了两个阶段：

1. `build`：下载仓库代码，安装指定版本的 Hugo，构建网站并上传构建产物。
2. `deploy`：把构建产物发布到 GitHub Pages。

`on.push.branches.main` 表示每次向 `main` 分支推送时自动运行。`workflow_dispatch` 则允许在 GitHub 页面上手动运行工作流。

如果本地安装的 Hugo 版本与示例不同，可以用下面的命令查看：

```powershell
hugo version
```

再把工作流中的 `hugo-version` 改成自己使用并验证过的版本。固定版本可以避免 GitHub 自动升级 Hugo 后出现意外兼容问题。

## 十三、进行第一次 Git 提交

先确认当前状态：

```powershell
git status
```

然后添加所有应提交的文件：

```powershell
git add .
```

创建第一次提交：

```powershell
git commit -m "Initial Hugo blog"
```

把默认分支名称统一为 `main`：

```powershell
git branch -M main
```

添加 GitHub 远程仓库：

```powershell
git remote add origin https://github.com/你的用户名/my-blog.git
```

可以使用下面的命令检查远程地址：

```powershell
git remote -v
```

最后推送：

```powershell
git push -u origin main
```

第一次推送时，Git Credential Manager 可能会打开浏览器，要求登录 GitHub 并授权。按照页面提示完成即可，不要把 GitHub 密码直接写进命令。

参数 `-u` 会让本地 `main` 跟踪远程 `origin/main`。以后在这个仓库中通常只需要执行：

```powershell
git push
```

## 十四、首次启用 GitHub Pages

第一次部署前，需要在仓库中手动启用一次 Pages：

1. 打开 GitHub 上的 `my-blog` 仓库。
2. 点击 `Settings`。
3. 在左侧找到 `Pages`。
4. 找到 `Build and deployment`。
5. 将 `Source` 选择为 `GitHub Actions`。

如果第一次工作流在启用 Pages 前运行，它可能会显示 `Get Pages site failed`。这不代表 Hugo 代码有问题，只是 Pages 站点还没有启用。

启用后，可以重新推送一次提交来触发部署。如果文件没有需要修改的内容，可以创建一个空提交：

```powershell
git commit --allow-empty -m "Trigger Pages deployment"
git push
```

## 十五、检查部署状态

打开 GitHub 仓库的 `Actions` 标签页。

应该能看到名为：

```text
Deploy Hugo site to Pages
```

的工作流。

状态含义：

- 黄色圆点：正在运行。
- 绿色对勾：部署成功。
- 红色叉号：部署失败，可以点击查看具体步骤和日志。

成功后，访问：

```text
https://你的用户名.github.io/my-blog/
```

首次部署一般需要等待一小段时间。如果刚部署完成仍然看到旧页面，可以刷新浏览器或等待缓存更新。

## 十六、以后如何发布新文章

### 1. 创建文章

```powershell
Set-Location D:\Blog
hugo new content posts/my-new-post.md
```

文件名建议使用小写英文和连字符，例如：

```text
solve-windows-path-problem.md
```

这样生成的 URL 更简单，也更容易分享。

### 2. 编写内容

```markdown
---
title: "解决 Windows 环境变量未生效的问题"
date: 2026-10-12
draft: false
tags: ["Windows", "开发环境"]
categories: ["技术问题"]
---

## 问题现象

描述看到的错误信息。

## 排查过程

记录检查过的步骤，以及哪些方法没有效果。

## 根本原因

解释问题为什么发生。

## 解决方案

给出最终有效的操作步骤。

## 验证结果

说明如何确认问题已经解决。
```

### 3. 本地预览

```powershell
hugo server
```

打开 `http://localhost:1313/` 检查文章排版、链接和代码块。

### 4. 构建检查

```powershell
hugo --cleanDestinationDir --minify
```

如果没有错误，就可以提交：

```powershell
git add .
git commit -m "Add article about Windows environment variables"
git push
```

推送完成后，GitHub Actions 会自动构建和发布，不需要手动上传 `public` 文件夹。

## 十七、常见问题

### 1. PowerShell 提示找不到 hugo

先关闭 PowerShell 并重新打开，再执行：

```powershell
hugo version
```

如果仍然找不到，可以检查：

```powershell
where.exe hugo
```

若没有输出，可以重新安装 Hugo Extended。

### 2. 本地首页显示 Page Not Found

检查：

- 是否在包含 `hugo.yaml` 的目录中运行 `hugo server`。
- `layouts/index.html` 是否存在。
- `layouts/_default/baseof.html` 是否存在。
- 模板文件是否有语法错误。

执行下面的命令通常能看到更具体的错误：

```powershell
hugo --cleanDestinationDir
```

### 3. 新文章没有显示

检查文章 Front Matter：

```yaml
draft: false
```

还要确认文章保存在：

```text
content/posts/
```

### 4. 本地正常，GitHub Pages 样式丢失

最常见原因是 `baseURL` 不正确。项目仓库 Pages 通常包含仓库名：

```yaml
baseURL: https://你的用户名.github.io/my-blog/
```

另外，在模板中尽量使用 Hugo 的 `relURL` 或 `.RelPermalink` 生成链接，不要把 `/css/site.css` 之类的根路径直接写死。

### 5. Git 推送时网络超时

先检查浏览器能否访问 GitHub，然后重新执行：

```powershell
git push
```

提交已经保存在本地时，推送失败不会丢失提交。网络恢复后再次推送即可。

### 6. GitHub Actions 成功，但网站仍然是 404

检查仓库：

```text
Settings → Pages → Build and deployment
```

确认 `Source` 已选择 `GitHub Actions`。还要确认访问地址包含正确的仓库名。

## 十八、备份和维护建议

虽然 GitHub 已经保存了一份博客源码，但仍建议偶尔把整个项目备份到其他磁盘。真正重要的是下面这些内容：

```text
content/
layouts/
static/
hugo.yaml
.github/
```

不要只备份 `public`，因为它只是 Hugo 生成的结果，Markdown 原文和模板才是能够长期维护的源文件。

图片数量较少时，可以直接放在：

```text
static/images/
```

然后在文章中使用：

```markdown
![图片说明](/my-blog/images/example.png)
```

更稳妥的做法是使用 Hugo 的相对链接处理方式，避免以后更换域名或部署路径时逐篇修改。

## 总结

完成以上步骤后，我们得到了一套不需要服务器和数据库的个人博客：

1. 使用 Markdown 写文章。
2. 使用 Hugo 在本地预览并生成静态网页。
3. 使用 Git 保存修改历史。
4. 使用 GitHub 保存博客源码。
5. 使用 GitHub Actions 自动构建。
6. 使用 GitHub Pages 免费发布。

这套架构的重点不是某一种页面样式，而是内容与展示相互分离：文章保存在 `content`，页面结构保存在 `layouts`，静态资源保存在 `static`。以后即使更换模板、主题或域名，Markdown 文章仍然可以继续使用。
