---
title: Git 操作简介
typora-root-url: ./
categories: [Technology]
tags: [Git, 留档备查]
date: 2021-06-29
---

此简易教程的初稿写于大二寒假，用于科协的 FC18 项目开发组的代码远程管理，因为涉及 Unity 项目，所以使用了 Git LFS 和 YAML Merge 等比较高级的工具。

在大三暑假时做过一次修改，用于管理电设小学期团队的机械臂项目 JetArm 的合作开发。新版删除了涉及 Unity 的部分，并做了精简优化，这里发出来作为存档，以便随时参考。

**注意：该教程使用 GitLab，但是与 GitHub 的操作差不多，可以作为参考**

<!--more-->

# Git简易教程

*作者：杨鼎*

*初版日期：2021/1/23*

*修改日期：2021/6/29*

------

注：此为简易教程，详细学习可以参考以下两个网站：

https://www.runoob.com/git/git-tutorial.html

https://www.liaoxuefeng.com/wiki/896043488029600

------

## CASE 1：准备工作

### 1、Git 下载

- 方法一：从官网 https://git-scm.com/downloads 下载（速度比较慢，建议挂VPN）
- 方法二：从我的清华云盘 https://cloud.tsinghua.edu.cn/f/68b3db8ad9b1456b815a/ 下载

### 2、Git 安装

安装过程中会有很多选项，看得懂的部分可以自定义，看不懂的就默认好了。

### 3、Git 简介

Git是分布式版本控制系统，每个人的电脑中都有完整的版本库。

3个区的概念：

- 工作区：在电脑中能看到文件和目录。
- 暂存区：工作区到版本库中间的缓冲，多次暂存，一次提交。
- 版本库：检查完毕，正式提交的内容会存到版本库。

### 4、注册GitLab账号

大家都有并且我已经把大家拉进项目开发组了

### 5、设置Personal Access Tokens

- 进入GitLab，点击右上角人像，选择`Settings`

<img src="/Git-操作简介/setting.png" alt="setting" style="zoom: 33%;" />

- 进入`Access Tokens`界面，输入用户名（随意取），勾选下面全部选项，点`Create personal access token`

![user setting](/Git-操作简介/user setting.png)

- 别忘了将生成的密钥保存下来

![created](/Git-操作简介/created.png)

- 以后，首次通过https方式克隆远程仓库时，需要输入用户名和密钥（就是上面新建的）来证明身份，以后进行远程访问的操作就不需要重复输入了

### 6、添加提交代码时使用的用户名和邮箱

打开 Git Bash，输入以下命令（只需要设置一次）

- 设置用户名，`git config --global user.name "你的姓名"`

  例如 `git config --global user.name "Alex"`

- 设置邮箱， `git config --global user.email 你的邮箱`

  例如 `git config --global user.email Alex123@qq.com`

## CASE 2：从远程仓库克隆到本地开发

*适用情形：已有远程仓库，需要将仓库中的代码克隆到本地进行开发。*

### 1、克隆远程仓库到本地

在你打算存放本地仓库的文件夹下，右键打开`Git Bash `命令行窗口。

输入`git clone https://git.tsinghua.edu.cn/yang-d19/jetarm.git`，clone 后面的就是本项目的地址。敲回车，之后远程仓库就被克隆到本地了。

你在当前文件夹下可以看到新的本地仓库文件夹`jetarm`。

### 2、修改

进入克隆出来的本地仓库，打开`Git Bash`，随便添加一点什么，比如新建一个`test.txt`，然后在里面写一句`hello world`，保存并关闭，此时工作区就被修改了。在命令行中输入`git status`，可以查看当前状态，应有提示`test.txt`还未被添加。

### 3、暂存

`git add . `将工作区中的修改添加到暂存区，这句命令中的`.`表示所有文件，也可以单独指明添加哪个文件，如`git add test.txt`。此时再用`git status`查看状态，应有提示`test.txt`还未被提交。

### 4、提交

`git commit -m "说明"`，例如`git commit -m "add a new file"`，暂存区的更改就被添加到版本库了。此时再输入`git status`查看状态，应有提示本地分支还未被推送到远程仓库。

### 5、推送

`git push`，将本地分支推送到远程仓库，此时用`git status`查看状态，会发现所有更改均已被提交，且本地与远程是同步的。

## CASE 3：将本地代码同步到远程仓库

*适用情形：远程仓库是空的，需将本地代码推送到远程仓库，让队友在你的代码基础上开发。*

### 1、与远程仓库连接

`git init` 初始化本地仓库

`git remote add origin https://git.tsinghua.edu.cn/yang-d19/jetarm.git`，origin 后面的就是远程仓库地址，完成本地仓库与远程仓库的连接。

### 2、暂存

`git add .` 添加所有文件到暂存区。

### 3、提交

`git commit -m 说明`，例如`git commit -m "Initialize"`，添加所有文件到版本库。

### 4、推送

`git push`，推送本地代码到远端。

## CASE 4：日常修改及提交

`git pull`，确保拉取远程仓库中最新的代码，

改完了之后`git commit -a -m 说明`，暂存并提交

`git push`，推送到远程仓库

## CASE 5：解决合并冲突

*适用情形：两人基于相同的版本修改代码，对方比你先将修改推送到远程仓库，此时你在本地的暂存和提交操作均可执行，但是推送到远程仓库时会报错。*

### 1、将远程仓库中的代码拉取到本地

`git pull`拉取远程代码到本地并尝试自动合并。

`git pull`是`git fetch`和`git merge FETCH_HEAD`命令的合体，拉取远程代码到本地后，Git会先尝试自动合并冲突，如果两人的修改不在一个文件中，或者在一个文件中，但互不影响，自动合并就是成功的，此时应当没有报错，直接跳转到第3步；否则进入第2步。

### 2、手动解决冲突

用编辑器如 VSCode 进入有冲突的文件，查找用`<<<<<<<`，`=======`，`>>>>>>>`这样的字符串包围的代码块，删掉对方或自己的改动，或者综合两者以解决冲突，保存并退出。

### 3、暂存且提交

```
git commit -a -m "说明"`，例如`git commit -a -m "resolve merge conflits"
```

这句命令等效于 `git add .` 和 `git commit -m “说明” `两句命令的综合

### 4、推送到远端

`git push`，此时应当没有问题了。
