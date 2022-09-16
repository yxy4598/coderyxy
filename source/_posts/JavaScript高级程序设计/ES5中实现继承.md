---
title: ES5实现继承
categories: JavaScript
tags: [JavaScript, 原型链]
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

### 创建对象的内存表现

![image-20220916083856044](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916083856044.png)

![image-20220916084016698](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916084016698.png)

![image-20220916084036389](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916084036389.png)

### constructor属性

- 事实上原型对象上面是有一个属性的：constructor
  - 默认情况下原型上都会添加一个属性叫做constructor，这个constructor指向当前的函数对象；

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

var p = new Person("yxy", 18)

console.log(p.name)
console.log(p.__proto__.constructor)
console.log(p.__proto__.constructor.name)
console.log(p.__proto__.constructor.length)
```

### 重写原型对象

- 如果我们需要在原型上添加过多的属性，通常我们会重写整个原型对象：

```js
function Person() {
  
}

console.log(Object.getOwnPropertyDescriptor(Person.prototype, "constructor"))

Person.prototype = {
  name: "yxy",
  age: 18,
  eating: function() {
    console.log("eating")
  }
}
```

- 前面我们说过, 每创建一个函数, 就会同时创建它的prototype对象, 这个对象也会自动获取constructor属性； 
  - 而我们这里相当于给prototype重新赋值了一个对象, 那么这个新对象的constructor属性, 会指向Object构造函数, 而不是 Person构造函数了

### 原型对象的constructor

- 如果希望constructor指向Person，那么可以手动添加：
- 上面的方式虽然可以, 但是也会造成constructor的[[Enumerable]]特性被设置了true. 
  - 默认情况下, 原生的constructor属性是不可枚举的. 
  - 如果希望解决这个问题, 就可以使用我们前面介绍的Object.defineProperty()函数了.

```js
Object.defineProperty(Person.prototype, "constructor", {
  value: Person,
  configurable: true,
  enumerable: false,
  writable: true
})
```

### 创建对象 – 构造函数和原型组合

- 我们在上一个构造函数的方式创建对象时，有一个弊端：会创建出重复的函数，比如running、eating这些函数
  - 那么有没有办法让所有的对象去共享这些函数呢? 
  - 可以，将这些函数放到Person.prototype的对象上即可；

```js
function Person() {
  this.name = "yxy";
}

Person.prototype.running = function() {
  console.log(this.name + "running")
}

function Student(sno) {
  this.sno = sno
}

// 创建中间继承的prototype
var p = new Person();
Student.prototype = p;

stu.running();
```

### 面向对象的特性 – 继承

- 面向对象有三大特性：封装、继承、多态 
  - 封装：我们前面将属性和方法封装到一个类中，可以称之为封装的过程； 
  - 继承：继承是面向对象中非常重要的，不仅仅可以减少重复代码的数量，也是多态前提（纯面向对象中）； 
  - 多态：不同的对象在执行时表现出不同的形态；
- 那么继承是做什么呢？ 
  - 继承可以帮助我们<font color="#f00">将重复的代码和逻辑抽取到父类</font>中，子类只需要直接继承过来使用即可； 
  - 在很多编程语言中，<font color="#f00">继承也是多态的前提</font>；
- **那么JavaScript当中如何实现继承呢？**
  - 我们先来看一下<font color="#f00">JavaScript原型链的机制</font>； 
  - 再利用原型链的机制实现一下继承；

### JavaScript原型链

- **在真正实现继承之前，我们先来理解一个非常重要的概念：原型链。**
  - 我们知道，从一个对象上获取属性，如果在当前对象中没有获取到就会去它的原型上面获取：

![image-20220916085728481](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916085728481.png)

### Object的原型

- 那么什么地方是原型链的尽头呢？比如第三个对象是否也是有原型__proto__属性呢？

![image-20220916085813618](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916085813618.png)

- **我们会发现它打印的是 [Object: null prototype] {}** 
  - 事实上这个原型就是我们最顶层的原型了 
  - 从Object直接创建出来的对象的原型都是 [Object: null prototype] {}。 
- [**Object: null prototype] {} 原型有什么特殊吗？** 
  - 特殊一：<font color="#f00">该对象有原型属性</font>，但是它的原型属性已经指向的是null，也就是已经是顶层原型了；
  - 特殊二：<font color="#f00">该对象上有很多默认的属性和方法</font>；

### 创建Object对象的内存图

![image-20220916085952847](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916085952847.png)

### 原型链关系的内存图

![image-20220916090019827](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916090019827.png)

### Object是所有类的父类

- **从我们上面的Object原型我们可以得出一个结论：<font color="#f00">原型链最顶层的原型对象就是Object的原型对象</font>**

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.running = function() {
  console.log(this.name + "running")
}

var p1 = new Person("yxy", 18)

console.log(p1)
console.log(p1.valueOf())
console.log(p1.toString())

```

![image-20220916090304429](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916090304429.png)

### 通过原型链实现继承

