---
title: 通关 Githug (一)
date: 2017-12-25
categories:
  - 工具
tags:
  - Git
excerpt: "通过 Githug 游戏学习 Git 命令，共 55 关，涵盖 init、add、commit、branch、merge、rebase 等核心操作。"
aiSummary: "Githug 是一个通过游戏关卡学习 Git 的工具，本文记录了作者通关 55 关的完整过程。内容涵盖 Git 核心命令：仓库初始化（git init）、文件操作（git add/rm/mv）、提交（git commit）、分支管理（git branch/checkout/merge）、远程仓库（git remote/push/pull）、历史查看（git log/diff/blame）、储藏（git stash）、标签（git tag）、版本重写（git rebase/amend）、调试（git bisect）等。是入门 Git 的实战指南。"
---




## 一. 序言

> 第一眼看到 githug, 以为是把 GitHub 拼错.
>
> 原来… 是个游戏! 



关于 Git，大四整理学习过一篇: [git 小玩](/posts/git-getting-started).

虽然很多 IDE 都集成了 Git 插件, 在了解 Git 命令和使用场景后再去使用 IDE 才会更加明白

<!-- more -->

## 二. 介绍

当然, githug 不是什么 RPG, FPS 之类的游戏.

而是一个围绕 Git 知识的小游戏, 每一关都会帮你自动搭建好实验场景, 告诉你任务背景, 然后完成这些关卡.

其实这个跟 GitHub 提供的在线练习很像, 但是我觉得 githug 更加好玩!

