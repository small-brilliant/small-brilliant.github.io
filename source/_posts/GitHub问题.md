---
title: Git 127.0.0.1:443端口拒绝连接？
tags: 技巧分享
categories: 学习记录
cover: false
date: 2021-03-12 00:00:00
keywords: 技巧
---
在上传，clone都会出现127.0.0.1：443端口拒绝连接的问题，查了很久的，点了个遍都没有解决问题，上传图床的时候也是同样的报错。

最终的解决方案是

1. 是否有代理
2. DNS 设置是否正常

我的问题是第二个方法解决的，打开电脑的网络中心

修改为下面的DNS服务器即可。

![image-20220223094436023](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202230944049.png)