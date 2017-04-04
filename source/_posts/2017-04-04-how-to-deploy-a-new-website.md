title: 前端工程师全栈之路之部署环境搭建篇
date: 2017-04-04 16:00:07
categories: 教程
tags: Nginx Node Linux 腾讯云
description: 越来越多追求全栈的工程师情怀，本文偏实践兼顾理论的介绍全栈工程师入门级别如何部署一个应用
---

又是一年清明雨上，去年写[前端工程师学习Nginx实践配置HTTP2.0篇](http://cnt1992.xyz/2016/04/08/upgrade-nginx-to-http2/)的时候心想着有钱能买个服务器，一年过去了，我终于有了一个腾讯云的服务器！！！（虽然只有一个月免费试用期... ）。

这篇文章想要偏向实践兼顾理论地系统性介绍如何在一个全新的Linux服务器上部署一个Node应用，我们知道Node依然很火，同时也越来越多全栈情怀的工程师诞生，这篇文章以`Nginx + Node + Mongodb`的技术架构来介绍如何从0开始一步步部署一个应用。

首先，想要实践的童鞋得先有一台Linux服务器，土豪直接去**腾讯云**或者**阿里云**等云平台购买，像我优雅的舍不得花钱的申请一个月的免费试用玩玩。

<!-- more -->

## 一点点知识储备

有了云服务器之后，第一步是登录到这台服务器，最简单的命令就是打开一个**终端**，然后输入：

```shell
$ ssh 账号@服务器IP
## 输入密码之后直接回车
```

登录上去之后只有一个命令行界面，那么需要懂一些基本的**Linux**命令了，这里列举一下我们将要用到的命令（诸如：ls/cat/cd/vi这些命令就不介绍了），具体使用过程也可以回头看：

- ps：显示当前进程的状态，我们经常需要查看某个进程是否开启着，例如想查看所有进程信息：`ps -ef`，关于参数介绍可以直接输入命令行`ps --help all`查看具体含义
- grep：过滤或者搜索特定的字符，例如结合`ps`命令我们想过滤出**nginx**的进程，那么可以用：`ps -ef | grep nginx`
- wget/curl：这两个货都可以用来下载指定URL的内容，比如下载二进制安装包，用法上有一些差别，同时功能上也有一些区别，例如下载某个URL的内容：`wget URL`/`curl -O URL`
- tar：解压命令，我们一般下载了二进制压缩包之后需要解压进行编译，假如解压xxx.tar.gz包直接用：`tar zxvf xxx.tar.gz`
- yum：一个包管理器，可以理解为Mac平台下的brew，或者是Node下的npm，这货提供了查找，增加，删除软件包(.rpm)的功能，例如我们在安装**Nginx**之前需要安装一个**openssl**库，那么可以直接使用：`yum -y install openssl`进行安装

值得一提的一点是，我们不可能记住所有命令提供的参数，这时候要么网上搜索，要么直接就用Linux自带的`man`命令（当然全英的，英文也是程序员必备技能）。

一切准备就绪，<del>撸</del>搞起来。

## 安装Nginx

其实我们完全可以不用**Nginx**，无非就是**Node**应用起来之后在访问的时候需要多一个端口号而已。但为了不暴露我们实际应用的端口号，用**Nginx**来做反向代理是非常有必要的，**Nginx**的介绍可以翻看我一年前写的文章：[传送门1](http://cnt1992.xyz/2016/03/18/simple-intro-to-nginx/)/[传送门2](http://cnt1992.xyz/2016/04/08/upgrade-nginx-to-http2/)。在安装**Nginx**之前得先安装一些必要的编译工具及库文件：

```shell
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

> 上面安装的库列表如果有一个出错，那么后面的库就不会自动安装，所以如果你看到命令行有出错并且提示哪个库，就排查掉那个库再继续安装

紧接着安装**PCRE**，这家伙提供了Rewrite功能：

```shell
## 先下载二进制包
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz

## 解压
tar zxvf pcre-8.35.tar.gz

## 进入安装目录编译安装
cd pcre-8.35
./configure
make && make install
```

现在就可以正式安装**Nginx**了，跟上面安装同样的方法：

```shell
## 先返回上一级目录
cd ..

## 下载二进制包
wget http://nginx.org/download/nginx-1.10.3.tar.gz(截止本文最新的稳定版本)

## 解压
tar zxvf nginx-1.10.3.tar.gz

## 进入安装目录编译安装
cd nginx-1.10.3
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35
make && make install
```

安装成功之后可以执行 `/usr/local/nginx/sbin/nginx -v`查看版本，但我们不想每次都输那么一长串命令，所以可以简化一些命令：

```shell
## 编辑~/.bashrc文件
vi ~/.bashrc

## 在~/.bashrc文件找到alias直接输入o在下一行加入下面命令：
alias nginx='/usr/local/nginx/sbin/nginx'

## 然后按一下`esc`键，再`shift+;`，输入`x`保存退出，然后还要让配置文件生效：
source ~/.bashrc
```

## 配置Nginx

安装好了**Nginx**之后我们还需要配置一下，这里就直接命令带过：

```shell
## 创建www组跟www用户用于nginx的进程
/usr/sbin/groupadd www 
/usr/sbin/useradd -g www www
```

编辑**Nginx**配置文件（文件路径在`/usr/local/nginx/conf/nginx.conf`），我们先只改user，也就是第一行被注释的改成：

```shell
user www www;
```

其它的等下再改，保存退出之后启动**Nginx**，输入下面命令：

```shell
## 配置文件检查
nginx -t

## 上面检查没问题之后启动
nginx
```

这个时候直接浏览器输入你的IP就可以访问到**Nginx**的欢迎界面了。

## 安装Node

在安装**Node**之前，其实我推荐先安装[nvm](https://github.com/creationix/nvm)，**Node**版本控制器，可以安装指定版本的**Node**同时又可以随时切换不同版本，只需要一条命令自动安装：

```shell
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash
```

上面命令成功之后，执行以下`source ~/.bashrc`让**nvm**命令生效，然后我们就可以愉快地安装最新稳定版的**Node**了：

```shell
nvm install v6.10.1
```

由于我们是第一次安装了**Node**，所以**nvm**会自动将这个版本的**Node**作为默认版本，这时可以输入`node -v`/`npm -v`看看我们安装的版本。

## 安装Mongodb

听说用**Node**的都很喜欢用**Mongodb**数据库？那么我们这个例子也安装一下：

```shell
## 下载二进制包
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz

## 解压
tar -zxvf mongodb-linux-x86_64-3.0.6.tgz 

## 直接将解压包移动到指定目录
mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb
```

**Mongodb**可执行的命令放在`/usr/local/mongodb/bin/`下面，为了方便不带这么长的前缀，可以将路径添加到**PATH**里面：

```shell
export PATH=/usr/local/mongodb/bin:$PATH
```

紧接着创建数据库目录，执行下面命令：

```shell
mkdir -p /data/db
```

> -p参数会自动级联创建文件夹，也就是data文件夹不存在先创建data文件夹再创建db文件夹

当然，如果你不想用默认的数据库目录，也可以指定`--dbpath=xxx`来启动，这时启动**Mongodb**：

```shell
mongod
```

## 上传代码

> 这里我们为了演示直接采用本地代码直接上传，实际项目中我们一般把代码提交到**github**然后利用`github hook`触发代码拉取。

假设我们本地代码路径在`~/node/test`，那么可以直接用`scp`命令上传到我们的服务器路径`/node/app`：

```shell
scp -r ~/node/test root@服务器IP:/node/test
## 输入密码回车
```

上传成功之后就可以用**Node**命令启动了，比如我的代码启动之后就下面这样子：

![node应用启动](/img/node-deploy/node_deploy.png)

然后直接在浏览器打开：[http://服务器IP:9990]()，就可以看到你的应用启动了！好了，结束，拜~

等等，作为有追求的全栈工程师，这还没完，这离我们想要的效果还有些出入：

- 访问可不可以去掉端口号？
- 启动**Node**应用就一直挂着命令行?

带着这两个问题，我们继续。

## 安装pm2

嗯，[pm2](https://github.com/Unitech/pm2)作为**Node**应用管理器，在生产环境下强烈建议使用，本地开发的建议使用[nodemon](https://github.com/remy/nodemon)，能够监听文件改动并自动重启应用。

**pm2**只是一个**npm**包，安装一下无痛感使用：

```shell
npm i -g pm2
```

这个时候我们就可以愉快地使用**pm2**来启动我们的应用了：

```shell
pm2 start index.js
```

更多**pm2**的命令可以阅读上面的链接。

## Nginx反向代理

要去掉端口号访问很简单，只需要利用**Nginx**反向代理到**Node**应用即可，直接在`/usr/local/nginx/conf/nginx.conf`找到`location`直接添加下面：

```shell
location /app/ {
    proxy_pass http://127.0.0.1:9990;
}
```

然后执行下面命令重新加载**Nginx**配置文件：

```shell
nginx -s reload
```

> 如果执行上面命令出现 `nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"`错误提示，那么先执行以下`nginx -c /usr/local/nginx/conf/nginx.conf`即可

这时就可以愉快的用 [http://IP/app]() 访问应用了。