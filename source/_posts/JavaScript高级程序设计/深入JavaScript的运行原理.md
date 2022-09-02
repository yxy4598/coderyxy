---
title: 深入JavaScript的运行原理
categories: JavaScript
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true
---

### JavaScript代码的执行

- **JavaScript代码下载好之后，是如何一步步被执行的呢？**
- **我们知道，浏览器内核是由两部分组成的，以webkit为例：**
  - <font color="#f00">WebCore</font>：负责HTML解析、布局、渲染等等相关的工作；
  - <font color="#f00">JavaScriptCore</font>：解析、执行JavaScript代码；

![image-20220902201251462](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902201251462.png)

- **另外一个强大的JavaScript引擎就是V8引擎。**

### V8引擎的执行原理

- **我们来看一下官方对V8引擎的定义：**
  - V8是用<font color="#f00">C ++</font>编写的Google开源高性能JavaScript和WebAssembly引擎，它用于Chrome和Node.js等。
  - 它实现ECMAScript和WebAssembly，并在Windows 7或更高版本，macOS 10.12+和使用x64，IA-32，ARM或MIPS处理 器的Linux系统上运行。
  - <font color="#f00">V8可以独立运行，也可以嵌入到任何C ++应用程序</font>中。

![image-20220902201347807](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902201347807.png)

### V8引擎的架构

- V8引擎本身的源码**非常复杂**，大概有超过**100w行C++代码**，通过了解它的架构，我们可以知道它是如何对JavaScript执行的：
- <font color="#f00">Parse</font>模块会将JavaScript代码转换成AST（抽象语法树），这是因为解释器并不直接认识JavaScript代码；
  - 如果函数没有被调用，那么是不会被转换成AST的；
  - Parse的V8官方文档：https://v8.dev/blog/scanner
- <font color="#f00">Ignition</font>是一个解释器，会将AST转换成ByteCode（字节码）
  - 同时会收集TurboFan优化所需要的信息（比如函数参数的类型信息，有了类型才能进行真实的运算）；
  - 如果函数只调用一次，Ignition会解释执行ByteCode；
  - Ignition的V8官方文档：https://v8.dev/blog/ignition-interpreter
- <font color="#f00">TurboFan</font>是一个编译器，可以将字节码编译为CPU可以直接执行的机器码；
  - 如果一个函数被多次调用，那么就会被标记为热点函数，那么就会经过TurboFan转换成优化的机器码，提高代码的执行性能；
  - 但是，<font color="#f00">机器码实际上也会被还原为ByteCode</font>，这是因为如果后续执行函数的过程中，<font color="#f00">类型发生了变化（比如sum函数原来执 行的是number类型，后来执行变成了string类型）</font>，之前优化的机器码并不能正确的处理运算，就会逆向的转换成字节码；
  - TurboFan的V8官方文档：https://v8.dev/blog/turbofan-jit

### V8引擎的解析图（官方）

![image-20220902202018869](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902202018869.png)

- **词法分析（英文lexical analysis）**
  - 将字符序列转换成token序列 的过程。
  - token是**记号化** （tokenization）的缩写 
  - **词法分析器**（lexical analyzer，简称lexer），也 叫**扫描器**（scanner）
- **语法分析（英语：syntactic analysis，也叫 parsing）**
  - **语法分析器也可以称之为 parser。**

### V8引擎的解析图

![image-20220902202335184](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902202335184.png)

### JavaScript代码执行原理 - 版本说明

- **在ECMA早期的版本中（ECMAScript3），代码的执行流程的术语和ECMAScript5以及之后的术语会有所区别：**
  - 目前网上大多数流行的说法都是**基于ECMAScript3版本**的解析，并且在**面试时**问到的大多数都是ECMAScript3的版本内容。
  - 但是ECMAScript3终将过去， **ECMAScript5必然会成为主流**，所以最好也理解ECMAScript5甚至包括ECMAScript6以及更 好版本的内容；
  - 事实上在TC39（ ECMAScript5 ）的最新描述中，和ECMAScript5之后的版本又出现了一定的差异；
