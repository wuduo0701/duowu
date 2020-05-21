---
title: Vue之虚拟DOM(vdom)
date: 2020-05-21 21:16:49
categories: 
  - ['技术', 'javascript']
  - ['技术','vue']
tags:
  - 虚拟dom
  - javascript
---
# 前言
> 以下内容是个人的一些学习总结，如有不对，欢迎大佬指正。
## 一.真实DOM和渲染流程
在开始虚拟DOM之前，让我们先来了解一下真实的DOM以及浏览器是怎么进行解析的。浏览器渲染引擎工作流程大致分为以下四类：**创建DOM树 -> 生成render树 -> 布局render树 -> 绘制render树**

 1. 创建DOM树：解析HTML生成DOM树 - 渲染引擎首先解析HTML文档，生成DOM树。 用CSS分析器，分析CSS文件和元素上的inline样式，生成页面的样式表。
 2. 生成render树：将DOM树和样式表，关联起来，构建一颗Render(渲染)树
 3. 布局render树：有了Render树，浏览器开始对渲染树的每个节点进行布局处理，确定其在屏幕上的显示位置。
 4. 绘制render树：遍历渲染树并用UI后端层将每一个节点绘制出来
![alt](https://user-gold-cdn.xitu.io/2020/3/7/170b3fde842e595e?w=624&h=289&f=jpeg&s=18227)
 
## 二.虚拟DOM
### 1.虚拟DOM是什么？
当用原生js或者jq去操作真实DOM的时候，浏览器会从构建DOM树开始从头到尾执行一遍流程。当操作次数过多时，之前计算DOM节点坐标值等都是白白浪费的性能，虚拟DOM由此诞生。
### 2.虚拟DOM有什么好处？
假设一次操作中有10次更新DOM的动作，虚拟DOM不会立即操作DOM，而是将这10次更新的diff内容保存到本地一个JS对象中，最终将这个JS对象一次性attch到DOM树上，再进行后续操作，避免大量无谓的计算量。所以，用JS对象模拟DOM节点的好处是，页面的更新可以先全部反映在虚拟DOM上，操作内存中的JS对象的速度显然要更快，等更新完成后，再将最终的JS对象映射成真实的DOM，交由浏览器去绘制。

## 三.Vue中的虚拟DOM
![alt](https://user-gold-cdn.xitu.io/2020/3/7/170b414d73437744?w=949&h=269&f=png&s=23906)
- **渲染函数**：渲染函数是用来生成虚拟DOM的。Vue推荐使用模板来构建应用界面，在底层实现中Vue将模板编译成渲染函数。
- **Vnode虚拟节点**：它可以代表一个真实的dom节点。通过createElement方法将vnode节点渲染成dom节点。
- **patch**：虚拟DOM最核心的部分，它可以将vnode渲染成真实的DOM，这个过程是对比新旧虚拟节点之间有哪些不同，然后根据对比结果找出需要更新的的节点进行更新

>参考文档：
> - [渲染函数Vue文档](https://cn.vuejs.org/v2/guide/render-function.html)
> - [vue 虚拟dom实现原理文章](https://blog.csdn.net/u010692018/article/details/78799335)
> - [patch算法源码](https://github.com/vuejs/vue/blob/dev/src/core/vdom/patch.js#L366-L366)

> ps: 笔者能力有限，原谅我patch源码篇看的不是很懂！在这请尤雨溪大大收下我的膝盖。
## 四.模拟Vue虚拟DOM

### 1.安装vue-cli脚手架，部署vue环境。
这里大家可以自行安装。

### 2.创建虚拟DOM树
先在element.js文件中实现如何创建虚拟DOM树。

```
// element.js

// 虚拟DOM元素类，用来描述DOM
function Element(type, props, children){
  this.type = type; //节点类型
  this.props = props; //属性
  this.children = children; //子节点
}
// 创建虚拟DOM
function createElement(type, props, children){
  return new Element(type, props, children);
}
//向外输出
export {
  Element,
  createElement
}
```

我们来到App.vue文件中调用createElement方法来创建一个DOM对象。
```
//App.vue
import {createElement} from './js/element.js'

let V_DOM = createElement('ul',{class:'list'},[
  createElement('li', {class:'item'},['item1']),
  createElement('li', {class:'item'},['item2']),
  createElement('li', {class:'item'},['item3'])
])
//打印虚拟DOM
console.log(V_DOM);
```
> 注：因为脚手架里的App.vue的内容没有删除，这里只贴了调用到的代码。

下面我们来看看浏览器里打印出来的虚拟DOM。
![alt](https://user-gold-cdn.xitu.io/2020/3/7/170b4a7e341625dc?w=465&h=402&f=png&s=25638)
### 三.模拟渲染函数渲染虚拟DOM

```
// element.js

function render(dom) {
  // 根据type类型来创建对应的元素
  let el = document.createElement(dom.type);
  
  // 再去遍历props属性对象，然后给创建的元素el设置属性
  for (let key in dom.props) {
      // 设置属性的方法
      el.setAttribute(key, dom.props[key]);
  }
  
  // 遍历子节点
  // // 如果子节点也是虚拟DOM，递归构建DOM节点
  // 不是就代表是文本节点，直接创建
  dom.children.forEach(child => {
      child = (child instanceof Element) ? render(child) : document.createTextNode(child);
      // 添加到对应元素内
      el.appendChild(child);
  });

  return el;
}

//向外输出
export {
  Element,
  createElement,
  render,
}
```
回到App.vue我们来调用**render函数**
```
import {createElement, render, renderDom} from './js/element.js'

let V_DOM = createElement('ul',{class:'list'},[
  createElement('li', {class:'item'},['item1']),
  createElement('li', {class:'item'},['item2']),
  createElement('li', {class:'item'},['item3'])
])
//打印虚拟DOM
console.log(V_DOM);
var el = render(V_DOM);
console.log(el);
document.body.appendChild(el);
```
来看看效果图。
![alt](https://user-gold-cdn.xitu.io/2020/3/7/170b4d7b39a06321?w=479&h=198&f=png&s=12226)
真实的DOM结构就构建出来了，接下来渲染到页面中。
![alt](https://user-gold-cdn.xitu.io/2020/3/7/170b4dacd16bede6?w=1433&h=968&f=png&s=64205)
好了，我们已经实现了模拟虚拟DOM并进行了渲染。而diff算法和patch算法才是虚拟DOM最核心的部分，笔者还在努力学习阶段。这里推荐一篇[宝藏文章](https://github.com/livoras/blog/issues/13)，这里比较详细的解析了虚拟DOM算法。
## 写在最后
如果文章有错误或者不妥之处，欢迎大家指正，谢谢。