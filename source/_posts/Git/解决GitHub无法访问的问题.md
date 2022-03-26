---
title: 解决GitHub无法访问的问题
categories: GitHub
tags: Github
toc: true # 是否启用内容索引
sidebar: true
---

# 站长工具查询DNS

访问 http://tool.chinaz.com/dns/ ，在输入框中填写 github.com，然后点击检测按钮，会列出响应ip

<img src="http://r9cax60vt.hd-bkt.clouddn.com/blog/old-1.png">

也可以通过 https://www.ipaddress.com/ 查询 IP

# 修改hosts文件

编辑C:\Windows\System32\drivers\etc\hosts文件，将列出的响应IP写入该文件中

**只适用于windows系统**

```
# github
52.74.223.119 github.com
13.229.188.59 github.com
```

# 刷新DNS缓存

在cmd命令行执行 ipconfig /flushdns ，刷新DNS缓存

```

C:\Users\test>ipconfig /flushdns
 
Windows IP 配置
 
已成功刷新 DNS 解析缓存。
```

