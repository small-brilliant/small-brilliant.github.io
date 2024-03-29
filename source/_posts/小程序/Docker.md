---
title: Docker(一)
tags: Docker
categories: 
- 学习记录
- 后端
- Docker
cover: https://i0.hippopx.com/photos/384/647/698/beaded-colour-pencils-in-the-water-air-bubbles-e7c732442fa6b2c185a89069f50abc3b.jpg
date: 2022-03-02 12:52:17
updated: 2022-03-02 12:52:20
keywords: Docker
---
# 1.Docker简介

## 1.1为什么需要Docker

要如何确保应用能够在不同环境中运行和通过质量检测？并且在部署过程中不出现令人头疼的版本、配置问题，也无需重新编写代码和进行故障修复？

Docker之所以发展如此迅速，也是因为它对此给出了一个标准化的解决方案-----**系统平滑移植，容器虚拟化技术。**安装的时候，把原始环境一模一样地复制过来。开发人员利用 Docker 可以消除协作编码时“在我的机器上可正常工作”的问题。

Docker的出现使得Docker得以打破过去「程序即应用」的观念。透过镜像(images)将作业系统核心除外，**运作应用程式所需要的系统环境，由下而上打包，达到应用程式跨平台间的无缝接轨运作**。

官网：[Empowering App Development for Developers | Docker](https://www.docker.com/)

仓库：[Docker Hub](https://hub.docker.com/)

## 1.2容器VS虚拟机

- 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；

- 容器内的应用进程直接运行于宿主的内核，**容器内没有自己的内核且也没有进行硬件虚拟**。因此容器要比传统虚拟机更为轻便。

-  每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。

## 1.3Docker基本组成

Docker 本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是image镜像文件。只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。

**镜像文件**

image 文件生成的容器实例，本身也是一个文件，称为镜像文件。

**容器实例**

一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器

**仓库**

就是放一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候再从仓库中拉下来就可以了。

# 2.安装及使用

## 2.1安装

**WIN安装**

官网下载

Docker必须依赖部署在Linux内核的系统上，所以win10要安装WSL2

**centos下安装**

1. 移除以前docker相关包

```
sudo yum remove docker*
```

2. 配置yum源

```
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

3. 安装docker

```
sudo yum install -y docker-ce docker-ce-cli containerd.io

#以下是在安装k8s的时候使用
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
```

4. 启动，设置开机启动

```bash
systemctl enable docker --now
```

5. 配置加速

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 2.2Docker常用命

![img](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205111427116.png)

### 1.找镜像

> 去[docker hub](http://hub.docker.com)，找到nginx镜像

```bash
docker pull nginx  #下载最新版

镜像名:版本名（标签）

docker pull nginx:1.20.1


docker pull redis  #下载最新
docker pull redis:6.2.4

## 下载来的镜像都在本地
docker images  #查看所有镜像

redis = redis:latest

docker rmi 镜像名:版本号/镜像id
```

### 2.启动容器

> 启动nginx应用容器，并映射88端口，测试的访问

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

【docker run  设置项   镜像名  】 镜像启动运行的命令（镜像里面默认有的，一般不会写）

# -d：后台运行
# --restart=always: 开机自启
docker run --name=mynginx   -d  --restart=always -p  88:80   nginx


# 查看正在运行的容器
docker ps
# 查看所有
docker ps -a
# 删除停止的容器
docker rm  容器id/名字
docker rm -f mynginx   #强制删除正在运行中的

#停止容器
docker stop 容器id/名字
#再次启动
docker start 容器id/名字

#应用开机自启
docker update 容器id/名字 --restart=always
```

### 3.修改容器内容

```
# 进入容器内部的系统，修改容器内容
docker exec -it 容器id  /bin/bash
```

```
docker run --name=mynginx   \
-d  --restart=always \
-p  88:80 -v /data/html:/usr/share/nginx/html:ro  \
nginx

# 修改页面只需要去 主机的 /data/html
```



### 4.提交改变

### 5.将应用打包成镜像

### 6.推送远程仓库

### 7.部署中间件