- 通过ECMAScript3中的概念学习<font color="#f00">JavaScript执行原理、作用域、作用域链、闭包</font>等概念；
- 通过ECMAScript5中的概念学习<font color="#f00">块级作用域、let、const</font>等概念；
- **事实上，它们只是在对某些概念上的描述不太一样，在整体思路上都是一致的。**

### JavaScript的执行过程

- 假如我们有下面一段代码，它在JavaScript中是如何被执行的呢？

<img src="https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902202711196.png" alt="image-20220902202711196" style="zoom:67%;" />

### 初始化全局对象

- js引擎会在<font color="#f00">**执行代码之前**</font>，会在<font color="#f00">堆内存中创建一个全局对象</font>：Global Object（GO）
  - 该对象 <font color="#f00">所有的作用域（scope）</font>都可以访问；
  - 里面会包含<font color="#f00">Date、Array、String、Number、setTimeout、setInterval</font>等等；
  - 其中还有一个<font color="#f00">window属性</font>指向自己；
- Global Object 的官方解释

![image-20220902202932866](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902202932866.png)

- GO在堆内存中的表示

<img src="https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902202956647.png" alt="image-20220902202956647" style="zoom:67%;" />

### 执行上下文（ Execution Contexts ）

-  js引擎内部有一个**执行上下文栈（Execution Context Stack，简称ECS）**，它是用于执行代**码的调用栈**
- 那么现在它要执行谁呢？执行的是**全局的代码块**：
  - 全局的代码块为了执行会构建一个 **Global Execution Context（GEC）**；
  - GEC会 <font color="#f00">被放入到ECS中</font> 执行；
- **GEC被放入到ECS中里面包含两部分内容：**
  - **第一部分：**在代码执行前，在<font color="#f00">parser转成AST的过程中</font>，会将<font color="#f00">全局定义的变量、函数</font>等加入到GlobalObject中，但是并<font color="#f00">不会 赋值</font>；
    - 这个过程也称之为<font color="#f00">变量的作用域提升（hoisting）</font>
  - **第二部分：**在代码执行中，对变量赋值，或者执行其他的函数；
- 官方对执行上下文的解释:

![image-20220902203616930](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902203616930.png)

### 认识VO对象（Variable Object）

- **每一个执行上下文会关联一个VO（Variable Object，变量对象），变量和函数声明会被添加到这个VO对象中。**

![image-20220902203757695](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902203757695.png)

- **当全局代码被执行的时候，VO就是GO对象了**

  ![image-20220902203833440](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902203833440.png)

### 全局代码执行过程（执行前）

![image-20220902203958821](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902203958821.png)

### 全局代码执行过程（执行后）

![image-20220902204040940](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204040940.png)

### 函数如何被执行呢？

- 在执行的过程中执行到一个函数时，就会根据函数体创建一个函数执行上下文（Functional Execution Context，简称FEC）， 并且压入到EC Stack中。
- **因为每个执行上下文都会关联一个VO，那么函数执行上下文关联的VO是什么呢？**
  - 当进入一个函数执行上下文时，会创建一个<font color="#f00">AO对象（Activation Object）</font>；
  - 这个AO对象会<font color="#f00">使用arguments作为初始化</font>，并且初始值是传入的参数；
  - 这个<font color="#f00">AO对象会作为执行上下文的VO来存放变量的初始化</font>；

![image-20220902204117644](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204117644.png)

### 函数的执行过程（执行前）

![image-20220902204209438](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204209438.png)

### 函数的执行过程（执行后）

![image-20220902204242262](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204242262.png)

### 作用域和作用域链（Scope Chain）

- **当进入到一个执行上下文时，执行上下文也会关联一个作用域链（Scope Chain）**
  - <font color="#f00">作用域链是一个对象列表</font>，用于变量标识符的求值
  - 当进入一个执行上下文时，这个<font color="#f00">作用域链被创建，并且根据代码类型，添加一系列的对象</font>；

![image-20220902204746401](https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204746401.png)

<img src="https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204954841.png" alt="image-20220902204954841" style="zoom: 67%;" />

<img src="https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204850933.png" alt="image-20220902204850933" style="zoom:60%;" />

<img src="https://coderyxy-1304887606.cos.ap-nanjing.myqcloud.com/blog/image-20220902204919680.png" alt="image-20220902204919680" style="zoom: 85%;" />