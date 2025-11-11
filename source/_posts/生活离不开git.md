---
title: 生活离不开git
description: "虽然知道工作要用但是还是喜欢复制粘贴重命名"
abbrlink: 42679
date: 2024-03-14 21:02:05
tags:
  - git
categories:
  - 开发技能
cover: 'https://image.aruoshui.fun/i/2024/12/31/vsi0ft-0.webp'
---


# 参考文章
{% link git指令学习游戏, https://learngitbranching.js.org/, http://image.aruoshui.fun/i/2024/12/31/qkt009-0.webp %}
Learning Git Branching 可以说是目前为止最好的教程了，详细的就不多说了，这里只是记录一些关键点

# 注意点
## git2.23版本 git switch取代 git checkout
checkout 作为单个命令有点超载（它承载了很多独立的功能），可以详见这[一篇文章](https://git-scm.com/docs/git-switch/zh_HANS-CN)
例如，切换到 "HEAD~3" 并创建分支 "fixup"：
`git switch -c fixup HEAD~3`

## 分离头HEAD
HEAD 是一个对当前检出记录的符号引用 —— 也就是指向你正在其基础上进行工作的提交记录。
- HEAD 总是指向当前分支上最近一次提交记录。大多数修改提交树的 Git 命令都是从改变 HEAD 的指向开始的。
- HEAD 通常情况下是指向分支名的（如 bugFix）。在你提交时，改变了 bugFix 的状态，这一变化通过 HEAD 变得可见。
- 如果想看 HEAD 指向，可以通过 cat .git/HEAD 查看， 如果 HEAD 指向的是一个引用，还可以用` git symbolic-ref HEAD `查看它的指向。
这里经常使用`^`和`~`来相对引用，例如`git branch -f main HEAD~3` 将 main 分支强制指向 HEAD 的第 3 级父提交。

## Git Tags
Git 的 tag 可以（在某种程度上 —— 因为标签可以被删除后重新在另外一个位置创建同名的标签）永久地将某个特定的提交命名为里程碑，然后就可以像分支一样引用了。
它们并不会随着新的提交而移动。你也不能切换到某个标签上面进行修改提交，它就像是提交树上的一个锚点，标识了某个特定的位置。——在tag上进行新`git commit --amend`会创建新分支。
场景：当你用 git bisect（一个查找产生 Bug 的提交记录的指令）找到某个提交记录时，或者是当你坐在你那刚刚度假回来的同事的电脑前时， 可能会用到这个命令。

语法是：git describe <ref>

<ref> 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（HEAD）
输出结果为：<tag>_<numCommits>_g<hash>

tag 表示的是离 ref 最近的标签， numCommits 是表示这个 ref 与 tag 相差有多少个提交记录， hash 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。
当 ref 提交记录上有某个标签时，则只输出标签名称

## Git Fetch
运行git fetch，远程最新提交被下载到本地，同时远程分支 o/main 也被更新

从远程仓库下载本地仓库中缺失的提交记录
更新远程分支指针(如 o/main)
远程分支反映了远程仓库在你最后一次与它通信时的状态，git fetch 就是你与远程仓库通信的方式了

不能做的事：不会改变你本地仓库的状态。它不会更新你的 main 分支，也不会修改你磁盘上的文件。

## 偏离的提交历史
例子
假设你周一克隆了一个仓库，然后开始研发某个新功能。到周五时，你新功能开发测试完毕，可以发布了。但是 —— 天啊！你的同事这周写了一堆代码，还改了许多你的功能中使用的 API，这些变动会导致你新开发的功能变得不可用。但是他们已经将那些提交推送到远程仓库了，因此你的工作就变成了基于项目旧版的代码，与远程仓库最新的代码不匹配了。

此时是不允许push的
它会强制你先合并远程最新的代码，然后才能分享你的工作。
演示
git push 失效，因为远程仓库有未拉取到本地的提交
你需要做的就是使你的工作基于最新的远程分支——最直接的方法就是通过 rebase 调整你的工作
方法1：

`git fetch; git rebase o/main; git push `或` git pull --rebase; git push`
方法2：

`git fetch; git merge o/main; git push `或`git pull; git push`

## 锁定的main
大型项目中，main（master）分支被锁定保护，需要一些Pull Request流程来合并修改





