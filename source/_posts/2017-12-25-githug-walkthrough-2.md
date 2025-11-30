---
title: 通关 Githug (二)
date: 2017-12-25 00:00:00
categories:
  - 工具
tags:
  - Git
excerpt: "Githug 游戏通关续篇，记录第 30-55 关攻略，涵盖 blame、branch、checkout、merge、rebase、stash、submodule 等进阶操作。"
aiSummary: "本文是 Githug 游戏通关教程的续篇，记录了第 30-55 关的通关攻略。内容涵盖进阶 Git 命令：git blame 查询代码历史、git branch 创建与管理分支、git checkout 切换分支与标签、git merge 合并分支、git rebase 变基操作、git stash 储藏工作现场、git bisect 二分法定位 bug、git cherry-pick 精选提交、git submodule 子模块管理等。是进一步掌握 Git 的实战指南。"
---

继续上一篇 [通关 Githug (一)](/posts/githug-walkthrough-1)





## 关卡列表

再附上 30-55 关目录：

<!-- more -->

| 关卡名称 | 学习内容 | Git 命令 |
| --- | --- | --- |
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



## Level 30: blame

> Someone has put a password inside the file ‘config.rb’ find out who it was.
>
> 有人在config.rb文件里放了个password, 找出来谁搞的?

```bash
[root@pipeline-cloud-test02 git_hug]# git blame config.rb
^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000  1) class Config
70d00535 (Bruce Banner      2012-03-08 23:07:41 +0000  2)   attr_accessor :name, :password
97bdd0cc (Spider Man        2012-03-08 23:08:15 +0000  3)   def initialize(name, password = nil, options = {})
^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000  4)     @name = name
97bdd0cc (Spider Man        2012-03-08 23:08:15 +0000  5)     @password = password || "i<3evil"
00000000 (Not Committed Yet 2017-12-22 16:23:59 +0800  6)
09409480 (Spider Man        2012-03-08 23:06:18 +0000  7)     if options[:downcase]
09409480 (Spider Man        2012-03-08 23:06:18 +0000  8)       @name.downcase!
09409480 (Spider Man        2012-03-08 23:06:18 +0000  9)     end
70d00535 (Bruce Banner      2012-03-08 23:07:41 +0000 10)
ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 11)     if options[:upcase]
ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 12)       @name.upcase!
ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 13)     end
ffd39c2d (Gary Rennie       2012-03-08 23:08:58 +0000 14)
^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000 15)   end
^5e8863d (Gary Rennie       2012-03-08 23:05:24 +0000 16) end
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Who made the commit with the password? Spider Man
Congratulations, you have solved the level!
```

这个命令会详细列出该文件每行代码提交的哈希值, 提交的人和日期

## Level 31: branch

> You want to work on a piece of code that has the potential to break things, create the branch test_code.
>
> 你想实验一些代码, 但害怕有问题. 创建一个新分支test_code

```bash
[root@pipeline-cloud-test02 git_hug]# git branch test_code
[root@pipeline-cloud-test02 git_hug]# git branch
* master
  test_code
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

```bash
git branch xxx # 创建一个分支
git branch # 列出所有分支
```

## Level 32: checkout

> Create and switch to a new branch called my_branch. You will need to create a branch like you did in the previous level.
>
> 创建一个新分支my_branch并切换到这个新分支

```bash
[root@pipeline-cloud-test02 git_hug]# git branch my_branch
[root@pipeline-cloud-test02 git_hug]# git checkout my_branch
Switched to branch 'my_branch'
[root@pipeline-cloud-test02 git_hug]# git branch
  master
* my_branch
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

**补充:**

```bash
git checkout xxxBranch  # 检出一个分支
git checkout xxxFile  # 之前讲过如何撤销进入暂存区的修改, 这个也可以
git checkout -b xxx  # == git branch xxx + git checkout xxx
git checkout - # 用于在最近的两个分支之间切换
```

## Level 33: checkout_tag

> You need to fix a bug in the version 1.2 of your app. Checkout the tag `v1.2`.
>
> 你需要切换到`tag v1.2`上去修复一些Bug

