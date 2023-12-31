---
title: Hello World
date: 2021-02-14
tags: Hexo
categories: 我的博客
description: Welcome to Hexo
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

[TOC]

## Quick Start

###  Install hexo
``` bash
$ npm install -g hexo-cli
```
### Create myblog
``` bash
$ hexo init myblog
```

**出现 hexo init 失败问题，[解决方法](https://blog.csdn.net/qq_43580193/article/details/117341489?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-0&spm=1001.2101.3001.4242)替换Github连接**

### 将Hexo部署到GitHub
```
deploy:
  type: git #冒号后面有空格
  repository: git@github.com:xuhuigeren/xuhuigeren.github.io.git  #ssh/https仓库地址
  branch: master
```
**回到 myblog 文件夹中，打开 Git Bash，安装Git部署插件，输入命令**

```
 npm install hexo-deployer-git --save
```

**然后分别输入以下三条命令：**

```bash
hexo clean   #清除缓存文件 db.json 和已生成的静态文件 public
hexo g       #生成网站静态文件到默认设置的 public 文件夹(hexo generate 的缩写)
hexo d       #自动生成网站静态文件，并部署到设定的仓库(hexo deploy 的缩写)
```

**完成以后，打开浏览器，输入 https://xuhuigeren.github.io  就可以打开你的网页了**

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
