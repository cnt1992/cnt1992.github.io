title: ie6下z-index解决tips
date: 2013-11-03 01:26
tags: css hack ie6 z-index
categories: 技术研究

---

IE6虽然已经被公认放弃，但是不少前端开发人员还是在苦逼的兼容IE6.我没有刻意去学习IE6的hack，只有在碰到的时候才会去兼容。至于z-index这个bug发生的条件有三个：

1. 父标签position属性为relative；
2. 问题标签无position属性（不包括static）；
3. 问题标签含有浮动(float)属性；

那解决办法肯定是针对这三个条件，去除任何一个即可。给出的方法也有三个：

1. position:relative改为position:absolute；
2. 去除浮动；
3. 浮动元素添加position属性（如relative，absolute等）。

关于ie hack，个人的感想是懂得解决一类bug的方法，而不是针对某个bug的特定方法，而且最重要的还是学会如何解决，像3px bug、注释bug、盒子bug等等，这些bug其实都大同小异。