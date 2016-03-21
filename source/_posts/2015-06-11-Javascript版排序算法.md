title: Javascript版排序算法
date: 2015-06-11 09:28
tags: javascript 算法
categories: 个人学习
avatar: "https://avatars2.githubusercontent.com/u/3118988?v=3&s=40"
---
### 必备知识点

- [JS原型](http://bonsaiden.github.io/JavaScript-Garden/zh/#object.prototype)
- 排序中的有序区和无序区
  简单来说，有序区就是在一组数据中已经排好序的部分，无序区就是该组数据中还没排好序的部分。
- [二叉树的基本知识](https://zh.wikipedia.org/zh/%E4%BA%8C%E5%8F%89%E6%A0%91)

### 封装原型方法

```javascript
			
	        Function.prototype.method = function ( name, func ) {
                this.prototype[ name ] = func;
                return this;
            }      

```

<!--more-->

### 算法思路及复杂度对比

| 名称 | 数据对象 | 时间复杂度 | 空间复杂度 | 描述 |
| :--: | :--: | :--: | :--: | :--: |
| 冒泡排序 | 数组 | O(n²) | O(1) | (无序区，有序区) 通过在无序区的相邻元素的比较和替换，使较小的元素浮到最上面 |
| 选择排序 | 数组、链表 | O(n²) | O(1) | (有序区，无序区) 
在无序区里找一个最小的元素跟在有序区的后面。对数组：比较得多，换得少 |
| 插入排序 | 数组、链表 | O(n²) | O(1) | (有序区，无序区) 把无序区的第一个元素插入到有序区的合适的位置。对数组：比较得少，换得多 |
| 堆排序 | 数组 | O(nlogn) | O(1) | (有序区，无序区) 在无序区里找一个最小的元素跟在有序区的后面。对数组：比较得多，换得少 |
| 归并排序 | 数组、链表 | O(nlogn) |  | (最大堆，有序区) 从堆顶把根卸下来放在有序区之前，再恢复堆 |
| 快速排序 | 数组 | O(nlogn)~O(n²) | O(logn)，O(n) | （小数，枢纽元，大数） |
| 希尔排序 | 数组 | O(nlog²n)~O(n²) | O(1) | 每一轮按照事先决定的间隔进行插入排序，间隔会依次缩小，最后依次一定要是1 |

### 算法实现

1.冒泡排序

```javascript
            
            Array.method('bubbleSort', function () {
                var len = this.length,
                    i, j, tmp;
                for ( i = 0; i < len; i++ ) {
                    for ( j = len - 1; j > i; j-- ) {
                        if ( this[j] < this[j-1] ) {
                            tmp = this[j-1];
                            this[j-1] = this[j];
                            this[j] = tmp;
                        }
                    }
                }
                return this;
            });

```

`改进冒泡排序`:如果在某次排序中没有出现交换的情况，那么说明在无序的元素现在已经是有序了，可以直接返回。

```javascript
            
            Array.method('rBubbleSort', function () {
                var len = this.length,
                    i, j, tmp, exchange;
                for ( i = 0; i < len; i++ ) {
                    exchange = 0;
                    for ( j = len - 1; j > i; j-- ) {
                        if ( this[j] < this[j-1] ) {
                            tmp = this[j-1];
                            this[j-1] = this[j];
                            this[j] = tmp;
                            exchange = 1;
                        }
                    }
                    if ( !exchange ) {
                        return this;
                    }
                }
                return this;
            });

```

2.选择排序

```javascript
            
            Array.method('selectSort', function () {
                var len = this.length,
                    i, j, k, tmp;
                for ( i = 0; i < len; i++ ) {
                    k = i;
                    for ( j = i + 1; j < len; j++ ) {
                        if ( this[j] < this[k] ) {
                            k = j;
                        }
                    }
                    if ( k!=i ) {
                        tmp = this[k];
                        this[k] = this[i];
                        this[i] = tmp;
                    }
                }
                return this;
            });

```

3.插入排序

```javascript
            
            Array.method('insertSort', function () {
                var len = this.length,
                    i, j, k, tmp;
                for ( i = 1; i < len; i++ ) {
                    tmp = this[i];
                    j = i - 1;
                    while ( j>=0 && tmp < this[j] ) {
                        this[j+1] = this[j];
                        j--;
                    }
                    this[j+1] = tmp;
                }
                return this;
            });

```

4.堆排序

    堆排序是一种树形选择排序方法(注意下标是从1开始的，也就是R[1...n])。
    堆排序思路：
    1) 初始堆：
    将原始数组调整成大根堆的方法——筛选算法：
    比较R[2i]、R[2i+1]和R[i]，将最大者放在R[i]的位置上(递归调用此方法到结束)
    2) 堆排序：
    每次将堆顶元素与数组最后面的且没有被置换的元素互换。

