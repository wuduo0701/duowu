---
title: this指向和call，apply， bind，new
date: 2020-05-20 21:26:17
categories: 
  - ['技术', 'javascript']
tags: 
  - this指向
  - javascript
---
## this指向
this永远指向最后调用它的对象，而不是创建的时候定义的
1. 函数作为对象的方法被调用时，this指向该对象
```js
var a = {
  name: 'lin',
  f1: function(){
    console.log(this.name)
  }
}
a.f1()
```
2. 作为普通函数，this指向window(在严格模式会是undefined)
```js
var name = 'lin'
function a(){
  console.log(this.name)
}
a()
```
3. 构造器调用时，this指向返回的这个对象
4.  箭头函数的this绑定看的是this所在函数定义在哪个对象下，就绑定哪个对象,如果有嵌套的情况，则this绑定到最近的一层对象上
+ 箭头函数this指向的固定化，并不上它内部有绑定this，而是箭头函数根本没用自己的this，他总是用外部代码的this

## 改变this指向
new > call/apply/bind > 对象.foo > 全局
1. 使用es6的箭头函数
2. 用that = this，绑定之前的this指向
3. 使用apply call bind改变
4. new的时候
### 使用
1. apply(接收的是一个包含多个参数的数组。)
```js
var a ={
  name : "Cherry",
  fn : function (a,b) {
    console.log( a + b)
  },
  fn1 : function () {
    console.log(this.name)
  }
}
var b = a.fn;
var b1 = a.fn1;
b1.apply(a);  //Cherry
b.apply(a,[1,2])     // 3
```
2. call(接受的是若干个参数列表)
```js
var a ={
  name : "Cherry",
  fn : function (a,b) {
    console.log( a + b)
  },
  fn1 : function () {
    console.log(this.name)
  }
}
var b = a.fn;
var b1 = a.fn1;
b1.call(a);  //Cherry
b.call(a,1,2)     // 3    差别在这
```
3. bind返回的是一个新的函数
```js
var a ={
  name : "Cherry",
  fn : function (a,b) {
    console.log( a + b)
  },
  fn1 : function () {
    console.log(this.name)
  }
}
var b = a.fn;
var b1 = a.fn1;
b1.bind(a)();  //Cherry
b.bind(a,1,2)()   // 3  返回的是新函数，重新执行
```
4. new的过程
  1. 创建一个新对象
  2. 把构造函数的作用域赋值给新对象，绑定this
  3. 执行构造函数的代码(为这个新对象添加属性)
  4. 如果无返回值或者返回一个非对象值，则将 obj 返回作为新对象；如果返回值是一个新对象的话那么直接直接返回该对象。
```js
function _new1(constructor, ...args){
  let target = {};  //1.定义新对象
  // 2. 绑定this指向，构造函数的作用域付给新对象
  target.prototype = constructor.prototype;
  // 3. 为这个新对象添加属性和方法
  let result = constructor.apply(target.prototype, args);
  // 4. 返回这个新对象
  if(result && (typeof result == "object" || typeof result == "function")){
    // 如果构造函数返回的结果是一个对象，就返回这个对象
    return result;
  }
  // 如果构造函数返回的不是一个对象，就返回创建的新对象。
  return target.prototype;
}
```