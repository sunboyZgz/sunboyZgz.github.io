---
title: 新的篇章
date: 2022-08-23 21:39:31
tags: 配置
categories: 工具
---
# 为什么会有这篇文章？
由于最近c盘紧张，我重装了一次系统同进行了一些卷容量的调整，因为最近没有怎么写过blog在进行格式化的时候将原来搭建的个人blog给清除了而且也没有做备份，git page上传的也只是一个经过处理后的html文件，所以我原来的blog文件都丢失了。因为我需要进行一些环境配置，所以写下了这篇文章。

# nodejs + npm + pnpm 配置

## 1.windows 下安装nodejs

https://nodejs.org/en/

按照步骤安卓即可，注意一下nodejs的安装目录即可

## 2.配置npm

```powershell
npm config get prefix
```

这条命令可以直接获取你的全局安装路径的前缀,这个前缀会用于

```powershell
npm install -g xxxx
```

```powershell
npm config set prefix "D:\Program Files\nodejs\node_global"

npm config set cache "D:\Program Files\nodejs\node_cache"
```

#### 环境配置

1.

`NODE_PATH    D:\Program Files\nodejs\node_global\node_modules`

`NODE_PATH` 用于进行全局的模块检索，例如加载npm全局下的`express`模块的时候

2.

`PATHD D:\Program Files\nodejs\node_global`

## 3.配置pnpm

注意此项可以不用配置，在没有配置的情况下会在当前卷的根目录创建`node_modules`进行管理。

如果想要pnpm跨磁盘或者卷进行管理可以按如下设置

```powershell
pnpm config set store-dir /path/to/.pnpm-store
```

参考链接：https://www.pnpm.cn/configuring、https://www.pnpm.cn/faq#store-path-is-not-specified

#### 4.修改install的镜像源

这里就是淘宝的，要是用别的请搜索对应uri

```powershell
npm config set registry https://registry.npmmirror.com
```

```powershell
npm config get registry
```



# python anaconda

## 1.安装anaconda

大陆用户推荐用清华的源进行安装解决安装包下载过慢的问题

https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/

## 2.解决一些典型问题（待补充）

#### 1.`[pip is configured with locations that require TLS/SSL]`

https://stackoverflow.com/questions/45954528/pip-is-configured-with-locations-that-require-tls-ssl-however-the-ssl-module-in

一般我们都会把 python解释器和pip的路径加入环境，但是这里我们也需要加入第三个路径到环境中。（这里需要的原因可能是我在使用anaconda这个发型版本）

## 3.修改local cache

下面这篇博文讲述了为什么需要这个cache

https://www.techiediaries.com/python-pip-local-cache/

配置方法:在`%HOME%`的`pip`目录下创建`pip.ini`（没有目录就自建一个）

找到`%HOME%`, 可以在文件资源管理器的路径栏输入`homepath`

```ini
[global]
cache-dir = E:/languages/anaconda3/mypip/cache
log-file = E:/languages/anaconda3/mypip/pip.log
```

## 4.修改pip安装源

https://mirrors.tuna.tsinghua.edu.cn/help/pypi/

#### 5. 检查一下配置情况

```powershell
pip config list
```

![image-20220823141458429](E:\tools\assets\image-20220823141458429.png)

## 6. 2022/8/27补充:

error:

> solving environment: failed with intitial frozen solve.Retrying with flexible solve.

![image-20220827185106780](E:\blog\sunboy\source\_posts\assets\image-20220827185106780.png)

#### 方案一：直面困难

1.修改`conda`安装的镜像源为后续操作提速

具体修改方案可以参考清华提供的方案`.condarc`

windows下使用命令生成在`%home%`目录下生成

```powershell
conda config --set show_channel_urls yes
```

然后将下方链接的中的配置复制到`.condarc`中

https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

2. ```powershell
   conda update --all
   ```

#### 方案二：曲线救国

使用pip install 安装，但是这样的话不能享受到conda create env带来的好处。但是真的简单。

#### 方案三：降级conda策略

网上自搜

# go

这个就是最为简单的了

去官网找个安装包跟着官网走就好了：https://go.dev/doc/install

安装好之后推荐修改一下`GOPATH`和`GOBIN`,目前版本` GO111MODULE`默认是设为`on`

小小的看一下`GOPATH`的目录结构，以及功能：http://c.biancheng.net/view/88.html

## 1.在（用户环境变量）中修改GOPATH

把对应的变量改为你自己的目录路径即可

#### 2.GOBIN

这个可以不做，我做这个主要是让后期`install`的文件和`bin`目录下的开发者工具相隔离

#### 3.修改GOCACHE

我也是简单地修改了一下`GOCACHE`,我是放在`GOPATH`之下的

```powershell
go env -w GOCACHE=yourpath
```

## 4.设置代理

https://cloud.tencent.com/developer/article/1773630