---
title: 新的博客
date: 2022-08-24 14:57:14
tags: 配置
categories: 工具 
---

# Github Page-Custom Domain

## 1.原因

众所周知`github`提供了一个`github-page`的服务，但是有些人很困惑为什么只能部署一个项目，但其实并不是这样的。`github-page`setting中的`custom domain`能很好的利用这个服务为我们部署多个项目。

本质上是利用`github`为每一个用户提供的一个个人站点(`username.github.io`)（相当于提供了静态服务器已经一个绑定的域名），进行`DNS`映射这样就能在剩下服务器的时候进行一些个人项目部署。

## 2.具体操作

#### 1.首先我们需要一个域名

如何获取域名自行解决，我是在阿里收购的域名提供商那边买的

#### 2.进行域名解析，使`username.github.io`与你购买的一级域相映射

这一步我是在阿里云的域名解析里做的，其他方式大同小异

1.在域名解析里采用CNAME的形式将一个对我自己的域名（zhugezhen.cn)和`sunboyzgz.github.io`

进行一个映射。

下一步在**setting**中的**pages**设置中操作

2.在**Build and deployment**的**source**选项下选择**deploy from a branch**，然后选择你要部署的目录，也就是你希望展示的`index.html`所存在的目录。不同于`deploy from a branch`github也为我们提供了一个**github actions**的选择，这里就是后面要讲的`github actions`。

3.选择一个`custom domain`这里就是我们自己购买的域名填入即可

**note**!!: 使用二级域名与`custom domain`相互映射的方案我们可以部署多个项目，并且这些项目并不需要都在`username.github.io`这个仓库下, 不过二级域名也是直接映射到`username.github.io`这个域。

这里假设我们的生成的二级域名是`book.zhugezhen.cn`

```
//域名解析中 CNAME形式，当然也可以使用ip的形式
zhugezhen.cn -> sunboyzgz.github.io
book.zhugezhen.cn -> sunboyzgz.github.io
//setting pages CNAME
custom domain save a zhugezhen.cn
custom domain save a book.zhugezhen.cn
```

# Github Actions

## 1.原因

本质上，这就是github为我们提供的一个CI/CD的能力

在`hexo`生成的blog项目中，我们虽然可以在本地进行`generate`然后将源代码同步到远程仓库的一个分支，然后再将需要展示的文件目录推送到另一个分支，但是这样会在每次操作时需要进行很多相同的重复操作。这项工作可以交给其他的工具进行比如[Travis CI](https://github.com/marketplace/travis-ci)，我只是抱着学习的态度想要尝试一下`action`。

## 2.如果你是hexo可以看下去，不是请跳过，直接到参考链接部分找寻答案

```yml
name: deploy blog to gh-pages  #这个是一个workflow name
on: #监听push的事件
  push:
    branches: #对于main分支的push事件进行监听
      - main
jobs: #workflow中的jobs列表，jobs默认并发进行，如需相互依赖参考 need 属性的使用
  generate-public: #job name
    runs-on: ubuntu-latest #运行的操作系统
    steps: #操作步骤一步一步向下执行
      - uses: actions/checkout@v3 #使用checkout action，提供摘取仓库的能力
      - name: Setup Node.js
        uses: actions/setup-node@v3 #一个配置node环境的action
        with:
          node-version: 16
      - name: Install packages
        run: npm install #就像在本地用使用命令行一样
        #这里要说一下，一个name下只能拥有一个run，有时可以像以下这样执行
      - name: Hexo generate public
        run: npx hexo clean && npx hexo generate
      - name: Add CNAME
        run: echo 'zhugezhen.cn' > ./public/CNAME
      - name: deploy
        uses: peaceiris/actions-gh-pages@v3 #将目标目录push gh-pages分支的action
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

这个`github actions`的配置其实就已经代替了`hexo`文档中https://hexo.io/zh-cn/docs/github-pages的操作说明了。

#### 3.参考链接

1.请先了解github actions:https://docs.github.com/cn/actions/learn-github-actions/understanding-github-actions

2.可以看看这里面的基本使用：https://github.com/jaywcjlove/github-actions

3.这个不重要随便看看就好：https://github.blog/2022-08-10-github-pages-now-uses-actions-by-default/