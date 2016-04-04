title: 前端工程师学习Nginx入门篇
date: 2016-03-18 14:10:49
categories: 教程
tags: Nginx
description: 本文手把手带领前端工程师走进Nginx配置
avatar: "https://avatars2.githubusercontent.com/u/3118988?v=3&s=40"
---

![Nginx Logo](/img/nginx/nginx.png)

## What is Nginx

`Nginx`(发音：engine X)是一款轻量级的`HTTP`服务器（相比于Apache、Lighttpd而言），同时是一个高性能的`HTTP`和[反向代理](#tips_1)服务器，如今国内主流网站基本搭建于`Nginx`之上，诸如新浪、腾讯、网易、豆瓣。

`Nginx`主要以**事件驱动**的方式编写，有兴趣可以移步[这里](https://github.com/nginx/nginx)看他们的源码，这让它拥有非常好的性能，同时也是一个非常高效的反向代理、负载均衡（不知道反向代理跟负载均衡的童鞋请自觉移步文章结尾恶补一下）。

[官方站点](http://nginx.org/en/)也指出了`Nginx`作为HTTP服务器的几项基本特性：

- 处理静态文件，索引文件以及自动索引；打开文件描述符缓冲
- 无缓存的反向代理加速，简单的负载均衡和容错
- FastCGI，简单的负载均衡和容错
- 模块化的结构，包括gzipping,byte ranges,chunked responses,以及SSI-filter等filter。
- 支持SSL和TLSSNI.

对于前端童鞋而言，可能基本不会碰到服务器的东西，但如果像我这样子有『全栈工程师』心结的话倒是可以研究一下，自己成功配置Nginx启动自己的服务，以后再碰到这些关于Nginx的问题自己能够解决，丰衣足食~

接下来我将手把手教大家从安装到配置，搭建起Nginx环境，走起~

<!-- more -->

## 安装并启动Nginx

由于我是用Mac办公的，所以安装`Nginx`是采用`brew`进行的，在`终端`输入下面命令安装好`Nginx`：

```bash
# 强烈建议每次brew安装软件的时候先执行brew update保持软件依赖包都是最新的
brew update && brew install nginx
```

紧接着就可以用浏览器打开[http://localhost:8080](http://localhost:8080)看到`Nginx`的欢迎信息。

跟`Linux`系统有些不同，在`Mac`下面`Nginx`默认监听了`8080`端口号，若强迫症（比如我）不希望每次打开网页都要输入端口号的话，那么请在`终端`执行下面命令：

```bash
# 下面的1.8.0请根据最新安装版本号对应修改
sudo chown root:wheel /usr/local/Cellar/nginx/1.8.0/bin/nginx
sudo chmod u+s /usr/local/Cellar/nginx/1.8.0/bin/nginx

# 用vi编辑器打开nginx配置文件，找到server字段的listen字段并将其值修改为80
vi /usr/local/etc/nginx/nginx.conf
```

修改完上面`配置信息`之后，执行下面命令检查配置文件语法是否有误并且重新加载配置：

```bash
nginx -t && nginx -s reload
```

更多关于`Nginx`命令的帮助可以输入`nginx -h`查看，若想每次开机自动开启`Nginx`，在`终端`执行下面命令即可：

```bash
ln -sfv /usr/local/opt/nginx/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
```

## Nginx配置不完全详解

`Nginx`能否发挥淋漓尽致，就看配置文件了，由于`Nginx`配置实在太多，不能一一解释，有兴趣移步[官方文档](http://nginx.org/en/docs/)。接下来我会给大家解释比较重要的配置，下面是我机器上的`Nginx`的配置文件（我会带上注释）：

| 强烈建议大家先打开自己的默认Nginx配置跟我的对比来看
| 可以在`终端`执行 `cat /usr/local/etc/nginx/nginx.conf.default` 查看默认配置文件

```
# user字段表明了Nginx服务是由哪个用户哪个群组来负责维护进程的，默认是nobody
# 我这里用了cainengtian用户，staff组来启动并维护进程
# 查看当前用户命令： whoami
# 查看当前用户所属组命令： groups ，当前用户可能有多个所属组，选第一个即可
user cainengtian staff;

# worker_processes字段表示Nginx服务占用的内核数量
# 为了充分利用服务器性能你可以直接写你本机最高内核
# 查看本机最高内核数量命令： sysctl -n hw.ncpu
worker_processes 4;

# error_log字段表示Nginx错误日志记录的位置
# 模式选择：debug/info/notice/warn/error/crit
# 上面模式从左到右记录的信息从最详细到最少
error_log  /usr/local/var/logs/nginx/error.log debug;

# Nginx执行的进程id,默认配置文件是注释了
# 如果上面worker_processes的数量大于1那Nginx就会启动多个进程
# 而发信号的时候需要知道要向哪个进程发信息，不同进程有不同的pid，所以写进文件发信号比较简单
# 你只需要手动创建，比如我下面的位置： touch /usr/local/var/run/nginx.pid
pid  /usr/local/var/run/nginx.pid;

events {
    # 每一个worker进程能并发处理的最大连接数
    # 当作为反向代理服务器，计算公式为： `worker_processes * worker_connections / 4`
    # 当作为HTTP服务器时，公式是除以2
    worker_connections  2048;
}

http {
    # 关闭错误页面的nginx版本数字，提高安全性
    server_tokens off;
    include       mime.types;
    default_type  application/octet-stream;

    # 日志记录格式，如果关闭了access_log可以注释掉这段
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                 '$status $body_bytes_sent "$http_referer" '
    #                '"$http_user_agent" "$http_x_forwarded_for"';

    # 关闭access_log可以让读取磁盘IO操作更快
    # 当然如果你在学习的过程中可以打开方便查看Nginx的访问日志
    access_log off;

    sendfile        on;

    # 在一个数据包里发送所有头文件，而不是一个接一个的发送
    tcp_nopush     on;

    # 不要缓存
    tcp_nodelay on;

    keepalive_timeout  65;

    gzip  on;
    client_max_body_size 10m;
    client_body_buffer_size 128k;

    # 关于下面这段在后面紧接着来谈！
    include /usr/local/etc/nginx/sites-enabled/*;
}
```

## Nginx配置最佳实践

上面的`配置文件`最后一行`include`关键词会将`/usr/local/etc/nginx/sites-enabled/`文件夹下面的所有文件都加载进当前的配置文件，这样子就可以将配置文件分离，`nginx.conf`这个`配置文件`修改之后以后基本不会修改，配置不同站点的时候只需要在`/usr/local/etc/nginx/sites-enabled/`不断增加新的文件即可，这是比较好的配置方式。

比如我在`/usr/local/etc/nginx/sites-enabled/`下面增加了两个文件，用来配置普通的`HTTP`服务还有`HTTPS`服务：

```bash
touch /usr/local/etc/nginx/sites-enabled/default
touch /usr/local/etc/nginx/sites-enabled/default-ssl
```

### default配置解析

`Nginx`整个配置的结构大致如下：

```
...
events {
    ...
}
http {
    ...
    server {
        ...
        location xxx {
            ...
        }
    }
}
```

对比上面我的`nginx.conf`文件可以知道`default`文件的内容就是配置`server`部分的，下面先弄一份最基本的配置（带有详细说明）：

```
server {
    # Nginx监听端口号
    listen       80;
    # 服务器的名字，默认为localhost，你也可以写成aotu.jd.com，这样子就可以通过aotu.jd.com来访问
    server_name  localhost;
    # 代码放置的根目录
    root /var/www/;
    # 编码
    charset utf-8;    

    location / {
        # index字段声明了解析的后缀名的先后顺序
        # 下面匹配到/的时候默认找后缀名为php的文件，找不到再找html，再找不到就找htm
        index index.php index.html index.htm;
        # 自动索引
        autoindex on;
        # 这里引入了解析PHP的东西
        include /usr/local/etc/nginx/conf.d/php-fpm;
    }    

    # 404页面跳转到404.html，相对于上面的root目录
    error_page  404              /404.html;
    # 403页面跳转到403.html，相对于上面的root目录
    error_page  403              /403.html;
    # 50x页面跳转到50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

上面的配置的意思就是：访问[http://localhost](http://localhost)『80端口号可以直接省略』的时候会在`/var/www/`下面找`index.php`文件，如果没有找到就找`index.html`，如果再没有找到那就找`index.htm`，如果还是没有找到的话就`404`跳转到`404.html`，如果你刚好将`/var/www/`设置为`root`用户访问的话，那么就会直接无访问权限`403`跳转到`403.html`。

值得注意的是`server`字段里面的`root`字段，这个字段需要跟`alias`字段区分开来，通过下面两段配置解释一下：

```
# 当用root配置的时候，root后面指定的目录是上级目录
# 并且该上级目录必须含有和location后指定的名称的同名目录，否则404
# root末尾的"/"加不加无所谓
# 下面的配置如果访问站点http://localhost/test1访问的就是/var/www/test1目录下的站点信息
location /test1/ {
    root /var/www/;
}

# 如果用alias配置，其后面跟的指定目录是准确的，并且末尾必须加"/"，否则404
# 下面的配置如果访问站点http://localhost/test2访问的就是/var/www/目录下的站点信息
location /test2/ {
    alias /var/www/;
}
```

大家在实践过程中注意区分即可，配置之后要是碰到`404`可以先考虑是否是这个原因。

### 配置反向代理

对于前端工程师而言，可能最容易成为`全栈`的技能就是`NodeJS`了，当我们用`express`框架写好了一个`Node`应用之后，比如启动的时候的访问地址是：`http://localhost:3000/`，但是在部署到服务器上去之后，我们当然不希望别人这样子访问，最好的情况肯定是隐藏掉端口号。

例如我有一个`Node`服务的名字是`o2blog_wx`，在启动`Node`的时候访问的地址是：`http://localhost:3000/`，但是对外网我们希望是：`http://aotu.jd.com/o2blog_wx`，接下来我们将通过`Nginx`进行配置（带有详细注释）。

```
server {
    listen 80;
    server_name aotu.jd.com;
    root /var/www/;
    location /o2blog_wx/ {
        # 反向代理我们通过proxy_pass字段来设置
        # 也就是当访问http://aotu.jd.com/o2blog_wx的时候经过Nginx反向代理到服务器上的http://127.0.0.1:3000
        # 同时由于解析到服务器上的时候o2blog_wx这个字段都要处理
        # 所以通过rewrite字段来进行正则匹配替换
        # 也就是http://aotu.jd.com/o2blog_wx/hello经过Nginx解析到服务器变成http://127.0.0.1:3000/hello
        proxy_pass http://127.0.0.1:3000;
        rewrite ^/o2blog_wx/(.*) /$1 break;
    }
}
```

### 配置临时跳转

有时候我们觉得一开始配置的URL不好想换掉，但又不想原先的链接失效，比如一开始对外网的链接是：http://aotu.jd.com/o2blog_wx/，后来想改成http://aotu.jd.com/wxblog，又不想原先的失效。

这个时候可以在`Nginx`上配置一个`302`临时跳转，如下（`server`部分跟前面的一样）：

```
location /o2blog_wx/ {
    # 当匹配到http://aotu.jd.com/o2blog_wx/的时候会跳转到http://aotu.jd.com/wxblog
    return 302 http://aotu.jd.com/wxblog
}
```

### 配置限制访问

在一台服务器上的资源不全部都是对外开放的，这个时候就需要通过`Nginx`配置一个限制访问，比如查看本服务器的`PHP`信息，我们就可以通过下面配置来实现限制访问：

```
# 当匹配到/info的时候只允许10.7.101.224访问，其它的全部限制
# 同时改写为/info.php
location = /info {
    allow 10.7.101.224;
    deny all;
    rewrite (.*) /info.php
}
```

这个时候只有`IP`为`10.7.101.224`的机器才可以访问：http://aotu.jd.com/info，其它机器都会`403`拒绝访问！

当然最佳的实践是将`IP`抽取出来变成白名单，这样子就可以实现部分`IP`可以访问，其它的不能访问。

## default-ssl 配置解析

我们都知道`HTTP`在传输的过程中都是明文的，这直接导致了在传输的任何一个过程中都容易被窃取信息，所以才有了`SSL`（安全套接层）以及升级版`TLS`（传输层安全协议）的出现，其实就是在`HTTP`应用层给`TCP/IP`传输层的中间增加了`TLS/SSL`层，统称为`HTTPS`。

那如何通过`Nginx`配置`HTTPS`站点呢，下面就是`default-ssl`配置文件的内容（详细解析）：

```
server {
    # 默认情况下HTTPS监听443端口
    listen  443 ssl;
    server_name  localhost;
    root  /var/www/;
    # 下面这些都是配置SSL需要的
    ssl on;
    # 下面两个字段需要的crt利用openssl生成，具体可以看[这里](http://nginx.org/en/docs/http/configuring_https_servers.html)
    ssl_certificate ssl/localhost.crt;
    ssl_certificate_key ssl/localhost.key;

    ssl_session_timeout 10m;

    ssl_protocols SSLv2 SSLv3 TLSv1;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;


    location = /info {
        allow 127.0.0.1;
        deny all;
        rewrite (.*) /info.php;
    }

    location /phpmyadmin/ {
        root /usr/local/share/phpmyadmin;
        index index.php index.html index.htm;
    }

    location / {
        include /usr/local/etc/nginx/conf.d/php-fpm;
    }

    error_page 403 /403.html;
    error_page 404 /404.html;
}
```

上面配置之后，就可以通过[https://localhost](https://localhost)访问我们的`Nginx`首页了。

当然若要在对外网使用，必须购买第三方信任证书才行，有兴趣的童鞋可以谷歌了解，这里不细谈。

## 小结

写到这里，最基本的`Nginx`配置就基本介绍完了，若按照我上面的配置一步步跟着改，基本上都可以跑起来`Nginx`服务了吧，若想更加深入学习`Nginx`的配置，强烈建议看[官方文档](http://nginx.org/en/docs/)，写得很清晰明了，还是那句老话：`授之以鱼不如授之以渔`。

## 反向代理

提到`反向代理`，必然先提到`正向代理`，正向代理(forward)是一个位于客户端【用户A】和原始服务器(origin server)【服务器B】之间的服务器【代理服务器Z】，为了从原始服务器取得内容，用户A向代理服务器Z发送一个请求并指定目标(服务器B),然后代理服务器Z向服务器B转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。如下图（图来自网络，如有侵权请联系我删除~）：

![正向代理示意图](/img/nginx/forwards_proxy.jpg)

从上图可以看出，所谓的`正向代理`就是`代理服务器替代访问方【用户A】去访问目标服务器【服务器B】`，在现实中的例子就是『翻墙』！但如果代理服务器Z被完全控制（或不完全控制），就变成了『肉鸡』了。

而`反向代理`与正向代理相反，对客户端而言代理服务器就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间（name-space）中的内容发送普通请求，接着反向代理将判断向何处（原始服务器）转交请求，并将获得的内容返回给客户端。

使用反向代理服务器主要核心作用如下：

1. 保护和隐藏原始资源服务器

![反向代理原理图](/img/nginx/backward_proxy_1.jpg)

从上图可以看出，用户A始终认为它访问的是代理服务器Z而不是原始服务器B，但实际上反向代理服务器接受用户A的应答，从原始资源服务器B中取得用户A的需求资源，然后发送给用户A。由于防火墙的作用，只允许代理服务器Z访问原始资源服务器B。尽管在这个虚拟的环境下，防火墙和反向代理的共同作用保护了原始资源服务器B，但用户A并不知情。

2. 负载均衡

![反向代理负载均衡示例图](/img/nginx/backward_balance.jpg)

当反向代理服务器不止一个的时候，我们甚至可以把它们做成集群，当更多的用户访问资源服务器B的时候，让不同的代理服务器Z（x）去应答不同的用户，然后发送不同用户需要的资源。

当然`反向代理服务器`像`正向代理服务器`一样拥有CACHE的作用，它可以缓存原始资源服务器B的资源，而不是每次都要向原始资源服务器B请求数据，特别是一些静态的数据，比如图片和文件，如果这些反向代理服务器能够做到和用户X来自同一个网络，那么用户X访问反向代理服务器X，就会得到很高质量的速度。这正是`CDN技术`的核心。如下图：

![CDN原理图](/img/nginx/cdn.jpg)