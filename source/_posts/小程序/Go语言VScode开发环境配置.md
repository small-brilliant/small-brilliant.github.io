---
title: Go语言VScode开发环境配置
tags: 学习记录
categories: 
- 学习记录
- 想要车小程序开发
- 后端
cover: https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241448220.png
date: 2022-02-24 15:19:00
updated: 2022-02-24 15:19:00
keywords: Go
---
# 配置国内镜像

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

# 在vscode中安装go插件

先在插件市场安装go语言插件

![image-20220224145409898](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241454987.png)

![image-20220224145516364](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241456776.png)

搜索`go：instal`，全选安装

![image-20220224145636572](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241456599.png)

完成安装

![image-20220224145833731](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241458757.png)

# server文件夹

创建文件夹server，然后再

添加的项目中。

注意：**要单独加进这个server文件夹，因为，这样操作，后端的根目录就是server这个文件夹。如果是再项目中加入，那么根目录就不是server**

在这个文件夹目录下输入

```
go mod init coolcar
```

就可以生成go语言的项目coolcar

![image-20220224150647437](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241506468.png)

运行hello world

![image-20220224151630357](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202241516392.png)