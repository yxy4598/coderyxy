---
title: 实现Promise/A+规范并结合ES6
categories: JavaScript
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true

---

### Promise的重要性

- 它是 async/await 语法的基础，是 JavaScript 中处理异步的标准形式。并且，未来的 Web API，只要是异步的，都会以 Promises 的形式出现。
- 如果不理解 Promises 相关的知识和运行机制，将来可能无法完成 Web 开发的日常工作。

### 前期工作

- 首先，你需要了解为什么要去使用Promise，之前的方法有什么缺点，Promise有什么优点
- 熟练地使用Promise，知道<a href="https://www.coderyxy.cn/2022/03/19/JavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/%E4%BB%80%E4%B9%88%E6%98%AFPromise/">Promise的各种特性</a>

### 实现Promise规范

- Promise的结构设计

  - 根据，平时使用Promise为切入点：

  ```javascript
  const promise = new Promise((resolve, reject) => {
  	resolve("data");
  	reject("err message")
  })
  ```

  - 创建Promise时需要传入一个回调函数，这个回调函数会被立即执行，并且传入两个参数，这两个参数为函数，并且接收参数：

  ```javascript
  class XYPromise {
    constructor(executor) {
      const resolve = function(value) {
        console.log("resolve函数执行", value)
      }
      const reject = function(reason) {
        console.log("reject函数执行", reason)
      }
      executor(resolve, reject);
    }
  }
  ```

  - Promise中创建的实例化对象中，resolve和reject不能同时执行，根据Promise使用过程分为三个阶段我们对回调函数的两个参数执行顺序进行重构

  ```javascript
  const PROMISE_STATUS_PENDING = "pending";
  const PROMISE_STATUS_FULFILLED = "fulfilled";
  const PROMISE_STATUS_REJECTED = "rejected";
  
  class XYPromise {
    constructor(executor) {
      this.status = PROMISE_STATUS_PENDING;
      // 这里我们将函数的定义换成箭头函数，是为了取到上层作用域的this，并拿到status的值
      const resolve = (value) => {
        if(this.status === PROMISE_STATUS_PENDING){
          this.status = PROMISE_STATUS_FULFILLED;
          console.log("resolve函数执行", value)
        }
      }
      const reject = (reason) => {
        if(this.status === PROMISE_STATUS_PENDING) {
          this.status = PROMISE_STATUS_REJECTED;
          console.log("reject函数执行", reason)
        }
      }
      executor(resolve, reject);
    }
  }
  ```

- Promise的then方法结构设计

  - 根据promise使用then方法为切入点

  ```
  promise.then(res => {
    console.log(res)
  }, err => {
    console.log(err)
  })
  ```

  - 在then方法中会传入两个回调函数

  ```javascript
  then(onfulfilled, onrejected) {
      this.onfulfilled = onfulfilled;
      this.onrejected = onrejected;
  }
  ```

  - 这两个回调函数需要在resolve或者reject中执行

  ```javascript
  this.onfulfilled(value);
  this.onrejected(reason);
  ```

  如果直接放入resolve和reject函数中，会得到 this.onfulfilled is not a function或者this.onrejected is not a function，这里出现错误的问题在于，创建Promise的时候已经执行了resolve或者reject，我们在创建完成后调用then方法，导致执行函数的时候拿不到传入的回调函数（两者不处于同一<font color="red"> 线程 </font>）

  解决这个问题，需要使用到微任务队列 (queueMicrotask)

  ```javascript
  queueMicrotask(() => {
            // console.log("resolve函数执行", value)
            this.onfulfilled(value);
  })
  queueMicrotask(() => {
            // console.log("reject函数执行", reason)
            this.onrejected(reason);
  })
  ```

