---
title: 介绍下 Set、Map、WeakSet 和 WeakMap 的区别？
date: 2020-06-16 22:50:34
categories: 
  - ['技术','javascript']
  - ['技术','ES6']
tags:
  - javascript
  - es6
  - 数组去重
---
## Set 和 Map
Set 和 Map 主要的应用场景在于 数据重组 和 数据储存
Set 是一种叫做**集合的数据结构**，Map 是一种叫做**字典的数据结构**

### Set(集合)
Set本身是一个构造函数，需要实例化一个实例,且Set结构式无重复的数据，可以用于数据去重。Set是一个只有键值，没有键名的结构
```js
const s = new Set();
[1, 2, 2, 3, 4, 4].forEach(item => s.add(item));
console.log(s)  //Set { 1, 2, 3, 4 }
```
数组去重方式：
```js
let arr = [1,2,2,3,3,4,4,4];
console.log([...new Set(arr)])  // [1, 2, 3, 4]
```
字符串去重：
```js
let str = 'abbbcccd';
console.log([...new Set(str)].join(''));  // 'abcd'
```

Set中加入值时，数字3和字符串3会被认为两个数据，不会发生类型转化，内部判断是否相等的算法是 === 全等类型，但是加入两次NaN时，会被认为两次NaN相等
```js
const set1 = new Set();
const set2 = new Set();
let a = NaN;
let b = NaN;
set1.add(a);
set1.add(b);
console.log(set1);    //Set { NaN }

set2.add('3');
set2.add(3);
console.log(set2)   //Set { '3', 3 }
```

### Set的属性和方法
- 属性
1. Set.prototype.constructor  构造函数，默认就是Set函数。
2. Set.prototype.size  返回Set实例的成员总数。
- 方法，分为操作方法和遍历方法
  - 操作方法
  1. Set.prototype.add(value)：添加某个值，返回 Set 结构本身。
  2. Set.prototype.delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
  3. Set.prototype.has(value)：返回一个布尔值，表示该值是否为Set的成员。
  4. Set.prototype.clear()：清除所有成员，没有返回值。
```js
const set = new Set();
set.add(2).add(3).add(3);
console.log(set.has(2));  //true
console.log(set);   //{2, 3}
set.delete(2);
console.log(set.has(2));  //false
```
- Array.form()方法可以将Set结构转化为数组, 这又提供了一种新的去重方法
+ 从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。
+ 支持三个参数，第一个arrayLike，类似数组的对象，第二个callback回调函数(可选)，指定必须每个参数都需要执行的回调函数，第三个thisArg(可选)， 表示执行的回调时的this指向。
```js
// 数组去重
let arr = [1, 2, 2, 3, 3];
console.log(Array.from(new Set(arr)));  // [1, 2, 3]
```
  - 遍历方法
  1. Set.prototype.keys()：返回键名的遍历器
  2. Set.prototype.values()：返回键值的遍历器
  3. Set.prototype.entries()：返回键值对的遍历器
  4. Set.prototype.forEach()：使用回调函数遍历每个成员
❗ 注意： Set的遍历顺序就是插入顺序，是按顺序调用的
- keys()，values()，entries()
Set.keys() 返回Set结构的键名，因为Set是没有键名的，所以会返回键值
Set.values() 返回Set结构的键值
Set.entries 返回Set结构的键名和键值。因为Set是没有键名，所以会返回两遍键值
```js
let set1 = new Set(['1', '2', '3'])
console.log(set1.keys()); //{ '1', '2', '3' }
console.log(set1.values()); //{ '1', '2', '3' }
console.log(set1.entries());  //{ [ '1', '1' ], [ '2', '2' ], [ '3', '3' ] }
```
- forEach
Set结构也有forEach方法，用于遍历，因为Set 结构的键名就是键值（两者是同一个值），因此第一个参数与第二个参数的值永远都是一样的。
```js
let set1 = new Set(['1', '2', '3']);
set1.forEach((key, value) => {
  console.log(key + ':' + value);
})
```

#### Set实现交集，并集，差集
```js
let set2 = new Set([1, 2, 3]);
let set3 = new Set([4, 5, 3]);

console.log([...set2, ...set3]);  //并集
console.log([...set2].filter(item => set3.has(item)));  //交集
console.log([...set2].filter(item => !set3.has(item)));  //差集(这里是a相对b的差集)
```

### WeakSet
与Set结构类似，也是不重复的值集合
- WeakSet 和 Set的区别
1. WeakSet 只能储存对象引用，不能存放值，而 Set 对象都可以
2. WeakSet 对象中储存的对象值都是被弱引用的，即垃圾回收机制不考虑 WeakSet 对该对象的应用，如果没有其他的变量或属性引用这个对象值，则这个对象将会被垃圾回收掉（不考虑该对象还存在于 WeakSet 中），所以，WeakSet 对象里有多少个成员元素，取决于垃圾回收机制有没有运行，运行前后成员个数可能不一致，遍历结束之后，有的成员可能取不到了（被垃圾回收了），WeakSet 对象是无法被遍历的（ES6 规定 WeakSet 不可遍历），也没有办法拿到它包含的所有元素

