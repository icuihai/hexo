---
title: 使用Hexo搭建免费Blog
date: 2018-03-15 15:25:40
tags: [Hexo,Blog]  
---

Hexo是基于Node.js开发的静态博客生成器，界面简洁优雅，深受程序员的喜爱。接下来我就简单介绍一下Hexo搭建的过程。

#### 使用工具

Node.js当然是必须的了，直接到官网下载 <https://nodejs.org/zh-cn/>

Git GitHub这两个不用说，程序员必备

#### 安装Hexo

**创建博客文件夹**

首先需要在你的电脑上新建一个文件夹(以后写博客所有的数据都在这个文件夹中)

比如我的直接在根目录新建一个hexo文件夹~~

**初始化**

在hexo文件夹中执行以下命令

`hexo init`

**安装依赖包**

这里会用到npm，最新版的node.js会自动安装npm

执行命令

`sudo npm install -g hexo`

到这里安装部分已经完成了，接下来就该到设置我们的GitHub了

####配置GitHub

新建repository,注意仓库名字最好以your_user_name.github.io命名。仓库建立好之后就可以跟我们的hexo关联起来了，首先打开hexo根目录下的_config.yml文件（sublime或者typora）找到deploy: 然后修改如下 

```html
deploy:
  type: git
  repo: 
	github: git@github.com:icuihai/icuihai.github.io,master
  	coding: git@git.coding.net:icuihai/icuihai.coding.me,master`
```
github一栏则是你刚刚创建的仓库，coding则是国内一个比较知名的代码仓库（不用也可以）

#### 部署

执行命令

```html
npm install hexo-deployer-git --save 
```

push到GitHub

```html
hexo deploy
```

到这一部已经基本完成，直接在浏览器输入你刚才创建的github仓库地址

```html
icuihai.github.io
```

#### 总结

使用Hexo搭建Blog操作简单，当面上面介绍的只是最基本的操作，如果你想让的你的博客更优雅，可以使用Coding（国内访问使用Coding,国外访问GitHub），也可以拥有一个你自己的域名（推荐狗爹）。从此之后就可以开开心心的写博客啦~~~