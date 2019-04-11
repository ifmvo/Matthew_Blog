---
title: Git 命令大全
date: 2016-12-25
tags: Git
---



|    命令   |	描述		|
|:-----------------|:--------------------:|
| git init|初始化仓库|
|git status|查看状态|
|git add|修改添加到缓存|
|git log|查看日志|
|git brach|查看分支列表
|git branch a|从当前分支另外再新建一个分支
|git checkout a|切换到a分支
|git checkout -b a|新建并切换（git branch a   +   git checkout a）
|git branch -d| 删除分支
|git branch -D| 强制删除分支
|git tag v1.0| 将当前分支新建一个标签
|git tag -d <tagname>|删除tag
|git checkout v1.0|切换到v1.0标签
|git stash|将当前的修改暂存起来 （git add 过的也可以）
|git stash list |查看缓存列表
|git stash apply|恢复暂存
|git stash drop |删除暂存
|git stash pop |恢复并删除（git stash apply  +  git stash drop）
|git merge develop |将develop 分支合并到当前分支
|git push origin --delete <branchname> |删除远程分支
|git push origin --delete <tagname>|删除远程tag
|git remote show origin|查看远程和本地的详细状态

### GitFlow(Git工作流)
一般开发来说,大部分情况下都会拥有两个分支	master	和	develop,他们的职责分别是:

master:永远处在即将发布(production-ready)状态
develop:最新的开发状态

确切的说master、develop分支大部分情况下都会保持一致,只有在上线前的测试阶段develop比master的代码要多,一旦测试没问题,准备发布了,这时候会将develop合并到master上。

但是我们发布之后又会进行下一版本的功能开发,开发中间可能又会遇到需要紧急修复bug,一个功能开发完成之后突然需求变动了等情况,所以GitFlow除了以上master和develop两个主要分支以外,还提出了以下三个辅助分支:

feature:开发新功能的分支,基于develop,	完成后merge回develop
release:准备要发布版本的分支,用来修复bug,基于develop,完成后	merge回develop和master
hotfix:修复master上的问题,	等不及release版本就必须马上上线.基于master,完成后merge回master和develop

什么意思呢?

举个例子,假设我们已经有master和develop两个分支了,这个时候我们准备做一个功能A,第一步我们要做的,就是基于develop分支新建个分支:

git branch feature/A看到了吧,其实就是一个规范,规定了所有开发的功能分支都以feature为前缀。但是这个时候做着做着发现线上有一个紧急的bug需要修复,那赶紧停下手头的工作,立刻切换到master分支,然后再此基础上新建一个分支:

git	branch	hotfix/B代表新建了一个紧急修复分支,修复完成之后直接合并到develop和master,然后发布。然后再切回我们的feature/A分支继续着我们的开发,如果开发完了,那么合并回develop分支,然后在develop分支属于测试环境,跟后端对接并且测试的差不多了,感觉可以发布到正式环境了,这个时候再新建一个release分支:

git branch release/1.0这个时候所有的api、数据等都是正式环境,然后在这个分支上进行最后的测试,发现bug直接进行修改,直到测试ok达到了发布的标准,最后把该分支合并到develop和master然后进行发布。