```bash
[root@pipeline-cloud-test02 git_hug]# git branch
* master
[root@pipeline-cloud-test02 git_hug]# git tag
v1.0
v1.2
v1.5
[root@pipeline-cloud-test02 git_hug]# git checkout v1.2
Note: checking out 'v1.2'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 060320d... Some more changes
[root@pipeline-cloud-test02 git_hug]# git branch
* (detached from v1.2)
  master
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

**注意:** 从前面已经看到`git checkout`后面可以跟分支, 标签, 文件. 但是作用不同.

## Level 34: checkout_tag_over_branch

> You need to fix a bug in the version 1.2 of your app. Checkout the tag `v1.2` (Note: There is also a branch named `v1.2`).
>
> 切换到`tag v1.2`上去修复一个bug(注意有一个分支也叫v1.2)

```bash
[root@pipeline-cloud-test02 git_hug]# git checkout tags/v1.2
Note: checking out 'tags/v1.2'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at fc5edda... Some more changes
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

这样区分就好了.

## Level 35: branch_at

> You forgot to branch at the previous commit and made a commit on top of it. Create branch test_branch at the commit before the last.
>
> 你忘记在上次提交之前先创建一个分支, 那么创建一个test_branch在上次提交之前

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
03a7f2c Updating file1 again
3bcda78 Updating file1
3873d24 Adding file1
[root@pipeline-cloud-test02 git_hug]# git branch test_branch 3bcda78
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 36: delete_branch

> You have created too many branches for your project. There is an old branch in your repo called ‘delete_me’, you should delete it.
>
> 删掉`delete_me`分支

```bash
[root@pipeline-cloud-test02 git_hug]# git branch
  delete_me
* master
[root@pipeline-cloud-test02 git_hug]# git branch -d delete_me
Deleted branch delete_me (was b60afe2).
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 37: push_branch

> You’ve made some changes to a local branch and want to share it, but aren’t yet ready to merge it with the ‘master’ branch. Push only ‘test_branch’ to the remote repository.
>
> 你本地分支做了些修改想分享它, 但是不准备合并到master上. 所以把他推送到test_branch分支远程仓库上.

```bash
[root@pipeline-cloud-test02 git_hug]# git remote
origin
[root@pipeline-cloud-test02 git_hug]# git branch
* master
  other_branch
  test_branch
[root@pipeline-cloud-test02 git_hug]# git push origin test_branch
Counting objects: 7, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 572 bytes | 0 bytes/s, done.
Total 6 (delta 3), reused 0 (delta 0)
To /tmp/d20171222-302-aokpd0/.git
 * [new branch]      test_branch -> test_branch
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 38: merge

> We have a file in the branch ‘feature’; Let’s merge it to the master branch.
>
> feature分支上有个文件, 把它合并到master上

```bash
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
e12277f added file1
[root@pipeline-cloud-test02 git_hug]# git log feature --oneline
cc8ea5a added file2
e12277f added file1
[root@pipeline-cloud-test02 git_hug]# git branch
  feature
* master
[root@pipeline-cloud-test02 git_hug]# git merge feature
Updating e12277f..cc8ea5a
Fast-forward
 file2 | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file2
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
cc8ea5a added file2
e12277f added file1
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

分支测试完成后就可以合并到主干, 主干上也会记录对应的日志.

## Level 39: fetch

> Looks like a new branch was pushed into our remote repository. Get the changes without merging them with the local repository
>
> 有一个新的分支推送到远程仓库了, 得到它但是不要合并到你的本地仓库

```bash
[root@pipeline-cloud-test02 git_hug]# git branch
* master
[root@pipeline-cloud-test02 git_hug]# git branch -r
  origin/master
[root@pipeline-cloud-test02 git_hug]# git fetch
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 2 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (2/2), done.
From /tmp/d20171222-797-1165a5c/
 * [new branch]      new_branch -> origin/new_branch
[root@pipeline-cloud-test02 git_hug]# git branch
* master
[root@pipeline-cloud-test02 git_hug]# git branch -r
  origin/master
  origin/new_branch
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

为什么不用`git pull`呢, 因为它不仅会拉取远程代码, 还会合并到本地代码, 相当于`git fetch`+`git merge`

