---
title: utils.js
categories: utils
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true
---

### 自动柯里化函数

- 将普通函数转换为柯里化函数
- 默认情况下都是没有使用this的情况

```js
function curringFn(fn) {
  return function curried(...args) {
    // 判断当调用的柯里化函数的参数等于被柯里化的函数时直接调用fn函数
    if(args.length >= fn.length) {
      return fn(...args)
    } else {
      // curriedFoo(10) 执行时 需要将第一次调用的参数保留下来所以重新返回一个新的函数
      // 然后通过concat()方法将第一次调用柯里化函数的值跟第二次...第n次的值合并成一个新的数组
      return function (...newArgs) {
        return curried(...args.concat(newArgs))
      }
    }
  } 
}
```

- 柯里化函数显示绑定this时,自动柯里化函数的处理

```js
function curringFn(fn) {
  return function curried(...args) {
    if(args.length >= fn.length) {
      return fn.apply(this, args)
    } else {
      return function (...newArgs) {
        return curried.apply(this, args.concat(newArgs))
      }
    }
  }
}
```

- 测试自动柯里化函数

```js
function log(date, type, message) {
  console.log(`[${date.getHours()}:${date.getMinutes()}] [${type}] [${message}]`)
}

var curriedLog = curringFn(log)
var curriedLogNow = curriedLog(new Date())

curriedLogNow("DEBUG")("嘿嘿发现了一个bug哦~")
```

### 组合函数

- 需要考虑更加复杂的情况：比如传入了更多的函数，在调用compose函数时，传入了更多的参数：

```js
function compose(...fns) {
  var length = fns.length;
  for (const fn of fns) {
    if (typeof fn !== "function") {
      throw new TypeError("Expected a function")
    }
  }
  // 取出所有的函数一次调用
  return function(...args) {
    var index = 0;
    var result = length ? fns[index].apply(this, args) : args
    while(++index < length) {
      result = fns[index].call(this, result)
    }
    return result
  }
}
```

- 测试组合函数

```js
function double(num) {
  return num * 2
}

function square(num) {
  return num ** 2
}

var calcFn = compose(square, double)
console.log(calcFn(10)) // 200
```

