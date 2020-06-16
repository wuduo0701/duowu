---
title: 对象，原型及原型链
date: 2020-06-12 21:47:48
categories: 
  - ['技术','对象']
  - ['技术','原型链']
tags:
  - javascript
  - 原型继承
---
## 对象

对象是一个包含相关数据和方法的集合(通常由一些变量和函数组成，我们称之为对象里面属性和方法)
- 对象也被称之为"关联数组"
```js
var person = {
  name: {
    first: "Lin",
    last: "JiaHao"
  }
}
//输出一样
console.log(person.name.first); //Lin
console.log(person['name']['first']); //Lin
```
问题&Q: 为什么能用数组形式访问呢？
回答&A: 对象做了字符串到值的映射，而数组做的是数字到值的映射。本质上是一样的

## 原型及原型链
javascript常被描述为"基于原型的语言"，每个对象都有一个原型对象，对象以原型为模板，从原型继承方法和属性。
原型对象也可能拥有原型，并从中继承方法和属性，一层一层、以此类推。这种关系常被称为原型链，**它解释了为何一个对象会拥有定义在其他对象中的属性和方法。**

❗ 注意：这些属性和方法定义在Object的构造器函数之上的prototype属性上，而非对象实例本身。

在传统的OOP中类中定义的所有属性和方法都被复制到实例中，而在javascript中，**是在构造器中和对象实例之间建立一个链接（它是__proto__属性，是从构造函数的prototype属性派生的）**，之后通过上溯原型链，在构造器中找到这些属性和方法。

### 理解实例对象的原型和构造函数的prototype之间的关系
前者是每个实例上都有的属性，后者是构造函数的属性。

实例对象的原型可以通过**__proto__**和**Object.getPrototypeOf(new People())**（People值构造函数）
也就是说实例对象的原型和构造函数的prototype指向同一个对象,即 Object.getPrototype(new People) 和 People.prototype指向同一个对象
```js
function People(name, age) {
  this.name = name;
  this.age = age;
}
var p1 = new People('朵雾', '20');

console.log(p1.__proto__ === People.prototype)    //true
``` 
总结：实例对象继承的是构造函数上的prototype上的属性和方法,并不是真正的继承而是**在对象实例和构造器上建立一个链接**, 这个链接是__proto__属性，是从构造函数的prototype属性派生的。
```js
function People(name, age) {
  this.name = name;
  this.age = age;
}
People.prototype.sex = '男';
console.log(People.prototype);

var p1 = new People('朵雾', '20');
console.log(p1.__proto__);  //People { sex: '男' }
console.log(p1.sex);  //男  这里并不是真正实例上的属性，而是实例通过上溯原型链，在构造器中找到这些属性
```
当你通过实例查找属性和方法时，他会先在这个实例查找有没有这个属性，没有的话就会通过__proto__这个属性上查找这个属性,也就是这里的People.prototype.sex上查找，如果 p1.__proto__ (People.prototype.sex)有这个属性，就会使用这个属性，没有的话就会继续往上查找p1.__proto__的__proto__属性，看它是否有这个属性。直到找到顶部全局的prototype上是否有这个属性，有就会使用这个属性，否则就为undefined
```js
function People(name, age) {
  this.name = name;
  this.age = age;
}
People.prototype.sex = '男';
console.log(People.prototype);

var p1 = new People('朵雾', '20');
console.log(p1.__proto__);
console.log(p1.sex);
p1.sex = '女'
console.log(p1.sex);
```

```js
// 顶部全局的属性
global.Object.prototype.sex = '超人';

function People(name, age) {
  this.name = name;
  this.age = age;
}
var p1 = new People('朵雾', '20');
console.log(p1.__proto__);
//  这里在p1上没找到sex属性，就会找p1.__proto__上的sex属性，也就是People.prototype上的sex属性,发现还是未找到，就会继续往上查找，找到顶层global.Object.prototype(这里的全局属性是global，浏览器是window)，发现找到了sex属性。
console.log(p1.sex);
p1.sex = '女'
//  这里直接在p1上找到属性，就不会继续查找
console.log(p1.sex);
```

❗ 注意：必须重申，原型链中的方法和属性没有被复制到其他对象——它们被访问需要通过前面所说的“原型链”的方式。