- **如果我们现在需要实现继承，那么就可以利用原型链来实现了：**

```js
function Person() {
  this.name = "yxy";
}

Person.prototype.running = function() {
  console.log(this.name + "running")
}

function Student(sno) {
  this.sno = sno
}

// 创建中间继承的prototype
var p = new Person();
Student.prototype = p;

Student.prototype.studing = function() {
  console.log(this.name + "studing")
}

var stu = new Student();
stu.running();

console.log(stu.sno)

```

### 继承创建对象的内存图

![image-20220916092727510](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916092727510.png)

### 原型链继承的弊端

- **但是目前有一个很大的弊端：某些属性其实是保存在p对象上的；**
  - 第一，我们通过直接打印对象是看不到这个属性的； 
  - 第二，这个属性会被多个对象共享，如果这个对象是一个引用类型，那么就会造成问题； 
  - 第三，不能给Person传递参数（让每个stu有自己的属性），因为这个对象是一次性创建的（没办法定制化）；

### 借用构造函数继承

- **为了解决原型链继承中存在的问题，开发人员提供了一种新的技术: constructor stealing(有很多名称: 借用构造函数或者称之 为经典继承或者称之为伪造对象)：**
- **借用继承的做法非常简单：在子类型构造函数的内部调用父类型构造函数**
  - 因为函数可以在任意的时刻被调用； 
  - 因此通过<font color="red">apply()和call()</font>方法也可以在新创建的对象上执行构造函数；

```js
function Student(sno, name, age) {
  Person.call(this, name, age)
  this.sno = sno
}

// 创建中间继承的prototype
var p = new Person();
// 组合继承最大的问题就是无论在什么情况下，都会调用两次父类构造函数。
Student.prototype = p;
```

### 原型式继承函数

- **原型式继承的渊源**
  - 这种模式要从**道格拉斯·克罗克福德（Douglas Crockford**，著名的前端大师，JSON的创立者）在2006年写的一篇文章说起: **Prototypal Inheritance in JavaScript**(在JavaScript中使用原型式继承) 
  - 在这篇文章中，它介绍了一种继承方法，而且这种继承方法不是通过构造函数来实现的. 
  - 为了理解这种方式，我们先再次回顾一下JavaScript想实现继承的目的：**重复利用另外一个对象的属性和方法.**
- **最终的目的：student对象的原型指向了person对象；**

```js
// 方案二:
var obj = {}
// obj.__proto__ = Person.prototype;
Object.setPrototypeOf(obj, Person.prototype)
Student.prototype = obj

// 方案三:
function F() {}
F.prototype = Person.prototype
Student.prototype = new F();

// 方案四:
// Object.create() 方法用于创建一个新对象，使用现有的对象来作为新创建对象的原型（prototype）。
var obj = Object.create(Person.prototype)
Student.prototype = obj
```

### 寄生式继承函数

- **寄生式(Parasitic)继承**
  - 寄生式(Parasitic)继承是与原型式继承紧密相关的一种思想, 并且同样由道格拉斯·克罗克福德(Douglas Crockford)提出和推 广的；
  - 寄生式继承的思路是**结合原型类继承和工厂模式**的一种方式；
  - 即**创建一个封装继承过程的函数, 该函数在内部以某种方式来增强对象，最后再将这个对象返回；**

```js
// 工具函数
// 创建对象的过程
function createObject(o) {
  function F() {}
  F.prototype = o.prototype;
  return new F();
}

function inherit(Subtype, Supertype) {
  // 兼容性问题
  // var obj = Object.create(Supertype.prototype)
  Subtype.prototype = createObject(Supertype)
  Object.defineProperty(Subtype, "constructor", {
    enumerable: false,
    configurable: true,
    writable: true,
    value: Supertype
  })
}
```

### 寄生组合式继承

- 现在我们来回顾一下之前提出的比较理想的组合继承  组合继承是比较理想的继承方式, 但是存在两个问题:
  - 问题一: **构造函数会被调用两次**: 一次在创建子类型原型对象的时候, 一次在创建子类型实例的时候.
  - 问题二: **父类型中的属性会有两份**: 一份在原型对象中, 一份在子类型实例中. 
- 事实上, 我们现在可以利用寄生式继承将这两个问题给解决掉. 
  - 你需要先明确一点: 当我们在子类型的构造函数中调用父类型.call(this, 参数)这个函数的时候, 就会将父类型中的属性和方法复 制一份到了子类型中. 所以父类型本身里面的内容, 我们不再需要. 
  - 这个时候, 我们还需要获取到一份父类型的原型对象中的属性和方法. 
  - 能不能直接让子类型的原型对象 = 父类型的原型对象呢?
  - 不要这么做, 因为这么做意味着以后修改了子类型原型对象的某个引用类型的时候, 父类型原生对象的引用类型也会被修改.
  - 我们使用前面的寄生式思想就可以了.

### 原型继承关系

![image-20220916093705114](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916093705114.png)

![image-20220916093715067](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220916093715067.png)