在 `git fetch` 之后用 `git branch -r` 查看时会发现新分支的名称

## Level 40: rebase

> We are using a git rebase workflow and the feature branch is ready to go into master. Let’s rebase the feature branch onto our master branch.
>
> 我们使用了git rebase工作流, 并准备把feature分支合并到master. 现在rebase feature分支到master分支之上吧.

```bash
[root@pipeline-cloud-test02 git_hug]# git branch
  feature
* master
[root@pipeline-cloud-test02 git_hug]# git checkout feature
Switched to branch 'feature'
[root@pipeline-cloud-test02 git_hug]# git log --oneline
ed0fdcf add feature
a78bcab init commit
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
98205e9 add content
a78bcab init commit
[root@pipeline-cloud-test02 git_hug]# git rebase master
First, rewinding head to replay your work on top of it...
Applying: add feature
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
98205e9 add content
a78bcab init commit
[root@pipeline-cloud-test02 git_hug]# git log --oneline
967920c add feature
98205e9 add content
a78bcab init commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

参考: [Git Rebase原理以及黄金准则详解](https://segmentfault.com/a/1190000005937408)

## Level 41: rebase_onto

> You have created your branch from `wrong_branch` and already made some commits, and you realise that you needed to create your branch from `master`. Rebase your commits onto `master` branch so that you don’t have `wrong_branch` commits.
>
> 你错误的从`wrong_branch`上创建了一个分支, 并且有了一些提交, 但是你意识到你应该是从`master`上创建分支, 现在rebase你的提交到`master`分支上去, 并且不保留`wrong_branch`的提交.

```bash
[root@pipeline-cloud-test02 git_hug]# git branch
  master
* readme-update
  wrong_branch
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
3d8ad8d Create authors file
[root@pipeline-cloud-test02 git_hug]# git log wrong_branch --oneline
a2c5c42 Wrong changes
3d8ad8d Create authors file
[root@pipeline-cloud-test02 git_hug]# git log --oneline
9c959f1 Add `Install` header in readme
082624b Add `About` header in readme
73dbf72 Add app name in readme
a2c5c42 Wrong changes
3d8ad8d Create authors file
[root@pipeline-cloud-test02 git_hug]# git rebase --onto master wrong_branch readme-update
First, rewinding head to replay your work on top of it...
Applying: Add app name in readme
Applying: Add `About` header in readme
Applying: Add `Install` header in readme
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
3d8ad8d Create authors file
[root@pipeline-cloud-test02 git_hug]# git log wrong_branch --oneline
a2c5c42 Wrong changes
3d8ad8d Create authors file
[root@pipeline-cloud-test02 git_hug]# git log --oneline
b113760 Add `Install` header in readme
809fc39 Add `About` header in readme
7adf9cd Add app name in readme
3d8ad8d Create authors file
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

`wrong_branch`和`master`分支的日志都没有变, 注意看`readme-update`的日志变化.

**补充:**

摘一段官网的内容:

> First let’s assume your topic is based on branch next. For example, a feature developed in topic depends on some functionality which is found in next.

```bash
o---o---o---o---o  master
     \
      o---o---o---o---o  next
                       \
                        o---o---o  topic
```

> We want to make topic forked from branch master; for example, because the functionality on which topic depends was merged into the more stable master branch. We want our tree to look like this:

```bash
o---o---o---o---o  master
    |            \
    |             o'--o'--o'  topic
     \
      o---o---o---o---o  next
```

> We can get this using the following command:`git rebase –onto master next topic`

## Level 42: repack

> Optimise how your repository is packaged ensuring that redundant packs are removed.
>
> 优化仓库的打包, 确保删除掉重复多余的包

