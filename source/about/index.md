title: 关于我
date: 2015-12-07 19:06:15
---

```javascript
function Me () {
  this.name = 'sky cai';
  this.work = 'JD';
  this.location = 'ShenZhen GuangDong';
  this.github = 'http://github.com/cnt1992';
  this.qq = '1019078860';
}

Me.prototype.sayHello = function(){
  alert("Hi, I am Sky, Welcome To My Blog");
}

var me = new Me();
me.sayHello();
```