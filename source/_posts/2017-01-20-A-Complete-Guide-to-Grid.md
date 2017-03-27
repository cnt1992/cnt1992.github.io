title: CSS Grid Layout 不完整指南
date: 2017-01-20 11:24:29
categories: 教程
tags: CSS
description: 介绍并实践下CSS Grid Layout
---

| 本文还没写完。。。

<!-- more -->

## 介绍

`CSS Grid Layout`，即网格布局系统，旨在用最简单的方式改变用户界面。

CSS发展至今，最基础也最核心的部分是布局，从一开始的用黑科技`float`进行多列布局，到后来的`flex`布局的出现，再到现在想讲的`Grid`布局系统，可以说大大加快网页布局的书写之外语义化也更加明确了。

## 兼容性

从下图看出，目前主流浏览器尚且不支持该属性（截至现在最新Chrome版本为55），所以想要体验`Grid`属性请先打开[chrome://flags/](chrome://flags/)将`experimental Web Platform features`开启。

<iframe style="border: none;width: 100%;height: 520px;background: " src="https://caniuse.bitsofco.de/embed/index.html?feat=css-grid&periods=future_3,current,past_1,past_2"></iframe>

## 示例一览

我们先从一个最经典的三列布局（左右固定宽度中间自适应）来看看如何用`grid`实现：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        .container {
            display: grid;
            grid-template-columns: 100px auto 150px;
            grid-template-areas: "left main right";
            
            min-height: 400px;
        }
        .left {
            grid-area: left;
        }
        .main {
            grid-area: main;
        }
        .right {
            grid-area: right;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="left">
            left
        </div>
        <div class="main">
            main
        </div>
        <div class="right">
            right
        </div>
    </div>
</body>
</html>
```

## 语法

1.Grid Container

想让一个容器变成`Grid`，只需要如下声明：

```css
.container {
  display: grid | inline-grid | subgrid;
}
```

| 属性 | grid | inline-grid | subgrid |
| :--: | :--: | :--: | :--: |
| 说明 | 块状级别grid | 内联级别grid | 自身已经是一个grid item，同时声明自己是一个grid container |

> 注意： `column`, `float`, `clear`, 以及 `vertical-align` 对声明了`display: grid;` 的元素无效。

2.Grid Items



## 参考资源

- [complete-guide-grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [3-new-css-features-to-learn-in-2017](https://bitsofco.de/3-new-css-features-to-learn-in-2017/)