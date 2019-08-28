title: 服务端预渲染在可视化搭建业务的实践
date: 2019-08-28 15:25:47
categories:
tags:
description:
---

最近在做`可视化搭建`的业务，为了追求页面的极致渲染，对比了`SSR`等服务端渲染方案之后，决定用比较轻量的`预渲染`来加速页面渲染速度，期间碰到的一些坑记录下。

<!-- more -->

关于 `CSR` / `SSR` / `同构` / `预渲染` 这几种渲染方式都有各自的优缺点，在我做的`可视化搭建`业务里面，`SSR`太“重”不适合，`同构`Node容易成为瓶颈，所以比较适合的是`预渲染`，这是以最小的代价达到相对比较好的收益。

### 如何预渲染

预渲染有各种手段可以实现，一般都用现成的谷歌维护的`Puppeteer`启动隐身浏览器去提前访问页面生成一份“快照”，真实用户访问时会直接将“快照”从服务端返回去，加快了`FCP`帧的时间。

拿我自己的例子来说，在页面发布阶段，通过`无头浏览器`提前访问页面拿到需要的`DOM`/`STYLE`数据发布到`OSS`去，渲染时从`OSS`取好数据放到模板里面返回，这样子渲染的`HTML`就不再是只有空的一个 `<div id="app"></div>` ，而是有一份完整的 `DOM`，具体时序图如下：

