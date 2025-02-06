---
title: "Git入门"
date: 2016-07-18 22:08:00
draft: false
categories: "Git"
---


#### 安装
* 下载git OSX版
* 下载git Windows版
* 下载git Linux版

##### 创建新仓库
创建新文件夹，打开，然后执行
``` bash
git init
```
以创建新的git仓库


##### 克隆仓库

创建一个本地仓库的克隆版本：
``` bash
git clone /path/to/repository
```

如果要克隆服务器上的仓库，你的命令会是这个样子：
``` bash
git clone username@host:/path/to/repository
```

##### 添加与提交
``` bash
git add "filename"git add *
git add .
```

这是git基本工作流程的第一步;使用如下命令以实际提交改动：
``` bash
git commit -m '代码提交信息'
```

##### 推送仓库
改动提交到远端仓库：
``` bash
git push origin master
```

可以把master 换成你想要推送的任何分支。

如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：
``` bash
git remote add origin <server>
```

##### 更新与合并
要更新你的本地仓库至最新改动，执行：
``` bash
git pull
```

以在你的工作目录中 获取（fetch） 并 合并（merge） 远端的改动。

要合并其他分支到你的当前分支（例如 master），执行：
``` bash
git merge <branch>
```

两种情况下，git 都会尝试去自动合并改动。不幸的是，自动合并并非次次都能成功，并可能导致 冲突（conflicts）。 这时候就需要你修改这些文件来合并这些冲突（conflicts）了。改完之后，你需要执行如下命令以将它们标记为合并成功：
``` bash
git add <filename>
```

在合并改动之前，也可以使用如下命令查看：
``` bash
git diff <source_branch> <target_branch>
```

##### 替换本地改动
假如你做错事（自然，这是不可能的），你可以使用如下命令替换掉本地改动：
``` bash
git checkout -- <filename>
```

此命令会使用 HEAD 中的最新内容替换掉你的工作目录中的文件。已添加到缓存区的改动，以及新文件，都不受影响。 假如你想要丢弃你所有的本地改动与提交，可以到服务器上获取最新的版本并将你本地主分支指向到它：
``` bash
git fetch origin
git reset --hard origin/master
```

##### Git整个流程如下
``` bash
1. git init
2. git add .
3. git commit -m "first commit"4. git remote add origin 远程仓库地址
5. git push -u origin master
```

可以通过如下命令进行代码合并【注：pull=fetch+merge]
``` bash
git pull --rebase origin master
```