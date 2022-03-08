---
title: 深入JavaScript学习之路（一）对象、类与面向对象编程
catalog: true
subtitle: 红宝书读书笔记 第八章 p205
date: 2022-03-07 16:50:52
header-img: /img/header_img/galaxy-ngc-3190-wallpaper-for-2880x1800-60-653.jpg
tags:
- 前端
- JavaScript
categories:
- 前端
  - 学习笔记
---

本章将学习：对象创建过程、原型链及继承实现方式、JS中的类等
<!-- more -->

# 理解对象
> ECMA-262将对象定义为**一组属性的无序集合**，每个属性或方法都有一个名称来表示，可将其想象为一张**散列表**，值可以为数据/函数
- 例1
```js
// 使用 new Object() 进行创建
let person = new Object();
person.name = "cosine";
person.age = 29
person.job = "Software Engineer"; 
person.sayName = function() { 
 console.log(this.name); 
}; 
// 使用对象字面量创建
let person = {
  name: 'cosine',
  age: 29, 
  job: "Software Engineer", 
  sayName() { 
    console.log(this.name); 
  }
}
```
以上两种对象是等价的，其属性和方法都一样
可以思考一下： **new 的过程中做了什么？** 后面也会提到

## 属性类型
ECMA-262 使用一些内部特性来描述属性的特征。这些特性是由为 JavaScript 实现引擎的规范定义的。因此，开发者不能在 JavaScript 中直接访问这些特性。为了将某个特性标识为内部特性，规范会用**两个中括号**把特性的名称括起来，比如`[[Enumerable]]`
属性分两种：**数据属性**和 **访问器属性**
### 数据属性
包含一个**保存数据值**的位置。值会从这个位置读取，也会写入到这个位置。数据属性有 4 个特性描述它们的行为

- `[[Configurable]]` 可配置
	- 表示属性是否可以通过  `delete` **删除**并**重新定义**
	- 是否可以**修改**它的特性
	- 是否可以把它**改为访问器属性**
	- 默认情况下，所有**直接定义**在对象上的属性的这个特性都是 `true`
- `[[Enumerable]]`  可枚举
	- 表示属性是否可以通过 `for-in` 循环返回
	- 默认： `true`
- `[[Writable]]`  可写
	- 表示属性的值**是否可以被修改**
	- 默认： `true`
- `[[Value]]`  可写
	- 包含属性**实际的值**
	- 这就是前面提到的那个**读取和写入属性值的位置**
	- 默认： `undefined`

