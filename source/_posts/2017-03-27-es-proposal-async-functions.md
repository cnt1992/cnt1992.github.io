title: 【译】ES建议：async函数
date: 2017-03-27 17:16:56
categories: 翻译
tags: ES7 async await
description: 简要介绍了异步函数进化史，同时推荐ES7的async函数，毕竟ES6的generator跟yield只是一个过渡
---

`Async`函数是`Brian Terlson`提出的`ECMAScript提案`，目前处于`stage 3(candidate 候选)`阶段。

| 译者注：ECMAScript提案分三个阶段：草图(Sketch) -> 提案(Proposal) -> 规范(Standard)，所以`Async`很大的希望能成为规范.

在开始解释`async`函数之前，我想先讲一下如何通过组合`Promises`和`generators`用**看起来同步的代码**来控制异步操作。

## 通过 Promises 和 generators 写异步代码

在众多的能用于异步的函数中，`Promises`，作为`ES6`的一部分，变得越来越受欢迎。一个例子就是客户端的`fetch API`，用来发送请求获取数据从而替代`XMLHttpRequest`。使用`fetch`看起来像下面这样：

```javascript
function fetchJson(url) {
    return fetch(url)
        .then(request => request.text())
        .then(text => {
            return JSON.parse(text);
        })
        .catch(error => {
            console.log(`ERROR: ${error.stack}`);
        });
}
fetchJson('http://example.com/some_file.json').then(obj => console.log(obj));
```

[co](https://github.com/tj/co)库通过使用`Promises`和`generators`能让代码看起来更同步，上面的例子用`co`来写就是下面这样子：

```javascript
const fetchJson = co.wrap(function* (url) {
    try {
        let request = yield fetch(url);
        let text = yield request.text();
        return JSON.parse(text);
    }
    catch (error) {
        console.log(`ERROR: ${error.stack}`);
    }
});
fetchJson('http://example.com/some_file.json').then(obj => console.log(obj));
```

每一次回调(一个`generator`函数)都会传递一个`Promises`给到`co`，然后回调进入暂停状态。一旦`Promise`执行完毕，`co`就会触发回调：如果`Promise`返回`fulfilled(成功)`，`yield`返回成功状态的值，如果返回`rejected(失败)`，`yield`就会抛出失败的错误。另外，`co`会将返回的回调包装成一个`Promise`(跟`then()`的用途一样)。

## Async 函数

上面的例子用`Async`函数的基本用法实现如下：

```javascript
async function fetchJson(url) {
    try {
        let request = await fetch(url);
        let text = await request.text();
        return JSON.parse(text);
    }
    catch (error) {
        console.log(`ERROR: ${error.stack}`);
    }
}
fetchJson('http://example.com/some_file.json').then(obj => console.log(obj));
```

在实现上，`async`函数更像`generators`，但是不会转换成`generatos`函数.

Internally, async functions work much like generators, but they are not translated to generator functions.

### Async 函数其他的一些用法

现在存在如下的`async`函数的变体：

- Async 函数声明： `async function foo() {}`
- Async 函数表达式： `const foo = async function() {}`
- Async 方法定义： `let obj = { async foo() {} }`
- Async 箭头函数：`const foo = async () => {};`

## 深入阅读

[Async Functions](https://github.com/tc39/ecmascript-asyncawait)(Brian Terlson)
[Simplifying asynchronous computations via generators](http://exploringjs.com/es6/ch_generators.html#sec_co-library)(“Exploring ES6”的章节)