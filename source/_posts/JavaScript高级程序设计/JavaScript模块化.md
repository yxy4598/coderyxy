---
title: JavaScript模块化
categories: JavaScript
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true
---

### 什么是模块化？

- 到底什么是模块化、模块化开发呢？
  - 事实上模块化开发最终的目的是将程序划分成一个个小的结构；
  - 这个结构中编写属于自己的逻辑代码，有自己的作用域，不会影响到其他的结构；
  - 这个结构可以将自己希望暴露的变量、函数、对象等导出给其结构使用；
  - 也可以通过某种方式，导入另外结构中的变量、函数、对象等；
- 上面说提到的结构，就是模块；按照这种结构划分开发程序的过程，就是模块化开发的过程；
- 无论你多么喜欢JavaScript，以及它现在发展的有多好，它都有很多的缺陷：
  - 比如var定义的变量作用域问题；
  - 比如JavaScript的面向对象并不能像常规面向对象语言一样使用class；
  - 比如JavaScript没有模块化的问题；
- Brendan Eich本人也多次承认过JavaScript设计之初的缺陷，但是随着JavaScript的<font color="red">发展以及标准化</font>，存在的缺陷问题基 本都得到了完善。
- 无论是web、移动端、小程序端、服务器端、桌面应用都被广泛的使用；

### 模块化的历史

- 在网页开发的早期，Brendan Eich开发JavaScript仅仅作为一种脚本语言，做一些简单的表单验证或动画实现等，那个时 候代码还是很少的：
  - 这个时候我们只需要讲JavaScript代码写到script标签中即可；
  - 并没有必要放到多个文件中来编写；甚至流行：通常来说 JavaScript 程序的长度只有一行。
- 但是随着前端和JavaScript的快速发展，JavaScript代码变得越来越复杂了：
  - ajax的出现，前后端开发分离，意味着后端返回数据后，我们需要通过JavaScript进行前端页面的渲染；
  - SPA的出现，前端页面变得更加复杂：包括前端路由、状态管理等等一系列复杂的需求需要通过JavaScript来实现；
  - 包括Node的实现，JavaScript编写复杂的后端程序，没有模块化是致命的硬伤；
- 所以，模块化已经是JavaScript一个非常迫切的需求：
  - 但是JavaScript本身，直到ES6（2015）才推出了自己的模块化方案；
  - 在此之前，为了让JavaScript支持模块化，涌现出了很多不同的模块化规范：<font color="red">AMD、CMD、CommonJS</font>等；

### 没有模块化带来的问题

- 早期没有模块化带来了很多的问题：比如命名冲突的问题
- **当然，我们有办法可以解决上面的问题：立即函数调用表达式（IIFE）**
  - **IIFE** (Immediately Invoked Function Expression)
- 但是，我们其实带来了新的问题：
  - 第一，我必须记得每一个模块中返回对象的命名，才能在其他模块使用过程中正确的使用；
  - 第二，代码写起来混乱不堪，每个文件中的代码都需要包裹在一个匿名函数中来编写；
  - 第三，在没有合适的规范情况下，每个人、每个公司都可能会任意命名、甚至出现模块名称相同的情况；
- **所以，我们会发现，虽然实现了模块化，但是我们的实现过于简单，并且是没有规范的。**
  - 我们需要制定一定的规范来约束每个人都按照这个规范去编写模块化的代码；
  - 这个规范中应该包括核心功能：模块本身可以导出暴露的属性，模块又可以导入自己需要的属性；
  - JavaScript社区为了解决上面的问题，涌现出一系列好用的规范，接下来我们就学习具有代表性的一些规范。

```javascript
// yxy.js
// 缺点：如果两个定义的变量相同，命名冲突
// 解决方式：通过调用立即执行函数解决。
var yxy = (function() {
  var name = 'coderyxy'
  var age = 18;
  var isFlag = true;
  return {
    name,
    age,
    isFlag
  }
})()

// demo.js
var demo = (function() {
  var name = 'demo';
  var isFlag = false;
  return {
    name,
    isFlag
  }
})()

// index.js
if (yxy.isFlag) {
  alert("你好 李银河")
}
```

