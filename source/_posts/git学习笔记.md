---
title: git学习笔记
date: 2018-07-28 20:52:49
uddated: 2018-07-28 22:09:35
categories: 
- 版本控制 
tags: 
- git
- 笔记 
description: 这是我的第一篇笔记，作为记录方便以后查看
---

本笔记为学习 [廖雪峰 Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000) 记录
---

# Git简介
## 安装Git
- CentOS安装git：yum install -y git
- 查看版本号：git --version
- 设置用户名：git config --global user.name "zzuhkp"
- 设置邮箱：git config --global user.email "zzuhkp@163.com"
## 创建版本库
- 创建版本库：
    - 创建目录：mkdir learngit && cd learngit
    - 初始化git版本库：git init  
    - 注意：git使用.git目录跟踪管理版本库，一般情况不要手动修改.git目录的文件

```
[zzuhkp@dp ~]$ mkdir learngit && cd learngit
[zzuhkp@dp learngit]$ pwd
/home/zzuhkp/learngit
[zzuhkp@dp learngit]$ git init
Initialized empty Git repository in /home/zzuhkp/learngit/.git/
```
 
- 添加文件到版本库：
    - 创建文件到版本库所在目录：vim readme.txt
    - 使用命令 git add把文件添加到版本库中的暂存区：git add readme.txt
    - 使用命令git commit把暂存区中的文件提交到仓库：git commit -m "wrote a readme file"  
    -m后面输入表示本次提交的说明  
   
```
[zzuhkp@dp learngit]$ vim readme.txt
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a version control system.
Git is free software.
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git commit -m "wrote a readme file"
[master (root-commit) be9249a] wrote a readme file
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt
```

- 查看仓库当前状态：git status  

```
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software.
[zzuhkp@dp learngit]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
```

- 查看文件修改内容：git diff  
    
```
[zzuhkp@dp learngit]$ git diff readme.txt 
diff --git a/readme.txt b/readme.txt
index 46d49bf..9247db6 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.
```

- 查看git提交日志：git log  [--pretty=oneline]  
    --pretty=oneline参数可以将每次提交的记录单行显示
    

```
[zzuhkp@dp learngit]$ git log
commit b884506c4823ae849e7b9b4ecb8270327f7cf3e5
Author: zzuhkp <zzuhkp@163.com>
Date:   Sun Jul 29 14:33:36 2018 +0800

    append GPL

commit 938ec6951b35b51867464a7182b422c7d35e076f
Author: zzuhkp <zzuhkp@163.com>
Date:   Sun Jul 29 14:22:44 2018 +0800

    add distributed

commit be9249a26e60398a13242dc0990d123265e9ae56
Author: zzuhkp <zzuhkp@163.com>
Date:   Sun Jul 29 14:18:41 2018 +0800

    wrote a readme file
```

```
[zzuhkp@dp learngit]$ git log --pretty=oneline
b884506c4823ae849e7b9b4ecb8270327f7cf3e5 append GPL
938ec6951b35b51867464a7182b422c7d35e076f add distributed
be9249a26e60398a13242dc0990d123265e9ae56 wrote a readme file
```
# 时光机穿梭
## 版本回退
- 回退文件到某一个历史版本：git reset --hard commit_id  
    - 在Git中，HEAD表示当前版本，也就是最新提交的版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，前100个版本就是HEAD~100
    - 回退上一个版本：

```
[zzuhkp@dp learngit]$ git reset --hard HEAD^
HEAD is now at 938ec69 add distributed
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software.
```

- 查看Git命令记录：git reflog

```
[zzuhkp@dp learngit]$ git reflog
b884506 HEAD@{0}: reset: moving to b8845
938ec69 HEAD@{1}: reset: moving to HEAD^
b884506 HEAD@{2}: commit: append GPL
938ec69 HEAD@{3}: commit: add distributed
be9249a HEAD@{4}: commit (initial): wrote a readme file
```

## 工作区和暂存区
- 工作区是能看到的目录，如前面创建的learngit目录
- 版本库是工作区隐藏的.git目录，不算工作区
- 版本库里面有称为stage或index的暂存区，还有git自动创建的第一个分支master，以及指向master的一个指针叫HEAD。
- 添加文件到Git版本库的执行步骤：
    - 第一步：使用git add把文件添加到暂存区
    - 第二步：使用git commit提交更改，即把暂存区中的所有内容提交到当前分支
## 撤销修改
撤销修改(包括删除操作)：
- 仅修改工作区文件内容，撤销工作区的修改：git checkout -- file_name
-  修改工作区文件内容并且添加到暂存区，撤销修改：
    - 撤销暂存区修改：git reset HEAD <file>
    - 撤销工作区修改：git cheakout -- file_name  
        git checkout 其实是用版本库中的版本替换工作区中的版本 
- 撤销本地版本库修改：git reset --hard commit_id 
```
仅修改工作区文件内容撤销修改
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of file.
My stupid boss still prefers SVN.
[zzuhkp@dp learngit]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
[zzuhkp@dp learngit]$ git checkout -- readme.txt 
[zzuhkp@dp learngit]$ git status
# On branch master
nothing to commit, working directory clean
```
   
```
修改工作区文件内容并添加到暂存区撤销修改
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of file.
My stupid boss still prefers SVN.
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   readme.txt
#
[zzuhkp@dp learngit]$ git reset HEAD readme.txt 
Unstaged changes after reset:
M	readme.txt
[zzuhkp@dp learngit]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
[zzuhkp@dp learngit]$ git checkout -- readme.txt 
[zzuhkp@dp learngit]$ git status
# On branch master
nothing to commit, working directory clean
```