- then方法进行优化

  - 优化一：

    - 多个promise调用，只会执行最后一个promise调用的then方法，后一个promise把前一个给覆盖了；

    ```javascript
    promise.then(res => {
      console.log("res1: ", res)
    }, err => {
      console.log("err1: ", err)
    })
    promise.then(res => {
      console.log("res2: ", res)
    }, err => {
      console.log("err2: ", err)
    })
    ```

    解决promise被覆盖的问题：只需要将then方法的回调函数放到数组里，对数组进行遍历来调用回调方法；

    ```javascript
    then(onfulfilled, onrejected) {
        if(onfulfilled) this.onfulfilledFns.push(onfulfilled);
        if(onrejected) this.onrejectedFns.push(onrejected);
      }
    ```

  - 优化二：

    - 延迟调用promise，不会执行XYPromise类中存放的 this.onfulfilledFns 和 this.onrejectedFns函数队列中的函数。

      原因是：在延迟调用then方法时，promise的状态处于fulfilled或者rejected，回调函数无法加入到存放回调函数的数组中；

      解决办法：单独执行延迟调用的函数；

    ```javascript
    then(onfulfilled, onrejected) {
        if(this.status === PROMISE_STATUS_FULFILLED) {
          onfulfilled(this.value);
          //这里需要拿到resolve中传递的值，我们只需要在resolve函数中将该方法绑定到this上即可
        }
        if(this.status === PROMISE_STATUS_REJECTED) {
          onrejected(this.reason);
        }
       	
        if(this.status === PROMISE_STATUS_PENDING) {
          if(onfulfilled) this.onfulfilledFns.push(onfulfilled);
          if(onrejected) this.onrejectedFns.push(onrejected);
        }
      }
    ```

    - 执行微任务队列中改变状态的语句应该放到微任务队列，不然直接改变状态后就会直接执行then方法的if语句直接执行一次resolve或者reject回调，不会执行回调函数的数组；

    ```javascript
    constructor(executor) {
        this.status = PROMISE_STATUS_PENDING;
        this.onfulfilledFns = [];
        this.onrejectedFns = [];
        this.value = undefined;
        this.reason = undefined;
        const resolve = (value) => {
          if(this.status === PROMISE_STATUS_PENDING){
            //微任务队列解决
            queueMicrotask(() => {
              if(this.status !== PROMISE_STATUS_PENDING) return;
              this.value = value;
              this.status = PROMISE_STATUS_FULFILLED;
              this.onfulfilledFns.forEach(fn => {
                fn(this.value);
              })
            })
          }
        }
        const reject = (reason) => {
          if(this.status === PROMISE_STATUS_PENDING) {
            queueMicrotask(() => {
              if(this.status !== PROMISE_STATUS_PENDING) return;
              this.reason = reason;
              this.status = PROMISE_STATUS_REJECTED;
              this.onrejectedFns.forEach(fn => {
                fn(this.reason);
              })
              
            })
          }
        }
        executor(resolve, reject);
      }
    ```

  - 优化三

    ```
    promsie.then(res => {
    	console.log("res1: ", res)
    }, err => {
    	console.log("err1: ", err)
    }).then(res => {
    	console.log("res2: ", res)
    }, err => {
    	console.log("err2: ", err)
    })
    ```

    - 根据Promise的特性可以知道，then方法的返回值应该也是Promise也能使用promise的方法，第一次返回的promise的返回值应该作为第二次promise的结果，相当于在返回的promise中使用resolve和reject方法，并且then方法的回调函数的返回值作为resolve和reject中传递的值；

    ```javascript
    then(onfulfilled, onrejected) {
        return new XYPromise((resolve, reject) => {
          if(this.status === PROMISE_STATUS_FULFILLED) {
            // try {
            //   const value = onfulfilled(this.value);
            //   resolve(value)
            // } catch (err) {
            //   reject(err);
            // }
            /*
             * 每次判断后都会执行try catch代码块 -> 将该代码块抽离出去成为一个单独的函数(execFuntionWithCatchError)
             */
            execFuntionWithCatchError(onfulfilled, this.value, resolve, reject)
          }
          if(this.status === PROMISE_STATUS_REJECTED) {
            execFuntionWithCatchError(onrejected, this.reason, resolve, reject)
          }
        })
      }
    ```

    execFuntionWithCatchError函数

    ```javascript
    // 工具函数
    function execFuntionWithCatchError(exec, value, resolve, reject) {
      try {
        const result = exec(value);
        resolve(result);
      } catch (err) {
        reject(err)
      }
    }
    ```

    - 这里会存在一个问题之前 处于 pending 状态的函数应该如何让它们返回promise？

    ```javascript
    if(this.status === PROMISE_STATUS_PENDING) {
        this.onfulfilledFns.push(this.onfulfilled);
        this.onrejectedFns.push(this.onrejected);
    }
    ```

    可以选择在resolve或者reject函数中执行then方法的回调函数，但是该操作十分麻烦，而且没必要，我们只需要在传入回调函数的时候外层再套一层函数，把回调函数放到内层进行执行，并且结合try catch：

    ```javascript
    if(this.status === PROMISE_STATUS_PENDING) {
        this.onfulfilledFns.push(() => {
          execFuntionWithCatchError(onfulfilled, this.value, resolve, reject)
        });
        this.onrejectedFns.push(() => {
          execFuntionWithCatchError(onrejected, this.reason, resolve, reject)
        });
    }
    ```

  - 优化四

    - 当then方法中的回调函数返回一个结果时，那么它处于fulfilled状态，并且会将结果作为resolve的参数；
      - 情况一：返回一个普通的值；
      - 情况二：返回一个Promise；
      - 情况三：返回一个thenable值；

    判断then返回值的结果时，只有在 execFuntionWithCatchError函数中可以拿到返回值，所以对该函数进行判断

    ```javascript
    // 工具函数
    function execFuntionWithCatchError(exec, value, resolve, reject) {
      try {
        const result = exec(value);
        //判断是否是XYPromise
        if(result instanceof XYPromise) {
          result.then(res => {
            resolve(res);
          }, err => {
            reject(err)
          })
        }else if(result.hasOwnProperty('then') && result.then instanceof Function){
          //判断是否是thenAble
          /**
           * 首先判断该对象里是否有then属性并且为function类型
           * 1. if(result.then)
           * 2. 'then' in result
           * 3. result.hasOwnProperty('then')
           */
          // result.then((res) => {
          //   resolve(res);
          // }, err => {
          //   reject(err);
          // })
          result.then(resolve, reject);
        }else {
          // 返回一个普通的值
          resolve(result);
        }
      } catch (err) {
        reject(err)
      }
    }
    ```

    到这里，Promise中最难的then方法已经编写完成了，其他的就是在then方法的基础上的其他方法

