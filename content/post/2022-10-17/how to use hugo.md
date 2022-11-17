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

使用Hugo创建一个新项目，具体方法如下。

``` bash
# 在当前路径中新建一个MySite文件夹，保存整个网站的相关文件
hugo new site MySite
```

### Choose a Hugo theme(Vitae)

选择一个[Hugo主题](https://themes.gohugo.io/)，并拷贝到本地。

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

[Github Pages](https://docs.github.com/en/pages)是Github提供给用户，用于展示一些开源项目、博客甚至是简历的工具。我们可以使用Hugo构建自己的博客，并使用Github Pages来向其他人展示。这主要有两种方法实现：

+ 可以使用Hugo生成静态资源文件到/docs目录，并设置Github repo根据/docs生成网页。
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
# generate static files
hugo -D
# serve without generating static files
hugo serve
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
        if: github.ref == 'refs/heads/master'
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
```

注意：GITHUB_TOKEN 不是 personal access token，因此可以不用其他设置就可运行。

接着，设置repo,将Github Page的源设置为gh pages的root目录

整体流程就是每次push到master分支后，Github Action触发。自动使用Hugo生成静态资源文件，并且将其commit到另一个branch（gh pages）中，最后Github Pages根据这个branch生成。

## Additional infomation

### Add your own domain name for github pages

简单来说，就是到大平台（阿里云、华为云或者腾讯云）购买域名，接着配置域名解析，使其能够转到你的github pages即可。具体可以参考[这里](https://www.zhihu.com/question/31377141)。

此外，注意当使用github actions部署网站时，要将CNAME文件添加在`publish_dir`中，否则个人域名会配置失败。具体可以参考[这里](https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-add-cname-file-cname)。

### Add icon for your website

首先，访问[favicon design](https://favicon.io/favicon-generator/)设计图标。

接着，下载生成文件，并将文件解压到根目录static文件夹中。

最后，有些主题需要修改html文件（Vatie不用），一般来说应该加一行`<link rel="shortcut icon" href="/favicon.ico">`就OK了。具体可以参考这篇[博文](https://ibrights.github.io/post/blog20210527/)。

## Reference

+ [Hugo quick start](https://gohugo.io/getting-started/quick-start/)

+ [Hugo host on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

+ [Github actions for hugo](https://github.com/marketplace/actions/hugo-setup)

+ [Github pages sources](https://docs.github.com/cn/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

+ [Add your own domain name for your github pages](https://www.zhihu.com/question/31377141)

+ [Add CNAME file in the publish_dir](https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-add-cname-file-cname)

+ [Add icon for your website](https://ibrights.github.io/post/blog20210527/)
