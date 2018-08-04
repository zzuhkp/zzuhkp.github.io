---
title: git学习笔记
date: 2018-07-28 20:52:49
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


- 删除本地远程库记录：git remote rm <name>

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


## 分支策略
- 合并分支时，使用--no-ff参数可以使用普通模式合并，合并后的历史有分支，能看出来曾经做过分支，而fast forward合并看不出来曾经做过分支。

```
[zzuhkp@dp learngit]$ git checkout -b dev
Switched to a new branch 'dev'
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git commit -m "add merge"
[dev a042964] add merge
 1 file changed, 1 insertion(+)
[zzuhkp@dp learngit]$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 4 commits.
  (use "git push" to publish your local commits)
[zzuhkp@dp learngit]$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
[zzuhkp@dp learngit]$ git log --graph --pretty=oneline --abbrev-commit
*   f6b9db0 merge with no-ff
|\  
| * a042964 add merge
|/  
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
```

## bug分支
- 修复bug时，我们会通过创建新的bug分支来进行修改，然后进行合并，最后删除
- 当手头工作没有完成时，可以通过git stash命令保存工作现场，然后修改bug，修改后，再使用git stash pop命令回到工作现场
- 保存工作现场：git stash
- 查看保存的工作现场列表：git stash list
- 恢复工作现场:
    - 方法1：使用git stash apply <stash@{0}>  
        使用该命令恢复后stash内容并不删除，需要用git stash drop命令删除
    - 方法2：使用git stash pop，恢复的时候同时把stash内容也删除了

```
[zzuhkp@dp learngit]$ git checkout -b dev
Switched to a new branch 'dev'
[zzuhkp@dp learngit]$ touch hello.py
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ git add hello.py
[zzuhkp@dp learngit]$ git status
# On branch dev
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   hello.py
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   readme.txt
#
[zzuhkp@dp learngit]$ git stash
Saved working directory and index state WIP on dev: f3bdbac merged bug fix 101
HEAD is now at f3bdbac merged bug fix 101
[zzuhkp@dp learngit]$ git status
# On branch dev
nothing to commit, working directory clean
[zzuhkp@dp learngit]$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 8 commits.
  (use "git push" to publish your local commits)
[zzuhkp@dp learngit]$ git checkout -b issue-101
Switched to a new branch 'issue-101'
[zzuhkp@dp learngit]$ vim readme.txt 
[zzuhkp@dp learngit]$ git add readme.txt 
[zzuhkp@dp learngit]$ git commit -m "fix bug 101"
[issue-101 bc66d34] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
[zzuhkp@dp learngit]$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 8 commits.
  (use "git push" to publish your local commits)
[zzuhkp@dp learngit]$ git merge --no-ff -m "merged bug fix 101" issue-101
Merge made by the 'recursive' strategy.
 readme.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
[zzuhkp@dp learngit]$ git checkout dev
Switched to branch 'dev'
[zzuhkp@dp learngit]$ git status
# On branch dev
nothing to commit, working directory clean
[zzuhkp@dp learngit]$ git stash list
stash@{0}: WIP on dev: f3bdbac merged bug fix 101
[zzuhkp@dp learngit]$ git stash pop
# On branch dev
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   hello.py
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   readme.txt
#
Dropped refs/stash@{0} (39b84099689796e851697a262d32191d587cbc75)
[zzuhkp@dp learngit]$ git stash list
[zzuhkp@dp learngit]$ 
```

## Feature分支
- 丢弃没有被合并过的分支：git branch -D <name>

```
[zzuhkp@dp learngit]$ git checkout -b feature-vulcan
Switched to a new branch 'feature-vulcan'
[zzuhkp@dp learngit]$ touch vulcan.py
[zzuhkp@dp learngit]$ git add vulcan.py 
[zzuhkp@dp learngit]$ git status
# On branch feature-vulcan
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	new file:   vulcan.py
#
[zzuhkp@dp learngit]$ git commit -m "add feature vulcan"
[feature-vulcan 0152991] add feature vulcan
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 vulcan.py
[zzuhkp@dp learngit]$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 10 commits.
  (use "git push" to publish your local commits)
[zzuhkp@dp learngit]$ git branch -d feature-vulcan
error: The branch 'feature-vulcan' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature-vulcan'.
[zzuhkp@dp learngit]$ git branch -D feature-vulcan
Deleted branch feature-vulcan (was 0152991).
```

## 多人协作
- 查看远程库信息:git remote [-v]  
    -v参数可以显示更详细的信息,如果没有推送信息则看不到push的地址

```
[zzuhkp@dp learngit]$ git remote
origin
[zzuhkp@dp learngit]$ git remote -v
origin	https://github.com/zzuhkp/learngit.git (fetch)
origin	https://github.com/zzuhkp/learngit.git (push)
```

- 推送分支：git push origin <branch_name>
    - master分支是主分支，需要时刻与远程同步
    - dev分支是开发分支，需要与远程同步
    - bug分支只用于在本地修复bug，不必推送到远程
    - feature分支是否推送取决于是否和其他人合作在上面开发

