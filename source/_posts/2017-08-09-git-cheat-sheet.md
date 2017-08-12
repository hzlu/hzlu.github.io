---
title: Git 命令小结
tags:
  - git
categories:
  - 命令行工具
date: 2017-08-09 22:36:57
---


## 配置

```bash
git config --global user.name "yourname"
git config --global user.email "yourname@example.com"
git config --global alias.co checkout # alias 设置别名
git config --global alias.unstage "reset HEAD"
git config -l # 列出全部配置
```

## git init

## git status

## git add

## git commit

```bash
git init # 初始化
git clone <url> # 克隆
git status # 仓库状态
git add . # 全部修改暂存
git add <file> <file> # 部分文件修改暂存
git add -p file
git commit
git commit -a # 把修改都添加到暂存区并提交，不包括untracked的文件
git commit --amend # 修改上次提交来修复错误，如果代码已发布千万不能使用
```

## git checkout

```bash
git checkout <branch> # 切换分支
git checkout -b <new-branch> # 新建分支并切换
git checkout -b <new-branch> <exist-branch> # 基于 <exist-branch> 创建新分支
git checkout -b <新分支> <远程>/<分支> # 基于远程分支创建新的分支
```

### git checkout --track

```bash
git checkout --track <远程名>/<分支名>
git checkout --track <new-branch> <remote-branch>
```

## git branch

```bash
git branch # 显示本地仓库的分支
git branch -r # 查看远程分支
git branch -a # 显示本地和远程仓库的全部分支
git branch -v # 查看本地各分支最后提交信息
git branch --merged # 查看已被合并到当前分支的分支
git branch --no-merged # 未合并到当前分支的分支
git branch <new-branch> # 创建新分支，但不会切换
git branch -d <branch> # 删除分支
git branch -D <branch> # 强制删除
git branch --track <new-branch> <remote-branch>
```

### 将本地分支关联远程分支

```bash
git branch -u upstream/foo
git branch -u upstream/foo foo
git branch --set-upstream-to upstream/foo [foo]
git branch --set-upstream master origin/master
```

> `-u`与`--set-upstream-to`作用一致

## git stash

```bash
git stash
git stash list
git stash pop
git stash apply
git stash drop
```

## git tag

```bash
git tag # 查看所有标签，按字母顺序
git tag <tag-name> # 给当前提交打标签
git tag <tag-name> <commitID> # 给历史提交打标签
git show <tag-name> # 查看标签信息
git tag -a <tag-name> -m <description> <commitID>
git push --tags
```

> 使用 `-a` 或 `-s` 或 `-u` 选项都会创建标签对象，并且需要填写标签信息，通过 `-m` 指定。

> 标签引用指向标签对象，而不是指向提交。

## git remote

```bash
git remote -v # 显示当前项目下所有远程仓库的信息
git remote show origin # 显示远程仓库的详细信息
git remote show upstream # 显示远程仓库的详细信息
git remote add <remote-shortname> <url> # 添加远程仓库
git remote add origin <url>
git remote set-url origin <new-url> # 修改远程仓库地址
git remote rename <old-shortname> <new-shortname> # 重命名远程仓库
git remote rm <remote-shortname> # 删除远程仓库
```

### git remote prune

删除本地仓库中相对于远程仓库已经不存在的分支

```bash
git remote prune origin
git remote prune upstream
```

## git fetch & git pull

`fetch`从远程分支拉取版本，但不自动集成到当前分支中

```bash
git fetch <remote>
```

`pull`从远程分支拉取版本，并使用`merge`合并到当前分支，可以理解成`git fetch` + `git merge` 的快捷命令，使用`--rebase`就是执行`rebase`操作，而不是`merge`操作。

```bash
git pull # 抓取远程仓库所有分支更新并合并merge到本地
git pull --no-ff # 不要进行fast-forward merge
git pull <remote> <branch>
git pull --rebase <remote> <branch>
```

## git push

```bash
git push # push所有分支
git push <remote> <branch>
# 将本地主分支推到远程主分支，如果没有远程主分支，则差创建，用于初始化远程仓库
git push -u origin master
```

### git push --delete

删除远程仓库上的分支

```bash
git push <remote> --delete <branch>
git push <remote> :<branch>
```

### git push --tag

```bash
git push --tags
```

## git merge & git rebase

```bash
git merge <branch>
git merge --no-ff <branch>
git rebase <branch> # 把 <branch> 分支 rebase 到当前分支
git rebase <other-branch> <branch> # 把 <other-branch> 分支 rebase 到 <branch> 分支
git rebase --abort
git rebase --continue
git rebase --abort
```

merge的两种方式：

- `fast-farward-merge` 默认执行，merge不会生成新节点
- 添加 `--no-ff` 参数，生成merge提交节点，为保证版本演进清晰，希望采用的方式。

### git mergetool

```bash
git mergetool
```

## git reset

```bash
git reset --hard HEAD # 回退版本
git reset --hard <commit> # 回退到特定版本，丢弃所有修改
git reset <commit> # 回退到特定版本，保留修改
git reset --keep <commit> # 回退到特定版本，保留修改
git checkout HEAD <file> # 撤销暂存区内文件修改
git checkout -- <file> # 撤销工作区内文件修改
```

## git revert

```bash
git revert <commit> #创建新的提交来撤销前期提交
```

## git cherry-pick

把其他分支上的提交 `cherry-pick` 到当前分支，产生新的 `commitID` ，保留版本信息。

```bash
git cherry-pick <commit>
```

使用 `-n` 选项不产生`commit`，只是应用修改。

```bash
git cherry-pick <commit> -n
```

## git diff

```bash
git diff # 显示工作区的修改
git diff --cache # 显示暂存区的修改
git diff <branch>..<other-branch> #  比较分支间的差异
git diff <branch> # 查看当前工作区与另外一个分支的区别
git diff HEAD -- ./lib #加上路径限定符，比较特定目录或文件
```

## git log

```bash
git log # 显示当前分支所有版本记录
git log --pretty=oneline
git log --pretty=short
git log --stat # 显示修改的日志统计
git log --pretty=format:'%h : %s' --graph # 可视化提交图
git log --pretty=format:'%h : %s' --topo-order --graph # 拓扑顺序显示
git log --pretty=format:'%h : %s' --date-order --graph # 时间顺序
git log --reverse # 正序
git log -p <file> # 文件修改记录
git log -- <file> # 文件版本日志
```

## git blame

```bash
git blame <file>
```

## gitk 工具

```bash
gitk <file>
```