```bash
[root@pipeline-cloud-test02 git_hug]# ls .git/objects/
4d  80  e6  info  pack
[root@pipeline-cloud-test02 git_hug]# git repack -d
Counting objects: 3, done.
Writing objects: 100% (3/3), done.
Total 3 (delta 0), reused 0 (delta 0)
[root@pipeline-cloud-test02 git_hug]# ls .git/objects/
info  pack
[root@pipeline-cloud-test02 git_hug]# ls .git/objects/pack/
pack-830ef7487354a6468143f53dd19ee16c25fc2837.idx  pack-830ef7487354a6468143f53dd19ee16c25fc2837.pack
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 43: cherry-pick

> Your new feature isn’t worth the time and you’re going to delete it. But it has one commit that fills in `README`file, and you want this commit to be on the master as well.
>
> 你的新功能废了你要删除它! 但是其中有一次README的提交还是有用的, 你把这次提交合并到主干上.

```bash
[root@pipeline-cloud-test02 git_hug]# git branch
* master
  new-feature
[root@pipeline-cloud-test02 git_hug]# git log new-feature --oneline
ea2a47c some small fixes
4a1961b Fixed feature
ca32a6d Filled in README.md with proper input
58a8c8e Added a stub for the feature
ea3dbcc Initial commit
[root@pipeline-cloud-test02 git_hug]# git log --oneline
6edea63 Added fancy branded output
232d266 Renamed project.js -> herdcore-math.js
b30c6a9 Added a hardcore math module
ea3dbcc Initial commit
[root@pipeline-cloud-test02 git_hug]# git cherry
cherry        cherry-pick
[root@pipeline-cloud-test02 git_hug]# git cherry
cherry        cherry-pick
[root@pipeline-cloud-test02 git_hug]# git cherry-pick ca32a6d
[master 25034ea] Filled in README.md with proper input
 Author: Andrey <aslushnikov@gmail.com>
 1 file changed, 1 insertion(+), 2 deletions(-)
[root@pipeline-cloud-test02 git_hug]# git log --oneline
25034ea Filled in README.md with proper input
6edea63 Added fancy branded output
232d266 Renamed project.js -> herdcore-math.js
b30c6a9 Added a hardcore math module
ea3dbcc Initial commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

`git cherry-pick hash-code`: 摘樱桃. 可以选取一个分支上某次提交的哈希值来合并到主线上.

## Level 44: grep

> Your project’s deadline approaches, you should evaluate how many TODOs are left in your code.
>
> 找下项目里还有多少待办事项(TODOs)

```bash
[root@pipeline-cloud-test02 git_hug]# git grep TODO
app.rb:# TODO Make site url variable.
app.rb:# TODO Make API version variable.
app.rb:# TODO Redirecting queries could be useful.
config.rb:    # TODO Move password to a configuration file.
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
How many items are there in your todolist? 4
Congratulations, you have solved the level!
```

```bash
git grep xxx # 全项目查找
git grep xxx file-name # 指定文件查找
```

## Level 45: rename_commit

> Correct the typo in the message of your first (non-root) commit.
>
> 修正第一次提交的拼写错误

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
5da15a1 Second commit
d1c06bd First commit
2b08caa Initial commit
[root@pipeline-cloud-test02 git_hug]# git rebase -i 2b08caa
[detached HEAD ac4092f] First commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file1
Successfully rebased and updated refs/heads/master.
[root@pipeline-cloud-test02 git_hug]# git log --oneline
208de39 Second commit
ac4092f First commit
2b08caa Initial commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

**补充:**

`git rebase -i hash-code`:

后面加了`-i`参数后就不再是分支合并的意思了. `-i`表示交互, 后面跟上提交的哈希码, 可以修改历史的提交.

执行该命令后, 会启动文本编辑器, 并在其中显示每次提交的记录信息(从前到后显示).

每行记录前面会有一个标识, 表示进行什么修改操作:

**Commands:**

- p, pick: 表示执行此次提交
- r, reword: 表示执行此次提交，但要修改备注内容
- e, edit: 表示可以修改此次提交，比如再追加文件或修改文件
- s, squash : 表示把此次提交的内容合并到上次提交中，备注内容也合并到上次提交中
- f, fixup : 和 “squash” 类似，但会丢弃掉此次备注内容
- x, exec : 执行命令行下的命令
- drop : 删除此次提交

本关用的就是reword.

## Level 46: squash

