---
title: VScode插件
categories: 工具
tags: 工具
toc: true
sidebar: true
---

# VScode插件推荐

**工欲善其事必先利其器，以下是本人为前端开发收集的vscode插件**

## 1.Chinese Language (vscode汉化)

![](https://gitee.com/billowcloud/picture_bed/raw/master/blog/vscode_chinese.png)

- 安装完成后重启即可

## 2.backgroud-cover

![](https://gitee.com/billowcloud/picture_bed/raw/master/blog/vscode_background.png)

- 安装完成后点击右下角，切换背景图然后选择一张喜欢的背景图片
- ![](https://gitee.com/billowcloud/picture_bed/raw/master/blog/vscode_back.png)
- **<font color=#FF0000 >出现的问题：</font>**不管是哪种方法，都是靠修改vscode底层文件的方法来实现背景图的改变，所以vscode会报警告，此时我们点忽略就ok了；但是在顶部会出现 *[不受支持]* 的字样。
- **<font color=#0000ff >解决方案：</font>** 使用扩展的 Fix VSCode Checksums 插件即可解决
- 同样安装好插件后 ctrl + shift +p 输入命令

```
Fix Checksums: Apply
```

## 3. Debugger for Chrome

![](https://gitee.com/billowcloud/picture_bed/raw/master/blog/vscode_debugger.png)

从VS Code调试在Google Chrome中运行的JavaScript代码。
用于在Google Chrome浏览器或支持Chrome DevTools协议的其他目标中调试JavaScript代码的VS Code扩展。