---
title: Git 初探
date: 2016-03-02
categories:
  - 工具
tags:
  - Git
excerpt: "介绍 Git 分布式版本控制系统，对比 SVN 与 Git 的区别，并记录 Git 的安装、配置和基础命令操作。"
aiSummary: "介绍了 Git 作为分布式版本控制系统相对于 SVN（集中式）的核心优势：每个电脑都是完整版本库，无需联网即可工作。文章详细记录了在 CentOS 上的安装配置过程，包括用户名邮箱全局配置、创建仓库（git init）、文件操作（git add/commit/status）、历史查看（git log）、远程仓库（git remote/push/pull）、分支管理等基础命令的使用。"
---

### 前言

在公司实习的几个月里没机会接触 SVN 和 Git 这类版本管理工具。

抱着对 Linus 大神的崇敬，以及对开源吧的向往，趁着不忙的几天赶紧来学习一下 Git，希望以后能够用到。

其实 Git 还是十分好学的。用不了多久，你就能体会到它的高效简洁之美！这里我是在本地虚拟机 CentOS 上来学习的，只是了解它的简单原理和操作，并没有真正去尝试大项目。

同时借鉴了网络，自己边操作边学习。很多地方简单的例子我都有做图解（做 PPT 真的好难阿！！！），并且花了两天时间来学习并整理了这个小的文档，希望能有所收获！

<!-- more -->

---

### 1. 介绍 Git

Git 是一款免费、开源、高效的分布式版本控制系统。

