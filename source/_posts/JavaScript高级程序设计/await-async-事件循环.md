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

  ```javascript
async function foo() {
    console.log("function start")
    console.log("中间代码")
    console.log("function end")
    // 2.返回thenable
    return {
      then: function(resolve, reject) {
        resolve("hahah")
      }
    }
  }
  ```
  
  - 情况三：如果我们的异步函数的返回值是一个对象并且实现了thenable，那么会由对象的then方法来决定；
  
  ```javascript
  async function foo() {
    console.log("function start")
    console.log("中间代码")
    console.log("function end")
    // 3.返回一个Promise
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve("123")
      }, 1000)
    })
  }
  
  ```
  
- 如果我们在async中抛出了异常，那么程序它并不会像普通函数一样报错，而是会作为Promise的reject来传递；

  ```javascript
  async function foo() {
    console.log("foo function start~")
    console.log("中间代码~")
    //异步函数中的异常，会被作为异步函数返回的Promise的reject的值
    //如果没有接受 异步函数抛出的异常那么node中就会报promise rejection 。。。错误
    throw new Error("err message")
    console.log("foo function end~")
  }
  // Promise在有异常的情况下必须用catch来捕获异常，否则在node里就会报错
  foo().catch(err => {
    console.log("err: ", err)
  })
  console.log("后续代码~")
  ```

### await关键字

- async函数另外一个特殊之处就是可以在它内部使用await关键字，而普通函数中是不可以的。

- await关键字有什么特点呢？

  - 通常使用await是后面会跟上一个表达式，这个表达式会返回一个Promise；
  - 那么await会等到Promise的状态变成fulfilled状态，之后继续执行异步函数；

  ```javascript
  // 网络请求函数
  function requestData() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve("msg");
        // reject("err message");
      }, 1000);
    })
  }
  ```

  ```javascript
  // 1.await跟上一个表达式（这个表达式会返回Promise的）
  async function foo() {
    //这里相当于生成器中yield返回的结果
    // await 表达式 返回值是Promise
    const res1 = await requestData();
    //res1 就是 返回的Promise就是resolve里的值
    //相当于第一次 then里面的代码
    console.log("代码", res1)
    const res2 = await requestData();
    console.log(res2)
  }
  foo();
  ```

- 如果await后面是一个普通的值，那么会直接返回这个值；

  ```javascript
  async function foo() {
    const res1 = await 123
    // 直接返回
    console.log(res1)
  }
  ```

- 如果await后面是一个thenable的对象，那么会根据对象的then方法调用来决定后续的值；

  ```javascript
  async function foo() {
    // 返回thenable对象中 resolve的值
    const res2 = await {
      then: (resolve, reject) => {
        resolve("abc")
      }
    }
    console.log(res2)
  }
  ```

- 如果await后面的表达式，返回的Promise是reject的状态，那么会将这个reject结果直接作为函数的Promise的 reject值；

  ```javascript
  async function foo() { 
    // 返回Promise中resolve的值
    const res3 = await new Promise((resolve, reject) => {
      resolve("yxy")
    })
    console.log(res3)
  }
  ```

  ```javascript
  // 3.网络请求函数中调用reject
  async function foo() {
    const res = await requestData();
    console.log(res)
    // 如果requsetData中promise调用的reject，则不会打印res
    // reject返回的值他会作为整个异步函数reject的值的
  }
  // reject抛出的值需要接收
  foo().catch(err => {
    console.log("err: ", err)
  })
  ```

### 进程和线程

- 线程和进程是操作系统中的两个概念：
  - 进程（process）：计算机已经运行的程序，是操作系统管理程序的一种方式；
  - 线程（thread）：操作系统能够运行运算调度的最小单位，通常情况下它被包含在进程中；
- 听起来很抽象，这里还是给出解释：
  - 进程：我们可以认为，启动一个应用程序，就会默认启动一个进程（也可能是多个进程）；
  - 线程：每一个进程中，都会启动至少一个线程用来执行程序中的代码，这个线程被称之为主线程；
  - 所以我们也可以说进程是线程的容器；
- 再用一个形象的例子解释：
  - 操作系统类似于一个大工厂；
  - 工厂中里有很多车间，这个车间就是进程；
  - 每个车间可能有一个以上的工人在工厂，这个工人就是线程；

### 操作系统 – 进程 – 线程

<img src="https://gitee.com/billowcloud/picture_bed/raw/master/coderyxy/blog/3.24-1.png">

### 操作系统的工作方式

- 操作系统是如何做到同时让多个进程（边听歌、边写代码、边查阅资料）同时工作呢？
  - 这是因为CPU的运算速度非常快，它可以快速的在多个进程之间迅速的切换；
  - 当我们进程中的线程获取到时间片时，就可以快速执行我们编写的代码；
  - 对于用户来说是感受不到这种快速的切换的；
- 你可以在Mac的活动监视器或者Windows的资源管理器中查看到很多进程：

### 浏览器中的JavaScript线程