```
撤销本地版本库修改回退到某一历史版本
[zzuhkp@dp learngit]$ git reflog
77a773f HEAD@{0}: commit: commit changes
410066c HEAD@{1}: commit: git track changes
dc8922f HEAD@{2}: commit: understand how stage work
b884506 HEAD@{3}: reset: moving to b8845
938ec69 HEAD@{4}: reset: moving to HEAD^
b884506 HEAD@{5}: commit: append GPL
938ec69 HEAD@{6}: commit: add distributed
be9249a HEAD@{7}: commit (initial): wrote a readme file
[zzuhkp@dp learngit]$ git reset --hard 410066c
HEAD is now at 410066c git track changes
```
## 删除文件
- 从版本库中删除文件
    - 1、从本地删除 rm <file>
    - 2、提交删除到暂存区 git rm <file>
    - 3、提交暂存区文件到版本库 git commit -m "remove test.txt"

```
[zzuhkp@dp learngit]$ touch test.txt
[zzuhkp@dp learngit]$ git add test.txt
[zzuhkp@dp learngit]$ git commit -m "add test.txt"
[master 16b8ca3] add test.txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test.txt
[zzuhkp@dp learngit]$ rm test.txt 
[zzuhkp@dp learngit]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add/rm <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	deleted:    test.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
[zzuhkp@dp learngit]$ git rm test.txt
rm 'test.txt'
[zzuhkp@dp learngit]$ git commit -m "remove test.txt"
[master ecb5583] remove test.txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 test.txt
```
# 远程仓库
## 添加远程库
- 关联远程库：git remote add origin git@github.com:zzuhkp/learngit.git  
    远程库的名字为origin，可以改成其他名字 
- 推送master分支内容到远程库：git push -u origin master  
    -u参数将本地master分支和远程master分支关联，以后本地提交后可以通过命令git push origin master将本地master分支最新修改推送至远程库

```
[zzuhkp@dp learngit]$ git remote add origin https://github.com/zzuhkp/learngit.git
[zzuhkp@dp learngit]$ git push -u origin master
Username for 'https://github.com': zzuhkp
Password for 'https://zzuhkp@github.com': 
Counting objects: 20, done.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (20/20), 1.63 KiB | 0 bytes/s, done.
Total 20 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4), done.
To https://github.com/zzuhkp/learngit.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

## 从远程库克隆
- 克隆远程仓库：git clone 仓库地址
```
[zzuhkp@dp ~]$ git clone git@github.com:zzuhkp/gitskills.git
Cloning into 'gitskills'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
[zzuhkp@dp ~]$ cd gitskills/
[zzuhkp@dp gitskills]$ ll
total 4
-rw-rw-r-- 1 zzuhkp zzuhkp 11 Jul 29 16:54 README.md
```
# 分支管理
## 创建与合并分支

- 创建+分支分支：git checkout -b <name>  
    相当于以下两个命令的结合
    - 创建分支：git branch <name>
    - 切换分支：git checkout <name>

```
[zzuhkp@dp learngit]$ git checkout -b dev
Switched to a new branch 'dev'
```

- 查看分支：git branch  
    *号表示当前分支

```
[zzuhkp@dp learngit]$ git branch
* dev
  master
```
- 切换分支：git checkout <name>

```
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes.
Creating a new branch is quick.
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git commit -m "branch test"
[dev 52957a1] branch test
 1 file changed, 1 insertion(+)
[zzuhkp@dp learngit]$ git checkout master
Switched to branch 'master'
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes.
```

- 合并某分支到当前分支：git merge <name>

```
[zzuhkp@dp learngit]$ git merge dev
Updating ecb5583..52957a1
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes.
Creating a new branch is quick.
```

- 删除分支：git branch -d <name>

```
[zzuhkp@dp learngit]$ git branch -d dev
Deleted branch dev (was 52957a1).
[zzuhkp@dp learngit]$ git branch
* master
```

- 解决冲突
    - Git用<<<<<<<，=======，>>>>>>>标记不同分支的内容
    - git log --graph命令可以看到分支合并图
```
[zzuhkp@dp learngit]$ git checkout -b feature1
Switched to a new branch 'feature1'
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git commit -m "AND simple"
[feature1 7504349] AND simple
 1 file changed, 1 insertion(+), 1 deletion(-)
[zzuhkp@dp learngit]$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git commit -m "& simpe"
[master a1d8d59] & simpe
 1 file changed, 1 insertion(+), 1 deletion(-)
[zzuhkp@dp learngit]$ git merge feature1
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
[zzuhkp@dp learngit]$ git status
# On branch master
# Your branch is ahead of 'origin/master' by 2 commits.
#   (use "git push" to publish your local commits)
#
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#
#	both modified:      readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes.
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ cat readme.txt 
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes.
Creating a new branch is quick and simple.
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git commit -m "conflict fixed"
[master e30137d] conflict fixed
[zzuhkp@dp learngit]$ git log --graph --pretty=oneline --avvrev-commit
fatal: unrecognized argument: --avvrev-commit
[zzuhkp@dp learngit]$ git log --graph --pretty=oneline --abbrev-commit
*   e30137d conflict fixed
|\  
| * 7504349 AND simple
* | a1d8d59 & simpe
|/  
* 52957a1 branch test
* ecb5583 remove test.txt
* 16b8ca3 add test.txt
* 410066c git track changes
* dc8922f understand how stage work
* b884506 append GPL
* 938ec69 add distributed
* be9249a wrote a readme file
[zzuhkp@dp learngit]$ git branch -d feature1
Deleted branch feature1 (was 7504349).
```



