---
title: 换电脑了，可是我的Bolg怎么办？
tags: 技巧分享
categories: 学习记录
cover: https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/202202231031988.jpg
date: 2021-03-12 00:00:00
keywords: 技巧
---

一不小心换了电脑，一不小心忘记了还有我的`blog`。使用了`Hexo+github`搭建的`blog`怎么使用`git`来同步文件呢？

# 1.思路

`Hexo`框架使用`hexo d`命令上传部署到`github`的其实是`hexo`编译后的静态网站，不包含源文件（看`blog`仓库应该就会发现）。而上传的这些文件都在`deploy_git`里面。其他所有文件，都没有上传到`github`。

这是我们可以利用`git`的分支管理，将源文件上传到`github`的另一分支，用这个分支来专门保存文件。

# 2.源文件上传操作

在`blog`仓库创建一个分支，专门用来存储`hexo`的一些文件。在换电脑的时候，只需要clone这些文件就可以愉快的写文章啦

## 2.1.建立分支

1. 在下面的搜索框里直接输入想要创建的分支名称即可创建一个分支。

![image-20210312230529040](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210312230536.png)

2. 之后将创建的分支设置为默认分支，这样每次同步的时候就不用指定分支，比较方便。

   ![image-20210312230839585](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210312230839.png)

## 2.2.上传文件到分支

1. 在本地的任意目录下，打开`git bash`输入下面命令，将这个分支克隆到本地，应为已经设置了`hexo`为默认的分支，所以不指定分支的情况下，就会`clone` `hexo`分支的内容。

   ```text
   git clone git@github.com:small-brilliant/small-brilliant.github.io.git
   ```

2. `clone`好后将**除**`.git`文件**之外**的所有文件全部删除，再执行下面命令。不出意外，`git`仓库的`hexo`分支所有文件都删除了。

   ```text
   git add -A
   git commit -m "some description"
   git push
   ```

   

3. 接下来就要上传我们的源文件了。将我们之前写博客的文件夹里面的所有文件复制过来，**除了**`.deploy_git`。**如果你有自定义的主题的话**，一定要删除主题文件中的`.git`文件夹，因为`git`不能嵌套上传。然后在这个文件夹打开`git bash`。输入下面命令，就上传了。

   ```text
   git add .
   git commit –m "add branch"
   git push 
   ```

![image-20210312232435711](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210312232435.png)

结束！

# 3.下载以及环境部署

在另一台电脑上，如下操作

1. 安装`git`和`node.js`

2. 设置`git`全局邮箱和用户名

```text
git config --global user.name "yourgithubname"
git config --global user.email "yourgithubemail"
```

3. 设置`ssh key`。

   1. 输入下面命令，会在`C:\Users\hui\.ssh`生成公钥和私钥。

      ```text
      ssh-keygen -t rsa -C "youremail"
      ```

   2. 将公钥`id_rsa.pub`打开复制里面的内容粘贴到下面图所示的`key`文本框中

      <img src="https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210312233257.png" alt="屏幕截图 2021-03-12 233244" style="zoom:50%;" />

      ![image-20210312233201949](https://cdn.jsdelivr.net/gh/small-brilliant/image/img1/20210312233202.png)

4. 验证是否成功

   ```text
   ssh -T git@github.com
   ```

5. 安装`hexo`但是不需要初始化。

   ```text
   npm install hexo-cli -g
   ```

6. 然后进入克隆到的文件夹。安装环境

   ```text
   npm install
   npm install hexo-deployer-git --save
   ```

7. 验证

   ```text
   hexo g
   hexo s
   ```

然后就可以开始写你的新博客了。每次写完都要把源文件上传一下。保持同步

```text
git add .
git commit –m "xxxx"
git push 
```

如果是在已经编辑过的电脑上，已经有clone文件夹了，那么，每次只要和远端同步一下就行了。

```text
git pull
```

写完博客用`hexo d`部署。

# 参考

https://www.zhihu.com/question/21193762

https://www.jianshu.com/p/fceaf373d797