- catch方法

  - 直接调用then方法将第一个值设为undefined；

  ```javascript
  catch(onrejected) {
    this.then(undefined, onrejected);
  }
  ```

  但是会存在问题，在执行回调函数写入回调函数的数组的时候，undefined也会写入，执行undefined()，就会报错

  需要判断传入的回调函数是否是undefined，然后再存入到数组中。

  但是这种方法是第一次promise的回调reject时的结果，而不是在第一次promise中catch到err。（就相当于第一次promise，reject的值返回给第二次promise时拿到结果）。我们需要将第一次promise.then方法中的reject为undefined时抛出给catch接收。

  ```javascript
  return new XYPromise((resolve, reject) => {
    //catch方法 捕获不到第一次promise1的数据，需要第一次promise直接抛出该err
    const defaultOnrejected = err => { throw err }
    onrejected = onrejected || defaultOnrejected;  
    
    if(this.status === PROMISE_STATUS_FULFILLED) {
      execFuntionWithCatchError(onfulfilled, this.value, resolve, reject)
    }
    if(this.status === PROMISE_STATUS_REJECTED) {
      execFuntionWithCatchError(onrejected, this.reason, resolve, reject)
    }
    if(this.status === PROMISE_STATUS_PENDING) {
      if(onfulfilled) this.onfulfilledFns.push(() => {
        execFuntionWithCatchError(onfulfilled, this.value, resolve, reject)
      });
      if(onrejected) this.onrejectedFns.push(() => {
        execFuntionWithCatchError(onrejected, this.reason, resolve, reject)
      });
    }
  })
  ```