> You have committed several times but would like all those changes to be one commit.
>
> 把多次提交合并成一次

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
a68644a Updating README (squash this commit into Adding README)
712a031 Updating README (squash this commit into Adding README)
da9df2a Updating README (squash this commit into Adding README)
e799cb3 Adding README
04a03cf Initial Commit
[root@pipeline-cloud-test02 git_hug]# git rebase -i 04a03cf
[detached HEAD 63c098c] Adding README
 1 file changed, 3 insertions(+)
 create mode 100644 README
Successfully rebased and updated refs/heads/master.
[root@pipeline-cloud-test02 git_hug]# git log --oneline
63c098c Adding README
04a03cf Initial Commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

本关用的就是squash.

## Level 47: merge_squash

> Merge all commits from the long-feature-branch as a single commit.
>
> 把`long-feature-branch`上的多次提交合并到主干上的一次提交

```bash
[root@pipeline-cloud-test02 git_hug]# git branch
  long-feature-branch
* master
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
b79bd86 Second commit
24289dd First commit
[root@pipeline-cloud-test02 git_hug]# git log long-feature-branch  --oneline
0a1ccc1 Time
36fb820 Takes
cef0b75 Developing new features
24289dd First commit
[root@pipeline-cloud-test02 git_hug]# git merge long-feature-branch --squash
Squash commit -- not updating HEAD
Automatic merge went well; stopped before committing as requested
[root@pipeline-cloud-test02 git_hug]# git commit -m "merge from long-fxxxx-branch"
[master 18b8931] merge from long-fxxxx-branch
 1 file changed, 3 insertions(+)
 create mode 100644 file3
[root@pipeline-cloud-test02 git_hug]# git log master --oneline
18b8931 merge from long-fxxxx-branch
b79bd86 Second commit
24289dd First commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

通常分支上会有很多个提交, 在合并到主干上时也会给主干带过去多次提交记录, 这样不便于追踪.

所以使用`git merge xxbranch_name --squash`命令把分支的多次提交搞成一次.

注意使用完合并之后, 需要在`commit`一下, 并写上注释信息. (不加`--squash`时会自动commit).

## Level 48: reorder

> You have committed several times but in the wrong order. Please reorder your commits.
>
> 调整提交顺序

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
ef9e60e Second commit
54c5e74 Third commit
b424f5a First commit
b435e3c Initial Setup
[root@pipeline-cloud-test02 git_hug]# git rebase -i b435e3c
Successfully rebased and updated refs/heads/master.
[root@pipeline-cloud-test02 git_hug]# git log --oneline
94b9586 Third commit
866d552 Second commit
b424f5a First commit
b435e3c Initial Setup
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 49: bisect

> A bug was introduced somewhere along the way. You know that running “ruby prog.rb 5” should output 15. You can also run “make test”. What are the first 7 chars of the hash of the commit that introduced the bug.
>
> 在开发过程中引入了一个 bug。已知运行 “ruby prog.rb 5” 应该输入 15，你也可以运行 “make test” 进行测试。你需要确定引入 bug 的那次提交的哈希值的前7位。

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
12628f4 Another Commit
9795761 Another Commit
028763b Another Commit
888386c Another Commit
bb736dd Another Commit
18ed2ac Another Commit
5db7a7c Another Commit
7c03a99 Another Commit
9f54462 Another Commit
5d1eb75 Another Commit
fdbfc0d Another Commit
a530e7e Another Commit
ccddb96 Another Commit
2e1735d Another Commit
ffb097e Another Commit
e060c0d Another Commit
49774ea Another Commit
8c992af Another Commit
80a9b3d Another Commit
f608824 First commit
[root@pipeline-cloud-test02 git_hug]# git bisect start
[root@pipeline-cloud-test02 git_hug]# git bisect good f608824
[root@pipeline-cloud-test02 git_hug]# git bisect bad 12628f4
Bisecting: 9 revisions left to test after this (roughly 3 steps)
[fdbfc0d403e5ac0b2659cbfa2cbb061fcca0dc2a] Another Commit
[root@pipeline-cloud-test02 git_hug]# make test
ruby prog.rb 5 | ruby test.rb
[root@pipeline-cloud-test02 git_hug]# git bisect good
Bisecting: 4 revisions left to test after this (roughly 2 steps)
[18ed2ac1522a014412d4303ce7c8db39becab076] Another Commit
[root@pipeline-cloud-test02 git_hug]# make test
ruby prog.rb 5 | ruby test.rb
make: *** [test] Error 1
[root@pipeline-cloud-test02 git_hug]# git bisect bad
Bisecting: 2 revisions left to test after this (roughly 1 step)
[9f54462abbb991b167532929b34118113aa6c52e] Another Commit
[root@pipeline-cloud-test02 git_hug]# make test
ruby prog.rb 5 | ruby test.rb
[root@pipeline-cloud-test02 git_hug]# git bisect good
Bisecting: 0 revisions left to test after this (roughly 1 step)
[5db7a7cb90e745e2c9dbdd84810ccc7d91d92e72] Another Commit
[root@pipeline-cloud-test02 git_hug]# make test
ruby prog.rb 5 | ruby test.rb
[root@pipeline-cloud-test02 git_hug]# git bisect good
18ed2ac1522a014412d4303ce7c8db39becab076 is the first bad commit
commit 18ed2ac1522a014412d4303ce7c8db39becab076
Author: Robert Bittle <guywithnose@gmail.com>
Date:   Mon Apr 23 06:52:10 2012 -0400

    Another Commit

:100644 100644 917e70054c8f4a4a79a8e805c0e1601b455ad236 7562257b8e6446686ffc43a2386c50c254365020 M      prog.rb
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
What are the first 7 characters of the hash of the commit that introduced the bug? 18ed2ac1522a014412d4303ce7c8db39becab076
Congratulations, you have solved the level!
```