### CommonJS规范和Node关系

- 我们需要知道CommonJS是一个规范，最初提出来是在浏览器以外的地方使用，并且当时被命名为ServerJS，后来为了 体现它的广泛性，修改为**<font color="red">CommonJS</font>**，平时我们也会简称为CJS。
  - Node是CommonJS在服务器端一个具有代表性的实现；
  -  Browserify是CommonJS在浏览器中的一种实现；
  - webpack打包工具具备对CommonJS的支持和转换；
- 所以，Node中对CommonJS进行了支持和实现，让我们在开发node的过程中可以方便的进行模块化开发：
  - 在Node中每一个js文件都是一个单独的模块；
  - 这个模块中包括CommonJS规范的核心变量：exports、module.exports、require；
  - 我们可以使用这些变量来方便的进行模块化开发；
- 前面我们提到过模块化的核心是导出和导入，Node中对其进行了实现：
  - exports和module.exports可以负责对模块中的内容进行导出；
  - require函数可以帮助我们导入其他模块（自定义模块、系统模块、第三方库模块）中的内容；

#### CommonJS模块化案例

- 基本使用

```javascript
// yxy.js
// 这里导出的对象和main.js文件中导入的对象是同一个对象
const name = "yxy";
const age = 18;
function foo() {
  console.log("foo函数执行")
}
module.exports = {
  // aaa: "aaa",
  // bbb: "bbb"
  name,
  age,
  foo
}
// main.js
// const yxy = require("./yxy.js");
// 对对象进行解构
const { name, age, foo } = require("./yxy.js")

console.log(name);
console.log(age);
foo()

// 导出对象的属性，只允许在导出的文件中使用，不允许在导入的文件中修改
// yxy.js
setTimeout(() => {
  name = "kobe"
}, 1000)

// main.js
setTimeout(() => {
  console.log(name);
  // name = "kobe"
})
```

#### exports导出

- 注意：exports是一个对象，我们可以在这个对象中添加很多个属性，添加的属性会导出；

```javascript
// 第二种导出方式
exports.name = name;
exports.age = age;
exports.foo = foo;

// 第一种导出方式
module.exports = {
  name,
  age,
  foo
}
```

- 另外一个文件中可以导入：

```javascript
const yxy = require("./yxy")
```

- 上面这行完成了什么操作呢？理解下面这句话，Node中的模块化一目了然
  - **意味着main中的yxy变量等于exports对象；**
  - 也就是require通过各种查找方式，最终找到了exports这个对象;
  - 并且将这个exports对象赋值给了yxy变量；
  - yxy变量就是exports对象了；

#### module.exports

- 但是Node中我们经常导出东西的时候，又是通过module.exports导出的：
  
  - module.exports和exports有什么关系或者区别呢？
- 我们追根溯源，通过维基百科中对CommonJS规范的解析：
  - CommonJS中是没有module.exports的概念的；
  - 但是为了实现模块的导出，Node中使用的是Module的类，每一个模块都是Module的一个实例，也就是 module；
  - 所以在Node中真正用于导出的其实根本不是exports，而是module.exports；
  - 因为module才是导出的真正实现者；
- 但是，为什么exports也可以导出呢？
  - 这是因为module对象的exports属性是exports对象的一个引用；
  - 也就是说 module.exports = exports = main中的yxy；
  
  

#### 改变代码发生了什么?

