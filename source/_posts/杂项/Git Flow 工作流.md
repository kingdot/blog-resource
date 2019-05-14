---
title: GitFlow工作流
date: 2019-5-14 21:39:01
tag: git
category: 杂项
---

#### 基本概念

1. git本地仓库的三个组成部分：工作区（Working Directoty），暂存区（Stage），历史记录区（History）

    - 工作区：在git管理下的正常目录都算是工作区，我们平时的编辑工作都是在工作区完成
    
    - 暂存区：临时区域。里面存放将要提交文件的快照
    
    - 历史区：git commit 后的记录区
    
    三个区的转换关系以及转换所使用的命令：
    
    ![](https://user-gold-cdn.xitu.io/2017/10/22/47ecaa2f458807fa3793b1b589aeaadd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

2. git reset 和 git reverse 和 git checkout 的区别

    - 共同点：用来撤销代码仓库中的某些更改
    
    - 不同点：
    
        1. git reset 可以将一个分支的末端指向之前的一个commit。然后在下次git执行垃圾回收的时候，会把这个commit之后的commit都扔掉。git reset还支持三种标记，用来标记reset指令影响的范围：
        
            - --mixed: 会影响暂存区和历史记录区。默认值
            
            - --soft：只影响历史记录区
            
            - --hard；三个区都有影响
            
        2. git checkout 可以将HEAD移到一个新的分支，并更新工作目录。因为可能会覆盖本地的修改，所以执行这个指令之前，需要stash或者commit暂存区和工作区的修改
        
        3. git revert 和 git reset 的目的是一样的，但是做法不同，他会以创建新的 commit的方式来撤销commit，这样能保留之前的commit历史，比较安全。
        
#### GitFlow 基本流程

名称 | 说明
:---: | :---
master | 主分支
develop | 主开发分支
feature | 新功能分支，一般一个新功能对应一个分支，对于功能的拆分需要比较合理，以避免一些后面不必要的代码冲突
release | 发布分支，发布时候用的分支，一般测试时候发现的bug在这个分支进行修复
hotfix | hotfix 分支，紧急修复 bug 的时候用

#### GitFlow 的优势

- 并行开发：每个新功能都会建立一个新的 feature 分支，从而和已经完成的功能隔离开来，而且只有在新功能完成开发的情况下，其对应的 feature 分支才会合并到主开发分支上（也就是我们经常说的 develop 分支）。另外，如果你正在开发某个功能，同时又有一个新的功能需要开发，你只需要提交当前 feature 的代码，然后创建另外一个 feature 分支并完成新功能开发。然后再切回之前的 feature 分支即可继续完成之前功能的开发。

- 协作开发：GitFlow 还支持多人协同开发，因为每个 feature 分支上改动的代码都只是为了让某个新的 feature 可以独立运行。同时我们也很容易知道每个人都在干啥。

- 支持紧急修复：GitFlow 还包含了 hotfix 分支。这种类型的分支是从某个已经发布的 tag 上创建出来并做一个紧急的修复，而且这个紧急修复只影响这个已经发布的 tag，而不会影响到你正在开发的新 feature。

#### GitFlow 流程图


![](https://user-gold-cdn.xitu.io/2017/10/22/a2616b167151327468d29f7853f3cfca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`feature` 分支都是从 `develop` 分支创建，完成后再合并到 `develop` 分支上，等待发布。


![](https://user-gold-cdn.xitu.io/2017/10/22/09158b9deb6e98109d34792d3efa6fc6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当需要发布时，我们从 `develop` 分支创建一个 `release` 分支


![](https://user-gold-cdn.xitu.io/2017/10/22/1153109c1d591d1015563bb286fcc34e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

然后这个 `release` 分支会发布到测试环境进行测试，**如果发现问题就在这个分支直接进行修复**。在所有问题修复之前，我们会不停的重复**发布->测试->修复->重新发布->重新测试**这个流程。


![](https://user-gold-cdn.xitu.io/2017/10/22/e5642b093c22bb22fbd230dd82f71e80?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

发布结束后，这个 release 分支会合并到 develop 和 master 分支，从而保证不会有代码丢失。

`master` 分支只跟踪已经发布的代码，合并到 `master` 上的 `commit` 只能来自 `release` 分支和 `hotfix` 分支。


![](https://user-gold-cdn.xitu.io/2017/10/22/8cdaeac5336b66d622551994ad805662?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`hotfix` 分支的作用是紧急修复一些 Bug。它们都是从 master 分支上的某个 tag 建立，修复结束后再合并到 develop 和 master 分支上。

> 注意：

1. 先有一个 master

2. 然后从 master 拉取一个 develop， 以后它负责维护最新的功能；

3. 从 develop 新建各种 feature 分支，负责各个功能开发

4. feature 通过 PR／MR 的方式合并回 develop 分支；

5. release 从 develop 拉取，进行测试环境测试

> 对于 release 流程，则是要注意几点（**重要**）：

- **如果 release 分支上有 bug 需要修复，直接在 release 分支上完成**；

- release 分支上的 bug 修复要持续通过 PR／MR 的方式**合并回 develop 分支**；

- 最后确认发版的时候才把 release 分支直接合并到 master 分支。

> 对于 hotfix 流程，则是要注意几点：

- 从 master 分支发起，修复完要同时合并到 develop 和 master。

#### PR 和 MR 的区别

PR 和 MR 的全称分别是 `pull request` 和 `merge request`。解释它们两者的区别之前，我们需要先了解一下 Code Review，因为 PR 和 MR 的引入正是为了进行 Code Review。

Code Review 是指在开发过程中，对代码的系统性检查。通常的目的是查找系统缺陷，保证代码质量和提高开发者自身水平。 Code Review 是轻量级代码评审，相对于正式代码评审，轻量级代码评审所需要的各种成本要明显低的多，如果流程正确，它可以起到更加积极的效果。

然后我们需要了解下 `fork` 和 `branch`，因为这是 `PR` 和 `MR` 各自所属的协作流程。

`fork` 是 `git` 上的一个协作流程。通俗来说就是**把别人的仓库备份到自己仓库，修修改改，然后再把修改的东西提交给对方审核，对方同意后，就可以实现帮别人改代码的小目标了**。

> fork 包含了两个流程：

- fork 并更新某个仓库(提交修改申请)

![](https://user-gold-cdn.xitu.io/2017/10/22/b0be58749566c533c105003aaabd5e03?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 同步 fork（获取远程最新）

![](https://user-gold-cdn.xitu.io/2017/10/22/625e29966b3287f81191a6331689f8f4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

和 fork 不同，branch 并不涉及其他的仓库，操作都在当前仓库完成。

![](https://user-gold-cdn.xitu.io/2017/10/22/32b40b18b29fb686b2ec80258cda6422?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

所以 PR 和 MR 的最大区别就在于此。

参考：

[git常见面试问题](https://juejin.im/post/59ecb3976fb9a0452724bde0#heading-4)

[git rebase 和 git merge](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)