- [Git 官网](https://git-scm.com)（支持命令行和 GUI 下载）
- [Git 官方帮助文档](https://gitref.org)

**SVN 和 Git 的主要区别：**

- **SVN 是集中式版本控制系统**：版本库是集中放在中央服务器的。干活的时候，用的都是自己的电脑，所以首先要从中央服务器那里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器（必须联网）。
- **Git 是分布式版本控制系统**：它没有所谓的“中央服务器”，每个人的电脑就是一个完整的版本库。这样工作的时候就不需要联网了，因为版本都在自己的电脑上。既然每个人都有完整的版本库，多个人如何协作呢？比如你在电脑上改了文件 A，其他人也改了文件 A，这时你们俩只需把各自的修改推送给对方，就可以互相看到对方的修改了（实际开发中通常还是会有一个充当“中央交换机”的远程仓库，比如 GitHub）。

### 2. 全局配置 Git

**安装与验证：**
```bash
sudo apt-get install git  # Ubuntu/Debian 系
yum install git           # CentOS/RedHat 系

git --version             # 查看版本
```

**全局配置（非常重要）：**
配置用户名和邮箱是为了提交代码时留下明确的团队身份标识。

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global color.ui true
```

> **提示**：
> - `git config --list`：查看当前 Git 的全局配置。
> - `cat ~/.gitconfig`：全局配置实际都保存在用户目录下的这个隐藏文件中，也可以直接用 `vi` 编辑它。
> - 获取帮助：使用 `git help` 或 `git help <特定指令>`。

### 3. 创建 Repository (仓库)

**方式一：初始化一个全新的 Git 仓库**

```bash
git init
```
这会在当前目录下初始化 Git 环境，并生成一个隐藏文件夹 `.git`（Git 仓库的核心，所有相关的数据、版本历史都保存在这儿）。
你可以通过 `ls -A` 查看到 `.git/` 目录。

**方式二：克隆现有的仓库**

```bash
git clone https://github.com/kennethreitz/requests.git
```
克隆网上的 Git 项目，会自动在本地创建并初始化好 Repository。

### 4. Git 的对象类型 (底层原理)

Git 中有四种基本对象类型，它们组成了 Git 高级的数据结构：

- **Blobs (数据对象)**：每个 blob 代表一个（版本的）文件。blob 只包含文件的数据内容，而忽略文件的其他元数据（如名字、路径、格式等）。
- **Trees (树对象)**：每个 tree 代表了一个目录的信息，包含了此目录下的 blobs、子目录（对应于子 trees）、文件名、路径等元数据。因此，对于有子目录的目录，Git 相当于存储了嵌套的 trees。
- **Commits (提交对象)**：每个 commit 记录了提交一个更新的所有元数据，如指向的 tree、父 commit、作者、提交日期、提交日志等。每次提交都指向一个 tree 对象，记录了当次提交时的目录快照。一个 commit 可以有多个（至少一个）父 commits。
- **Tags (标签对象)**：tag 用于给某个上述类型的对象指配一个便于开发者记忆的名字，通常用于标记某次重要的 commit（如里程碑版本）。

### 5. 提交及添加文件

首先，使用 `git status` 可以查看 Git 仓库当前的状态。新建的文件通常会显示为 `Untracked files`（未追踪的文件）。

**a) 添加到暂存区**
```bash
git add hello.py
```

**b) 提交到版本库**
```bash
git commit -m "init commit"
```
（如果不加 `-m` 参数，Git 会默认打开 vi 编辑器让你输入提交描述。这里的 "init commit" 表示首次提交的说明。）

**c) 跳过暂存区直接提交**
如果文件**已经被 Git 追踪过**（之前 add 过），可以直接使用以下命令将其修改一步提交到仓库：
```bash
git commit -am "modify hello.txt"
```
> **注意**：这并不能自动提交“未追踪（Untracked）”的新文件，新文件必须老老实实先 `git add`。

**核心概念：Git 的三个工作区域**

- **Working Directory (工作区)**：你平时能看到的目录，用于编辑、修改文件。
- **Staging Area / Index (暂存区)**：暂时存放已经修改的文件快照，准备下一次提交。
- **Git Repository / History (本地仓库)**：最终确定的文件保存到仓库，成为一个新的版本，并且对他人可见。

### 6. 查看 Git 状态与文件差别

使用 `git status -s` 可以列出简要的状态信息。文件前面通常有两个标识位（flag）：
- 第一个位代表 **Staging area（暂存区）**的状态。
- 第二个位代表 **Working directory（工作区）**的状态。

当文件被修改时，会出现 `M`（Modified）。如果两个区域完全一致，标志位就是空白的。

**查看文件差别：**
```bash
# 查看 工作区 vs 暂存区 的差异
git diff

# 查看 暂存区 vs 本地仓库(History) 的差异
git diff --staged

# 查看 工作区 vs 本地仓库 的差异
git diff HEAD

# 简化输出统计信息
git diff --stat
```

### 7. 移除及重命名文件

**删除文件：**
1. 从 Git 中删除文件：`git rm filename`
2. 提交操作：`git commit -m "delete filename"`
> *注意：这只是删除了当前版本中的文件，该文件依然被记录在 Git 的历史版本中。*
> *如果只想把文件从暂存区移除（不再追踪），但保留在物理磁盘上，使用：`git rm --cached filename`*

**重命名文件：**
```bash
git mv old_name new_name
git commit -m "rename file"
```
这其实相当于执行了三步操作：
1. `mv old_name new_name` (系统级别改名)
2. `git rm old_name`
3. `git add new_name`

### 8. 暂存工作区现场 (Stash)

假设出现这种情况：你在工作区修改了代码（修改1）并 `add` 到了暂存区，还未 `commit`。此时突然来了一个紧急 Bug，你需要回到修改1之前的状态去修复（修改2），但是又不想放弃修改1。

此时可以使用 **Stash** 暂存工作区：

```bash
# 暂存当前未提交的现场
git stash

# 此时工作区变得干净了，你可以安心修复紧急 Bug 并提交
# git commit -am "quick fix"

# 查看暂存的工作区列表
git stash list

# Bug 修复完后，弹出刚才暂存的现场继续工作
git stash pop
```
执行 pop 后，你之前的“修改1”就恢复到了工作区。

### 9. 理解 tree-ish 表达式与底层对象

我们可以通过底层命令去窥探 Git 是如何存储 commit 的。
每个 commit 后面都有一个 Hash 码来唯一标识。`HEAD` 默认指向当前分支的最新 commit。

```bash
# 查看对象的类型 (blob / tree / commit)
git cat-file -t <hash>

# 打印对象的内容
git cat-file -p <hash>

# 查看单行历史
git log --oneline

# 查看 HEAD 的真实指向
git rev-parse HEAD
```

**Tree-ish 表达式：**
为了更方便地定位到历史中的某个对象或文件，我们可以使用 tree-ish 表达式，而不必每次都去查长长的 Hash 值。
- 定位到前第 3 个 commit 的 tree 对象：`HEAD~3^{tree}`
- 定位到前第 3 个 commit 中的某个具体文件：`HEAD~3:hello.py`

### 10. 分支管理 (Branch)

Git 的分支极其轻量级，本质上它只是一个指向某个 commit 的可变指针。

- **查看分支**：`git branch`
- **新建分支**：`git branch newBranch`
- **切换分支**：`git checkout newBranch`
  *(创建并切换可以合并为一步：`git checkout -b newBranch`)*
- **删除分支**：`git branch -d newBranch`

> *注意：切换分支，实际上就是把 `HEAD` 指针指向了不同的 branch。*

### 11. 合并分支 (Merge)

合并分支有两种主要的机制：

**机制一：Fast-forward (快进合并)**
当你在 `master` 分支上拉出一个 `newBranch`，在 `newBranch` 上提交了新代码，而这段时间内 `master` 没有发生任何变化。
此时合并 `newBranch` 到 `master`，Git 只需要将 `master` 指针“快进”指向 `newBranch` 的最新 commit 即可。这种合并非常快，不会产生新的合并提交对象。

**机制二：3-way merge (三方合并)**
如果在两个分支（例如 `master` 和 `bugFix`）上都各自进行了新的代码提交，此时再进行合并，就无法使用快进机制了。
Git 会进行“三方合并”：
1. 找到这两个分支最近的**共同祖先**（基准节点）。
2. 将两个分支的最新提交与这个祖先进行比对，提取出差异。
3. 将两边的差异合并在一起，生成一个新的 commit 对象（Merge Commit），然后 `master` 指向这个新对象。分支合并完毕！

### 12. Git 远程仓库

本地玩转了 Git，接下来就是与团队协作了。远程仓库的实现通常有两种：

1. **使用现有的代码托管平台**：
   - GitHub: [https://github.com](https://github.com) (全球最大的开源社区)
   - GitLab / Gitee (国内常用的企业/个人托管平台)
   - Bitbucket: [https://bitbucket.org](https://bitbucket.org)
2. **搭建自己的私有 Git 服务器**（如基于 GitLab 社区版搭建企业内部仓库）。