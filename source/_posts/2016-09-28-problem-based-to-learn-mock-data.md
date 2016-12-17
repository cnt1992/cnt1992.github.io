title: 问题式学习Mock开发
date: 2016-09-28 10:06:23
categories: 教程
tags: Mock NodeJs
description: 前后端分离与解耦是目前开发的一个大趋势，本文通过问题式的方式来总结一下前端工程师如何通过Mock进行开发。
---

> 本文适合于重构转前端开发或对前后端分离的开发模式尚且没有真正的实践的童鞋，对这套模式有深入实践并且有一套心得的大神欢迎提出更多的建议。

<!-- more -->

## 问题一：我跟后端约定好了接口的数据格式，接下来我应该怎么进行开发呢？

这是前后端分离的第一步也是很重要的一步，跟后端约定好了数据格式之后两边可以并行开发而不需要等待另外一方提供真正的接口出来，假设数据格式如下（一般数据都会是**JSON**格式）：

```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "name": "天哥"
        },
        {
            "id": 2,
            "name": "sky"
        }
    ]
}
```

前端只要约定好了数据格式那就可以愉快进行开发了，假设用现成的**jQuery**库来开发，核心代码就可以如下：

```javascript
;(function($) {
    const mockData = {
        "success": true,
        "data": [
            {
                "id": 1,
                "name": "天哥"
            },
            {
                "id": 2,
                "name": "sky"
            }
        ]
    };
    $.ajax({
        url: 'ajax-url',
        type: 'GET',
        dataType: 'JSON',
        success: function(data) {
            // 由于上面ajax请求的url并不会返回真实数据，所以会直接走到error函数去
        },
        error: function() {
            let data = mockData;
            if (data.success === 'true') {
                // 其它逻辑开发
            }
        }
    })
})(jQuery);
```

上面代码是**Mock**最简单的方法，当然会存在一些问题，接下来看。

## 问题二：我的mock数据是比较多的，也写在脚本里面混在一起么？

你能想到的问题大多数都要解决方案的，对于数据量相对较大而且又需要随机的，那就直接用 [mock](https://github.com/nuysoft/Mock/wiki/Getting-Started) 库，语法直接在他们的站点看挺简单的。

**Mock**数据最终是不会发布的，所以为了避免后期又需要改动很多代码，建议的做法就是将这些数据移到一个`mock`文件夹，代码结构可以如下：

```
app
 |- index.html
 |- js
    |- index.js
 |- mock
    |- mock.js
```

那么这个时候`index.html`的代码大致如下：

```html
<!DOCTYPE html>
<html>
<head>
    <title>mock demo</title>
</head>
<body>
<script src="mock/mock.js"></script>    
<script src="js/index.js"></script>
</body>
</html>
```

**mock.js**的内容其实就是上面问题一的`const mockData = xxx`挪过去，而剩下的逻辑就是`index.js`的内容。

这里延伸一下，`<script src="mock/mock.js></script>"` 这段代码只在开发环境下才需要引入，而在生产环境是不需要引入的，有没有办法可以自动化根据不同环境引入跟不引入呢？答案是肯定的。这块用**gulp**来做就比较合适了，这里不赘述，如有需要可私下继续讨论。

## 问题三：我如何直接就写好接口地址并且返回数据写正常逻辑？

这个时候`NodeJs`就出场了，后端接口地址给你了，只是没有数据，那就让`Node`去做这一层代理，给出一个示例：

```javascript
// app.js
'use strict';

const http = require('http');
const fs = require('fs');
const path = require('path');
const url = require('url');

const hostname = '127.0.0.1';
const port = 9000;

const server = http.createServer((req, res) => {
    let params = url.parse(req.url, true);
    let content = fs.readFileSync('mock/mockdata.js').toString();
    if (params.query && params.query.callback) {
        let str = `${params.query.callback}(${content})`;
        res.end(str);
    } else {
        res.end(content);
    }    
});

server.listen(port, hostname, () => {
    console.info(`server listen on ${hostname}:${port}`);
});
```

这个时候只要启动 `node app.js`，再加上`hosts`配置，就可以把接口地址匹配到该服务返回数据进行开发了。