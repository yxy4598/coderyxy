---
title: 什么是Promise
categories: JavaScript
tags: JavaScript
toc: true
sidebar: true
---

### 原始的异步任务的处理

- 我们以实际的例子来作为切入点：
  - 我们调用一个函数，这个函数中发送网络请求（我们使用定时器来模拟网络请求）；
  - 如果发送网络请求成功了，那么告知调用者发送成功，并且将相关数据返回；
  - 如果发送网络请求失败了，那么告知调用者发送失败，并告知错误信息

```javascript
function requestData(url, successCallback, failtureCallback) {
  setTimeout(() => {
    if(url === "coderyxy") {
      let names = ['yxy', 'abc']
      successCallback(names);
    }else {
      let msg = "请求失败"
      failtureCallback(msg);
    }
  }, 1000)
}

requestData("coderyxy", res => {
  console.log(res)
}, err => {
  console.log(err)
})
```

### 什么是Promise呢？

- 在上面的解决方案中，我们确确实实可以解决请求函数得到结果之后，获取到对应的回调，但是它存在两个主要的问题
  - 第一，我们需要自己来设计回调函数、回调函数的名称、回调函数的使用等；
  - 第二，对于不同的人、不同的框架设计出来的方案是不同的，那么我们必须耐心去看别人的源码或者文档，以 便可以理解它这个函数到底怎么用；
- Promise的API是怎么样的：
  - Promise是一个类，可以翻译成 承诺、许诺 、期约；
  - 当我们需要给予调用者一个承诺：待会儿我会给你回调数据时，就可以创建一个Promise的对象；
  - 在通过new创建Promise对象时，我们需要传入一个回调函数，我们称之为executor
    - 这个回调函数会被立即执行，并且给传入另外两个回调函数resolve、reject；
    - 当我们调用resolve回调函数时，会执行Promise对象的then方法传入的回调函数；
    - 当我们调用reject回调函数时，会执行Promise对象的catch方法传入的回调函数；

### Promise的代码结构

```javascript
const promise = new Promise((resolve, reject) => {
    resolve('成功');
    reject('失败');
})

promise.then(res => {
    console.log(res);
}, err => {
    console.log(err);
})
```

- Promise的使用过程，分为三个阶段
  - 待定（pending）：初始状态，既没有被兑现，也没有被拒绝；
    - 当执行executor中的代码时，处于该状态；
  - 已兑现（fulfilled）: 意味着操作成功完成；
    - 执行了resolve时，处于该状态；
  - 已拒绝（rejected）: 意味着操作失败；
    - 执行了reject时，处于该状态；

### Promise对原始异步请求重构

```javascript
function requestData(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if(url == 'coderyxy') {
        let names = ['xy', 'kobe', 'james']
        resolve(names)
      }else {
        let msg = "err Url"
        reject(msg)
      }
    }, 1000);
  })
}
```

### Executor

- Executor是在创建Promise时需要传入的一个回调函数，这个回调函数会被立即执行，并且传入两个参数；

  ```javascript
  new Promsie((resolve, reject) => {})
  ```

- 通常我们会在Executor中确定我们的Promise状态：

  - 通过resolve，可以兑现（fulfilled）Promise的状态，我们也可以称之为已解决（resolved）；
  - 通过reject，可以拒绝（reject）Promise的状态；

- 这里需要注意：一旦状态被确定下来，Promise的状态会被 锁死，该Promise的状态是不可更改的

  - 在我们调用resolve的时候，如果resolve传入的值本身不是一个Promise，那么会将该Promise的状态变成 兑 现（fulfilled）；
  - 在之后我们去调用reject时，已经不会有任何的响应了（并不是这行代码不会执行，而是无法改变Promise状 态）；

### resolve不同值的区别

- 情况一：resolve中传入的是一个普通的值或对象，那这个值就会作为then回调的参数；

  ```javascript
  new Promise((resolve, reject) => {
  	resolve('msg')
  }).then(res => console.log(res))
  ```

  

- 情况二：resolve中传入的是一个Promise对象，那么这个新的Promise决定原来Promise的状态

  ```javascript
  new Promise((resolve, reject) => {
      resolve new Promise((resolve, reject) => {
          resolve("新的Promise的resolve")
      })
  }).then(res => console.log(res))
  ```

  

- 情况三：resolve中传入的是一个thenable对象，会执行该then方法，并且根据then方法的结果来决定Promise的状态；

  ```javascript
  new Promise((resolve, reject) => {
      resolve({
          then: (resolve, reject) => {
              resolve('thenable对象的msg')
          }
      })
  }) .then(res => console.log(res))
  ```

### then方法



