---
title: JSON-数据存储
categories: JavaScript
tags: JavaScript
toc: true # 是否启用内容索引
sidebar: true
---

### JSON的由来

- 在目前的开发中，JSON是一种非常重要的数据格式，它并不是编程语言，而是一种可以在服务器和客户端之间传输的数据格式
- JSON的全称是JavaScript Object Notation（JavaScript对象符号）：
  - JSON是由Douglas Crockford构想和设计的一种轻量级资料交换格式，算是JavaScript的一个子集；
  - 但是虽然JSON被提出来的时候是主要应用JavaScript中，但是目前已经独立于编程语言，可以在各个编程语言中使用；
  - 很多编程语言都实现了将JSON转成对应模型的方式；
- 其他的传输格式：
  - **XML**：在早期的网络传输中主要是使用XML来进行数据交换的，但是这种格式在解析、传输等各方面都弱于JSON，所以目前已经很 少在被使用了；
  - **Protobuf**：另外一个在网络传输中目前已经**越来越多使用**的传输格式是protobuf，但是直到2021年的3.x版本才支持JavaScript，所 以目前在前端使用的较少；
- 目前JSON被使用的场景也越来越多：
  - 网络数据的传输JSON数据；
  - 项目的某些配置文件；
  - 非关系型数据库（NoSQL）将json作为存储格式；

### JSON基本语法

- JSON的顶层支持三种类型的值：

  - 简单值：数字（Number）、字符串（String，不支持单引号）、布尔类型（Boolean）、null类型；

    ```json
    "abc"
    ```

  - 对象值：由key、value组成，key是字符串类型，并且必须添加双引号，值可以是简单值、对象值、数组值；

    ```json
    {
      "name": "yxy",
      "age": 18,
      "friend": {
        "name": "kobe"
      },
      "hobbies": ["篮球", "足球"]
    }
    ```

  - 数组值：数组的值可以是简单值、对象值、数组值；

    ```json
    [
      "abc",
      123,
      {
        "name": "abc"
      }
    ]
    ```

### JSON序列化

- 某些情况下我们希望将JavaScript中的复杂类型转化成JSON格式的字符串，这样方便对其进行处理：

  - 比如我们希望将一个对象保存到localStorage中；
  - 但是如果我们直接存放一个对象，这个对象会被转化成 [object Object] 格式的字符串，并不是我们想要的结果；

  ```javascript
  const obj = {
    name: "yxy",
    age: 18,
    friends: {
      name: "kobe"
    },
    hobbies: ["篮球", "足球"]
  }
  
  localStorage.setItem("obj", obj)
  const string = localStorage.getItem("obj");
  console.log(string);
  
  // 将对象存储在localStorage
  // 这样存储是有问题的，第二个参数是传入 String类型，会将对象类型转换成string。
  // obj -> [object Object]
  ```

#### JSON序列化方法

- 在ES5中引用了JSON全局对象，该对象有两个常用的方法：

  - stringify方法：将JavaScript类型转成对应的JSON字符串；
  - parse方法：解析JSON字符串，转回对应的JavaScript类型；

- 那么上面的代码我们可以通过如下的方法来使用：

  ```javascript
  const objString = JSON.stringify(obj);
  console.log(objString);
  // 这是浏览器的一种存储方式，不能在Node环境下测试，需要在浏览器下测试
  localStorage.setItem("obj", objString)
  
  const jsonString = localStorage.getItem("obj");
  // 将JSON格式的字符串转回对象
  const info = JSON.parse(jsonString)
  console.log(info)
  ```

#### Stringify的参数replace

- JSON.stringify() 方法将一个 JavaScript 对象或值转换为 JSON 字符串：

  - 如果指定了一个 replacer 函数，则可以选择性地替换值；
  - 如果指定的 replacer 是数组，则可选择性地仅包含数组指定的属性；

  ```javascript
  // 2.stringify第二个参数replacer
  // 2.1 传入数组：设定哪些是需要转换
  // 只有name属性和friends属性转换成为JSON字符串格式
  const jsonString2 = JSON.stringify(obj, ["name", "friends"]);
  console.log(jsonString2)
  
  // 2.2传入回调函数
  const jsonString3 = JSON.stringify(obj, (key, value) => {
    if (key === 'age') {
      return value + 1;
    }
    return value;
  })
  console.log(jsonString3)
  ```

#### Stringify的参数space

- 当然，它还可以跟上第三个参数space：

  ```javascript
  // 3.stringify第三个参数 space
  // 控制转换的缩进 默认是number类型，也可以是字符串
  // const jsonString4 = JSON.stringify(obj, null, "^^^");
  const jsonString4 = JSON.stringify(obj, null, 2);
  console.log(jsonString4)
  ```

#### toJSON方法

```javascript
const obj = {
  name: "yxy",
  age: 18,
  friends: {
    name: "kobe"
  },
  hobbies: ["篮球", "足球"],
  toJSON() {
    return "123"
  }
}

// 4.如果obj对象中有toJSON方法
// 该方法返回什么调用 JSON.stringify()的对象就返回什么
```

#### parse方法

- JSON.parse() 方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。

  - 提供可选的 reviver 函数用以在返回之前对所得到的对象执行变换(操作)。

  ```javascript
  const JSONString = '{"name":"yxy","age":18,"friends":{"name":"kobe"},"hobbies":["篮球","足球"]}';
  
  // 第二个参数 reviver(兴奋剂), 也是一个回调函数
  // 相当于对转回对象的过程做了一个拦截
  const info = JSON.parse(JSONString, (key, value) => {
    if(key === 'age') {
      return value - 1;
    }
    return value;
  });
  console.log(info)
  ```

### 使用JSON序列化深拷贝

- 另外我们生成的新对象和之前的对象并不是同一个对象：

  - 相当于是进行了一次深拷贝；

  ```javascript
  const obj = {
    name: "yxy",
    age: 18,
    friends: {
      name: "kobe"
    },
    hobbies: ["篮球", "足球"],
    foo() {
      console.log("hello")
    }
  }
  
  // 将obj的内容放到info变量中
  // 1.引用赋值
  const info = obj;
  obj.age = 17;
  console.log(info.age);
  
  // 2.浅拷贝
  const info1 = {...obj};
  obj.age = 16;
  // 浅拷贝只拷贝第一层
  console.log(info1.age);
  // 不拷贝第二层
  obj.friends.name = "james";
  console.log(info1.friends)
  
  // 3.stringify和prase实现深拷贝
  
  const JSONString = JSON.stringify(obj);
  const info2 = JSON.parse(JSONString);
  
  console.log(JSONString);
  // 不打印function
  obj.age = 10;
  console.log(info2.age);
  obj.friends.name = "curry";
  console.log(info2.friends)
  // 但是会存在问题，如果拷贝的对象是存在函数的，那么利用JSON序列化的方式就不能拷贝函数
  ```

- 注意：这种方法它对函数是无能为力的

  - 创建出来的info中是没有foo函数的，这是因为stringify并不会对函数进行处理；









