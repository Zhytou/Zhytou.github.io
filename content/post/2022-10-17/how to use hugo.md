---
title: "Hugo搭建博客"
date: 2022-10-17T13:34:43+08:00
---

一直想搞一个自己的网站，记录一下自己的想法。之前尝试用过[JekyII](https://jekyllrb.com/)和[Hexo](https://hexo.io/zh-cn/)，但研究了很久还是最终选择使用[Hugo](https://gohugo.io/),于是就有了这篇踩坑记录贴。

## Hugo

[Hugo](https://gohugo.io/)是一个静态网页生成器，能够根据/content目录和config.toml设置生成一些列Html文件供用户使用。

### Install Hugo

根据[Hugo](https://gohugo.io/getting-started/installing/)官网的教程，我们可以直接下载压缩包,解压到Hugo/bin文件夹中。

此外，可以将Hugo/bin添加至环境变量，方便后续使用。具体方法：我的电脑——属性——高级系统设置——环境变量——Path添加/Hugo/bin。

### Build your proj

``` bash
# 在当前路径中新建一个MySite文件夹，保存整个网站的相关文件
hugo new site MySite
```

### Choose a Hugo theme(Vitae)

``` bash
cd MySite
git init
git submodule add https://github.com/dataCobra/hugo-vitae themes/vitae
```

## Hugo-Vitae

[Vitae](https://github.com/dataCobra/hugo-vitae)是一种hugo主题，下面介绍一下怎么如何在config.toml中修改配置它。

``` toml
# 选择vitae主题
theme = 'vitae'

[params]

# 启用homepage
homepage = true

# 头像
avatar = 'https://avatars.githubusercontent.com/u/56868292'

# 次级标题
subtitle = 'May the force be with me.'

# 页面标题
pagetitle = 'May the force be with me.'

# Home tab显示/content/homepage.md
[[menu.main]]
name = "Home"
url = "/"
weight = 1

# Posts tab显示/content/post/*.md
[[menu.main]]
name = "Posts"
url = "/post"
weight = 2

# About tab显示/content/about.md
[[menu.main]]
name = "About"
url = "/about"
weight = 3

# github图标
[[params.social]]
name = "Github"
icon = "fab fa-github"
url = "https://github.com/Zhytou"

```

## Github Pages

用户以使用[Github Pages](https://docs.github.com/en/pages)来展示一些开源项目、主持博客甚或分享您的简历。

将Hugo生成的静态网页资源，推送到GitHub的repo中并构建网站主要有两种方法：

+ 可以将 GitHub Pages 站点配置为在将更改推送到特定分支时进行发布，
+ 也可以编写 GitHub Actions 工作流来发布站点。

两种方法相比，第一种配置简单好理解，但每次都要手动操作有点麻烦；另一种全自动，但配置相对复杂一点。

### Deploy on Github Pages using /docs

首先，在Hugo的配置文件config.toml中修改publishDir和baseUrl，具体如下：

``` toml
# for your user repository https://<USERNAME>.github.io 
# for a project repository https://<USERNAME>.github.io/<REPOSITORY_NAME> 
baseUrl = 'https://zhytou.github.io'
publishDir = 'docs'
```

接着，在Github相应的repo中修改设置，选择任意分支上的 docs 文件夹作为发布源，具体如下：

+ 在侧边栏"Code and automation" 中选择Pages。
+ 在“生成和部署”的“源”下，选择“从分支进行部署”。
+ 在“生成和部署”的“分支”下，使用“无”或“分支”下拉菜单并选择发布源。

最后，每次生成静态网页资源到docs文件夹中，然后push到remote repo即可。

``` bash
hugo -D
```

### Deploy on Github PageS using Github Action

首先，在根目录中添加/.github/workflows/gh-actions.yml,配置Github Action

``` yml
name: github pages

on:
  push:
    branches:
      - master  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
```

接着，设置repo,将Github Page的源设置为Github Action

整体流程就是每次push上去后，Github Action帮用户用Hugo在一个branch（gh-action）生成静态资源文件到docs，然后Github Pages根据这个branch生成。

## Reference

[Hugo quick start](https://gohugo.io/getting-started/quick-start/)
[Hugo host on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
[Github Pages Sources](https://docs.github.com/cn/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
