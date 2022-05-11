---
title: Kubernates实战
tags: Kubernates
categories: 
- 学习记录
- 后端
- Docker
cover: https://i0.hippopx.com/photos/384/647/698/beaded-colour-pencils-in-the-water-air-bubbles-e7c732442fa6b2c185a89069f50abc3b.jpg
date: 2022-05-11 19:08:17
updated: 2022-05-11 19:08:20
keywords: Kubernates
---
# 1、Namespace

**资源创建方式**

- 命令行
- YAML 

> 名称空间用来隔离资源

**命令**创建和删除

```bash
kubectl create ns hello
kubectl delete ns hello
```

**yaml**实现创建和删除

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: hello
```

# 2、Pod

## 2.1单个容器Pod

> 运行中的一组容器，Pod是kubernetes中应用的最小单位.Pod中可以有多个容器

创建

- 命令

```bash
kubectl run mynginx --image=nginx
```

集群中的任意一个机器以及任意的应用都能通过Pod分配的ip来访问这个Pod

- yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mynginx
  name: mynginx
#  namespace: default
spec:
  containers:
  - image: nginx
    name: mynginx
```

对于pod的一些命令

```bash
# 查看default名称空间的Pod
kubectl get pod 

# 查看pod的Events
kubectl describe pod 你自己的Pod名字

# 删除
kubectl delete pod Pod名字
# 删除yaml文件也会删除
kubectl delete -f pod.yaml

# 查看Pod的运行日志
kubectl logs Pod名字

# 打印更详细的信息，每个Pod - k8s都会分配一个ip
kubectl get pod -owide

# 使用Pod的ip+pod里面运行容器的端口
curl 192.168.169.136
```

集群中的任意一个机器以及任意的应用都能通过Pod分配的ip来访问这个Pod

## 2.2多容器Pod

创建

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: myapp
  name: myapp
spec:
  containers:
  - image: nginx
    name: nginx
  - image: tomcat:8.5.68
    name: tomcat
```

![image-20220511193325607](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205111933649.png)

```bash
# 访问tomcate
curl 192.168.169.136:8080
# 访问nginx
curl 192.168.169.136:80
```

**一个Pod中不能占用相同端口**

例如下面就会出错，因为都在80端口：

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: myapp
  name: myapp
spec:
  containers:
  - image: nginx
    name: nginx
  - image: nginx
    name: nginx
```

# 3、Deployment

> 控制Pod，使Pod拥有多副本，自愈，扩缩容等能力

清除所有Pod:先查看pod再

```bash
kubectl delete pod myapp mynginx -n default
```

比较下面两个命令有何不同效果？

```bash
kubectl run mynginx --image=nginx
kubectl create deployment mytomcat --image=tomcat:8.5.68
```

> 删除它们所起的服务发现deployment删除后会立马**重启**一个服务

删除deployment

```bash
kubectl get deploy
kubectl delete deploy 名字
```

## 3.1、多副本

```
kubectl create deployment my-dep --image=nginx --replicas=3
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-dep
  name: my-dep
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-dep
  template:
    metadata:
      labels:
        app: my-dep
    spec:
      containers:
      - image: nginx
        name: nginx
```

## 3.2扩缩容

扩容和缩容的时候，就指定replicas的值就可以

```
kubectl scale --replicas=5 deployment/my-dep
```

```
kubectl scale --replicas=3 deployment/my-dep
```

> 还可以通过yaml来修改

```bash
#使用命令打开yaml
kubectl edit deployment/my-dep
#然后修改replicas的值
```

## 3.3滚动更新

更新一个Pod下线旧一个Pod，这是其他的旧Pod都还在。直到更新完。

```bash
# 修改版本号实现更新
kubectl set image deployment/my-dep nginx=nginx:1.16.1 --record
# 查看变化
kubectl get pod -w
```

## 3.5版本回退

```bash
#查看更新历史记录
kubectl rollout history deployment/my-dep

#查看某个历史详情
kubectl rollout history deployment/my-dep --revision=2

#回滚(回到上次)
kubectl rollout undo deployment/my-dep

#回滚(回到指定版本)
kubectl rollout undo deployment/my-dep --to-revision=2
```

> 更多：
>
> 除了Deployment，k8s还有 `StatefulSet` 、`DaemonSet` 、`Job`  等 类型资源。我们都称为 `工作负载`。
>
> 有状态应用使用  `StatefulSet`  部署，无状态应用使用 `Deployment` 部署
>
> https://kubernetes.io/zh/docs/concepts/workloads/controllers/

![image-20220511201447890](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202205112014970.png)

# 4、Service