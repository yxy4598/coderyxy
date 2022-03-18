---
title: Gitee+hexo博客搭建流程
date: 21/2/11
tags: gitee
categories: 博客
keywords: hexo博客
description: gitee+hexo博客搭建流程
top_img: https://gitee.com/billowcloud/picture_bed/raw/master/img/img%20(45).jpg
cover: https://gitee.com/billowcloud/picture_bed/raw/master/img/img%20(45).jpg
toc: True

---
# 环境变量

- Git
- Node.js
  - 最新版的Node.js已经集成了npm
- npm

**<font color=#FF0000 >官网地址：</font>**

- git： https://git-scm.com/download
- Node.js：http://nodejs.cn/download/



# 开始搭建



## 1.安装 Hexo

```
npm install -g hexo   # 通过npm安装hexo
# -g 指定全局安装，可以使用hexo命令
```

## 2.初始化文件夹

```
hexo init blog  # blog 就是创建的博客文件夹
cd blog    # 进入blog目录
npm install   # 进一步安装hexo所需文件
```

### 初始化的文件目录如下:

```
├── .deploy       #需要部署的文件
├── node_modules  #Hexo插件
├── public        #生成的静态网页文件
├── scaffolds     #模板
├── source        #博客正文和其他源文件等都应该放在这里
|   ├── _drafts   #草稿
|   └── _posts    #文章
├── themes        #主题
├── _config.yml   #全局配置文件
└── package.json
```

## 3.启动 Hexo

```
hexo clean   # 清除所有记录
hexo generate  # 配置信息
hexo server    # 启动本地服务
```

简便方法，一次到位：

```
hexo cl && hexo g && hexo s
```



启动程序后就可以访问 http://localhost:4000。

就可以看到 Hexo 部署的默认博客样式。

从这里，你就成功**1/3**了。

## 4.部署到gitee

码云（gitee）：**https://gitee.com/**

###  注册gitee账户，并且创建仓库。

