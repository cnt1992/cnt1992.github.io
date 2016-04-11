title: 前端工程师学习Nginx实践配置HTTP2.0篇
date: 2016-04-08 15:41:54
categories: 教程
tags: Nginx HTTP2.0 前端 
description: HTTP2.0很火？想知道Nginx如何配置HTTP2.0么？跟着我一步步实际操作体验秒开网页。
---

## 科普时间

作为有 `全栈工程师` 情节并且 **乐于折腾** 的我在上一篇 [前端工程师学习Nginx入门篇](http://cnt1992.xyz/2016/03/18/simple-intro-to-nginx/) 之后继续捣鼓将 `Nginx` 配置支持 `HTTP2.0`，这是为了等以后有钱可以买个服务器的时候直接配置服务器体验一把网页`秒开`的快感，为啥配置`HTTP2.0`可以秒开网页？请静心往下看(图片摘自网络，如有侵略版权内容请联系我删除哈)：

### `HTTP 2.0` 带来了哪些新特性 ？

> `HTTP 2.0` 带来了很多新的特性，这里不会一一详细介绍，只列举了几项，如有兴趣研究的童鞋请移步 [官方RFC文档](https://tools.ietf.org/html/rfc7540)。

1.**增加**二进制分帧

`HTTP`协议从`0.9`版本开始不断增加新的功能特性，但长远来看都是 `向前兼容` 的，`HTTP 2.0` 在 `应用层` 跟 `传输层` 之间增加了一个 `二进制分帧层`，从而能够达到 “在不改动HTTP的语义、HTTP方法、状态码、URI及首部字段的情况下，突破HTTP 1.1的性能限制，改进传输性能，实现低延迟和高吞吐量。”

![HTTP 2.0二进制分帧层](/img/nginx-http/binary.png)

如上图所示，在 `二进制分帧层` 上， `HTTP 2.0` 会将所有传输的信息分割为更小的消息和帧，并对它们采用二进制格式的编码，其中 `HTTP 1.1` 的首部信息会被封装到 `Headers` 帧，而 `request body` 被封装到 `Data` 帧里面。

2.压缩头部

如下图所示：`HTTP 2.0` 在 `客户端` 和 `服务端` 使用 **首部表** 来跟踪和存储之间发送的 `键-值` 对，对相同请求而言不需要再次发送请求和相应发送，通信期间几乎不会改变的通用 `键-值` （如：用户代理、可接受的媒体类型）只需发送一次。

- 如果请求不包含首部（如：对同一资源的轮询请求），那首部开销为零字节
- 如果首部发生变化，那只需发送变化的数据在 `Headers` 帧里面，新增或修改的 `首部帧` 会被追加到 **首部表**

![HTTP 2.0头部压缩](/img/nginx-http/header.png)

3.多路复用

记得大学 `计算机网络` 跟 `计算机组成原理` 这两门课都讲到了 `多路复用` 这个概念，当时一知半解（可能是老师讲得太抽象了o(╯□╰)o）。相比一个入了门的前端开发在谈到 `性能优化` 的方法时都可以轻轻松松列举如下几点：

* CSS雪碧图合并 - 减少请求
* 合并压缩CSS跟JavaScript代码 - 减少请求
* CSS代码放在header头部里面，JavaScript代码放到body结束之前 - 因为JavaScript代码执行会阻塞

然后我们可以自豪地晒出下面的代码片段：

```
<!DOCTYPE HTML>
<html>
    <head>
        <link rel="stylesheet" href="xxx.cdn.com/??a.css,b.css" />
    </head>
    <body>
        ...
        <script src="xxx.cdn.com/??a.js,b.js"></script>
    </body>
</html>
```

但 `HTTP 2.0` 的 `多路复用` 让我们回到了最原始最自然的写码状态，先看下图：

![HTTP 2.0 多路复用](/img/nginx-http/multiplexing.png)

对 `HTTP 1.1` 而言，浏览器通常有链接的限制，即使开启多个链接，也需要付出相应的代码，而 `多路复用` 允许同时通过单一的 `HTTP 2.0` 连接发起多重的 `请求-相应` 消息。

这意味着 `HTTP 2.0` 的通信都在一个连接上完成了，这个连接可以承载任意数量的 **双向数据流** ，直观来说，就是我们又可以开开心心写出下面代码：

```
<!DOCTYPE HTML>
<html>
    <head>
        <link rel="stylesheet" href="a.css" />
        <link rel="stylesheet" href="b.css" />
    </head>
    <body>
        ...
        <script src="a.js"></script>
        <script src="b.js"></script>
    </body>
</html>
```

<!-- more -->

4.请求优先级

既然所有资源都可以并行交错发送，会不会导致下面这种情况呢？

> 浏览器： 服务器，请给我需要的CSS文件
> 服务器： Biu，发送图片中...
> 浏览器： 服务器，请给我需要的JS文件
> 服务器： Biu，发送图片中...
> 浏览器： 服务器，你TM的倒是给我发送我需要的CSS文件跟JS文件呀！

这个时候 `HTTP 2.0` 的 `请求优先级` 特性出场了，在每个 `HTTP 2.0` 的 `流` 里面有个 **优先值** ，这个 **优先值** 确定着客户端跟服务器处理不同的 `流` 采取不同的 **优先级策略** ，高优先级的应该优先发送，但这不会绝对的（绝对等待会导致 `首队阻塞` 问题）。在分配处理资源和客户端与服务器间的宽带，不同优先级的混合都是必须的。

5.服务器提示

一般情况下，客户端需要请求啥东西告诉服务器，然后服务器返回对应资源回到客户端，这也是请求很慢的原因之一。`HTTP 2.0` 新增加 `服务器提示` ，可以先于客户端检测到将要请求的资源，提前通知客户端，服务器不发送所有资源的实体，只发送资源的 `URL`。客户端接到提示后会进行验证缓存，如果发现需要这些资源，则正式发起请求。

这个技术跟我们常用的 `预加载` 技术实现差不多，但 `服务器提示` 是通过 `HTTP Link Header` 和 `Link Prefetching` 语义重叠的部分来实现的。

### Nginx 如何实现 HTTP 2.0 ?

打开 [Nginx官网](http://nginx.org/) 可以看到其中有一条新闻如下：

![Nginx的HTTP 2.0模块](/img/nginx-http/nginx-release-http2.0.png)

显然，`Nginx` 是已经有了一个 `ngx_http_v2_module` 模块用来支持 `HTTP 2.0` 的，但是还不是稳定版，所以想要尝鲜那就需要 **手动升级Nginx** 。

科普时间到此结束，下面就让我们正式进入今天的主题： `一步步实现Nginx配置HTTP 2.0`。

## Nginx配置HTTP 2.0正式起航

> 下面内容有条件的同学请对比实践操作，体会更深。

我将从下面几步来介绍：

- 准备工作
- 源码安装升级 Nginx 到最新版（当前是1.9.14）
- 生成 `SSL/TLS` 安全证书
- 修改Nginx配置

### 准备工作

在开始我们的任务之前，请先下载 [OpenSSL](https://www.openssl.org/source/openssl-1.0.2g.tar.gz)，[pcre](http://downloads.sourceforge.net/project/pcre/pcre/8.38/pcre-8.38.tar.gz?r=https%3A%2F%2Fsourceforge.net%2Fprojects%2Fpcre%2Ffiles%2Fpcre%2F8.38%2F&ts=1459928198&use_mirror=liquidtelecom)，[Zlib](http://zlib.net/zlib-1.2.8.tar.gz ) 跟 [Nginx源码](http://nginx.org/download/nginx-1.9.14.tar.gz) ，并且全部解压到同一个目录，假设为 `nginx-build`，这时候的代码结构如下：

```
nginx-build
  nginx-1.9.14
  pcre-8.38
  openssl-1.0.2g
  zlib-1.2.8
```

### 源码安装升级 Nginx 到最新版（当前是1.9.14）

> 若之前用 `brew` 安装过 `Nginx` ，请先执行 `brew uninstall nginx` 卸载。 

准备工作做好之后，打开 `Terminal` 切换到上面的 `nginx-build` 目录，然后执行下面命令编译Nginx：

```
# 先进入nginx源码目录
cd nginx-1.9.14

# 配置Nginx源码
# --prefix：配置nginx最终安装目录，可以修改为你想要的目录
# --with-http_ssl_module 跟 --with-http_v2_module 必带，因为 HTTP 2.0 采用 HTTPS ，HTTPS 基于 SSL/TLS
sudo ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-pcre=../pcre-8.38 --with-openssl=../openssl-1.0.2g --with-zlib=../zlib-1.2.8 --with-http_v2_module
> 输入管理员密码然后回车
```

配置源码成功之后，为了避免 `make` 失败，因为我失败了（可能是人品不好...），反正我 `make` 的时候报了下面的错误：

![Nginx make 出错](/img/nginx-http/make-error.png)

所以建议大家跟我一样先修改一个配置文件，如下命令(相对于`nginx-build`目录)：

```
cd nginx-1.9.14/objs
sudo vi Makefile
> 输入管理员密码然后回车
# 在 vi 模式下输入 '/' ，然后输入./config --prefix定位到类似我下面的片段：
# && ./config --prefix=/Users/cainengtian/Downloads/software/nginx-1.8.0/../openssl-1.0.2d/.openssl no-shared  \
# 将前面的 ./config 改为 ./Congigure darwin64-x86_64-cc ，其他的不要改动， 然后保存即可
```

接着回到`nginx-build`目录，然后执行下面命令编译：

```
cd nginx-1.9.14
sudo make && make install
```

如果没有报错那就编译成功，在终端试试 `sudo nginx` 启动，然后打开 [http://localhost](http://localhost) 能看到欢迎页，然后在终端输入 `nginx -v` 看到 `nginx version: nginx/1.9.14`。

### 生成 `SSL/TLS` 安全证书

要配置 `HTTP 2.0` 就需要启动 `HTTPS`，那就需要生成一个 `SSL/TLS` 证书，而 `OpenSSL` 正好可以做这件事情，买不起证书但自己造一个来临时用还是可以的，直接执行下面命令：

```
# 切换到nginx的根目录（我这里是/usr/local/nginx，请根据上一步编译时候指定的--prefix对应修改）
cd /usr/local/nginx
cd conf

# 利用openssl命令生成公钥跟密钥
openssl genrsa -des3 -passout pass:x -out cert.pass.key 2048
openssl rsa -passin pass:x -in cert.pass.key -out cert.key
openssl req -new -key cert.key -out cert.csr
openssl x509 -req -days 365 -in cert.csr -signkey server.key -out cert.crt

# 将crt跟key合并生成pem文件
cat server.crt server.key > server.pem

# 删除掉我们不需要用到的文件
rm -rf cert.crt cert.csr
```

### 修改Nginx配置

离成功之后一步之遥了，接下来就是最重要的一步，配置Nginx支持HTTP 2.0，修改Nginx配置文件`/usr/local/nginx/conf/nginx.conf`： 

1.将所有HTTP请求重定向到HTTPS请求

```
location / {
    return 301 https://$host$remote_port$request_uri;
}  
```

2.配置HTTPS server

```
server {
    listen 443 ssl http2 default_server;
        server_name  localhost;
        ssl_certificate      cert.pem;
        ssl_certificate_key  cert.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        # 我用原始的下面这段启动报错了，所以注释掉改用了后面那段
        #ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_ciphers 'CHACHA20:EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH:ECDHE-RSA-AES128-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA128:DHE-RSA-AES128-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA128:ECDHE-RSA-AES128-SHA384:ECDHE-RSA-AES128-SHA128:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES128-SHA128:DHE-RSA-AES128-SHA128:DHE-RSA-AES128-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA384:AES128-GCM-SHA128:AES128-SHA128:AES128-SHA128:AES128-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4;';
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }
```

可以注意到上面 `HTTPS` 的配置比之前多了一个 `http2` ，这就是 `ngx_http_v2_module` 模块，若没有安装该模块启动一样会报错。

3.重启服务器

配置完成之后，重新启动服务器即可：

```
sudo nginx -s reload
```

4.预览

打开 [http://localhost](http://localhost) ，自动跳转到 [https://localhost](https://localhost) ，Done！

## 一点思考

虽然 `HTTP 2.0` 也已经不算是非常新的东西，要普及确实也需要很长一段时间，希望不久将来有钱买个服务器部署到自己网站上。

只有动手实践才知道看起来简单的东西实际上还是会碰到很多坑，anyway，一直在成长的路上~