---
title: Git 常用操作
date: 2020-2-23 15:06:01
tag: Git
category: Git
---

## 引言

`git` 的命令有很多，全部都掌握的话比较困难，但根据二八定理，我们只需要熟练掌握一些常用的操作即可满足日常工作所需，以下列举了一些常用场景下的解决方案。

### 关于 git add

> 慎用 `git add .`

它表示把当前工作区的改动全部添加 `trace`，很有可能就会把一些无关文件加进去，一旦 `commit -> push` 就会到远程分支。因此，在执行 `git add` 之前，最好先 `git status` 康一下目前的改动状态，确认可以用的时候再用。

如果不小心使用了它，并且加入了一些不想要的文件，此时该怎么办呢？

- 如果还未进行 `commit`: 可以使用 `git checkout 文件 [-r 目录]` 将其还原

- 如果已经 `commit`: 可以使用 `git rm --cached 文件`，将其从暂存区移除，如果需要还原改动，再使用 `git checkout 文件`。

### 关于 git checkout

>  git checkout [-q] [commit id] [--] <paths>

该命令主要用于检出某一个指定文件。

如果不填写commit id，则默认会从暂存区检出该文件，如果暂存区为空，则该文件会回滚到最近一次的提交状态。当暂存区为空，如果我们想要放弃对某一个文件的修改，可以用这个命令进行撤销：

如果填写commit id（既可以是commit hash也可以是分支名称还可以说tag，其本质上都是commit hash），则会从指定commit hash中检出该文件。用于恢复某一个文件到某一个提交状态。

> git checkout -b new-branch origin/new-branch

项目从远程拉下来后，本地只有一个 `master` 分支，该命令用于在本地新建一个远程分支的拷贝。

### 关于 git rm

> git rm -r --cached file 　　//不删除本地文件，只是移除暂存区

> git rm -r --f file 　　//删除并移除暂存区

### 关于 git push

大多数情况下，我们都是使用这个命令的简写： `git push orign xxbranch`，它的完整形式为：

> git push 远程名 本地分支名:远程分支名

所以，以下等价：

```
git push 

git push 默认远程 当前分支:远程同名分支
```

### 关于分支关联 

给一个项目的本地分支关联到远程对应分支（设定默认trace），此后可以直接使用 `git pull/push` 等简写：

```
官方释义： Branch 'dev_wangdian' set up to track remote branch 'dev_wangdian' from 'origin'.
```

> git branch --set-upstream-to=origin/branch1 本地branch1   
    
可以在推送的时候同时设置 `track`：

> git push -u 远程名 本地分支名

### 关于删除分支

要想删除一个远程分支，可以通过把 `当前分支` 设为空，即：

> git push origin :远程待删除分支

补充一个删除本地分支的命令：

> git branch -d (branchname)

### 配置 gitbash 为中文

> git config --global core.quotepath false