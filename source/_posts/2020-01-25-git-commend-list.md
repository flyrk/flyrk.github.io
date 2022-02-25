---
title: git命令手册（持续更新）
date: 2020-01-25 16:47:41
toc: true
categories:
- 工具
tags:
- git
---

本篇不定时更新git实践技巧，方便查阅git命令。
<!--more-->

# 分支相关操作
## git checkout
切换分支，例如：`git checkout -b ${branch_name}`

## git show
显示各种类型的对象。可以是blobs，树，标签和提交。

## git remote
对本地远程增删改查

git remote -v 获取远程分支信息

## git fetch
从远端拉取代码，默认拉取`origin`。

## git merge
合并分支操作，例如：`git merge master`，把`master`分支的内容合并到当前分支。

## git pull
拉取代码，用得最多的命令。pull = fetch + merge

## git push
推送代码到远程分支。

# 修改相关操作
## git add
添加变更，最常用的命令：`git add .`，添加当前所有变更。

## git status
查看当前git状态。

## git diff
对比两个分支的差异。

## git stash
缓存当前变更，一般在要切换到别的分支，而当前分支的变更又不想提交时用。`git stash`，然后恢复的时候：`git stash apply`。

# 提交相关操作
## git commit
提交变更，最常用的命令：`git commit -m "commit message"`。

## git revert
撤销某次版本，`git revert -n 版本号`

## git log
查看git提交信息。

# 奇淫技巧
- git blame 查看文件每一行修改记录
- git reflog 查看操作记录
- git rebase -i HEAD-n 修改已提交的前n个commit操作
- 更新子模块url：`git submodule sync --recursive`