```
[zzuhkp@dp learngit]$ git push origin master
Username for 'https://github.com': zzuhkp
Password for 'https://zzuhkp@github.com': 
Counting objects: 26, done.
Compressing objects: 100% (24/24), done.
Writing objects: 100% (24/24), 2.05 KiB | 0 bytes/s, done.
Total 24 (delta 11), reused 0 (delta 0)
remote: Resolving deltas: 100% (11/11), completed with 1 local object.
To https://github.com/zzuhkp/learngit.git
   ecb5583..1f8f980  master -> master
[zzuhkp@dp learngit]$ git push origin dev
Username for 'https://github.com': zzuhkp
Password for 'https://zzuhkp@github.com': 
Counting objects: 4, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 294 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/zzuhkp/learngit.git
 * [new branch]      dev -> dev
```

- 抓取分支：git clone git@github.com:zzuhp/learngit.git
- 在本地创建和远程分支对应的分支：git checkout -b branch-name origin/branch-name
    默认情况抓取分支时只能看到本地的master分支
- 多人协作工作模式：
    - 1、首先，可以试图使用git push origin <branch-name>推送自己的修改
    - 2、如果推送失败，则因为远程分支比自己本地更新，需要先用git pull试图合并
    - 3.1、如果合并有冲突，则解决冲突，并在本地提交
    - 3.2没有冲突或者解决掉冲突后，再利用git push origin <branch-name>推送
    - tip：如果git pull 提示no tracking information,则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>


```
$ git clone git@github.com:zzuhp/learngit.git
Cloning into 'learngit'...
ERROR: Repository not found.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub
$ git clone git@github.com:zzuhkp/learngit.git
Cloning into 'learngit'...
remote: Counting objects: 46, done.
remote: Compressing objects: 100% (26/26), done.
remote: Total 46 (delta 15), reused 46 (delta 15), pack-reused 0
Receiving objects: 100% (46/46), done.
Resolving deltas: 100% (15/15), done.

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub
$ cd learngit/

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub/learngit (master)
$ git branch
* master

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub/learngit (master)
$ git checkout -b dev origin/dev
Switched to a new branch 'dev'
Branch 'dev' set up to track remote branch 'dev' from 'origin'.

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub/learngit (dev)
$ touch env.txt

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub/learngit (dev)
$ git add env.txt

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub/learngit (dev)
$ git commit -m "add env"
[dev 49b54ca] add env
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 env.txt

hkp20@LAPTOP-JQGTORVI MINGW64 /g/Project/GitHub/learngit (dev)
$ git push origin dev
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 219 bytes | 219.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:zzuhkp/learngit.git
   a6ae793..49b54ca  dev -> dev
```

```
[zzuhkp@dp learngit]$ vim env.txt
[zzuhkp@dp learngit]$ git add env.txt 
[zzuhkp@dp learngit]$ git commit -m "add new env"
[dev ad5dbb6] add new env
 1 file changed, 1 insertion(+)
 create mode 100644 env.txt
[zzuhkp@dp learngit]$ git push origin dev
Username for 'https://github.com': zzuhkp
Password for 'https://zzuhkp@github.com': 
To https://github.com/zzuhkp/learngit.git
 ! [rejected]        dev -> dev (fetch first)
error: failed to push some refs to 'https://github.com/zzuhkp/learngit.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first merge the remote changes (e.g.,
hint: 'git pull') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
[zzuhkp@dp learngit]$ git pull
remote: Counting objects: 5, done.
remote: Compressing objects: 100% (3/3), done.
Unpacking objects: 100% (5/5), done.
remote: Total 5 (delta 1), reused 5 (delta 1), pack-reused 0
From https://github.com/zzuhkp/learngit
   a6ae793..64fcabc  dev        -> origin/dev
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> dev

[zzuhkp@dp learngit]$  git branch --set-upstream-to=origin/dev dev
Branch dev set up to track remote branch dev from origin.
[zzuhkp@dp learngit]$ git pull
Auto-merging env.txt
CONFLICT (add/add): Merge conflict in env.txt
Automatic merge failed; fix conflicts and then commit the result.
[zzuhkp@dp learngit]$ vim env.txt
[zzuhkp@dp learngit]$ git add env.txt 
[zzuhkp@dp learngit]$ git commit -m "fix env conflict"
[dev 8da0df2] fix env conflict
[zzuhkp@dp learngit]$ git push origin dev
Username for 'https://github.com': zzuhkp
Password for 'https://zzuhkp@github.com': 
Counting objects: 7, done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 448 bytes | 0 bytes/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/zzuhkp/learngit.git
   64fcabc..8da0df2  dev -> dev
```

# 标签管理

- 新建标签：git tag <tagname> [commit_id],默认为HEAD,也可以指定一个commit id
- 新建标签并指定标签信息：git tag -a <tagname> -m "tag info"
- 查看所有标签：git tag
- 查看标签说明文字：git show <tagname>
- 推送本地标签：git push origin <tagname>
- 推送本地全部未推送标签：git push origin --tags
- 删除本地标签：git tag -d <tagname>
- 删除远程标签：git push origin :refs/tags/<tagname>

# 自定义Git

- 显示颜色：git config --global color.ui true
- 忽略某些文件：在工作区中创建文件.gitignore文件，在.gitignore文件中编写忽略的文件名称，每行写一个文件名称
- 强制提交忽略文件到暂存区：git add -f <file_name>
- 检查文件被忽略配置：git check-ignore -v <file_name>
- 配置别名：git config --global alias.<alias_name> origin_name  
    --global针对当前用户起作用，如果不加针对当前仓库起作用
- 每个仓库的配置放在.git/config文件中，当前用户的Git配置文件放在用户主目录下的.gitonfig中

```
[zzuhkp@dp ~]$ git config --global alias.st status
```

