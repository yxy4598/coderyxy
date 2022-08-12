---
title: 深入理解BFC概念
categories: CSS高级
tags: CSS高级
toc: true # 是否启用内容索引
sidebar: true
---

### FC – Formatting Context

- 什么是FC呢？
- FC的全称是Formatting Context，元素在标准流里面都是属于一个FC的
  - 来自w3c官网解释

![image-20220812164658578](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220812164658578.png)

- 块级元素的布局属于Block Formatting Context（BFC）
  - 也就是block level box都是在BFC中布局的；
- 行内级元素的布局属于Inline Formatting Context（IFC）
  - 而inline level box都是在IFC中布局的；



### BFC – Block Formatting Context

- block level box都是在BFC中布局的，那么这个BFC在哪里呢？

![image-20220812170358674](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220812170358674.png)

- MDN上有整理出在哪些具体的情况下会创建BFC：
  - 根元素（）
  - 浮动元素（元素的 float 不是 none）
  - 绝对定位元素（元素的 position 为 absolute 或 fixed）
  - 行内块元素（元素的 display 为 inline-block）
  - 表格单元格（元素的 display 为 table-cell，HTML表格单元格默认为该值），表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
  - 匿名表格单元格元素（元素的 display 为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、 row、tbody、thead、tfoot 的默认属性）或 inline-table）
  - overflow 计算值(Computed)不为 visible 的块元素
  - 弹性元素（display 为 flex 或 inline-flex 元素的直接子元素）
  - 网格元素（display 为 grid 或 inline-grid 元素的直接子元素）
  -  display 值为 flow-root 的元素

### BFC有什么作用呢？

- 我们来看一下官方文档对BFC作用的描述：

![image-20220812170523213](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220812170523213.png)

- 简单概况如下：
  - 在BFC中，box会在<span style="color: red;">垂直方向上一个挨着一个</span>的排布；
  - <span style="color: red;">垂直方向的间距由margin属性</span>决定；
  - 在**同一个BFC**中，<span style="color: red;">相邻两个box之间的margin会折叠（collapse）</span>；
  - 在BFC中，每个元素的<span style="color: red;">左边缘是紧挨着包含块的左边缘</span>的；
- 那么这个东西有什么用呢？
  -  解决margin的折叠问题；
  - 解决浮动高度塌陷问题；

### BFC的作用一：解决折叠问题（权威）

- 在同一个BFC中，相邻两个box之间的margin会折叠（collapse）
  - 官方文档明确的有说
  - <span style="color: red;">The vertical distance between two sibling boxes is determined by the 'margin' properties. Vertical margins between adjacent block-level boxes in a block formatting context collapse.</span>
  - 那么如果我们让两个box是不同的BFC呢？那么就可以解决折叠问题。

![image-20220812170837023](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220812170837023.png)

### BFC的作用二：解决浮动高度塌陷（权威）

- 网上有很多说法，BFC可以解决浮动高度塌陷，可以实现清除浮动的效果。

  - 但是从来没有给出过BFC可以解决高度塌陷的原理或者权威的文档说明；
  - 他们也压根没有办法解释，为什么可以解决浮动高度的塌陷问题，但是不能解决绝对定位元素的高度塌陷问题呢？

- 事实上，BFC解决高度塌陷需要满足两个条件：

  - 浮动元素的父元素触发BFC，形成独立的块级格式化上下文（Block Formatting Context）；
  - 浮动元素的父元素的浮动元素的父元素的<span style="color: red;">**高度是auto**</span>的；

- BFC的高度是auto的情况下，是如下方法计算高度的

  - 1.如果只有inline-level，是行高的顶部和底部的距离；
  - 2.如果有block-level，是由最底层的块上边缘和最底层 块盒子的下边缘之间的距离
  - 3.如果有绝对定位元素，将被忽略；
  - <span style="color: red;">4.如果有浮动元素，那么会增加高度以包括这些浮动元 素的下边缘</span>

- 这是官方文档对解决浮动问题的解答

  <img src="C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220812171109695.png" alt="image-20220812171109695" style="zoom:90%;" />