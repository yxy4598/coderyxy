---
title: ES5实现继承
categories: JavaScript
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true
---

### 认识对象的原型

- **JavaScript当中每个对象都有一个特殊的内置属性 [[prototype]]，这个特殊的对象可以指向另外一个对象。** 
- **那么这个对象有什么用呢？** 
  - 当我们通过引用对象的<font color="#f00">属性key来获取一个value</font>时，它会<font color="#f00">触发 [[Get]]</font>的操作；
  - 这个操作会首先<font color="#f00">检查该对象是否有对应的属性</font>，如果有的话就使用它；
  - <font color="#f00">如果对象中没有改属性，那么会访问对象[[prototype]]内置属性指向的对象上的属性；</font>
- 那么如果通过字面量直接创建一个对象，这个对象也会有这样的属性吗？如果有，应该如何获取这个属性呢？
  - 答案是有的，只要是对象都会有这样的一个内置属性； 
- **获取的方式有两种：**
  - 方式一：通过对象的 `__proto__`  属性可以获取到（但是这个是早期浏览器自己添加的，存在一定的兼容性问题）；
  - 方式二：通过 Object.getPrototypeOf 方法可以获取到；

### 函数的原型 prototype

- **那么我们知道上面的东西对于我们的构造函数创建对象来说有什么用呢？** 
  - 它的意义是非常重大的，接下来我们继续来探讨；
- **这里我们又要引入一个新的概念：所有的函数都有一个prototype的属性（注意：不是`__proto__`）**

```js
function foo() {}

// 所有的函数对象都有 prototype属性
console.log(foo.prototype)
```

- 是不是因为函数是一个对象，所以它有prototype的属性呢？ 
  - 不是的，因为它是一个函数，才有了这个特殊的属性； 
  - 而不是它是一个对象，所以有这个特殊的属性；

```js
var obj = {}
obj.prototype // obj不存在这个属性
```

### new操作符

- **new关键字的步骤如下：** 
  - 在内存中创建一个新的对象（空对象）； 
  - 这个对象内部的[[prototype]]属性会被赋值为该构造函数的prototype属性； 
- 那么也就意味着我们通过Person构造函数创建出来的所有对象的[[prototype]]属性都指向Person.prototype：

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

var p = new Person("yxy", 18)

console.log(p.__proto__)
console.log(Person.prototype)
console.log(Person.prototype === p.__proto__)
```

