title: 从程序猿角度来看2016年为啥那么6
date: 2016-01-04 10:01:36
categories: 技术研究
tags: 算法
description: 2016年是一个非常6的一年，为啥这样子说？请往下看。
---

最近网上流传一张图来说明`2016年很6`，大概就是下面的意思：

```
2016=666+666+666+6+6+6
2016=777+777+77+77+77+77+77+77
2016=888+888+88+88+8+8+8+8+8+8+8+8
2016=999+999+9+9
```

看起来确实很6吧，不过作为`程序猿`的我看到上面的数字第一想法就是：咦，这个的算法是咋实现的？

于是，带着上面的疑问，简单去实现了一个算法来计算出上面这样的结果。

## 先瞎掰一下

咳咳，在开始算法之前，请允许我先扯一下 `6` 这个词，网络用语意思是 `溜溜溜`，形容很牛逼；
但是，在伟大的数学当中，`6` 可是一个 `完美数`，不知道啥是完美数？移步 [维基百科](https://zh.wikipedia.org/wiki/%E5%AE%8C%E5%85%A8%E6%95%B0)。作为第一个完美数，`6` 当然拥有至高无上的地位！纯属个人瞎掰，下面正式开始。

## 抽象化

算法最有趣的地方应该就是将现实复杂的事物简单抽象化，以此例子抽象化之后的意思大概就是：

> 给定一个数字（假设是2016），求将其分解成 **N** 个 `各位数都相等` 的数的和（假设这里的位数分别为6、7、8、9）

## 步骤分解

知道了算法的意思之后，就可以分解步骤了，大概如下：

1.将位数分别为6、7、8、9存进一个数组，用来遍历求出不同结果；
2.求出`各位数都相等`的数，这里基于第一步的基数；
3.将2016进行分解；
4.将结果串联起来

## 具体实现

我也不知道该怎么扯了，非常扯淡的算法如下：

```javascript
function spiltYear( year ){
    // 将year显式转为数字，避免传入字符串报错
    year = parseInt( year, 10 );    
    if( year < 1000 ){
        return;
    }                
    var baseArr = [6,7,8,9];
    baseArr.forEach(function( base, i ){
        var result = [];
        var val1 = base + 10 * base,                // 十位数
            val2 = base + 10 * base + 100 * base;   // 百位数
        var num2 = Math.floor( year / val2 ),
            num1 = Math.floor( ( year - num2 * val2 ) / val1 ),
            numBase = ( year - num2 * val2 - num1 * val1 ) / base,
            flag = numBase == Math.floor( numBase ) ? true : false;
        if( !flag ){
            console.error('当前年份不符合条件');
            return ;
        }
        while( num2 ){
            result.push(val2);
            num2--;
        }
        while( num1 ){
            result.push(val1);
            num1--;
        }
        while( numBase ){
            result.push(base);
            numBase--;
        }
        console.log('2016=%s',result.join('+'));
    });
}
spiltYear(2016);
```

## 写在后面

2016年本想写一篇长长的总结2015年，然而下笔却发现写不了啥东西，2016年希望所有人都666！
