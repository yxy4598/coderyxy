---
title: Gitee+hexo博客搭建遇到的BUG
date: 21/2/11
tags: gitee
categories: 博客
keywords: BUG
description: 搭建博客踩坑
top_img: https://gitee.com/billowcloud/picture_bed/raw/master/img/img%20(15).jpg
cover: https://gitee.com/billowcloud/picture_bed/raw/master/img/img%20(15).jpg
toc: True
---

# 搭建中踩的第一个坑

<img src="https://gitee.com/billowcloud/picture_bed/raw/master/blog/gitee_bug_1.png" style="zoom:80%;" />

一个hexo方便上传的配置文件憋了一天一夜也是没找到解决方法。

但是功夫不负有心人，最后找到了问题所在:

- nodejs 更迭过快，node14版本会存在一些版本过高的问题。
- npm镜像的问题，下载hexo的deploy插件会出现问题导致上方图片的问题，上传不到远端仓库。

解决方案如下:

- nodejs版本过高，使用低版本nodejs

  node有一个模块n，是专门用来管理node.js的版本的。

  安装 n 模块

  ```
  npm install -g n
  ```

  升级 node.js到最新稳定版

  ```
  n stable
  ```

  安装指定版本

  ```
  n v12.13.11
  ```

- npm问题，通过使用阿里定制的cnpm代替npm，或者npm换成淘宝镜像

  使用阿里定制的cnpm命令行工具代替默认的npm

  ```
  npm install -g cnpm --registry=https://registry.npm.taobao.org
  ```

  检查是否安装成功

  ```
  cnpm -v
  ```

  npm换镜像

  ```
  npm config set registry https://registry.npm.taobao.org
  ```

**<font color = "#ff0000">最后，重新安装插件即可</font>**