![预渲染时序图](https://gw.alicdn.com/tfs/TB16_hYeAT2gK0jSZPcXXcKkpXa-1732-1426.png)

### 预渲染/无预渲染 对比图

> todo: 待大量数据压测，目前只是直观数据

#### 无预渲染
FCP时间：1226.6ms，可以看到中间有一段“白屏”

![无预渲染.png](https://gw.alicdn.com/tfs/TB1N.VQepT7gK0jSZFpXXaTkpXa-2956-1594.png)


#### 有预渲染
FCP时间: 567.9ms，基本没有“白屏”

![预渲染.png](https://gw.alicdn.com/tfs/TB1PWphep67gK0jSZPfXXahhFXa-2952-1632.png)

### 问题记录

#### 部署到服务器之后提示chromium启动失败

```
"Failed to launch chrome!
/${app_path}/node_modules/_puppeteer@1.18.1@puppeteer/.local-chromium/linux-672088/chrome-linux/chrome: error while loading shared libraries: libX11.so.6: cannot open shared object file: No such file or directory


TROUBLESHOOTING: https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md
"
```

**原因：linux机器chromium缺少动态依赖库导致（不像mac/windows，linux服务器无UI）**

解法：

1、[可选，也可以直接执行第二步安装所有依赖]上服务器执行下面的命令查看缺少哪一些动态依赖库，关于**ldd**命令介绍可戳[这里](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/ldd.html)

```bash
# 下面的appName替换成自己的应用名称，或者直接从错误日志里面直接复制也行
cd /home/admin/${appName}/target/${appName}
ldd ./node_modules/_puppeteer@1.18.1@puppeteer/.local-chromium/linux-672088/chrome-linux/chrome | grep not
```

2、根据缺少的依赖库利用 **yum** 进行安装，将安装的命令写到 `config/docker/Dockerfile` (本应用采用了egg框架，其他类型应用自行修改) ，如下所示：

```bash

...

# linux下puppeteer依赖库安装
RUN yum -y install libX11 libXcomposite libXcursor libXdamage libXext libXi libXtst cups-libs libXScrnSaver libXrandr alsa-lib pango atk at-spi2-atk gtk3 
RUN yum -y install ipa-gothic-fonts xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi xorg-x11-utils xorg-x11-fonts-cyrillic xorg-x11-fonts-Type1 xorg-x11-fonts-misc

...
```

3、提交代码重新部署即可

#### 本地跟部署之后的结果不一致

现象：本地调试时根据`puppeteer`获取到的`html`内容符合期望，但是部署到服务器之后只能获取到原始的`html`而非异步渲染完成之后的`html`

原因排查思路：

1、是否mac跟linux的chromium导致异步渲染问题？经过demo测试排除了环境差异
2、服务器无hosts绑定，是否服务器请求异常？利用`puppeteer`的劫持测试发现本地发起的请求数量跟服务端发起的请求数量确实不一致，而且服务端少了关键的数据请求。进一步直接在服务器测试是否所有请求链接都是通的，发现其中有一个域名是不通的，这时候基本定位出原因了：服务端的机器跟日常的域名是不通的导致了请求的资源挂了所以没有走到最后数据渲染那一步

解法思路：

利用`puppeteer`的劫持重定向，下面是解法代码片段

```javascript
...

// 绕过CSP，同时设置劫持
await page.setBypassCSP(true);
await page.setRequestInterception(true);

page.on('request', interceptedRequest => {
  const reqUrl = interceptedRequest.url();
  // 为了隐藏业务信息，下面的 error-host-cdn / correct-host-cdn 只是替代了需要重定向的cdn地址
  if (reqUrl.includes('error-host-cdn')) {
    const newReqUrl = reqUrl.replace(
      'error-host-cdn',
      'correct-host-cdn'
    );
    interceptedRequest.continue({
      url: newReqUrl,
    });
  } else {
    // 其他请求继续走
    interceptedRequest.continue();
  }
});

...
```

#### 劫持接口数据如何返回？

由于预渲染好的`DOM`是不能带有任何跟用户相关信息，所以接口的返回数据也需要屏蔽掉数据，所以对`XHR/FETCH`请求做了一层劫持如下代码：

```javascript
page.on('request', interceptedRequest => {
  const reqUrl = interceptedRequest.url();
  if (reqUrl.includes('error-host-cdn')) {
    const newReqUrl = reqUrl.replace(
      'error-host-cdn',
      'correct-host-cdn'
    );
    interceptedRequest.continue({
      url: newReqUrl,
    });
  } else if (resourceType === 'fetch') {
    // 判断是fetch请求之后返回统一的数据格式
    return interceptedRequest.respond({
      status: 200,
      contentType: 'application/json;charset=utf-8',
      body: JSON.stringify({
        success: true,
        data: {
          loading: true,
          dataSource: [],
        },
      }),
    });
  } else {
    // 其他请求继续走
    interceptedRequest.continue();
  }
});

```

本以为完美的逻辑，结果崩了！ 因为`fetch`请求返回的数据结构不一定都是

```json
{
  success: true,
  data: {
  	loading: true,
    dataSource: []
  }
}
```

解法：

1、前端生成请求对应的组件列表如下：

```json
interceptMap: {
  "https://xxx.yyy/zzz.json":"TablePc"
}
```

2、后端配置一份组件映射到返回数据结构在配置中心：

```json
{
  "TablePc": {
    success: true,
    data: {
      dataSource: [],
      loading: true,
    }
  }
}
```

3、预渲染拦截的时候判断在 **interceptMap** 里面的才做拦截返回，其它的继续走请求


### 参考文章

预渲染实践过程参考了业界一些比较优秀的方案，`工欲善其事，必先利其器`，在具体实践之前整体了解同时通读文档是及其必要的，这样子才能避免踩一些不必要的坑

- [《构建时预渲染：网页首帧优化实践》](https://tech.meituan.com/2018/11/15/first-contentful-paint-practice.html)
- [Puppeteer文档](https://pptr.dev/#?product=Puppeteer&version=v1.18.1&show=api-class-page) / [Puppeteer中文文档](https://www.kancloud.cn/luponu/puppeteer/870156)
- [Headless Chrome: an answer to server-side rendering JS sites](https://developers.google.com/web/tools/puppeteer/articles/ssr) ：这篇自带梯子访问，用`express`框架的比较适合