- 我们这里从几个方面来研究修改代码发生了什么？

  - 在三者项目引用的情况下，修改exports中的name属性到底发生了什么？

    - 修改name属性，同时main.js导入的name属性也会发生改变

  - 在三者引用的情况下，修改了main中的yxy的name属性，在yxy模块中会发生什么？

    - 不推荐这样去修改
    - 只允许在导入的包中修改导入的变量，不允许在导出的包中修改变量

  - 如果module.exports不再引用exports对象了，那么修改export还有意义吗？

    ```javascript
    // 两种不能导出的方式
    // 第二种
    exports.name = name;
    exports.age = age;
    exports.foo = foo;
    
    module.exports = {
    
    }
    // module.exports 创建了导出的新对象，而exports的操作还在针对于以前的对象
    // 最终能导出的一定是module.exports
    
    // 第一种
    exports = {
      name,
      age,
      foo
    }
    // 将新创建的对象赋值给exports,并不是直接操作的module.exports对象
    ```

#### require细节

- 我们现在已经知道，require是一个函数，可以帮助我们引入一个文件（模块）中导出的对象。

- 那么，require的查找规则是怎么样的呢？

  - NodeJS中require的查找规则 —》https://nodejs.org/dist/latest-v14.x/docs/api/modules.html

- **总结比较常见的查找规则**：导入格式如下：require(X)

- 情况一： X是一个Node的核心模块，比如path、http

  ```javascript
  const path = require("path")
  const fs = require("fs");
  // node核心模块可以进行相关操作
  path.resolve();
  path.extname();
  
  fs.readFile();
  ```

- 情况二：X是以 ./ 或 ../ 或 /（根目录）开头的

  - 第一步：将X当做一个文件在对应的目录下查找；
    - 如果有后缀名，按照后缀名的格式查找对应的文件
    - 如果没有后缀名，会按照如下顺序：
      - 1> 直接查找文件X
      - 2> 查找X.js文件
      - 3> 查找X.json文件
      - 4> 查找X.node文件
  - 第二步：没有找到对应的文件，将X作为一个目录
    - 查找目录下面的index文件
      - 1> 查找X/index.js文件
      - 2> 查找X/index.json文件
      - 3> 查找X/index.node文件
  - 如果没有找到，那么报错：not found

  ```javascript
  const foo = require("./foo.js")
  ```

- 情况三：直接是一个X（没有路径），并且X不是一个核心模块

  ```javascript
  const axios = require('axios');
  const why = require("yxy");
  
  // 从下方的路径从上往下找
  console.log(module.paths)
  ```

  ```javascript
  [
    'E:\\javaScript\\30_JS模块化解析\\02_CommonJS\\04_require细节\\node_modules',
    'E:\\javaScript\\30_JS模块化解析\\02_CommonJS\\node_modules',
    'E:\\javaScript\\30_JS模块化解析\\node_modules',
    'E:\\javaScript\\node_modules',
    'E:\\node_modules'
  ]
  ```

  - 如果上面的路径中都没有找到，那么报错：not found

#### 模块的加载过程

- 结论一：模块在被第一次引入时，模块中的js代码会被运行一次
- 结论二：模块被多次引入时，会缓存，最终只加载（运行）一次
  - 为什么只会加载运行一次呢？
  - 这是因为每个模块对象module都有一个属性：loaded
  - 为false表示还没有加载，为true表示已经加载；
- 结论三：如果有循环引入，那么加载顺序是什么？
- 如果出现下图模块的引用关系，那么加载顺序是什么呢？
  - 这个其实是一种数据结构：图结构；
  - 图结构在遍历的过程中，有深度优先搜索（DFS, depth first search）和广度优先搜索（BFS, breadth first search）；
  - Node采用的是深度优先算法：main -> aaa -> ccc -> ddd -> eee ->bbb

<img src="http://r9cax60vt.hd-bkt.clouddn.com/image-20220329103529433.png" width="50%">

#### CommonJS规范缺点

- CommonJS加载模块是同步的：
  - 同步的意味着只有等到对应的模块加载完毕，当前模块中的内容才能被运行；
  - 这个在服务器不会有什么问题，因为服务器加载的js文件都是本地文件，加载速度非常快；
