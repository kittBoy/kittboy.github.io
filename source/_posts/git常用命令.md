---
"title": "git常用命令"
"date": "2015/5/12 20:46:25"
categories:
- git

tags:
- git
---


### 版本库
- 创建版本库 git init
- 添加文件到git版本库 git add
- 提交暂存区修改到仓库  git commit
- 查看当前仓库状态 git status
- 查看修改内容  git diff
- 回退到历史版本  git reset --hard commit_id
- 查看提交历史   git log
- 查看命令历史  git reflog
-  丢弃工作区修改  git checkout -- file
-  回退暂存区修改  git reset HEAD file
- 删除文件 git rm

### 远程仓库

- 添加远程仓库 git remote add origin git@github.com:michaelliao/learngit.git
- 推送到远程仓库 git push -u origin master
- 从远程仓库克隆  git clone

### 分支管理
- 查看分支  git branch
- 创建分支  git branch name
- 切换分支  git checkout name
- 创建并切换分支  git checkout -b name
- 合并分支到当前分支 git merge name
- 删除分支  git branch -d name
- 查看分支合并图   git log --graph
- 普通模式合并分支  git merge --no-ff
- 隐藏当前工作区  git stash
- 创建标签  git tag