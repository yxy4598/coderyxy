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

### 实现Promise规范

- 前期准备工作

  - 首先，你需要了解为什么要去使用Promise，之前的方法有什么缺点，Promise有什么优点
  - 其次，熟练地使用Promise，知道Promise的各种特性

  