- 如果将它应用于浏览器呢？
  - 浏览器加载js文件需要先从服务器将文件下载下来，之后再加载运行;
  - 那么采用同步的就意味着后续的js代码都无法正常运行，即使是一些简单的DOM操作；
- 所以在浏览器中，我们通常不使用CommonJS规范：
  - 当然在webpack中使用CommonJS是另外一回事；
  - 因为它会将我们的代码转成浏览器可以直接执行的代码；
- 在早期为了可以在浏览器中使用模块化，通常会采用AMD或CMD：
  - 但是目前一方面现代的浏览器已经支持ES Modules，另一方面借助于webpack等工具可以实现对CommonJS或者ES Module代码的转换；
  - AMD和CMD已经使用非常少了。

### AMD规范

- AMD主要是应用于浏览器的一种模块化规范：
  - AMD是Asynchronous Module Definition（异步模块定义）的缩写；
  - 它采用的是异步加载模块；
  - 事实上AMD的规范还要早于CommonJS，但是CommonJS目前依然在被使用，而AMD使用的较少了；
- 我们提到过，**规范只是定义代码的应该如何去编写**，只有有了具体的实现才能被应用
  - AMD实现的比较常用的库是require.js和curl.js；

#### require.js的使用

- 第一步：下载require.js

  - 下载地址：https://github.com/requirejs/requirejs
  - 找到其中的require.js文件；

- 第二步：定义HTML的script标签引入require.js和定义入口文件：

  - **data-main属性**的作用是在加载完src的文件后会加载执行该文件；

  ```javascript
  <script src="./lib/require.js" data-main="./src/main.js"></script>
  ```

```javascript
// main.js
(function () {
  require.config({
    baseUrl: "", // 默认情况下baseUrl是 "./src"
    paths: {
      foo: "./src/foo",
      bar: "./src/bar"
    }
  })
  // 在这里请求文件
    
  // 请求foo.js文件和bar.js文件的时候就会执行里边的代码，然后在回调中拿到值
  require(["foo", "bar"], function(foo) {
    console.log("main: ", foo.name)
  })
  
})()
```

```javascript
// bar.js
// define(function() {
//   console.log("--------------")
//   // 这种请求方法保证一定是异步的
//   require(["foo"], function(foo) {
//     console.log("bar: ", foo)
//   })
// })

define(["foo"], function(foo) {
  console.log("---------------")
  console.log("bar: ", foo)
})
```

```javascript
// foo.js
define(function() {
  const name = "yxy"
  const age = 18
  function foo() {
    console.log("foo")
  }
  return {
    name,
    age,
    foo
  }
})
```

### CMD规范

- CMD规范也是应用于浏览器的一种模块化规范：
  - CMD 是Common Module Definition（通用模块定义）的缩写；
  - 它也采用了异步加载模块，但是它将CommonJS的优点吸收了过来；
  - 但是目前CMD使用也非常少了；
- CMD也有自己比较优秀的实现方案：
  - SeaJS

#### SeaJS的使用

- 第一步：下载SeaJS

  - 下载地址：https://github.com/seajs/seajs
  - 找到dist文件夹下的sea.js

- 第二步：引入sea.js和使用主入口文件

  - seajs是指定主入口文件的

  ```javascript
  <script src="./lib/sea.js"></script>
  
  <script>
  seajs.use("./src/main.js")
  </script>
  ```

```javascript
// main.js
define(function(require, exports, module) {
  const foo = require("./foo");
  console.log("main: ", foo)
})

// foo.js
define(function(require, exports, module) {
  const name = "yxy";
  const age = 18;
  function foo() {
    console.log("foo")
  }

  // exports.name = name;
  // exports.age = age;
  // exports.foo = foo;

  module.exports = {
    name,
    age,
    foo
  }
})
```

### 认识 ES Module