- finally方法

  - 根据promise中调用finally的使用方法

  ```javascript
  promise.then(res => {
    console.log(res)
    return "aaa"
  }).catch(err=> {
    console.log(err)
  }).finally(() => {
    console.log("finally执行")
  })
  ```

  - 知道finally的执行方式，无论是处于fulfilled或者rejected他都会执行

  ```javascript
  finally(onfinally) {
    this.then(onfinally, onfinally)
  }
  ```

  但是这里会存在问题，我们只有在调用XYPromise方法的reject()回调他才会执行finally方法。

  这是因为catch相当于第二次promise但是promise中并未使用 resolve方法，第二次promise就未返回新的promise所以无法调用finally方法，就无法打印。

  ```javascript
  //finally方法,由于promise2成功的回调传入的是undefined，所以不会执行，最终不会有返回的XYPromise就无法执行 finally所以这里我们创建 undefined时的成功的回调
    const defaultOnfulfilled = value => { return value }
    onfulfilled = onfulfilled || defaultOnfulfilled;
  ```

  手动在then方法中添加，第二次回调为undefined的默认返回值。

- resolve和reject类方法

  ```javascript
  static resolve(value) {
    return new XYPromise((resolve) => resolve(value))
  }
  
  static reject(reason) {
    return new XYPromise((resolve, reject) => reject(reason))
  }
  ```

  直接创建static类方法，并返回对应操作的新的promise

- all和allSettled类方法

  - 根据两个方法的不同特性

  ```javascript
  static all(promises) {
  return new XYPromise((resolve, reject) => {
    const values = []
    promises.forEach(promise => {
      promise.then(res => {
        values.push(res);
        if(values.length === promises.length) {
          resolve(values)
        }
      }).catch(err => {
        reject(err)
      })
    })
  })
  }
  
  static allSettled(promises) {
  return new XYPromise((resolve, reject) => {
    const result = [];
    promises.forEach(promise => {
      promise.then(res => {
        result.push({status: PROMISE_STATUS_FULFILLED, value: res})
        if(result.length === promises.length){
          resolve(result)
        }
      }, err => {
        result.push({status: PROMISE_STATUS_REJECTED, value: err})
        if(result.length === promises.length){
          resolve(result)
        }
      })
    })
  })
  }
  ```

- race和any类方法

  - 根据两个方法的不同特性

  ```javascript
  static race(promises) {
      return new XYPromise((resolve, reject) => {
        promises.forEach(promise => {
          promise.then(res => {
            resolve(res)
          }).catch(err => {
            reject(err);
          })
        })
      })
    }
    static any(promises) {
      return new XYPromise((resolve, reject) => {
        const reasons = [];
        promises.forEach(promise => {
          promise.then(res => {
            resolve(res)
          }).catch(err => {
            reasons.push(err);
            if(reasons.length === promises.length) {
              reject(new AggregateError(reasons))
            }
          })
        })
      })
    }
  }
  ```

### **澄清迷思**

- **能手写 promises 实现，一定精通 promises？**

在计算机行业，盛行着一种朴素还原论的迷思。即认为越接近底层，技术含量越高。每个程序员都有读懂底层源代码的追求。

这在一定程度上是正确的。不过，我们也应该看到，一旦底层和表层之间，形成了领域鸿沟。精通底层，并不能代表在表层的水平。

比如游戏的开发者，不一定是游戏中的佼佼者。这在 FPS 射击游戏或者格斗游戏里尤为明显，这些游戏里的绝大部分顶尖玩家，完全不会写代码。

如果将精通 promises 定义为，善于在各种异步场景中使用 promises 解决问题。