对我没用.. 略

## Level 50: stage_lines

> You’ve made changes within a single file that belong to two different features, but neither of the changes are yet staged. Stage only the changes belonging to the first feature.
>
> 一个文件里有两个功能, 现在只想把其中一个功能提交到暂存区.

```bash
[root@pipeline-cloud-test02 git_hug]# git status -s
 M feature.rb
[root@pipeline-cloud-test02 git_hug]# git diff feature.rb
diff --git a/feature.rb b/feature.rb
index 1a271e9..4a80dda 100644
--- a/feature.rb
+++ b/feature.rb
@@ -1 +1,3 @@
 this is the class of my feature
+This change belongs to the first feature
+This change belongs to the second feature
[root@pipeline-cloud-test02 git_hug]# git add feature.rb --edit
[root@pipeline-cloud-test02 git_hug]# git status -s
MM feature.rb
[root@pipeline-cloud-test02 git_hug]# git diff feature.rb
diff --git a/feature.rb b/feature.rb
index 3bccd0e..4a80dda 100644
--- a/feature.rb
+++ b/feature.rb
@@ -1,2 +1,3 @@
 this is the class of my feature
 This change belongs to the first feature
+This change belongs to the second feature
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

`git add file_name --edit`会在添加文件时打开编辑器, 并显示文件diff结果. 可以根据编辑的结果进行add操作.

## Level 51: find_old_branch

> You have been working on a branch but got distracted by a major issue and forgot the name of it. Switch back to that branch.
>
> 你忘了刚才时在哪个分支上工作了, 找到它. 切换过去

```bash
[root@pipeline-cloud-test02 git_hug]# git reflog
894a16d HEAD@{0}: commit: commit another todo
6876e5b HEAD@{1}: checkout: moving from solve_world_hunger to kill_the_batman
324336a HEAD@{2}: commit: commit todo
6876e5b HEAD@{3}: checkout: moving from blowup_sun_for_ransom to solve_world_hunger
6876e5b HEAD@{4}: checkout: moving from kill_the_batman to blowup_sun_for_ransom
6876e5b HEAD@{5}: checkout: moving from cure_common_cold to kill_the_batman
6876e5b HEAD@{6}: commit (initial): initial commit
[root@pipeline-cloud-test02 git_hug]# git checkout solve_world_hunger
Switched to branch 'solve_world_hunger'
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

`git reflog`不光记录了Commit信息, 还记录了分支切换的信息.