## Map(字典)

JavaScript的对象本质上是键值对的集合(Hash 结构)，但是传统上只能用字符串当作键。这给它的使用带来了很大的限制。
Es6的Map提供了这一解决办法，Map键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。

- Map 和 Set的区别
1. 共同点：集合、字典 可以储存不重复的值
2. 不同点：集合 是以 [value, value]的形式储存元素，字典 是以 [key, value] 的形式储存。
```js
const m = new Map()
const o = {p: 'haha'}
m.set(o, 'content')
m.get(o)	// content

m.has(o)	// true
m.delete(o)	// true
m.has(o)	// false
```
Map对一个键值多次赋值后，前一个会被后一个覆盖
```js
const  map1 = new Map();
map1.set('a', 1);
map1.set('a', 2);

console.log(map1) //'a' => 2
```
❗ 注意，只有对同一个对象的引用，Map 结构才将其视为同一个键。这一点要非常小心。

```js
const map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined
```
### Map的属性和方法
- 属性
1. constructor：构造函数
2. size：返回字典中所包含的元素个数
```js
const map = new Map([
  ['name', 'An'],
  ['des', 'JS']
]);
map.size // 2
```
- 方法
  - 操作方法
  1. set(key, value)：向字典中添加新元素
  2. get(key)：通过键查找特定的数值并返回
  3. has(key)：判断字典中是否存在键key
  4. delete(key)：通过键 key 从字典中移除对应的数据
  5. clear()：将这个字典中的所有元素删除
  ```js
  const m = new Map();
  m.set('a', 2);
  m.get('a'); //2
  m.has('a'); //true

  m.delete('a');
  m.has('a'); //false
  ```
  - 遍历方法    遍历的顺序就是插入数据的顺序
  1. Map.prototype.keys()：返回键名的遍历器。
  2. Map.prototype.values()：返回键值的遍历器。
  3. Map.prototype.entries()：返回所有成员的遍历器。
  4. Map.prototype.forEach()：遍历 Map 的所有成员。

### Map与数据结构的相互转化
1. Map转化为数组
```js
const map = new Map();
map.set('a', 1).set('b', 2);
console.log([...map]);  //通过扩展运算符
```
2. 数组转化为Map
```js
const map1 = new Map([['a', 1], ['abc']]);
console.log(map1)
```
3. Map转化为对象
```js
const map2 = new Map();
map2.set('c', 1).set('b', 2);
function MaptoObj(map) {
  let Obj = Object.create(null);
  for(let [key, value] of map) {
    Obj[key] = value;
  }
  return Obj;
}
```
4. 对象转化为Map
```js
var Obj1 = {
  'x': 2,
  'y': 3
}
function ObjtoMap(obj) {
  let map3 = new Map();
  for(let key of Object.keys(obj)) {
    map3.set(key, obj[key]);
  }
  return map3;
}
```
5. Map转化为JSON
```js
function mapToJson(map) {
  return JSON.stringify([...map]);
}

let map4 = new Map().set('name', 'An').set('des', 'JS');
console.log(mapToJson(map4));
```
6. JSON转化为Map
```js
function jsonToStrMap(jsonStr) {
  return ObjtoMap(JSON.parse(jsonStr));
}

console.log(jsonToStrMap('{"name": "An", "des": "JS"}'));
```
## WeakMap
WeakMap 对象是一组键值对的集合，其中的键是弱引用对象，而值可以是任意。
❗ 注意：WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。

WeakMap 中，每个键对自己所引用对象的引用都是弱引用，在没有其他引用和该键引用同一对象，这个对象将会被垃圾回收（相应的key则变成无效的），所以，WeakMap 的 key 是不可枚举的。
- 属性
1. constructor：构造函数
- 方法
1. has(key)：判断是否有 key 关联对象
2. get(key)：返回key关联对象（没有则则返回 undefined）
3. set(key)：设置一组key关联对象
4. delete(key)：移除 key 的关联对象

## 总结
- Set
1. 成员为一，无序且不重复
2. 只有键值，没有键名，也可以说是键值与键名一致[value, value]
3. 可以用于数组去重
4. 可以遍历，方法有：add、delete、has
5. 很容易实现交集，并集，差集
- WeakSet
1. 成员都是对象
2. 成员都是弱引用，可以被垃圾回收机制回收，可以用来保存DOM节点，不容易造成内存泄漏
3. 不能遍历，方法有add、delete、has
- Map
1. 本质上是键值对的集合，类似集合,但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。
2. 可以遍历，方法很多可以跟各种数据格式转换
- WeakMap
1. 只接受对象作为键名（null除外），不接受其他类型的值作为键名
2. 键名是弱引用，键值可以是任意的，键名所指向的对象可以被垃圾回收，此时键名是无效的
3. 不能遍历，方法有get、set、has、delete