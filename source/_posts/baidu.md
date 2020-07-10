---
title: 百度一二面经
date: 2020-07-04 19:57:03
comments: true
categories: 
  - ['技术','面试']
tags:
  - 面试
---
## 百度一面 1h
几乎全程在手写算法...
1. 算法：去除一个数组中的全部0.
```js
function deleteZero(arr) {
  let newArr = [];
  for(let i = 0; i < arr.length; i++) {
    if(arr[i] !==  0) {
      newArr.push(arr[i]);
    }
  }
  return newArr;
}
```
追问：如果不用新数组怎么处理
```js
function deleteZero(arr) {
  // let newArr = [];
  for(let i = 0; i < arr.length; i++) {
    if(arr[i] === 0) {
      arr.splice(i, 1);
      i--;
    }
  }
  return arr;
}
```
追问：如果在增加去除字符串0呢，和空字符串呢？
开始没想起来，后面面试官提示了我看你是用"==="来判断的，知道"==="和"=="的区别吗?
后来就想起来了，不能用"==="，用"=="就可以了，就可以判断空字符串和"=="这些了

2. 算法：一个长度为100的数组，里面是0 - 100的随机数，用一个新数组统计出里面的0-9的数，提示: 99是两个9,也要统计出两个9
一开始没听清面试官的题目，只是统计了0-9的数。后面有改了
```js
//  开始数组乱序都不知道怎么写了，还问了面试官，尴尬
var arr = [];
for(let i = 0; i < 100; i++) {
  arr.push(Math.floor(Math.random() *100))
} 

function num(arr) {
  var newArr = Array.from({length: 10}).fill(0);
  let big = 0, small = 0; //整数部分，和个位数
  arr.forEach(item => {
    if(item < 10) {
      newArr[item]++;
    } else {
      small = item % 10;
      big = (item - small) / 10;
      newArr[small]++;
      newArr[big]++;
    }
  })
  return newArr;
}
```

3. 冒泡知道吗？讲一下，事件委托知道吗？实现一个

。。。 手写代码
4. 实现垂直居中的办法
5. 上下两列布局，下面高度为40px，上面自适应
6. 说说vue的虚拟DOM，以及优缺点
7. 看你之前博客写过手动生成虚拟DOM的，用一个构造函数来实现下。
  - 我忘记了，写的有点时间长了，后面就和面试官说了一些步骤
8. 反问阶段 


## 百度二面(挂) 1.5h
1. 自我介绍
2. 介绍项目以及难点
3. es6的箭头函数，参数问题
4. 聊了下跨域以及项目中的跨域
5. 问了我axios的一些知识，以及手写axios
 - 没写出来
6. aa-bb-cc变成AaBbCc用函数实现下
 - 写出来了但是后面优化不会
7. 异步编程的一些问题
8. 反问阶段

最后还是挂了，有点可惜/(ㄒoㄒ)/~~
