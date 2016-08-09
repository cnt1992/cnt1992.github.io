title: React组件之高阶函数替换Mixins注入
date: 2016-08-09 20:38:57
categories: 个人学习
tags: React Mixins
description: 用ES6写React组件的时候，不用Mixins而直接用高阶函数来代替
---

值此`七夕佳节`，在这里记录一下今天用React写一个倒计时组件的笔记。

<!-- more -->

## 实现功能

基于`ES6`用`React`实现一个倒计时组件，格式：6时6分6秒，然后每隔一秒不断倒计时。

## 回顾Mixins

当然，完全可以直接写在一个`React.createClass`函数里面，但`可复用性`不高，所以换做在之前官方推荐的做法就是利用**Mixins**特性，所谓**Mixins**我理解就是`注入`，类似于`Java Spring`的`依赖注入`，简单代码如下：

```javascript
// 定义一个倒计时Mixins
var SetIntervalMixin = {
    componentWillMount: function() {
        this.intervals = [];
    },
    setInterval: function() {
        this.intervals.push(setInterval.apply(null, arguments));
    },
    componentWillUnmount: function() {
        this.intervals.forEach(clearInterval);
    }
};

// 注入Mixins到组件中
var Timer = React.createClass({
    mixins: [SetIntervalMixin],
    getInitialState: function() {
        return {
            timestamp: this.props.timestamp
        }
    },
    componentDidMount: function() {
        this.setInterval(this.changeState, 1000);
    },
    changeState: function() {
        this.setState({
            timestamp: this.state.timestamp - 1000
        })
    },
    render: function() {
        //取得属性值
        var timestamp = this.state.timestamp;        
        return <p>{this.renderTime(timestamp)}</p>
    },
    renderTime(timestamp){
        // 省略掉时间计算转换
        ...
        return 'XX天XX时XX分XX秒';
    }
});

//渲染React元素
ReactDOM.render(
    <Timer timestamp="66666666"/> ,
    document.querySelector("body")
);
```

用`Mixins`的最大好处就是`复用`，保证组件的最小颗粒化，同时也保证了功能相对单一。

## ES6的高阶函数

但是用`ES6`来写的话，`Mixins`显得有点无力，没有办法注入，这个时候高阶函数就派上用场了，其实`Redux`的`connect`就是利用了这一特性，改写上面的代码如下：

```javascript
// 注意这里传入一个React组件然后返回一个React组件
var SetIntervalMixin = ComposedComponent => class extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            timestamp: this.props.timestamp
        }
    }
    componentDidMount() {
        this.interval = setInterval(this.changeState.bind(this), 1000);    
    }
    componentWillUnmount() {
        clearInterval(this.interval);
    }
    changeState() {
        this.setState({
            timestamp: this.state.timestamp - 1000
        });
    }
    render() {
        return <ComposedComponent {...this.props} {...this.state} />;
    }
};

class Timer extends React.Component {
    constructor(props) {
        super(props);        
    }
    render() {
        // 注意，这里通过props取得属性值
        var timestamp = this.props.timestamp;        
        return <p>{this.renderTime(timestamp)}</p>
    }
    renderTime(timestamp){
        // 省略掉时间计算转换
        ...
        return 'XX天XX时XX分XX秒';
    }
}
export default SetIntervalMixin(Timer);

//渲染React元素
ReactDOM.render(
    <Timer timestamp="66666666"/> ,
    document.querySelector("body")
);
```

## 总结

两者方法其实我更倾向于第二种，也不单单是用`ES6`写起来舒服些，`函数式编程`以及`中间件`的思维在里面，总而言之，写组件多考虑抽取`可复用`组件出来，避免逻辑一大堆糅合在一起。

稍微再记一下今天碰到的一个坑：用`ES6`的`for...of`去遍历数组的时候，利用`Babel`转换在`chrome 49`以下会被转换成`Symbol`导致报错 o(╯□╰)o


