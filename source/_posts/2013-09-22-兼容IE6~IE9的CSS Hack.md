title: 兼容IE6~IE9的CSS Hack
date: 2013-09-22 10:16:29
tags: css hack
categories: 技术研究
avatar: "https://avatars2.githubusercontent.com/u/3118988?v=3&s=40"
---
```css
div {
  background-color:#f00\0; /* ie 8/9 */
  background-color:#f00\9\0; /* ie 9 */
  *background-color:#0f0; /* ie 7 */
  _background-color:#00f; /* ie 6 */
}
```

其中，‘\0’对ie8和ie9都支持，而‘\9\0’只支持ie9，所以顺序不可以颠倒。