- JavaScript没有模块化一直是它的痛点，所以才会产生我们前面学习的社区规范：CommonJS、AMD、CMD等， 所以在ES推出自己的模块化系统时，大家也是兴奋异常。
- ES Module和CommonJS的模块化有一些不同之处：
  - 一方面它使用了import和export关键字；
  - 另一方面它采用编译期的静态分析，并且也加入了动态引用的方式；
- ES Module模块采用export和import关键字来实现模块化:
  - export负责将模块内的内容导出；
  - import负责从其他模块导入内容；
- 了解：采用ES Module将自动采用严格模式：**use strict**

#### 案例代码结构组件

- 这里我在浏览器中演示ES6的模块化开发：

  ```javascript
  <script src="./main.js" type="module"></script>
  ```

- 如果直接在浏览器中运行代码，会报如下错误：

  <img src="http://r9cax60vt.hd-bkt.clouddn.com/image-20220329113314818.png">

- 这个在MDN上面有给出解释：

  - https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules
  - 你需要注意本地测试 — 如果你通过本地加载Html 文件 (比如一个 file:// 路径的文件), 你将会遇到 CORS 错误，因为 Javascript 模块安全性需要。
  - 你需要通过一个服务器(本地服务器)来测试。

- VSCode中有一个插件：Live Server；webstorm本身就是开服务启动

#### exports关键字

- export关键字将一个模块中的变量、函数、类等导出；
- 我们希望将其他中内容全部导出，它可以有如下的方式
  - 方式一：在语句声明的前面直接加上export关键字
  - 方式二：将所有需要导出的标识符，放到export后面的 {}中
    - **注意**：这里的 **{}**里面不是ES6的对象字面量的增强写法，**{}**也不是表示一个对象的；
    - 所以： export {name: name}，是错误的写法；
- 方式三：导出时给标识符起一个别名

```javascript
// 1. 第一种方式：export 声明语句
export const name = "foo";
export const age = 19;

export function foo () {
  console.log("foo")
}
export class Person{

}

// 2.第二种方式：export 导出和声明分开
const name = "why";
const age = 18;
function foo() {
  console.log("foo")
}

// 这里的大括号是个固定的语法，而不是一个对象
export {
  name,
  age,
  foo
}

// 3.第三种方式：第二种导出的时候起别名

export {
  name as fName,
  age as fAge,
  foo as fFoo
}

// 三种导出方式，前两种导出方式运用最多，起别名常用在导入的时候起别名

```

#### import关键字

- import关键字负责从另外一个模块中导入内容
- 导入内容的方式也有多种：
  - 方式一：import {标识符列表} from '模块'；
    - 注意：这里的{}也不是一个对象，里面只是存放导入的标识符列表内容；
  - 方式二：导入时给标识符起别名
  - 方式三：通过 * 将模块功能放到一个模块功能对象（a module object）上

```javascript
// 浏览器加载的时候 url scheme 不允许使用 file:// 可以是 https:// http://

// 1.导入的方式一：普通的导入
import { name, age } from "./foo.js"

// 2.导入方式二：起别名
import { name as fName, age as fAge } from "./foo.js"

// 3.导入方式三：将所有导出的内容发哦到一个标识符中
import * as foo from "./foo.js"

console.log(foo.name);

// console.log(name);
// console.log(age)
```

#### export和import结合使用

- 补充：export和import可以结合使用
  - 该例子中的导出方式二：

```javascript
// 导出方式一：
import { sum, sub } from "./math.js"
import { timeFormat, priceFormat } from "./format.js"
import { request } from "./request.js"
export {
  sum,
  sub,
  timeFormat,
  priceFormat,
  request
}

// 导出方式二：
// 推荐方式
// 阅读性好
export { sum, sub } from "./math.js"
export { timeFormat, priceFormat } from "./format.js"
export { request } from "./request.js"

// 导出方式三：
// 阅读性差
export * from "./math.js"
export * from "./format.js"
export * from "./request.js"

```

- 为什么要这样做呢？
  - 在开发和封装一个功能库时，通常我们希望将暴露的所有接口放到一个文件中；
  - 这样方便指定统一的接口规范，也方便阅读；
  - 这个时候，我们就可以使用export和import结合使用；

#### default用法

- 前面我们学习的导出功能都是有名字的导出（named exports）：
  - 在导出export时指定了名字；
  - 在导入import时需要知道具体的名字；
- 还有一种导出叫做默认导出（default export）
  - 默认导出export时可以不需要指定名字；
  - 在导入时不需要使用 {}，并且可以自己来指定名字；
  - 它也方便我们和现有的CommonJS等规范相互操作；
- **注意**：在一个模块中，只能有一个默认导出（default export）；

```javascript
const name = "foo";
const age = 19;

const foo = "foo value"

// 导出方式叫做 named exports
export {
  name,
  age,

  // 默认导出 default export
  // 方式一
  foo as default
}

// 常用写法 
// 方式二
export default foo;
// 一个文件导出最重要的东西就使用默认导出；

// 默认导出只能有一个
```

#### import函数

- 通过import加载一个模块，是不可以在其放到逻辑代码中的，比如：

- 为什么会出现这个情况呢？

  - 这是因为ES Module在被JS引擎解析时，就必须知道它的依赖关系；
  - 由于这个时候js代码没有任何的运行，所以无法在进行类似于if判断中根据代码的执行情况；
  - 甚至下面的这种写法也是错误的：因为我们必须到运行时能确定path的值；

  ```javascript
  if (true) {
    import sub from './modules/foo.js'
  }
  ```

- 但是某些情况下，我们确确实实希望动态的来加载某一个模块：

  - 如果根据不懂的条件，动态来选择加载模块的路径；
  - 这个时候我们需要使用 import() 函数来动态加载；

  ```javascript
  if (flag) {
    import('./modules/aaa.js').then(res => {
      console.log(res);
    })
  }
  ```

  ```javascript
  // 这种方式的导入相当于是同步的
  // import { name, age, foo } from "./foo.js"
  
  // 直接使用import函数，相当于是异步的
  // import函数的返回值是promise
  import("./foo.js").then(res => {
    console.log("res: ", res)
  })
  
  console.log("后续代码不会运行~~")
  ```

#### import meta

- import.meta是一个给JavaScript模块暴露特定上下文的元数据属性的对象

  - 它包含了这个模块的信息，比如说这个模块的URL；
  - 在ES11（ES2020）中新增的特性；

  ```javascript
  // ES11新增属性
  // meta属性本身也是对象: { url: "当前模块所在的路径" }
  // {url: 'http://127.0.0.1:5500/30_JS%E6%A8%A1%E5%9D%97%E5%8…0/05_ESModule/04_import%E5%87%BD%E6%95%B0/main.js'}
  console.log(import.meta)
  ```

#### ES Module的解析流程(ES Module原理)

- ES Module是如何被浏览器解析并且让模块之间可以相互引用的呢？
  - https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/
- ES Module的解析过程可以划分为三个阶段：
  - 阶段一：构建（Construction），根据地址查找js文件，并且下载，将其解析成模块记录（Module Record）；
  - 阶段二：实例化（Instantiation），对模块记录进行实例化，并且分配内存空间，解析模块的导入和导出语句，把模块指向 对应的内存地址。
  - 阶段三：运行（Evaluation），运行代码，计算值，并且将值填充到内存地址中；

<img src="http://r9cax60vt.hd-bkt.clouddn.com/image-20220329115638408.png">

##### 阶段一：构建阶段

<img src="http://r9cax60vt.hd-bkt.clouddn.com/image-20220329115722408.png">

##### 阶段二和三：实例化阶段 – 求值阶段

<img src="http://r9cax60vt.hd-bkt.clouddn.com/image-20220329115801173.png">