## Level 52: revert

> You have committed several times but want to undo the middle commit. All commits have been pushed, so you can’t change existing history.
>
> 你提交了很多次, 而且已经推送到服务器上了,所以你不能改变已经存在的历史. 你想取消中间的某次提交.

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
498b9be Second commit
5a42119 Bad commit
4d925f6 First commit
[root@pipeline-cloud-test02 git_hug]# git revert 5a42119 --no-edit
[master c608236] Revert "Bad commit"
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 file3
[root@pipeline-cloud-test02 git_hug]# git log --oneline
c608236 Revert "Bad commit"
498b9be Second commit
5a42119 Bad commit
4d925f6 First commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

既然不能改变历史, 那就用`git revert`增加一次取消的逆处理.

```bash
git revert hash-code # 调用文本编辑器让你编写注释
git revert hash-code --no-edit # 自动生成注释
```

## Level 53: restore

> You decided to delete your latest commit by running `git reset --hard HEAD^`. (Not a smart thing to do.) You then change your mind, and want that commit back. Restore the deleted commit.
>
> 你用`git reset --hard HEAD^`删掉了最后一次提交(傻), 然后你改变主意了, 想回退恢复那条被删除的提交.

```bash
[root@pipeline-cloud-test02 git_hug]# git log --oneline
d56c44f First commit
9fd667b Initial commit
[root@pipeline-cloud-test02 git_hug]# git reflog
d56c44f HEAD@{0}: reset: moving to HEAD^
dfabe57 HEAD@{1}: commit: Restore this commit
d56c44f HEAD@{2}: commit: First commit
9fd667b HEAD@{3}: commit (initial): Initial commit
[root@pipeline-cloud-test02 git_hug]# git reset --hard dfabe57
HEAD is now at dfabe57 Restore this commit
[root@pipeline-cloud-test02 git_hug]# git log --oneline
dfabe57 Restore this commit
d56c44f First commit
9fd667b Initial commit
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 54: conflict

> You need to merge mybranch into the current branch (master). But there may be some incorrect changes in mybranch which may cause conflicts. Solve any merge-conflicts you come across and finish the merge.
>
> 你要合并mybranch分支到当前分支master, 可能有冲突, 解决!

```bash
[root@pipeline-cloud-test02 git_hug]# git merge mybranch
Auto-merging poem.txt
CONFLICT (content): Merge conflict in poem.txt
Automatic merge failed; fix conflicts and then commit the result.
[root@pipeline-cloud-test02 git_hug]# vim poem.txt
[root@pipeline-cloud-test02 git_hug]# git status
# On branch master
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#
#       both modified:      poem.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
[root@pipeline-cloud-test02 git_hug]# git add poem.txt
[root@pipeline-cloud-test02 git_hug]# git commit -m "merge mybranch to master"
[master 7c3ba83] merge mybranch to master
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 55: submodule

> You want to include the files from the following repo: `https://github.com/jackmaney/githug-include-me`into a the folder `./githug-include-me`. Do this without cloning the repo or copying the files from the repo into this repo.
>
> 你想把 `https://github.com/jackmaney/githug-include-me` 这个仓库的代码引入到自己项目的 `./githug-include-me` 目录，这个方法不需要克隆第三方仓库，也不需要把第三方仓库的文件复制到你的项目中。

```bash
[root@pipeline-cloud-test02 git_hug]# git submodule add https://github.com/jackmaney/githug-include-me
Cloning into 'githug-include-me'...
remote: Counting objects: 9, done.
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 9
Unpacking objects: 100% (9/9), done.
[root@pipeline-cloud-test02 git_hug]# githug
********************************************************************************
*                                    Githug                                    *
********************************************************************************
Congratulations, you have solved the level!
```

## Level 56: contribute

> This is the final level, the goal is to contribute to this repository by making a pull request on GitHub. Please note that this level is designed to encourage you to add a valid contribution to Githug, not testing your ability to create a pull request. Contributions that are likely to be accepted are levels, bug fixes and improved documentation.

最后一关, 是让你去GitHub贡献一个request.

其实到这已经全部通关!
