---
title: CentOS 中 SVN 服务器实现自动更新
date: 2017-03-09 00:00:00
categories:
  - DevOps
tags:
  - SVN
  - Linux
  - Shell
excerpt: "解决 SVN 仓库与 Web 目录不在同一台服务器时，如何实现代码提交后自动同步到远程服务器的问题。"
aiSummary: "本文针对 SVN 仓库与 Web 目录不在同一台机器的场景，提供了两种自动更新方案。方案一采用循环定时更新（轮询）方式，通过后台脚本定时执行 svn up；方案二通过 SVN 的 post-commit 钩子触发本地脚本，再使用 ssh 远程执行更新命令。文章详细给出了两种方案的 Shell 脚本实现，并说明了各自的优缺点和适用场景。"
---

### 前言

在上一篇《[CentOS 搭建 SVN 服务器](/posts/centos-svn-server)》中，曾提到使用钩子（Hooks）实现自动更新和同步。

> 但现实往往比较骨感：难受的是我们的 SVN 服务器不允许直接登录存放 hooks。为了避免每次都被喊去手动执行更新，我尝试了以下两种解决方案。

**背景说明**：
利用 SVN 钩子实现自动同步通常有一个前提条件，即 **SVN 服务器版本库** 和 **要更新的 Web 目录** 必须在同一台机器（Server-A）上。
其本质是利用 `hooks/post-commit` 的“提交后自动触发”功能：在 `post-commit` 中编写 `svn up` 脚本，当客户端（Server-B）提交代码后，触发 Server-A 的 hook，自动执行脚本以更新 Server-A 中的 Web 目录。

但如果 SVN 服务器版本库在 Server-A，而要更新的 Web 目录在 Server-B 呢？

<!-- more -->

---

### 方案一：循环定时更新（轮询）

先介绍一种相对“傻”的方法（事实上我刚开始也是这么做的）：
在 Web 目录所在机器（Server-B）上，编写一个循环定时更新的 Shell 脚本，让它在后台不断轮询。

例如，每过 10 秒执行一次 `update`：

```bash
#!/bin/bash
while [ 1 -gt 0 ]
do
  sleep 10
  svn update /home/www/project/ &>/dev/null
done
```

如果需要记录更新日志，可以改进为以下方式：

```bash
#!/bin/bash
pre_update="$(svn log /home/www/project/ | head -2 | awk 'NR==2{print $1}')"

while [ 1 -gt 0 ]
do
  svn update /home/www/project/ &>/dev/null
  now_update="$(svn log /home/www/project/ | head -2 | awk 'NR==2{print $1}')"
  
  if [[ "${now_update}" > "${pre_update}" ]]
  then
    echo " update_time : $(date) " >> /tmp/svn_update.log
    pre_update=${now_update}
  fi
  sleep 5
done
```

---

### 方案二：利用钩子触发跨服务器更新

回到正轨，依然利用 `post-commit` 的提交后自动触发功能。但因为要更新的 Web 目录在另一台机器上，所以我们需要在 Hook 触发后进行**远程文件传输**。

#### 1. 核心命令说明

这里需要用到两个常用的文件传输命令：

- **`scp`**：用于在 Linux 下进行远程拷贝文件。
  类似本机的 `cp` 命令，但 `scp` 支持跨服务器且传输过程是加密的。
- **`rsync`**：一个远程数据同步工具，可通过 LAN/WAN 快速同步多台主机间的文件。
  `rsync` 采用“rsync 算法”，只传送两个文件的差异部分，而不是每次都整份传送，因此同步速度极快。

> **注意**：虽然在通常情况下 `rsync` 的增量同步比 `scp` 更快，但当面临大量小文件时，`rsync` 会导致硬盘 I/O 非常高，此时使用 `scp` 反而对系统性能影响更小。

#### 2. 配置 SSH 免密登录

使用脚本进行远程拷贝，必须解决权限验证问题，因此我们需要配置 SSH 免密登录：

1. **生成密钥**：在机器 A 上执行 `ssh-keygen` 命令，在 `~/.ssh` 目录下生成 `id_rsa`（私钥）和 `id_rsa.pub`（公钥）。
2. **分发公钥**：将 `id_rsa.pub` 拷贝到机器 B 上，并将其内容追加到 B 机器的 `~/.ssh/authorized_keys` 文件中。
   *注：请确保 `.ssh` 目录（700）及 `authorized_keys` 文件（600）的权限正确。*

> 可以参考: [SSH免密码远程登录Linux](http://www.cnblogs.com/bootoo/p/5068514.html)

配置完成后，可以在 A 机器上测试：`ssh root@[B的IP]`。如果无需输入密码即可登录，则配置成功。

#### 3. 编写 post-commit 脚本

最后，在 SVN 版本库的 `hooks` 目录下创建并编辑 `post-commit` 文件：

```bash
#!/bin/bash

export LANG=en_US.utf8
SVN=/usr/bin/svn
WEB=/home/www/project/
RSYNC=/usr/bin/rsync
LOG=/tmp/uplog_xfy.log
WEBIP="192.168.100.212"

echo " start update ! " >> $LOG

# 先更新本机（SVN 服务器）的一个临时/镜像 WEB 目录
$SVN update $WEB --username 'xiefy' --password 'xiefy'

# 判断上一步 update 是否执行成功
if [ $? -eq 0 ]
then
   echo $(date) >> $LOG 
   
   # 选择 1：scp 方式
   scp -r $WEB root@$WEBIP:/home/xiefy/ >> $LOG
   
   # 选择 2：rsync 同步方式
   # rsync -vaztpH --timeout=90 $WEB root@$WEBIP:/home/xiefy/ >> $LOG   
fi
```

好了，作为一名开发狗，Linux 下的 SVN 服务器暂时就折腾到这里……

**The End**