![image-20210215010257285](https://gitee.com/billowcloud/picture_bed/raw/master/blog/gitee_1.png)

这里仓库名称要尽量和你的**个人空间地址**保持一致，

不一致也没有关系。

### 生成/添加SSH公钥

码云 Gitee 、GitHub 提供了基于 SSH 协议的 Git 服务，在使用 SSH 协议访问仓库仓库之前，需要先配置好账户/仓库的 SSH 公钥。

<font color="#ff0000">**配置 ssh 账户和邮箱**</font>

```
git config --global user.email **@163.com # 设置邮箱
git config --global user.name ''   # 设置用户名
```

<font color="#ff0000">**查看账户和邮箱**</font>

```
git config --global user.name 
git config --global user.email

```

<font color="#ff0000">**本地生成ssh公钥**</font>

邮箱为配置好的账户邮箱

```
ssh-keygen -t rsa -C "**@163.com"
```

按照提示无需配置公钥密码直接**设空**即可，

按照提示完成三次回车，即可生成 ssh key。

<font color="#ff0000">**查看本地公钥**</font>

在你电脑

/C:/user/主用户/.ssh/id_rsa.pub 文件里

<img src="https://gitee.com/billowcloud/picture_bed/raw/master/blog/gitee_2.png" style="zoom:80%;" />

粘贴此处就可以生成本机与gitee的公钥

<font color="#ff0000">**说明一下**</font>

```
gitee、github可以使用一个公钥
```

<font color="#ff0000">**测试是否连接成功**</font>

```
ssh -T git@gitee.com
```

```
# 表示连接成功
Hi “您的用户名”! You've successfully authenticated, but gitee does not provide shell access.
```

### 配置连接到Gitee

打开新创建的仓库，复制项目的地址到hexo的根目录，打开配置文件**_config.yml**

```
deploy:
  type: git #方式为git
  repo: https://gitee.com/***/***.git #仓库的url
  branch: master
  commit: "blog"
```

<font color="#ff0000">**注意：冒号后面一定要有空格，否则不能正确识别。**</font>

###  安装部署博客插件

```
npm install hexo-deployer-git --save
```

这里建议使用淘宝镜像进行安装。

```
cnpm install hexo-deployer-git --save
```

然后一键部署博客

```
hexo cl && hexo g && hexo d
```

你的博客就成功提交到gitee的仓库了。

## 5. 启动 Gitee Pages 服务

打开你的码云仓库找到**管理**

<img src="https://gitee.com/billowcloud/picture_bed/raw/master/blog/gitee_3.png" style="zoom:80%;" />

点击   **初始化readme启用svn访问**

然后就可以启动你的服务了

<img src="https://gitee.com/billowcloud/picture_bed/raw/master/blog/gitee_4.png" style="zoom: 80%;" />







<img src="https://gitee.com/billowcloud/picture_bed/raw/master/blog/gitee_5.png" style="zoom:80%;" />

每一次上传之后都找到服务这里点击**更新**。

如果博客的样式不对，则需要在**_config.yml**中配置下博客地址和路径：

```
url: “https://**.gitee.io”
root: /
```

再执行一键部署代码。

```
hexo cl && hexo g && hexo d
```

就可以看到在线博客啦！

## 6.主题配置

<font color = "#f00">**官网地址：**</font>[https://hexo.io/themes/](https://hexo.io/themes/)

下载自己喜欢的主题，从github上**clone**下来放到**hexo根目录**下的**themes**文件夹里

然后配置 **_config.yml**

```
theme: “您的主题文件夹完整名称”  
# 这里需要注意:后面要有一个空格，名称要和theme下的主题目录名称相同。
```

本地运行 **hexo s** 测试，成功后上传部署到码云。



## 7. 自定义域名

经过上面操作，我们用 Hexo 搭建好自己的 Blog 后，我选择了托管在码云上,现在通过 https://xx0817.gitee.io/blog 这个地址就可以访问了。但是我想通过自己的域名进行访问，要实现这个功能。

###  购买域名

这里不多说，某里云、某讯云、某为云都可以，看自己喜欢。

###  域名解析

我们需要通过 GitHub 网址 ping 出服务器的 IP 地址。可以在本地 cmd 中 ping。也可以在网站上 ping。

我选择网站 ping。

网站：**http://ip.tool.chinaz.com/**

访问后输入自己的 GitHub 部署的博客网址就能 ping 出来了。

进入域名购买的控制台，在解析中添加记录：

添加记录：

主机记录为@，记录类型为A，解析线路选择默认，记录值设置为上面ping出来的IP地址。最少要设置一个，我是四个全部设置了。

再添加记录：

主机记录为www，记录类型为 CNAME，解析线路选择默认，记录值为你的 GitHub 域名，我的为 gdfuturexx.github.io。

上面设置的意思为：

- 设置 A 记录的意思是，当我输入 hongxin.online 这个域名的时候，访问的是 185.199.110.153等这4个IP地址其中一个；
- 设置 CNAME 的意思是，当我访问 gdfuturexx.github.io 这个地址的时候，会跳转到 hongxin.online，之后的过程就和 A 记录相同了，即访问 185.199.110.153等4个IP地址其中一个。

###  添加CNAME文件

在Hexo本地文件夹的source文件夹中，增加一个名为CNAME的无后缀文件。

如果你想地址栏中显示www就输入www.xxx.com，否则输入xxxx.com（你的域名）就行。

之后重新部署即可。

###  GitHub Pages 绑定域名

登录你的GitHub，进入仓库，打开设置。

找到下图位置，在 Custom domain 添加你的自定义域名。

之后刷新一下页面，如果 能勾选Enforce HTTPS就要勾选上。如果不勾选的话访问域名会显示不安全。



本文参考链接 : [https://www.cnblogs.com/yizhixue-hx/](https://www.cnblogs.com/yizhixue-hx/)