```javascript
            
            Array.method('createHeap', function(low, high){
                var i=low, j=2*i, tmp=this[i];
                while(j<=high){
                    // 从左右子节点中选出较大的节点
                    if(j< high && this[j]<this[j+1]) j++;
                    if(tmp < this[j]){                    
                        //根节点(tmp)<较大的节点
                        this[i] = this[j];
                        i = j;
                        j = 2*i;
                    }else break;
                }
                //被筛选的元素放在最终的位置上
                this[i] = tmp;                            
                return this;
            });
            Array.method('heapSort', function(){
                var i, tmp, len=this.length-1;
                for(i=parseInt(len/2); i>=1; i--) this.createHeap(i, len);
                for(i=len; i>=2; i--){
                    tmp = this[1];
                    this[1] = this[i];
                    this[i] = tmp;
                    this.createHeap(1, i-1);
                }
                return this;
            });

```

5.归并排序
    归并排序思路：
    1) 归并
    从两个有序表R[low...mid]和R[mid+1...high]，每次从左边依次取出一个数进行比较，将较小者放入tmp数组中，最后将两段中剩下的部分直接复制到tmp中。
    这样tmp是一个有序表，再将它复制加R中。(其中要考虑最后一个子表的长度不足length的情况)
    2) 排序
    自底向上的归并，第一回：length=1；第二回：length=2*length ...

```javascript
            
            Array.method('createHeap', function(low, high){
                var i=low, j=2*i, tmp=this[i];
                while(j<=high){
                    // 从左右子节点中选出较大的节点
                    if(j< high && this[j]<this[j+1]) j++;
                    if(tmp < this[j]){                    
                        //根节点(tmp)<较大的节点
                        this[i] = this[j];
                        i = j;
                        j = 2*i;
                    }else break;
                }
                //被筛选的元素放在最终的位置上
                this[i] = tmp;                            
                return this;
            });
            Array.method('heapSort', function(){
                var i, tmp, len=this.length-1;
                for(i=parseInt(len/2); i>=1; i--) this.createHeap(i, len);
                for(i=len; i>=2; i--){
                    tmp = this[1];
                    this[1] = this[i];
                    this[i] = tmp;
                    this.createHeap(1, i-1);
                }
                return this;
            });

```

6.快速排序
    快速排序思路：
    1) 假设第一个元素为基准元素 
    2) 把所有比基准元素小的记录放置在前一部分，把所有比基准元素大的记录放置在后一部分，并把基准元素放在这两部分的中间(i=j的位置)

```javascript
            
            Array.method('quickSort', function(s, t){
                var i=s, j=t,
                    tmp;
                if(s < t){
                    tmp = this[s];
                    while(i!=j){
                        while(j>i && this[j]>tmp) j--;//右—>左
                        R[i] = R[j];
                        while(i<j && this[j]<tmp) i++;//左—>右
                        R[j] = R[i]; 
                    }
                    R[i] = tmp;
                    this.quickSort(s, i-1);
                    this.quickSort(i+1, t);
                }
                return this;
            });

```

7.希尔排序

    希尔排序思路:
    我们在第 i 次时取gap = n/(2的i次方)，然后将数组分为gap组(从下标0开始，每相邻的gap个元素为一组)，接下来我们对每一组进行直接插入排序。

```javascript
            
            Array.method('shellSort', function(){
                var len = this.length, gap = parseInt(len/2), 
                    i, j, tmp;
                while(gap > 0){
                    for(i=gap; i<len; i++){
                        tmp = this[i];
                        j = i - gap;
                        while(j>=0 && tmp < this[j]){
                            this[j+gap] = this[j];
                            j = j - gap;
                        }
                        this[j + gap] = tmp;
                    }
                    gap = parseInt(gap/2);
                }
                return this;
            });

```