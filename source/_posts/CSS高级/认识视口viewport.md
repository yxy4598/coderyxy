---
title: 认识视口
categories: CSS高级
tags: 移动端开发
toc: true # 是否启用内容索引
sidebar: true
---

### 认识视口viewport

- 在前面我们已经简单了解过视口的概念了：
  - 在一个浏览器中，我们可以看到的区域就是视口（viewport）；
  - 我们说过fixed就是相对于视口来进行定位的；
  - 在PC端的页面中，我们是不需要对视口进行区分，因为我们的布局视口和视觉视口是同一个；
- 但是在移动端，不太一样，你布局的视口和你可见的视口是不太一样的。
  - 这是因为移动端的网页窗口往往比较小，我们可能会希望一个大的网页在移动端可以完整的显示；
  - 所以在默认情况下，移动端的布局视口是大于视觉视口的；
- 所以在移动端，我们可以将视口划分为三种情况：
  - 布局视口（layout viewport）
  - 视觉视口（visual layout）
  - 理想视口（ideal layout）
- 这些概念的区分，事实上来自ppk，他也是对前端贡献比较大的一个人（特别是在移动端浏览器）
  - https://www.quirksmode.org/mobile/viewports2.html

![image-20220813165506824](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220813165506824.png)

### 布局视口和视觉视口

- 布局视口（layout viewport）
- 默认情况下，一个在PC端的网页在移动端会如何显示呢？
  - 第一，它会按照宽度为980px来布局一个页面的盒子和内容；
  - 第二，为了显示可以完整的显示在页面中，对整个页面进行缩小；
- 我们相对于980px布局的这个视口，称之为**<span style="color: red;">布局视口（layout viewport）；</span>**
  - 布局视口的默认宽度是980px；
- 视觉视口（visual viewport）
  - 如果默认情况下，我们按照980px显示内容，那么右侧有一部分区域 就会无法显示，所以手机端浏览器会默认对页面进行缩放以显示到用 户的可见区域中；
  - 那么显示在可见区域的这个视口，就是视觉视口（visual viewport）
- 在Chrome上按shift+鼠标左键可以进行缩放。

<img src="C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220813181006646.png" alt="image-20220813181006646" style="zoom:67%;" />

### 理想视口（ideal viewport）

- 如果所有的网页都按照980px在移动端布局，那么最终页面都会被缩放显示
  - 事实上这种方式是不利于我们进行移动的开发的，我们希望的是设置100px，那么显示的就是100px；
  - 如何做到这一点呢？通过设置理想视口（ideal viewport）；
- 理想视口（ideal viewport）：
  - 默认情况下的layout viewport并不适合我们进行布局；
  - 我们可以对layout viewport进行宽度和缩放的设置，以满足正常在一个移动端窗口的布局；
  - 这个时候可以设置meta中的viewport；

![image-20220813181211055](C:\Users\86186\AppData\Roaming\Typora\typora-user-images\image-20220813181211055.png)