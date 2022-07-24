---
title: Hexo 搭建 & 使用指南
date: 2022-05-17 23:27:57
categories: 服务搭建
tags: Hexo
---

## 什么是Hexo
> Hexo 是一个可以易于上手、部署的静态博客架构。使用node.js作为构建引擎，插件库丰富，可扩展性好。支持MarkDown作为书写语言，可以满足日常博客编写需求

## Hexo 安装
安装Hexo前，需要本地环境已经安装有这两个东西:
* Node.js
* Git

(这二者的安装就不在这里单独赘述了)

如果你的环境没有问题了，那么就可以开始安装Hexo了:

```bash
npm install hexo-cli -g
```

安装完成后，进入要作为博客资源的目录，执行:

```bash
hexo init blog  # 初始化配置
cd blog
npm install  # 安装依赖包
hexo server  # 在本地启动一个临时服务，可以用来预览和调试
```

## 撰写博文
撰写博文也非常简单，在`blog`目录下执行:

```bash
hexo new post <title>
```

例如想要创建一篇名为`hello`的博文:

```bash
hexo new post hello
```

生成的文件路径是:

```bash
INFO Created: <blog-dir>/source/_posts/hello.md
```

打开文件，我们可以post模板已经自动帮我们生成了`yaml`文件头。其中`title`是文章的标题，`tags`是文章的标签。

编辑好文章内容后，执行`hexo server`就可以看见这篇文章已经发布到博客中了

## Hexo 部署
上文提到的`hexo server`的方式，只是在本地搭建了一个临时服务器，只能在本地访问。Hexo本身是一个静态页面的博客系统，因此对服务器的要求极低，Github免费提供的`Github Pages`就足以满足需求

### 1.创建托管仓库
这里建议创建一个名为`<username>.github.io`的仓库来托管我们的网页(因为正常情况下，我们创建的仓库的github page的url是`<username>.github.io/<repo name>/`, 在域名之后会有一级仓库名称，这会导致很多页面、主题的相对路径功能出现问题；而使用这种仓库名和域名相同的方式创建的仓库的page的url中就不会包含仓库名)

### 2. 部署配置
在博客的根目录下有一个`_config.yml`文件，这是博客的主配置文件。最后的`Deployment`配置项代表我们要使用的部署方式，如果我们要使用Git方式部署，那么需要安装一个hexo-deployer-git插件:

```bash
npm install hexo-deployer-git --save
```

然后编辑这里的配置:

```bash
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: <repo url>  # 建议使用ssh的方式
  branch: [branch]  # 默认可不填
  message: [message]  # 默认可不填
```

保存配置文件后，执行:

```bash
hexo generate
hexo deploy
```

上述命令可以简化成:

```bash
hexo g -d
```

现在已经将博客部署到Github了。可以用过`http://<username>.github.io`来访问自己的博客了

## Hexo 配置

在博客的主配置文件中除了配置部署相关的内容，还可以配置一些自定义的配置:

```bash
# Site
title: ''  # 站名
subtitle: ''  # 副标题
description: ''  # 对搜索引擎收录博客会有帮助
author: ''
language: zh-Hans  # 配置中文显示或者英文显示
timezone: ''
```

```bash
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: https://yoursite.com/ 
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```
这部分配置只需要将url配置成自己的博客地址即可，例如部署在github page，就可以填 `https://<username>.github.io/project`

```bash
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: ayer  
```
主题配置，Hexo社区有大量的主题可供选择, 可以参考[官方列表](https://hexo.io/themes/)

## Hexo 主题
主题的使用也非常简单，这里以`AYER`主题为例。

```bash
git clone https://github.com/Shen-Yu/hexo-theme-ayer.git themes/ayer
```

把你想要的主题`git clone`下来，放到`themes/`下对应的目录中，然后我们修改博客主配置中`theme`配置，保存后，再次执行`hexo server`命令，预览一下变化

主题的配置一般放在`themes/<themename>/_config.yml`文件下，具体配置参考[官方文档](https://shen-yu.gitee.io/2019/ayer/)即可
