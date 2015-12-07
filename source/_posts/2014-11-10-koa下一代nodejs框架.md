title: koa下一代nodejs框架
date: 2014-11-10 11:21:50
tags: koa,node.js,nvm,javascript
categories: 技术研究

---
[koa](http://koajs.com)是下一代的NodeJS框架，由原先[Express](http://expressjs.com/)团队开发，致力于更小，更有表现力的web应用程序。

若想在自己的机器上运行koa应用程序，必须是node 0.11版本以上，建议下载node版本控制器[n](https://github.com/visionmedia/n)或者[nvm](https://github.com/creationix/nvm),自己试用了一下，感觉比较喜欢nvm，下面是koa入门的步骤。

> 注：本实验平台基于mac osx ，若是windows平台，有一些对应步骤可能不同。

<!--more-->

1. 利用命令行直接安装nvm
```javascript
curl https://raw.githubusercontent.com/creationix/nvm/v0.18.0/install.sh | bash
```

2. 利用nvm安装指定版本（必须是0.11.*以上版本）
```javascript
nvm install 0.11.12
```
或者安装node最新版本
```javascript
nvm install latest
```

3. 指定使用安装的node 版本
```javascript
nvm use 0.11.12
```

4. 新建app.js文件，录入下面经典的hello world代码
```javascript
var koa = require('koa');
var app = koa();

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);
```

5. 进入应用根目录，下载依赖module
```javascript
npm install koa
```

6. 用node的harmony模式启动服务器
```javascript
node --harmony koa
```
为了以后方便使用，可以将node设置为默认启动harmony模式的别名：
```javascript
alias node='node --harmony'
```




    
