---
title: git学习笔记
date: 2018-07-28 20:52:49
uddated: 2018-07-28 22:09:35
categories: 版本控制 
tags: [git,笔记] 
description: 这是我的第一篇笔记，方便以后查看
---

- CentOS安装：yum -install -y git
- 查看版本号：git --version
- 设置用户名：git config --global user.name "zzuhkp"
- 设置邮箱：git config --global user.email "zzuhkp@163.com"
- 创建版本库：
    - 创建目录：mkdir learngit && cd learngit
    - 把目录变成git可以管理的仓库：git init  

```
    [root@dp learngit]# git init  
    Initialized empty Git repository in /opt/java/practice/learngit/.git/
```
 
    注意：git使用.git目录跟踪管理版本库，一般情况不要手动修改.git目录的文件
- 添加文件到版本库：
    - 创建文件到版本库所在目录：touch readme.txt
    - 使用命令 git add把文件添加到版本库中的暂存区：git add readme.txt
    - 使用命令git commit把暂存区中的文件提交到仓库：git commit -m "wrote a readme file"  
    -m后面输入表示本次提交的说明  
   
```
    [root@dp learngit]# git commit -m "wrote a readme file"
    [master (root-commit) 992fc83] wrote a readme file
    1 file changed, 2 insertions(+)
    create mode 100644 readme.txt
```

- 查看仓库当前状态：git status  

```
    [root@dp learngit]# git status
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
    [root@dp learngit]# git diff readme.txt 
    diff --git a/readme.txt b/readme.txt
    index 46d49bf..9247db6 100644
    --- a/readme.txt
    +++ b/readme.txt
    @@ -1,2 +1,2 @@
    -Git is a version control system.
    +Git is a distributed version control system.
    Git is free software.
```

- 查看git提交日志：git log  --pretty=oneline  
    --pretty=oneline参数可以将每次提交的记录单行显示
    
```
    [root@dp learngit]# git log
    commit 72ef8d1f20a60dc335683e533d8ec213df0c9040
    Author: zzuhkp <zzuhkp@163.com>
    Date:   Sat Jul 28 10:09:55 2018 +0800

        append GPL

    commit 91e7cb59a9659e96462102bcc169571fc4a532a1
    Author: zzuhkp <zzuhkp@163.com>
    Date:   Sat Jul 28 10:07:18 2018 +0800

        add distributed

    commit 992fc83e7ab904fd594c3503f77cae20b0f3fdd3
    Author: zzuhkp <zzuhkp@163.com>
    Date:   Sat Jul 28 09:52:13 2018 +0800
    
        wrote a readme file
```

- 回退文件到某一个历史版本：git reset 版本号  
    - 在Git中，HEAD表示当前版本，也就是最新提交的版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，前100个版本就是HEAD~100
    - 回退上一个版本：

```
    [root@dp learngit]# git reset --hard HEAD^
    HEAD is now at 91e7cb5 add distributed
```

查看Git命令记录：git reflog

```
    [root@dp learngit]# git reflog
    72ef8d1 HEAD@{0}: reset: moving to 72ef8d
    91e7cb5 HEAD@{1}: reset: moving to HEAD^
    72ef8d1 HEAD@{2}: commit: append GPL
    91e7cb5 HEAD@{3}: commit: add distributed
    992fc83 HEAD@{4}: commit (initial): wrote a readme file
```




