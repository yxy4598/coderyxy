---
title: 深入理解javascript闭包
categories: JavaScript
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true
---

### 又爱又恨的闭包

- 闭包是JavaScript中一个非常容易让人迷惑的知识点：
- 在你不知道的javascript这本书中作者曾这样写到:

![image-20220902213748733](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902213748733.png)

- 闭包确实是JavaScript中一个很难理解的知识点

### JavaScript的函数式编程

- **JavaScript是支持<font color=#f00>函数式编程</font>的**
- **在JavaScript中，函数是非常重要的，并且是一等公民：**
  - 那么就意味着函数的使用是非常灵活的；
  - 函数可以作为另外一个函数的参数，也可以作为另外一个函数的返回值来使用；
- **所以JavaScript存在很多的高阶函数：**
  - 自己编写高阶函数 
  - 使用内置的高阶函数
- **目前在vue3+react开发中，也都在趋向于函数式编程：**
  - vue3 composition api: setup函数 -> 代码（函数hook，定义函数）；
  - react：class -> function -> hooks

### 闭包的定义

- **这里先来看一下闭包的定义，分成两个：<font color=#f00>在计算机科学中和在JavaScript中</font>**
- **在计算机科学中对闭包的定义（维基百科）：**
  - 闭包（英语：Closure），又称**词法闭包**（Lexical Closure）或函数闭包（function closures）；
  - 是在支持 **头等函数** 的编程语言中，实现词法绑定的一种技术； 
  - 闭包在实现上是一个<font color=#f00>**结构体**</font>，它存储了<font color=#f00>**一个函数和一个关联的环境**</font>（相当于一个符号查找表）；
  - 闭包跟函数最大的区别在于，当捕捉闭包的时候，它的 **自由变量** 会在捕捉时被确定，这样即使脱离了捕捉时的上下文，它也能照常运行；
- **闭包的概念出现于60年代，最早实现闭包的程序是 <font color=#f00>Scheme</font>，那么我们就可以理解为什么JavaScript中有闭包：**
  - javascript的作者Brendan Eich 在设计的时候非常喜欢scheme这门语言,所以借鉴了很多里边的使用方式
  - 因为JavaScript中有大量的设计是来源于Scheme的；
- **我们再来看一下MDN对JavaScript闭包的解释：**
  -  一个函数和对其周围状态（lexical environment，**词法环境**）的引用捆绑在一起（或者说函数被引用包围），这样的组合就是**闭包**（closure）；
  - 也就是说，闭包让你可以在一个内层函数中访问到其外层函数的作用域；
  - 在 JavaScript 中，每当创建一个函数，闭包就会在函数创建的同时被创建出来；
- **理解和总结：**
  - 一个普通的函数function，如果它可以访问外层作用域的自由变量，那么这个函数和周围环境就是一个闭包；
  - **<font color=#f00>从广义的角度来说：JavaScript中的函数都是闭包；</font>**
  - **<font color=#f00>从狭义的角度来说：JavaScript中一个函数，如果访问了外层作用域的变量，那么它是一个闭包；</font>**

### 闭包的访问过程

- **如果我们编写了如下的代码，它一定是形成了闭包的：**

![image-20220902214428745](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902214428745.png)

![image-20220902214447609](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902214447609.png)

### 闭包的执行过程

- **那么函数继续执行呢？**
  - 这个时候makeAdder函数执行完毕，正常情况下我们的AO对象会被释放；
  - 但是因为在0xb00的函数中有作用域引用指向了这个AO对象，所以它不会被释放掉；

![image-20220902214557084](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902214557084.png)

### 闭包的内存泄漏

- **那么我们为什么经常会说闭包是有内存泄露的呢？**
  - 在上面的案例中，如果后续我们不再使用add10函数了，那么该函数对象应该要被销毁掉，并且其引用着的父作用域AO也应 该被销毁掉；
  - 但是目前因为在全局作用域下add10变量对0xb00的函数对象有引用，而0xb00的作用域中AO（0x200）有引用，所以最终 会造成这些内存都是无法被释放的；
  - 所以我们经常说的闭包会造成内存泄露，其实就是刚才的引用链中的所有对象都是无法释放的；
- **那么，怎么解决这个问题呢？**
  - 因为当将add10设置为null时，就不再对函数对象0xb00有引用，那么对应的AO对象0x200也就不可达了；
  - 在GC的下一次检测中，它们就会被销毁掉；

```js
add10 = null
```

### 闭包的内存泄漏测试

![image-20220902214757277](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902214757277.png)

### AO不使用的属性优化

- **我们来研究一个问题：AO对象不会被销毁时，是否里面的所有属性都不会被释放？**
  - 下面这段代码中name属于闭包的父作用域里面的变量；
  - 我们知道形成闭包之后count一定不会被销毁掉，那么name是否会被销毁掉呢？
  - 这里我打上了断点，我们可以在浏览器上看看结果；

![](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902214855105.png)

- 浏览器自己做了GC垃圾处理的优化措施释放了没有引用的name变量释放了一定的内存