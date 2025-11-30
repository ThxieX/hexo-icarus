---
title: SVN 分支管理
date: 2017-12-22
categories:
  - 工具
tags:
  - SVN
  - Archived
excerpt: "介绍 SVN 的标准目录结构（Trunk、Branches、Tag、Release）及分支的创建、合并、切换等常用操作命令。"
aiSummary: "本文记录了 SVN 分支管理的常用操作，包括 SVN 的标准目录结构概念：Trunk（主干）、Branches（分支）、Tag（基线/快照）、Release（发布）。详细介绍了新建分支（svn copy）、切换分支（svn switch）、合并分支（svn merge）、解析冲突等实际开发中常用的命令和流程。"
---

> 前言：工作中遇到了相关需求，简单记录下 SVN 中的分支管理备忘。
> SVN的标准目录结构：欲了解详细，请参照：http://www.cnmiss.cn/?p=296

### 一、基础概念

在进行分支操作前，先来了解几个 SVN 的标准目录结构基本概念：

* **Trunk (主干库)**：存放核心项目代码，通常是最新的开发主线。
* **Branches (分支库)**：存放为不同用户定制化的版本、或阶段性的稳定 release 版本。这些版本是可以继续独立进行开发和维护的。
* **Tag (基线库)**：存档目录。通常用于里程碑版本的快照，**原则上不允许修改**。
* **Release (发布库)**：存放历次发布内容。

<!-- more -->

---

### 二、常见操作命令

> 假设主干仓库地址为: `svn://192.168.100.212/xxws-web/trunk`

#### 1. 新建分支 (branch1)

```bash
# 将主干复制一份作为新分支，执行后 branch1 下就会有 trunk 上的完整代码，且后续开发互不影响
svn cp svn://192.168.100.212/xxws-web/trunk svn://192.168.100.212/xxws-web/branch1 -m "create branch1 version"
```

#### 2. 删除分支 (branch1)

```bash
svn rm svn://192.168.100.212/xxws-web/branch1 -m "remove branch1"
```

#### 3. 检出分支来开发

```bash
svn co svn://192.168.100.212/xxws-web/branch1
```

#### 4. 合并主干最新代码到分支

当主干（Trunk）有了新代码，而你的分支（Branch）也需要这些更新时：

```bash
# 先进入分支所在的本地工作目录
cd branch1

# 执行合并操作
svn merge svn://192.168.100.212/xxws-web/trunk

# (可选) 在合并前，可以先预览该合并操作会带来哪些变更版本
svn mergeinfo svn://192.168.100.212/xxws-web/trunk --show-revs eligible
```

#### 5. 分支合并回主干

当分支开发测试完毕，需要将其合并回主干发布时：

```bash
# 先进入主干所在的本地工作目录
cd trunk

# 使用 --reintegrate 参数将分支合并回主干
svn merge --reintegrate svn://192.168.100.212/xxws-web/branch1
```

> **历史遗留疑问**：网上教程常说，分支合并到主干中完成后应当删除该分支，因为在 SVN 中该分支已经不能进行刷新也不能合并到主干了 ? ? ?
>
> **【解惑补充】**：
> 这个说法在 **SVN 1.8 版本之前**是正确的。因为使用 `--reintegrate` 合并后，SVN 的元数据会认为该分支的历史已经全部注入主干，如果继续在这个分支开发并再次合并，会导致树冲突（Tree Conflict）。所以在老版本中，标准的做法是合并后删掉该分支，下次需要时再重新基于主干拉一个新分支。
> 但是，**从 SVN 1.8 开始**，SVN 引擎已经变得更聪明了，它支持自动重组合并（Automatic Merge），你甚至不需要加 `--reintegrate` 参数，SVN 就能自动判断合并方向。因此在新版本中，合并后不删除分支继续开发也是可以的。

#### 6. 合并指定版本到当前分支

如果只需要合并某几次特定的提交记录：

```bash
# 例如只把版本 148 到 149 的变更合并过来
svn -r 148:149 merge http://svn_server/xxx_repository/trunk
```