- [git命令在线练习](https://try.github.io/levels/1/challenges/1)
- [git分支在线练习](https://learngitbranching.js.org/)

> GitHub 地址: <https://github.com/Gazler/githug>

花了一天时间去通关. 当中也有过不去的然后去翻译和查通关攻略了

## 三. 安装使用

### 1. 安装

Githug可以使用在Linux, Windows, OS X下.

我以 CentOS 7 为例:

在安装Githug之前首先确认有Ruby环境.

查看是否安装: `ruby --version`

这里我选择自动安装的方式: `sudo yum install ruby`

接着通过gem安装githug: `gem install githug`

安装完成.

### 2. 使用

Githug只有四个游戏命令, 很简单:

- `githug play`: 开始玩, 会验证是否完成当前关卡, 完成则进入下一关(可缩写成`githug`),

  没完成则会提示你并提示任务要求

- `githug hint`: 给你一点当前关卡的提示

- `githug reset`: 重置当前关卡

- `githug levels`: 列出所有关卡

在第一次输入`githug`开始游戏时, 会提示`No githug directory found, do you wish to create one?`

意思要初始化一个游戏目录. 输入`y`即可创建.

然后会出现一个git_hug目录, 以后所有关卡都会在该目录下生成文件和git对象. 每完成一关都会重新初始化游戏场景.

### 3. 关卡列表

为了方便查看, 所以列出所有关卡. 这里摘抄一个wiki.

| 关卡名称 | 学习内容 | Git 命令 |
| --- | --- | --- |
| [第1关 init](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-1.html) | 初始化仓库                              | git init          |
| [第2关 config](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-2.html) | 设置用户名和电子邮箱地址                | git config        |
| [第3关 add](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-3.html) | 把文件添加到暂存区                      | git add           |
| [第4关 commit](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-4.html) | 提交                                    | git commit        |
| [第5关 clone](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-5.html) | 克隆远程仓库                            | git clone         |
| [第6关 clone_to_folder](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-6.html) | 克隆远程仓库，并指定本地目录名          | git clone         |
| [第7关 ignore](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-7.html) | 配置不被 Git 管理的文件                 | vim .gitignore    |
| [第8关 include](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-8.html) | 配置不被 Git 管理的文件                 | vim .gitignore    |
| [第9关 status](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-9.html) | 查看仓库状态                            | git status        |
| [第10关 number_of_files_committed](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-10.html) | 查看仓库状态                            | git status        |
| [第11关 rm](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-11.html) | 删除文件                                | git rm            |
| [第12关 rm_cached](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-12.html) | 从暂存区中移除文件，系 git add 的逆操作 | git rm –cached    |
| [第13关 stash](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-13.html) | 保存而不提交                            | git stash         |
| [第14关 rename](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-14.html) | 文件改名                                | git mv            |
| [第15关 restructure](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-15.html) | 整理目录结构                            |                   |
| [第16关 log](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-16.html) | 查询日志                                | git log           |
| [第17关 tag](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-17.html) | 打标签                                  | git tag           |
| [第18关 push_tags](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-18.html) | 把标签推送到远程仓库                    | git push –tags    |
| [第19关 commit_amend](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-19.html) | 修改最后一次提交                        | git commit –amend |
| [第20关 commit_in_future](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-20.html) | 指定提交的日期                          | git commit –date  |
| [第21关 reset](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-21.html) | 从暂存区中移除文件，系 git add 的逆操作 | git reset         |
| [第22关 reset_soft](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-22.html) | 撤销提交，系 git commit 的逆操作        | git reset –soft   |
| [第23关 checkout_file](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-23.html) | 撤销对一个文件的修改                    | git checkout      |
| [第24关 remote](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-24.html) | 查询远程仓库                            | git remote        |
| [第25关 remote_url](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-25.html) | 查询远程仓库的 URL                      | git remote -v     |
| [第26关 pull](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-26.html) | 从远程仓库拉取更新                      | git pull          |
| [第27关 remote_add](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-27.html) | 添加远程仓库                            | git remote        |
| [第28关 push](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-28.html) | 把提交推送到远程仓库                    | git push          |
| [第29关 diff](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-29.html) | 查看文件被修改的细节                    | git diff          |
| [第30关 blame](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-30.html) | 查询每一行代码被谁编辑过                | git blame         |
| [第31关 branch](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-31.html) | 创建分支                                | git branch        |
| [第32关 checkout](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-32.html) | 切换分支                                | git checkout      |
| [第33关 checkout_tag](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-33.html) | 切换到标签                              | git checkout      |
| [第34关 checkout_tag_over_branch](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-34.html) | 切换到标签                              | git checkout      |
| [第35关 branch_at](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-35.html) | 在指定的提交处创建分支                  | git branch        |
| [第36关 delete_branch](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-36.html) | 删除分支                                | git branch -d     |
| [第37关 push_branch](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-37.html) | 推送分支到远程仓库                      | git push          |
| [第38关 merge](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-38.html) | 合并分支                                | git merge         |
| [第39关 fetch](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-39.html) | 从远程仓库抓取数据                      | git fetch         |
| [第40关 rebase](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-40.html) | 变基合并                                | git rebase        |
| [第41关 repack](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-41.html) | 重新打包                                | git repack        |
| [第42关 cherry-pick](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-42.html) | 合并分支上指定的提交                    | git cherry-pick   |
| [第43关 grep](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-43.html) | 搜索文本                                | git grep          |
| [第44关 rename_commit](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-44.html) | 修改历史提交的说明                      | git rebase -i     |
| [第45关 squash](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-45.html) | 把多次提交合并成一次提交                | git rebase -i     |
| [第46关 merge_squash](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-46.html) | 合并分支时把多次提交合并成一次提交      | git merge –squash |
| [第47关 reorder](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-47.html) | 调整提交顺序                            | git rebase -i     |
| [第48关 bisect](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-48.html) | 用二分法定位 bug                        | git bisect        |
| [第49关 stage_lines](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-49.html) | 添加文件的部分行到暂存区                | git add –edit     |
| [第50关 file_old_branch](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-50.html) | 查看 Git 上的操作历史                   | git reflog        |
| [第51关 revert](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-51.html) | 取消已推送到远程仓库的提交              | git revert        |
| [第52关 restore](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-52.html) | 恢复被删除的提交                        | git reset –hard   |
| [第53关 conflict](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-53.html) | 解决冲突                                |                   |
| [第54关 submodule](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-54.html) | 把第三方库当作子模块                    | git submodule     |
| [第55关 contribute](http://wiki.jikexueyuan.com/project/githug-walkthrough/level-55.html) | 捐献                                    |                   |

## 四. 开始游戏

### Level 1: init

> A new directory, `git_hug`, has been created; initialize an empty repository in it.
>
> 在`git_hug`目录下初始化一个仓库

```bash
[root@pipeline-cloud-test02 git_hug]# git init
Initialized empty Git repository in /home/githug/git_hug/.git/
[root@pipeline-cloud-test02 git_hug]# ls -a
.  ..  .git  .gitignore  .profile.yml
[root@pipeline-cloud-test02 git_hug]# githug
Congratulations, you have solved the level!
```

空目录和有文件的目录都可以初始化, 可以看到隐藏文件`.git`, 如同`.svn`一样. 该目录会被git所管理.

### Level 2: config

> Set up your git name and email, this is important so that your commits can be identified.
>
> 配置用户名和邮箱是为了提交代码时的团队标识

```bash
[root@pipeline-cloud-test02 git_hug]# git config --add user.name thank
[root@pipeline-cloud-test02 git_hug]# git config --add user.email coderthank@163.com
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
What is your name? thank
What is your email? coderthank@163.com
Your config has the following name: thank
Your config has the following email: coderthank@163.com
Congratulations, you have solved the level!
```

**补充**:

```bash
git config --add [--global/--local] user.name xxx # 增加name配置信息
git config --get [--global/--local] user.name xxx # 查看name配置信息
# --global/--local参数用来表示是全局配置还是本地配置
# email的话换成user.email

git config –list # 查看git全局配置
```

当然, 有配置就会有配置的记录文件. 你可以到`~/.gitconfig`去编辑配置信息

### Level 3: add

> There is a file in your folder called ‘README’, you should add it to your staging area.
>
> 添加README文件到暂存区

```bash
[root@pipeline-cloud-test02 git_hug]# git add README
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

**补充:**

首先补充一个图:

![man](http://img.blog.csdn.net/20160325192928814?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

关于这三个工作区域:

- History 也可以叫repository(git仓库): 最终确定的文件能保存到仓库, 成为一个新的版本, 并且对他人可见
- Staging area (暂存区,Cache, Index): 暂存已经修改的文件
- Working directory(工作区): 也就是你实际看见的工作目录

### Level 4: commit

> The ‘README’ file has been added to your staging area, now commit it.
>
> 把暂存区的README文件提交

```bash
[root@pipeline-cloud-test02 git_hug]# git commit -m "add README file"
[master (root-commit) a3317a2] add README file
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 5: clone

> Clone the repository at <https://github.com/Gazler/cloneme>.

```bash
[root@pipeline-cloud-test02 git_hug]# git clone https://github.com/Gazler/cloneme
Cloning into 'cloneme'...
remote: Counting objects: 7, done.
remote: Total 7 (delta 0), reused 0 (delta 0), pack-reused 7
Unpacking objects: 100% (7/7), done.
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

回一下第1关, 可以总结下创建git仓库大致有两种: `git init`初始化和`git clone`克隆远程git项目

### Level 6: clone to folder

> Clone the repository at <https://github.com/Gazler/cloneme> to ‘my_cloned_repo’.
>
> 还是克隆远程git项目, 只不过换个名字

```bash
[root@pipeline-cloud-test02 git_hug]# git clone  https://github.com/Gazler/cloneme my_cloned_repo
Cloning into 'my_cloned_repo'...
remote: Counting objects: 7, done.
remote: Total 7 (delta 0), reused 0 (delta 0), pack-reused 7
Unpacking objects: 100% (7/7), done.
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 7: ignore

> The text editor ‘vim’ creates files ending in ‘.swp’ (swap files) for all files that are currently open. We don’t want them creeping into the repository. Make this repository ignore ‘.swp’ files.
>
> 有些零食文件什么的不需要git来管理. 忽略他们

```bash
[root@pipeline-cloud-test02 git_hug]# ls -a
.  ..  .git  .gitignore  .profile.yml  README.swp
[root@pipeline-cloud-test02 git_hug]# vim .gitignore
[root@pipeline-cloud-test02 git_hug]# more .gitignore
.profile.yml
.gitignore
*.swp
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

`.gitignore` 在仓库的根目录下, 用于配置可忽略文件的规则

### Level 8: include

> Notice a few files with the ‘.a’ extension. We want git to ignore all but the ‘lib.a’ file.
>
> 在忽略的里面排除掉不忽略的

```bash
[root@pipeline-cloud-test02 git_hug]# ls -a
.  ..  first.a  .git  .gitignore  lib.a  .profile.yml  second.a
[root@pipeline-cloud-test02 git_hug]# vim .gitignore
[root@pipeline-cloud-test02 git_hug]# more .gitignore
.profile.yml
.gitignore
*.a
!lib.a  # 加"!"就是不忽略
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 9: status

> There are some files in this repository, one of the files is untracked, which file is it?
>
> 也就是去找没有登记的文件, Untracked File

```bash
[root@pipeline-cloud-test02 git_hug]# git status -s
A  Guardfile
A  README
A  config.rb
A  deploy.rb
A  setup.rb
?? database.yml
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
What is the full file name of the untracked file? database.yml
Congratulations, you have solved the level!
```

通过`git status`查看到的文件有这三种状态:

- untracked: 未被登记的
- modified: 修改过的
- staged: 暂存区的

### Level 10: number_of_files_committed

> There are some files in this repository, how many of the files will be committed?
>
> 找出有几个要被提交的文件, 什么状态的文件能被提交? 只有暂存区的了

```bash
[root@pipeline-cloud-test02 git_hug]# git status  -s
A  rubyfile1.rb
M  rubyfile4.rb
 M rubyfile5.rb
?? rubyfile6.rb
?? rubyfile7.rb
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
How many changes are going to be committed? 2
Congratulations, you have solved the level!
```

**补充:**

如果你用`git status`, 你会明显看到有两个文件处于Changes to be commited.

如果加`-s`查看简要列表, 你需要明白前面的标志位是什么意思.

首先第一个标志位: 是Staging area中的变化, 第二个标志位: 是Working directory中发生的更改.

标志位上的字母`A, M, D, ?`就不用说了

### Level 11: rm

> A file has been removed from the working tree, however the file was not removed from the repository. Find out what this file was and remove it.
>
> 有个文件只在工作空间被删了, 然而没有从仓库里删掉. 找到它并删掉rm_cached

```bash
[root@pipeline-cloud-test02 git_hug]# git status -s
 D deleteme.rb
[root@pipeline-cloud-test02 git_hug]# git rm deleteme.rb
rm 'deleteme.rb'
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

在删除git文件时, 不要用`rm`的系统级别删除, 这样git会失去对这个文件的追踪. 而应该用`git rm`删除.

### Level 12: rm_cached

> A file has accidentally been added to your staging area, find out which file and remove it from the staging area. *NOTE* Do not remove the file from the file system, only from git.
>
> 有个文件被错误的添加到暂存区, 找到他, 并从**暂存区**删除, 注意不是从文件系统删除.

```bash
[root@pipeline-cloud-test02 git_hug]# git status -s
A  deleteme.rb
[root@pipeline-cloud-test02 git_hug]# git rm --cached deleteme.rb
rm 'deleteme.rb'
[root@pipeline-cloud-test02 git_hug]# git status -s
?? deleteme.rb
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 13: stash

> You’ve made some changes and want to work on them later. You should save them, but don’t commit them.
>
> 你已经在工作空间做了一些改动, 可能出现别的任务要你改动, 你需要保存之前的改动稍后处理, 但并不是提交.

```bash
[root@pipeline-cloud-test02 git_hug]# git status -s
 M lyrics.txt
[root@pipeline-cloud-test02 git_hug]# git stash
Saved working directory and index state WIP on master: 0206059 Add some lyrics
HEAD is now at 0206059 Add some lyrics
[root@pipeline-cloud-test02 git_hug]# git status -s
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

**补充:**

储藏(Stashing): 经常有这样的事情发生，当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作，否则以后你无法回到这个工作点。解决这个问题的办法就是`git stash`命令。

“储藏”可以获取你工作目录的中间状态——也就是你修改过的被追踪的文件和暂存的变更——并将它保存到一个未完结变更的堆栈中，随时可以重新应用。

使用储藏命令后, 它会帮你把工作环境回到 最后一次提交的状态, 也就是一个完全干净的工作环境.

```bash
git stash list # 查看暂存工作区的列表
git stash pop # 弹出这个现场
```

### Level 14: rename

> We have a file called ‘oldfile.txt’. We want to rename it to ‘newfile.txt’ and stage this change.
>
> 将这个文件改名, 并希望在暂存区也生效.

```bash
[root@pipeline-cloud-test02 git_hug]# ls
oldfile.txt
[root@pipeline-cloud-test02 git_hug]# git mv oldfile.txt newfile.txt
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

如同`git rm`一样, `git mv`也会把改动的结果记录到暂存区.

### Level 15: restructure

> You added some files to your repository, but now realize that your project needs to be restructured. Make a new folder named ‘src’ and using Git move all of the .html files into this folder.
>
> 你需要重构下仓库里的文件了, 新建一个src目录, 并把html文件放进去(用Git能够追踪的方式)

```bash
[root@pipeline-cloud-test02 git_hug]# mkdir src
[root@pipeline-cloud-test02 git_hug]# git mv *.html /s
sbin/ srv/  sys/
[root@pipeline-cloud-test02 git_hug]# git mv *.html ./src/
[root@pipeline-cloud-test02 git_hug]# ls src/
about.html  contact.html  index.html
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 16: log

> You will be asked for the hash of most recent commit. You will need to investigate the logs of the repository for this.
>
> 查下最近一次commit的哈希值

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
7b8d2cd THIS IS THE COMMIT YOU ARE LOOKING FOR!
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
What is the hash of the most recent commit? 7b8d2cd
Congratulations, you have solved the level!
```

每个`git commit`都会留下一条日志.

### Level 17: tag

> We have a git repo and we want to tag the current commit with ‘new_tag’.
>
> 给当前提交打一个新标签

```bash
[root@pipeline-cloud-test02 git_hug]# git tag new_tag
[root@pipeline-cloud-test02 git_hug]# git tag
new_tag
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

```bash
git tag xxx # 给最近一次提交打标签
git tag xxx hashCodeXxx # 给某次提交打标签
git tag # 列出所有标签
git tag -d xxx # 删除某个标签
```

### Level 18: push tags

> There are tags in the repository that aren’t pushed into remote repository. Push them now.
>
> 推送本地仓库的标签到远程仓库

```bash
[root@pipeline-cloud-test02 git_hug]# git push --tags
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 19: commit_amend

> The ‘README’ file has been committed, but it looks like the file ‘forgotten_file.rb’ was missing from the commit. Add the file and amend your previous commit to include it.
>
> README文件已经被提交, 但是另外一个文件忘记了, 添加这个文件到上一次提交中

```bash
[root@pipeline-cloud-test02 git_hug]# git status -s
?? forgotten_file.rb
[root@pipeline-cloud-test02 git_hug]# git add forgotten_file.rb
[root@pipeline-cloud-test02 git_hug]# git commit --amend -C HEAD
[master c90accd] Initial commit
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README
 create mode 100644 forgotten_file.rb
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

一般是有文件漏提交了才会用`--amend`. 会把你漏的东西加到上一次提交里.

```bash
git commit --amend
git commit --amend -m "xxx"  # 并用新的注释覆盖上一次的
git commit --amend -C HEAD   # 还是用上一次的注释
```

### Level 20: commit_in_future

> Commit your changes with the future date (e.g. tomorrow).
>
> 提交时候改变下日期

```bash
[root@pipeline-cloud-test02 git_hug]# date
Fri Dec 22 14:07:17 CST 2017
[root@pipeline-cloud-test02 git_hug]# git commit --date="Fri Dec 22 14:07:17 CST 2018" -m "midify commit date"
[master (root-commit) 83bca42] midify commit date
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

没什么鸟用吧?

### Level 21: reset

> There are two files to be committed. The goal was to add each file as a separate commit, however both were added by accident. Unstage the file ‘to_commit_second.rb’ using the reset command (don’t commit anything).
>
> 有两个文件要被提交, 但是你想分别提交, 把to_commit_second.rb从暂存区中拿出来

```bash
[root@pipeline-cloud-test02 git_hug]# git reset to_commit_second.rb
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

这一关是让你从暂存区中取回文件到工作区, 目的是不想让这些文件被提交.

回忆第12关, 也是从暂存区中取. 所以你用`git rm --cached to_commit_second.rb`也能完成该任务.

区别在于, `git rm`适合于取回新增的文件, `git reset`适合取回修改的文件.

当然`git reset`还有更强的恢复功能, 后面会遇到.

### Level 22: reset_soft

> You committed too soon. Now you want to undo the last commit, while keeping the index.
>
> 取消最后一次提交, 并保持暂存区不变

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
02dd7f7 Premature commit
40820f0 Initial commit
[root@pipeline-cloud-test02 git_hug]# git reset --soft HEAD^
[root@pipeline-cloud-test02 git_hug]# git log --oneline
40820f0 Initial commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

在`git reset`命令中, `--soft HEAD^`就表示撤销的事最近一次提交. 并且暂存区不受影响.

回忆下19关, 是将某个遗漏的文件添加到上一次提交中去. 用的是`git commit --amend -C HEAD`

那它就等同于先执行`git reset --soft HEAD^`再执行`git commit`的合体.

### Level 23: checkout_file

> A file has been modified, but you don’t want to keep the modification. Checkout the ‘config.rb’ file from the last commit.
>
> 一个文件已经被修改了, 但你并不想保留这份修改. 把文件撤销到最后一次提交的状态.

```bash
[root@pipeline-cloud-test02 git_hug]# git status -s
 M config.rb
[root@pipeline-cloud-test02 git_hug]# git checkout config.rb
[root@pipeline-cloud-test02 git_hug]# git status -s
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 24: remote

> This project has a remote repository. Identify it.
>
> 找到这个项目的远程仓库名

```bash
[root@pipeline-cloud-test02 git_hug]# git remote
my_remote_repo
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
What is the name of the remote repository? my_remote_repo
Congratulations, you have solved the level!
```

### Level 25: remote_url

> The remote repositories have a url associated to them. Please enter the url of remote_location.
>
> 找到这个项目的远程仓库的URL

```bash
[root@pipeline-cloud-test02 git_hug]# git remote -v
my_remote_repo  https://github.com/Gazler/githug (fetch)
my_remote_repo  https://github.com/Gazler/githug (push)
remote_location https://github.com/githug/not_a_repo (fetch)
remote_location https://github.com/githug/not_a_repo (push)
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
What is the url of the remote repository? remote_location https://github.com/githug/not_a_repo
Congratulations, you have solved the level!
```

### Level 26: pull

> You need to pull changes from your origin repository.
>
> 从远程仓库origin拉取更新

```bash
[root@pipeline-cloud-test02 git_hug]# git pull origin master
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 3
Unpacking objects: 100% (3/3), done.
From https://github.com/pull-this/thing-to-pull
 * branch            master     -> FETCH_HEAD
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

### Level 27: remote_add

> Add a remote repository called ‘origin’ with the url <https://github.com/githug/githug>
>
> 添加一个远程仓库, 仓库名origin, 地址<https://github.com/githug/githug>

```bash
[root@pipeline-cloud-test02 git_hug]# git remote
[root@pipeline-cloud-test02 git_hug]# git remote -v
[root@pipeline-cloud-test02 git_hug]# git remote add origin https://github.com/githug/githug
[root@pipeline-cloud-test02 git_hug]# git remote -v
origin  https://github.com/githug/githug (fetch)
origin  https://github.com/githug/githug (push)
[root@pipeline-cloud-test02 git_hug]# git remote
origin
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

clone的项目git会自动保存地址, 以上这种方式适合于手工添加远程仓库

### Level 28: push

> Your local master branch has diverged from the remote origin/master branch. Rebase your commit onto origin/master and push it to remote.
>
> 你本地的master分支是从远程仓库(origin/master)上创建的, rebase你的提交到远程仓库(origin/master).

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
fd962e8 Third commit
98f106a Second commit
3186c7d First commit
[root@pipeline-cloud-test02 git_hug]# git log origin/master --oneline
6c41ca7 Fourth commit
[root@pipeline-cloud-test02 git_hug]# git rebase
First, rewinding head to replay your work on top of it...
Applying: First commit
Applying: Second commit
Applying: Third commit
[root@pipeline-cloud-test02 git_hug]# git push origin master
Counting objects: 7, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 591 bytes | 0 bytes/s, done.
Total 6 (delta 2), reused 0 (delta 0)
To /tmp/d20171222-60085-1tu401p/.git
   6c41ca7..1154eda  master -> master
[root@pipeline-cloud-test02 git_hug]# git log origin/master --oneline
1154eda Third commit
f2580d0 Second commit
944547b First commit
6c41ca7 Fourth commit
```

**补充:**

关于推送到远程:

```bash
# 推送到远程
git push remote-name branch-name
git push -u remote-name branch-name # 推送的同时记住仓库名和分支
git push # 记住后这样推送
```

在多人协作的项目中, 大家都需要推送自己的更改到远程仓库.

但是推送也是有时间顺序的. 如果在你推送之前有人已经推送了, 那么你会收到`"non-fast forward"`的提示.

所以需要先获取远程的最新代码到本地, 有如下两种方式:

1. `git pull`: 把远程仓库的最新代码合并到本地，然后再提交。这时本地的提交和远程的提交**按时间顺序混合排列**。

2. `git rebase`: 把本地仓库的更新排到远程仓库更新之后，那这时候本地仓库的所有提交都排在远程仓库的最后一次提交之后。rebase翻译过来叫变基, 后面还会有关于它的应用.

   

### Level 29: diff

> There have been modifications to the ‘app.rb’ file since your last commit. Find out which line has changed.
>
> 最后一次提交之后你又修改了app.rb文件, 找到哪行被修改了

```bash
[root@pipeline-cloud-test02 git_hug]# git diff app.rb
diff --git a/app.rb b/app.rb
index 4f703ca..3bfa839 100644
--- a/app.rb
+++ b/app.rb
@@ -23,7 +23,7 @@ get '/yet_another' do
   erb :success
 end
 get '/another_page' do
-  @message = get_response('data.json')
+  @message = get_response('server.json')
   erb :another
 end

[root@pipeline-cloud-test02 git_hug]# vim app.rb
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
What is the number of the line which has changed? 26
Congratulations, you have solved the level!
```

**补充:**

`git diff` 是查看第二个flag(也就是Staging area和Working directory) 具体变化信息的命令

被修改后的文件是modified状态:

```bash
git diff –staged # 可以查看第一个flag(也就是Staging arae和History 之间的变化), 产生相同的效果
git diff HEAD # 可以看History 和 Working 之间的变化
git diff –stat # 后面加stat可以简化变化信息
```

未完待续: [通关 Githug (二)](/posts/githug-walkthrough-2)