那么，能手写 promises 实现，对精通 promises 帮助不大。

Promise/A+ 规范，重点围绕的是 How to implement。

而不是 How to use。

通过实现规范，或者所谓的吃透底层源码，以期打通任督二脉的武学幻想，是不切实际的。

想要精通 promises，还得在日常开发的各个异步场景中，多加思考和训练。

有同学一定好奇，如何解释许多精通某技术的开发者，确实能读懂和实现规范？

其实，这是一个典型的【把相关当因果】的案例。

他们不是因为读懂和实现了规范而精通。而是精通该技术使他们产生读规范和实现规范的兴趣。

很少看到有电工试图通过学习电磁学，去精通电工实操，却常常看到开发者框架都还没玩熟，就想啃框架源码，代码还写得一塌糊涂，就想看语言规范。

想跳过扎实的学习和训练过程，直接走最后一步。有点像企图通过只吃最后一那口饭，来吃饱。

- **promises 是比 callback 更先进的异步方案？**

  网络上，许多文章介绍 JavaScript 里的异步方案的演进时，是用下面这种顺序：

  callback -> promise -> generator -> async/await

  其中，callback 被认为是最差的方案，而 async/await 则是所谓的异步终极解决方案。

  从流行趋势来看，这种说法不无道理。我们确实从 callback style 一步步走到了今天大量 async/await 的阶段。

  不过，有些一些有趣的内容，我们可以了解一下。

  - **promises 也属于 callback style 的一种**

    在 Promises/A+ 规范的第一段，我们能看到一个明确的表述：

    

    promise 是通过 then 方法去注册 callbacks，其中 onFulfilled callback 处理 value，而 onRejected callback 处理 reason。

    我们一直诟病的 callback style，通常是指 nodejs 那种 Error-First Callbacks，或者其它 raw callback。

    

    callback 自身是一个很宽泛的概念。

    仿照 state management 的说法，我们可以将 promises 理解为是一种 callback management。

    而 rxjs 则是另一种 callback management，相比 promises 只关心单个异步操作的 value 和 reason，rxjs 可以胜任多个异步操作的 value, error 和 complete 信号。

    如果说 promises 可以处理 0~1 个异步结果，rxjs 则可以处理 0~Infinite 无限多个异步结果。

    一个很合理的设想是，我们能否用 rxjs 实现 Promises/A+ 规范？

    看起来，无非是缩小 rxjs 的处理范围罢了。

    答案是，可以的。

    
    
    如果将 promises 视为一种 callback management，那么之前我们认为的
    
    callback -> promise 的演进，就变成了用一种 callback style 取代另一种 callback style 的过程。 
    
  - **generator function 也是一种 callback style**

至此，我们可以将 JavaScript 里的异步方案演进的表述，重新梳理成如下形式：

Raw Callback Style -> Promise Callback Style -> Generator Callback Style -> Async/Await Callback

它们分别是：

1）Raw Callback Style: 朴素函数作为 callback，接受 error, data 等参数。

2）Promise Callback Style：通过 { then } 对象，去处理 onFulfilled 和 onRejected 两个 callback 函数。

3) Generator Callback Style：通过 * 号和 yield 关键字，将多个嵌套 callback 扁平化的语法糖。

4）Async/Await Callback Style：通过 async 和 await 关键字，将 Promise + Generator Callback Style 语义化和标准化的产物。

新的表述跟旧的表述，总体上是一致的。只是描述口径从 4 个不同事物的演进，变成了同种事物的不同形态的演进。

在这个主流演进路径之外，还存在 Rxjs Callback Style 等其它形态。

对 callback 的本质和背后的概念感兴趣的同学，可以搜索 Continuation 这个术语。



- **async/await 是异步终极解决方案？**

我不太确定当人们说 async/await 是异步终极解决方案时，所描述的终极在什么维度上衡量。

如果是指表达能力的维度，有几个有趣的事实，我们可以了解一下。