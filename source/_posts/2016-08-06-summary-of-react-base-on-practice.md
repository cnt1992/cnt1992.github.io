title: 关于React的一些实践思考与总结
date: 2016-08-06 09:41:56
categories: 心得体会
tags: React Redux Flux
description: 结合近段时间使用React开发项目碰到的疑惑总结一下自己对于React的一些实践思考以及总结
---

本文不是关于 `React` 的入门介绍教程，假设还没有对 `React` 这门技术有所了解或者没有真正在项目中实践过的童鞋可以移步 [React官网](https://facebook.github.io/react/) 或者 [React中文网](http://reactjs.cn/)，推荐看阮一峰的 [React入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react.html)。对 `React` 有实际项目的应用并且也用过 `Redux`/`Flux`/`Reflux` 其中的一个或者多个框架开发过应用的童鞋欢迎共同 <del>吐槽</del>
**探讨** 关于 `React` 这门技术使用过程中碰到的问题以及一些总结。

最近这段时间用了 `Flux` 跟 `Redux` 这两个框架开发 `React` 应用，并且完全用 `ES6` 语法开发，利用 `Webpack` 作为脚手架结合 `Babel` 编译成向下兼容的脚本。在实际项目开发过程中，碰到的问题挺多，所以才有了此文。

<!-- more -->

## 如何保护进程不因React脚本错误而终止

近些年一直被业界推崇的`组件化`开发，用`React`开发起来还是挺不错的，一个页面用多个组件堆砌起来，组件能够复用，但这个时候会碰到一个问题，在开发的过程中如果某一个组件报错了，那么这个页面就会被直接抛出错误而导致Crash，其他组件也直接不渲染出来。

这样子我们在控制台里面只会看到报错的内容以及官方的链接让你去解决问题，其实不能直观让我们知道是哪一个组件报错了。而且从`分而治之`的思想来看，组件与组件之间不应该直接相互干扰，一个组件报错也不应该导致其他组件直接不渲染了，那么最好的方法就是直接用 `装饰者模式` 给 `React` 带上美丽的装饰。

每一个`React`组件都会调用`createElement`方法生成组件，那么只需要在这个地方做文章即可，如下：

```javascript
import React from 'react';
import safe from './Safe';
let _func = React.createElement;

React.createElement = function (...args) {
    if (typeof args[0] === 'function' && !args[0]._isSafe) {
        safe(args[0]);
    }
    return _func.apply(this, args);
};
```

上面代码将`React.createElement`由`safe`方法来接管，那么这个时候我们就有办法在`Safe.js`里面保证在`createElement`的时候不直接因为报错而导致页面Crash，只要把会报错的生命周期的其中两个：`componentWillMount`/`render`改写一些即可，大致如下：

```javascript
import React from 'react';

export default function safe (target) {
    let p = target.prototype;
    let list = [
        'render',
        'componentWillMount'
    ];
    list.forEach(name => {
        if (name in p && typeof p[name] === 'function') {
            let _func = p[name];
            p[name] = function (...args) {
                try {
                    return _func.apply(this, args);
                }
                catch(e) {
                    // 这里捕捉到React渲染报错，你可以啥都不做
                    // 也可以将错误信息直接渲染在页面 
                    // ErrorResult是你自定义的错误展示组件 msg是你自定义的错误信息
                    const error = {
                        msg
                    };
                    return (
                        <ErrorResult {error} {...this.props} />
                    )
                }
            }
        }
    });
    target._isSafe = true;
}
```

这种`Safe`机制极大帮助我们快速知道哪个组件有问题，减少定位问题组件的时间成本，上面这种方案也是我们项目基于`Redux`的基础上封装成的基础`SDK`，基于这个`SDK`上进行页面的组件堆砌。

## 关于传值与传址的问题

这个其实`Javascript`里面已经说得非常清楚，简单复习一些**传值**与**传址**的区别，一个简单的Demo即可：

```javascript
// 传值是Javascript基本数据类型（数字、字符串、布尔值）被操作的过程
// 实际上是拷贝了一份存在另一个变量里面，相互不干扰
var a = 666, 
    b = a;
b = 555;
console.log(a);  // 555

// 而传址在Javascript中主要指针对引用类型(对象、数组、函数)的值的操作过程
// 虽然也拷贝了一份存在另一个变量里面，但副本指针指向跟原本同一个位置
var c = new Object(),
    d = c;
c.name = 'SkyCai';
console.log(d.name);   // 'SkyCai'
```

简单描述一下我在项目需要实现的需求：无线端页面往下拉到差不多最底部的时候触发一次异步请求拿到下一页的数据，但由于我们的整个页面数据是由`SDK`进行管控的，这个时候就需要将请求拿到的数据跟之前的数据进行合并之后触发一次`SDK`提供的`updateStore`方法去更新`store`里面的数据，然后`增量更新`页面数据从而渲染。

简单看下我一开始写下的核心代码逻辑：

```javascript
// 用于存储页面合并之后的数据
var pageData = {};

// 模拟加载第一页数据
loadData(1);

// 模拟加载第二页数据
loadData(2);

// 模拟加载第三页数据
loadData(3);

function loadData(page) {
    let promise = xxx; // promise请求
    promise.then(function(res){
        // res是一个对象
        if (res.success) {
            if (!pageData.data) {
                // 初次请求直接就赋值给pageData
                pageData = res.data;
                // 初次请求直接渲染页面
                renderPage(pageData);          
            } else {
                // 后面的请求就将请求回来的数据合并到之前保存的数据里面
                Object.assign(pageData, res.data);
                // 后面合并数据之后触发updateStore更新store数据
                updateStore(pageData);
            }
        }
    },function(err){
        // err handle
    });    
}
```

乍看之下因为没什么问题了，But，任凭你怎么刷新页面永远都是第一次渲染出来，后面的就不管你了。

首先，犯了一个很低级的错误，关于`cloneDeep`的问题，翻一下 [MDN文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 可以知道 `Object.assign(target, ...sources)` 是浅拷贝的，也就是我上面第二次请求回来的数据合并之后并不是正确的数据，因为`res.data`里面还有多层`object`，这个容易解决，写一个`cloneDeep`方法实现即可，这里就不再贴出代码。

合并之后的数据正确了，但是触发`updateStore`的时候也碰到一个问题，那就是第一次触发`updateStore`的时候能够正确拿到数据并且渲染第二页的数据，但是再次触发的时候就不渲染第三页的数据了。这就是提到的关于**传值**跟**传址**的问题，当第一次触发`updateStore`的时候传了一个`object`过去，后面再合并数据之后同样传了一个`object`过去，但这个`object`是基于上一个`object`的基础上复制的，所以导致了指向同一个指针，最简单粗暴的方法就是：

```javascript
// 简单粗暴的让pageData变成值传递过去
updateStore(JSON.parse(JSON.stringify(pageData))); 
```

## 尽量避免写依赖函数名的代码

啥意思？接着上面那个问题，`updateStore`实际上是触发了`updateStoreAction`对`store`进行更新的，来看一下`updateStoreAction`的核心代码逻辑：

```javascript
function updateStoreData(data) {
    return {
        type : 'UPDATESTORE',
        data : data
    };
}

let updateStoreAction = function updateStoreAction(data) {
    return (dispatch, getState) => {        
        dispatch(updateStoreData(data));
    };
}

export default updateStoreAction;
```

上面代码逻辑还是很清晰的，如果接受到`UPDATESTORE`的`type`时就`dispatch`出去`updateStoreData`，这个时候这个`dispatch`走到`SDK`里面的一段逻辑：

```javascript
// 上面的updateStoreAction会传入到Config.actionMiddlewareList
let middleware = Config.actionMiddlewareList || [];
middleware.forEach(function(mw) {
    let funcName = mw.name; // 这里拿出函数名字，也就是updateStoreAction
    if(!funcName) {
        funcName = mw.toString().match(/^function\s*([^\s(]+)/)[1];
    }
    ModularizationActions[funcName] = mw;
});
}
```

这个时候看到`SDK`里面有一段代码依赖于函数名字了，那么问题就来了，在本地开启`webpack-dev-server`进行开发的时候没啥问题，但是提交生产环境的代码上去之后问题就来了。

这个“锅”让`webpack`来背比较好，看下面一段配置：

```
plugins: [
    ...
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      minimize: true
    }),
    ...
]
```

生产环境下我们会压缩JS代码，但`webpack`官方文档没有详细对`UglifyJsPlugin`的参数进行解释，含糊的说默认情况下不进行开启**代码混淆**，也就是你不声明的话就不混淆，但实际上默认是进行了代码混淆，有兴趣的童鞋可以试试看`build`之后的代码。

基本解决方法当然是在`webpack`配置上将参数配好，增加一个参数：

```
plugins: [
    ...
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      mangle: false, // 代码不进行混淆
      minimize: true
    }),
    ...
]
```

但是我们仔细思考一下，怪`webpack`？作为基础`SDK`依赖于函数名进行操作其实是非常不合理的，底层通用脚手架必然承当更大的使命，所以当然要对依赖函数名字的代码进行重构才行，这才是解决问题本质的方法。

## 关于Redux/Flux/Reflux框架选型思考

经过对这三个框架实际项目的实践来看，个人而言，难易程度以此为：Redux > Flux > Reflux。当然这只是个人见解，不同的人使用起来感触都有所不同。

盗图(若侵权请联系我删除)来对比一下这三个框架的区别：

- Flux

`Flux`是`Facebook`官方实现的一套框架，基本上整个流程都已经有了，但是感觉操作起来还是挺麻烦，你每次增加一个功能都得改很多个文件，准确来说它应该是一种模式，基于此模式才有了Reflux跟Redux等优秀框架的出现。

![flux流程图](/img/think-of-react/flux.png)

- Reflux

流程相对简单，操作起来也非常方便。

![flux流程图](/img/think-of-react/reflux.png)

- Redux

这个社区应该是最多人在用了的，`单一数据源`/`Store是只读的`/`使用纯函数来进行修改` 这三个原则是`Redux`的最大特点。但就是由于`单一数据源`，所有的数据都存储在同一个`object tree`里面导致了调试很麻烦，跟`Flux`一样，增加一个功能你也得改很多个文件。

![redux流程图](/img/think-of-react/redux.jpeg)

所以根据不同业务场景使用恰当的框架才是最正确的解锁方式，不要为了使用框架而使用框架。而且也不要盲目地看这个好用这个，根据业务定制最好的框架。而且，你不可能实现一个简简单单的需求就动辄用一个那么大的框架进来吧？

## 写在后面

`React`出来也一年多，涵盖了`web`/`IOS`/`Android`三端，社区也很火。有人看好有人看衰，有人说前端越来越浮躁，又有人喊web标准一去不复返。但我觉得框架亦或是工具只是为我们提供便利加速开发的，是我们在使用它们而不是它们在控制我们。所以，适合就用，不适合就不用，没有必要跟大众走。

嗯，用`ES6`写`React`还是爽！