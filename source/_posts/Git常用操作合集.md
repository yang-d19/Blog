---
title: Git常用操作合集
typora-root-url: ./
categories: [技术]
tags: [Git, 留档备查]
date: 2022-05-09
---

记录了一些 Git 的常用操作，免得每次都重复搜索。

<!--more-->

### 1、基础操作

在当前文件夹初始化一个仓库：

```
git init
```

克隆远程仓库：

```
git clone <storage-url>
```

暂存：

```
git add .
```

提交：

```
git commit -m "说明"
```

推送：

```
git push
```

### 2、远程相关

添加远程仓库：

```
git remote add <strorage-name> <storage-url>
```

查看所有已添加的远程仓库：

```
git remote -v
```

更新仓库信息：

```
git fetch <strorage-name>
```

------

在本地新建一个与远程仓库某分支对应的新分支，并切换过去：

```
git checkout -b <channel-name> <storage-name>/<remote-channel-name>
```

将当前分支上的更新推送到远程仓库的某分支：

```
git push <storage-name> <remote-channel-name>
```

### 3、分支相关

列出所有的分支：

```
git branch
```

创建分支但不切换：

```
git branch <branch-name>
```

删除指定分支，会自动阻止删除包含未合并更改的分支：

```
git branch -d <branch-name>
```

强制删除指定分支：

```
git branch -D <branch-name>
```

重新命名当前分支：

```
git branch -m <branch-name>
```

------

切换到指定分支：

```
git checkout <channel-name>
```

放弃当前工作区所有未暂存的修改：

```
git checkout .
```

放弃工作区某未暂存的文件：

```
git checkout -- <filename>
```

放弃工作区和暂存区的所有修改：

```
git checkout -f
```

------

获取当前分支名：

```
git rev-parse --abbrev-ref HEAD
```

获取当前 commit 完整 hash 值：

```
git rev-parse HEAD
```

获取当前 commit 短 hash 值：

```
git rev-parse --short HEAD
```

### 4、修改记录相关

查看提交历史记录

```
git log
```

几个常用的选项：

```
--oneline          以精简模式显示
--graph            以图形模式显示
--stat             显示文件更改列表
--author='name'    显示某个作者的日志
-p filepath        查看某个文件的详细修改
-L start,end:filepath 查看某个文件某几行范围内的修改记录
--stat commitId    查看某一次提交的文件修改列表
```