在像前面例子中那样将属性显式添加到对象之后，`[[Configurable]]`、`[[Enumerable]]` 和 `[[Writable]]`都会被设置为 true，而 `[[Value]]` 特性会被设置为指定的值。
#### Object.defineProperty()方法
要修改属性的默认特性，就必须使用  [`Object.defineProperty()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)方法。这个方法接收 3 个参数：
- `obj` 待添加属性的**对象**
- `prop` 待定义或修改的**属性名称**或 **Symbol** 
- `descriptor` 要定义或修改的**属性描述符**

设置方法如下例
```js
// Object.defineProperty()设置属性
let person = {};
Object.defineProperty(person, "name", {
  writable: false,    // 看这儿！ 不可修改
  value: "cosine"
});
console.log(person.name); // cosine
person.name = "NaHCOx";   // 试图修改 
console.log(person.name); // 修改无效 print: cosine 
```
创建了一个名为 `name` 的属性并给它赋予了一个**只读**的值，则这个属性的值不能修改了
- **非严格模式**下，尝试给这个属性重新赋值会被**忽略**。
- **严格模式**下，尝试修改只读属性的值会**抛出错误**

类似的规则也适用于创建**不可配置**的属性
把 `configurable` 设置为 `false`，意味着这个属性**不能从对象上删除**。

注意：**一个属性被定义为不可配置之后，就不能再变回可配置的了！！**
> 再次调用 `Object.defineProperty()` 并修改任何非 `writable` 属性则会导致错误

在调用 `Object.defineProperty()`时，`configurable`、`enumerable` 和 `writable` 的值如果不指定，则都默认为 `false`。
### 访问器属性
访问器属性**不包含数据值**。相反，它们包含一个获取（[`getter`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/get)）函数和一个设置（[`setter`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/set)）函数，不过这两个函数不是必需的。
- 读取访问器属性时，会调用 `getter` 函数，返回一个有效的值
- 写入访问器属性时，会调用 `setter` 函数并传入新值，这个函数必须决定对数据做出什么修改


访问器属性有 4 个特性描述它们的行为:

- `[[Configurable]]`：表示属性
	- 是否可以通过 delete 删除并重新定义
	- 是否可以修改它的特性
	- 是否可以把它**改为数据属性**
	- 默认：`true`
- `[[Enumerable]]`：表示属性是否可以通过 for-in 循环返回。默认`true`
- `[[Get]]`：获取函数，在读取属性时调用。默认值为 `undefined`
- `[[Set]]`：设置函数，在写入属性时调用。默认值为 `undefined`

注意，上述属性的默认值都表示在直接定义对象上时的默认值，若用`Object.defineProperty()`，则没有定义的都为 `undefined`

下面是一个例子
```js
// 访问器属性定义
// 定义一个对象，包含伪私有成员 year_和公共成员 edition 
let book = { 
  year_: 2017, 
  edition: 1 
};
Object.defineProperty(book, "year", {
  get() {
    return this.year_;
  },
  set(newVal) {
    if(newVal > 2017) {
      this.year_ = newVal;
      this.edition += newVal - 2017;
    }
  }
});
book.year = 1999
console.log(book.year);   // 2017
console.log(book.edition);  // 1
book.year = 2018
console.log(book.year);   // 2018
console.log(book.edition);  // 2
```

对象 book 有两个默认属性：year_和 edition
**year_中的下划线常用来表示该属性并不希望在对象方法的外部被访问**
另一个属性 year 被定义为一个访问器属性，其中，
- getter简单返回 year_的值
- setter会做一些计算以决定正确的版本（edition）

因此，把 year 属性修改为 2018 会导致 year_变成 2018，edition 变成 2。而试图修改为1999则不会有变化，这是访问器属性的典型使用场景，即设置一个属性值会导致一些其他变化发生。

获取函数和设置函数不一定都要定义。
- 只定义`getter` 函数意味着属性是**只读**的，尝试修改属性会被**忽略**。在严格模式下，尝试写入只定义了获取函数的属性会抛出错误。
- 类似地，只有定义`setter`的属性是**不能读取**的，非严格模式下读取会返回 `undefined`，严格模式下会抛出错误。

## 其他定义属性函数
其他还可通过 [`Object.defineProperties()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties) 定义多个属性，使用 `Object.getOwnPropertyDescriptor()`方法可以取得指定属性的属性描述符 ， `Object.getOwnPropertyDescriptors()` 方法可以取得每个自有属性的属性描述符并在一个新对象中返回。
### 合并对象
把源对象所有的属性一起复制到目标对象上，这种操作也被称为“混入”（mixin），因为目标对象通过混入源对象的属性从而得到了增强。
[`Object.assign()` ](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 方法用于将所有**可枚举属性/自有属性**的值从一个或多个源对象分配到目标对象，返回目标对象。
```js
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const returnedTarget = Object.assign(target, source);

console.log(target);
// expected output: Object { a: 1, b: 4, c: 5 }

console.log(returnedTarget);
// expected output: Object { a: 1, b: 4, c: 5 }
```
`Object.assign()` 实际上执行的是**浅复制**，只会复制对象的引用
- 若多个源对象都有相同的属性，则用**最后一个复制的值**。
- 从源对象**访问器属性**取得的值，比如获取函数，会**作为一个静态值**赋给目标对象。也就是说：**不能在两个对象间转移获取函数和设置函数。**
- 赋值期间出错，则操作会中止并退出，同时抛出错误。没有“回滚”之前赋值的概念，因此它是一个尽力而为、**可能只会完成部分复制**的方法。

## 对象标识及相等判定
ES6之前，有些情况利用===操作符进行判断是无法成功的，如
```js
// 这些情况在不同 JavaScript 引擎中表现不同，但仍被认为相等
console.log(+0 === -0); // true 
console.log(+0 === 0); // true 
console.log(-0 === 0); // true 
// 要确定 NaN 的相等性，必须使用极为讨厌的 isNaN() 
console.log(NaN === NaN); // false 
console.log(isNaN(NaN)); // true 
```
ES6规范改善了这类情况，新增了 [`Object.is()` ](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 方法用于判断两个值是否为同一个值
```js
// 正确的 0、-0、+0 相等/不等判定
console.log(Object.is(+0, -0)); // false 
console.log(Object.is(+0, 0)); // true 
console.log(Object.is(-0, 0)); // false 
// 正确的 NaN 相等判定
console.log(Object.is(NaN, NaN)); // true
```
要检查超过两个值，可递归地利用相等性传递即可
```js
function recursivelyCheckEqual(x, ...rest) {
  return Object.is(x, rest[0]) && 
  (rest.length < 2 || recursivelyCheckEqual(...rest));
}
```
## ES6 语法糖
ECMAScript 6 为定义和操作对象新增了很多极其有用的语法糖特性。这些特性都没有改变现有引擎的行为，但极大地提升了处理对象的方便程度。
### 属性名简写
给对象添加变量的时候，经常会发现属性名和变量名是一样的。这个时候就可以使用变量名，不用再写冒号，如果没有找到同名变量，则会抛出 `ReferenceError`。
例如：
```js
let name = 'cosine'; 
let person = { name: name }; 
console.log(person); // { name: 'cosine' }
// 使用语法糖 等价于上面那个
let person = { name }; 
console.log(person); // { name: 'cosine' }
```
代码压缩程序会在不同作用域间保留属性名，以防止找不到引用。
### 可计算属性
可在对象字面量中直接动态的命名属性：
```js
const nameKey = 'name'; 
const ageKey = 'age'; 
const jobKey = 'job'; 
let uniqueToken = 0; 
function getUniqueKey(key) { 
 return `${key}_${uniqueToken++}`; 
} 
let person = { 
 [nameKey]: 'cosine', 
 [ageKey]: 21, 
 [jobKey]: 'Software engineer',
 // 也可以是表达式！
 [getUniqueKey(jobKey+ageKey)]: 'test'
}; 
console.log(person); 
// { name: 'cosine', age: 21, job: 'Software engineer', jobage_0: 'test' }
```
### 简写方法名
直接看：
```js
let person = { 
let person = { 
    // sayName: function(name) { // 旧
    //     console.log(`My name is ${name}`); 
    // } 
    sayName(name) { // 新
        console.log(`My name is ${name}`); 
    } 
}; 
person.sayName('Matt'); // My name is Matt 
```
简写方法名对获取函数和设置函数也是适用的，并且简写方法名与可计算属性键相互兼容，也为后文的类打下了基础

### 对象解构
```js
// 对象解构
let person = { 
    name: 'cosine', 
    age: 21 
}; 
let { name: personName, age: personAge } = person; 
console.log(personName, personAge); // cosine 21 
// 让变量直接使用属性的名称 定义默认值 若未定义默认值则不存在的则为undefined
let { name, age, job = 'test', score } = person; 
console.log(name, age, job, score); // cosine 21 test undefined
```
解构在内部使用函数 `ToObject()`（不能在运行时环境中直接访问）把源数据结构转换为对象。
这意味着在对象解构的上下文中，原始值会被当成对象。也就是说：`null` 和 `undefined` 不能被解构，否则会抛出错误
```js
let { length } = 'foobar'; 
console.log(length); // 6 
let { constructor: c } = 4; 
console.log(c === Number); // true 
let { _ } = null; // TypeError 
let { _ } = undefined; // TypeError
```
想要给**事先声明的变量**解构赋值，则赋值表达式必须**包含在一对括号**中
```js
let personName, personAge; 
let person = { 
 name: 'cosine', 
 age: 21
}; 
({name: personName, age: personAge} = person); 
console.log(personName, personAge); // cosine 21
```
#### 1、嵌套解构
解构对于引用嵌套的属性或赋值目标没有限制。为此，可以通过解构来复制对象属性(浅复制)
```js
let person = { 
    name: 'cosine', 
    age: 21, 
    job: { 
        title: 'Software engineer' 
    } 
}; 
// 声明 title 变量并将 person.job.title 的值赋给它
let { job: { title } } = person; 
console.log(title); // Software engineer 
```
在外层属性没有定义的情况下不能使用嵌套解构。无论源对象还是目标对象都一样

#### 2、部分解构
涉及多个属性的解构赋值是一个输出无关的顺序化操作。如果一个解构表达式涉及多个赋值，开始的赋值成功而**后面的赋值出错**（对不能解构的undefined || null进行解构），则整个解构赋值**只会完成一部分**

#### 3、参数上下文匹配
在函数参数列表中也可以进行解构赋值。对参数的解构赋值不会影响 arguments 对象，但可以在函数签名中声明在函数体内使用局部变量：
```js
let person = { 
    name: 'cosine', 
    age: 21
}; 
function printPerson(foo, {name, age}, bar) { 
    console.log(arguments); 
    console.log(name, age); 
} 
function printPerson2(foo, {name: personName, age: personAge}, bar) { 
    console.log(arguments); 
    console.log(personName, personAge); 
} 
printPerson('1st', person, '2nd'); 
// ['1st', { name: 'cosine', age: 21 }, '2nd'] 
// 'cosine' 21 
printPerson2('1st', person, '2nd'); 
// ['1st', { name: 'cosine', age: 21 }, '2nd'] 
// 'cosine' 21 
```
# 创建对象
ES6开始，正式支持了类和继承，不过这种支持其实是封装了ES5.1构造函数加原型继承的语法糖。
## 工厂模式
在设计模式那篇博客有提到一些设计模式 ([前端设计模式应用笔记](https://ysx.cosine.ren/cn/%E3%80%90%E7%AC%AC%E4%BA%8C%E5%B1%8A%E9%9D%92%E8%AE%AD%E8%90%A5-%E5%AF%92%E5%81%87%E5%89%8D%E7%AB%AF%E5%9C%BA%E3%80%91-%20%E5%89%8D%E7%AB%AF%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E5%BA%94%E7%94%A8/))， 而工厂模式也是一种广泛使用的设计模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。
```js
function createPerson(name, age, job) { 
    let o = new Object(); 
    o.name = name; 
    o.age = age; 
    o.job = job; 
    o.sayName = function() { 
        console.log(this.name); 
    }; 
    return o; 
} 
let person1 = createPerson("cosine", 21, "Software Engineer"); 
let person2 = createPerson("Greg", 27, "Doctor"); 
console.log(person1);   // { name: 'cosine', age: 21, job: 'Software Engineer', sayName: [Function (anonymous)] }
person1.sayName();  // cosine
console.log(person2); // { name: 'Greg', age: 27, job: 'Doctor', sayName: [Function (anonymous)] }
person2.sayName();  // Greg
```
这种模式可以解决创建多个类似对象的问题，但没有解决对象标识问题（即新创建的对象是**什么类型**）

## 构造函数模式
自定义构造函数，以函数的形式为自己的对象类型定义属性和方法。
```js
function Person(name, age, job){ 
    this.name = name; 
    this.age = age; 
    this.job = job; 
    this.sayName = function() { 
        console.log(this.name); 
    }; 
} 
let person1 = new Person("cosine", 21, "Software Engineer"); 
person1.sayName(); // cosine
```
- 没有显式地创建对象
- 属性和方法直接赋值给了 this
- 没有 return
- 要创建 Person 的实例，应使用 new 操作符

### new 过程中发生了什么？
划重点，使用new调用构造函数会执行如下几个操作：
1. 在内存中创建一个新对象
2. 将新对象内部的 `[[Prototype]]` 赋值为构造函数的prototype属性。
3. 构造函数内部的 `this` 指向这个新对象
4. 执行构造函数内部的代码（为对象添加属性）
5. **若构造函数返回非空对象，则返回该对象。否则，返回刚创建的新对象！**

上一个栗子的最后，person1有一个constructor属性指向Person
```js
console.log(person1.constructor)    // [Function: Person]
console.log(person1.constructor === Person)    // true
console.log(person1 instanceof Object); // true 
console.log(person1 instanceof Person); // true 
```
定义自定义构造函数可以**确保实例被标识为特定类型**，相比于工厂模式，这是一个很大的好处。
person1 之所以也被认为是 Object 的实例，是因为所有自定义对象都继承自 Object（后文会提到）

注意以下几点：

 1. **构造函数也是函数** ：任何函数只要使用 new 操作符调用就是构造函数，而不使用 new 操作符调用的函数就是普通函数
 2. **构造函数的主要问题**：其定义的方法会在每个实例上都创建一遍，因此不同实例上的函数虽然同名却不相等，而因为都是做一样的事，所以没必要定义两个不同的 Function 实例。

this 对象可以把函数与对象的绑定**推迟到运行时**，所以可以将函数定义转移到构造函数外部。
这样虽然解决了相同逻辑的函数重复定义的问题，但全局作用域也因此被搞乱了，因为那个函数实际上只能在一个对象上调用。如果这个对象需要多个方法，那么就要在全局作用域中定义多个函数。这会导致自定义类型引用的代码不能很好地聚集一起。而这个新问题可以通过原型模式来解决

## 原型模式
- 每个函数都会创建一个 `prototype` 属性指向原型对象
- 在原型对象上定义的属性和方法可以被所有对象实例共享
- 原来在构造函数中赋给对象实例的值，可以直接赋值给它们的原型

### 1、理解原型
- 无论何时，只要创建一个函数，就会按照特定的规则为这个函数创建一个 `prototype` 属性指向原型对象
- 所有原型对象会获得一个名为 `constructor` 的属性，指回与之关联的构造函数
	-  如`Person.prototype.constructor` 指向 `Person` 
- 然后，可通过构造函数给原型对象添加其他属性和方法
- 原型对象默认只会获得 `constructor` 属性，其他的所有方法都继承自 `Object`
- 每次调用构造函数创建一个新实例，其的内部`[[Prototype]]`指针就会被赋值为构造函数的原型对象
- 脚本中没有访问这个 `[[Prototype]]` 特性的标准方式，但 Firefox、Safari 和 Chrome 会在每个对象上暴露 `__proto__` 属性，通过这个属性可以访问**对象的原型**