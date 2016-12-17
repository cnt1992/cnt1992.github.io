title: 一句话介绍
date: 2015-12-07 19:06:15
---

> 懂设计（PS、Sketch），会视频制作（AE、PR），能写后端语言（PHP），喜欢看书，追求“全栈工程师”的程序猿。

```javascript
class Person {
    constructor(name, work, location) {
        [this.name, this.work, this.location] = [name, work, location];
    }
    sayHello() {
        console.info(`Hi, I am ${this.name}, now work for ${this.work} on ${this.location}`);
    }
}

class Me extends Person {
    constructor(name, work, location, github, qq) {
        super(name, work, location);
        [this.github, this.qq] = [github, qq];
    }
    sayHello() {
        console.info(`Hi, I am ${this.name}, now work for ${this.work} on ${this.location}, You can chat with me by qq:${this.qq} or follow my github page: ${this.github}`);
    }
    showMotto(msg) {
        console.log(msg);
    }
}

let me = new Me('SkyCai', 'Alibaba', 'HangZhou', 'https://github.com/cnt1992', '1019078860');

me.sayHello();
me.showMotto('where there is a will there is a way');
```