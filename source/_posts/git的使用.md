---
title: git的使用
date: 2015-12-20 15:30:25
categories: [工具]
tags: [Git]
---

## 配置

- 配置可以通过gui工具进行。
- git config --list 查看所有配置
- git config user.name 查看用户名。

<!-- more -->

## 基础
- **git init** 在目录中初始化仓库，使用git管理。
 初始化仓库之后， 可以git remote add [shortname] [url]，添加到远程仓库中。
 > git remote add cpf https://github.com/cpf929/test.git
- **git clone** 克隆仓库的命令格式为 `git clone [url]`
- 文件跟踪和更新到仓库

工作目录下面的所有文件都不外乎这两种状态：已跟踪或未跟踪。已跟踪的文件是指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，它们的状态可能是未更新，已修改或者已放入暂存区。而所有其他文件都属于未跟踪文件。它们既没有上次更新时的快照，也不在当前的暂存区域。初次克隆某个仓库时，工作目录中的所有文件都属于已跟踪文件，且状态为未修改。

git文件状态变化周期.

![](http://7xsj41.com2.z0.glb.clouddn.com/git.png)


- **git status** 查看文件状态
- **git add** 跟踪文件(把文件放入暂存区，文件处于暂存状态)
根据目标文件的状态不同，此命令的效果也不同：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等
- **git diff** 
比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。
若要看已经暂存起来的文件和上次提交时的快照之间的差异，使用 `git diff --staged`。
- **git commit** 提交暂存区的文件。
git commit -m '注释'
git commit -a 把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤
- **git rm [filename]** 从暂存区和工作空间中移除文件。
git rm --cached readme.txt, 只从暂存区移除，而保留在工作空间。
- **重命名** `git mv from_name to_name`
相当于执行了三条命令。
```
mv from_name to_name
git rm from_name
git add to_name
```

- **git log** 显示日志
git log -p -2 , `-p` 选项展开显示每次提交的内容差异，用 `-2` 则仅显示最近的两次更新

### 撤销操作。
- 修改上次的提交.
有时候我们提交完了才发现漏掉了几个文件没有加，或者提交信息写错了。想要撤消刚才的提交操作，可以使用 `--amend` 选项重新提交.
```
git commit -m 'initial commit'
git add forgotten_file 
git commit --amend
```
上面的三条命令最终只是产生一个提交，第二个提交命令修正了第一个的提交内容。
- **git reset HEAD fileName** 取消暂存的文件。
```
git checkout --filename 撤销工作区文件的修改
```
### 远程仓库
- 查看远程仓库
git remote -v ，查看远程仓库和克隆的地址
- 添加远程仓库
git remote add [shortname] [url]
现在可以用[shortname]代替远程地址。如： git fetch [shortname]
- 从远程仓库抓取数据
```
git fetch [remote-name]
```
会到远程仓库中拉取所有你本地仓库中还没有的数据或更新，只是拉取数据，但是并不自动合并到当前工作分支，
- 推送数据到远程仓库
```
git push [remote-name] [branch-name]
```
克隆操作会自动使用默认的 master 和 origin 名字,所以可以直接git push

