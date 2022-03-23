---
title: async-await
categories: JavaScript
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true
---

### 异步函数 async function

- async关键字用于声明一个异步函数：
  - async是asynchronous单词的缩写，异步、非同步；
  - sync是synchronous单词的缩写，同步、同时；
- async异步函数可以有很多中写法：

```javascript
// await/async
async function foo() {

}

const foo2 = async function() {

}

const foo3 = async () => {

}

class Foo {
  async bar() {

  }
}
```

### 异步函数的执行流程

- 异步函数的内部代码执行过程和普通的函数是一致的，默认情况下也是会被同步执行。

```javascript
async function foo() {
  console.log("function start")

  console.log("内部代码执行~")

  console.log("function end")
}
console.log("script start")
// 这个异步函数，如果没有await，就跟普通函数的执行流程一样
// foo();

// 异步代码等执行完普通代码后执行
// setTimeout(() => {
//   console.log("内部代码执行~")
// }, 1000);
console.log("script end")
```

- 异步函数有返回值时，和普通函数会有区别：

  - 情况一：异步函数也可以有返回值，但是异步函数的返回值会被包裹到Promise.resolve中；

    ```javascript
    async function foo() {
      console.log("function start")
      console.log("中间代码")
      console.log("function end")
      // 1.返回一个普通值 /undefined/number~~~
      return 1
    }
    ```

    

  - 情况二：如果我们的异步函数的返回值是Promise，Promise.resolve的状态会由Promise决定；

  - 情况三：如果我们的异步函数的返回值是一个对象并且实现了thenable，那么会由对象的then方法来决定；