- 我们经常会说JavaScript是单线程的，但是JavaScript的线程应该有自己的容器进程：浏览器或者Node。
- 浏览器是一个进程吗，它里面只有一个线程吗？
  - 目前多数的浏览器其实都是多进程的，当我们打开一个tab页面时就会开启一个新的进程，这是为了防止一个页 面卡死而造成所有页面无法响应，整个浏览器需要强制退出；
  - 每个进程中又有很多的线程，其中包括执行JavaScript代码的线程；
- JavaScript的代码执行是在一个单独的线程中执行的：
  - 这就意味着JavaScript的代码，在同一个时刻只能做一件事；
  - 如果这件事是非常耗时的，就意味着当前的线程就会被阻塞；
- 所以真正耗时的操作，实际上并不是由JavaScript线程在执行的：
  - 浏览器的每个进程是多线程的，那么其他线程可以来完成这个耗时的操作;
  - 比如网络请求、定时器，我们只需要在特性的时候执行应该有的回调即可；

### 浏览器的事件循环

- 如果在执行JavaScript代码的过程中，有异步操作呢？
  - 中间我们插入了一个setTimeout的函数调用；
  - 这个函数被放到入调用栈中，执行会立即结束，并不会阻塞后续代码的执行；

```javascript
function sum(num1, num2) {
  return num1 + num2;
}
function bar() {
  return sum(20, 30);
}
setTimeout(() => {
  console.log("settimeout");
})
const result = bar();
console.log(result);
```



### 宏任务和微任务

- 但是事件循环中并非只维护着一个队列，事实上是有两个队列:
  - 宏任务队列（macrotask queue）：ajax、setTimeout、setInterval、DOM监听、UI Rendering等;
  - 微任务队列（microtask queue）：Promise的then回调、 Mutation Observer API、queueMicrotask()等；
- 那么事件循环对于两个队列的优先级是怎么样的呢？
  - 1.main script中的代码优先执行（编写的顶层script代码）；
  - 2.在执行任何一个宏任务之前（不是队列，是一个宏任务），都会先查看微任务队列中是否有任务需要执行
    - 也就是宏任务执行之前，必须保证微任务队列是空的；
    - 如果不为空，那么就优先执行微任务队列中的任务（回调）；

### Promise面试题

- 第一个面试题

  ```javascript
  setTimeout(function () {
    console.log("setTimeout1");
  
    new Promise(function (resolve) {
      resolve();
    }).then(function () {
      new Promise(function (resolve) {
        resolve();
      }).then(function () {
        console.log("then4");
      });
      console.log("then2");
    });
  });
  
  new Promise(function (resolve) {
    console.log("promise1");
    resolve();
  }).then(function () {
    console.log("then1");
  });
  
  setTimeout(function () {
    console.log("setTimeout2");
  });
  
  console.log(2);
  
  queueMicrotask(() => {
    console.log("queueMicrotask1")
  });
  
  new Promise(function (resolve) {
    resolve();
  }).then(function () {
    console.log("then3");
  });
  
  // promise1
  // 2
  // then1
  // queueMicrotask1
  // then3
  // setTimeout1
  // then2
  // then4
  // setTimeout2
  
  ```

- 第二个面试题（字节面试题）

  ```javascript
  async function async1 () {
    console.log('async1 start')
    await async2();
    console.log('async1 end')
  }
   
  async function async2 () {
    console.log('async2')
  }
  
  console.log('script start')
   
  setTimeout(function () {
    console.log('setTimeout')
  }, 0)
   
  async1();
   
  new Promise (function (resolve) {
    console.log('promise1')
    resolve();
  }).then (function () {
    console.log('promise2')
  })
  
  console.log('script end')
  
  // script start
  // async1 start
  // async2
  // promsie1
  // script end
  // async1 end
  // promise2
  // setTimeout
  ```

- 第三个面试题（困难）

  ```javascript
  Promise.resolve().then(() => {
    console.log(0);
    // 1.直接return一个值 相当于resolve(4)
    // return 4;
  
    // 2.return thenable的值
    // return {
    //   then: function(resolve) {
    //     resolve(4)
    //   }
    // }
    
    // 3.return Promise
    return Promise.resolve(4);
  }).then((res) => {
    console.log(res)
  })
  
  
  Promise.resolve().then(() => {
    console.log(1);
  }).then(() => {
    console.log(2);
  }).then(() => {
    console.log(3);
  }).then(() => {
    console.log(5);
  }).then(() =>{
    console.log(6);
  })
  ```

- 第四个面试题（基于Node）

  ```javascript
  async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
  }
  
  async function async2() {
    console.log('async2')
  }
  
  console.log('script start')
  
  setTimeout(function () {
    console.log('setTimeout0')
  }, 0)
  
  setTimeout(function () {
    console.log('setTimeout2')
  }, 300)
  
  setImmediate(() => console.log('setImmediate'));
  
  process.nextTick(() => console.log('nextTick1'));
  
  async1();
  
  process.nextTick(() => console.log('nextTick2'));
  
  new Promise(function (resolve) {
    console.log('promise1')
    resolve();
    console.log('promise2')
  }).then(function () {
    console.log('promise3')
  })
  
  console.log('script end')
  
  // script start
  // async1 start
  // async2
  // promise1
  // promise2
  // script end
  // nextTick1
  // nextTick2
  // async1 end
  // promise3
  // setTimeout0
  // setImmediate
  // setTimeout2
  ```

  

