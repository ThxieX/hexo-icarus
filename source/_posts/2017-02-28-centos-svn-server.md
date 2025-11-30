---
title: CentOS 搭建 SVN 服务器
date: 2017-02-28
categories:
  - DevOps
tags:
  - SVN
  - Linux
  - Shell
excerpt: "在 CentOS 上搭建 SVN 服务器，创建版本库，配置用户权限，并利用 Hooks 实现代码提交后自动同步到 Web 目录。"
aiSummary: "本文记录在 CentOS 上搭建 SVN 服务器的完整过程，包括安装 Subversion、创建版本库（svnadmin create）、配置用户和权限（svnserve.conf、passwd、authz）、启动服务（svnserve -d），以及通过 Hooks 钩子（post-commit）实现代码提交后自动同步到 Web 目录的自动化部署流程。"
---

### 序言

工作所需不想每次都被喊去手动执行 `svn up` 更新代码！

**预期目的：**
1. 搭建 SVN 服务器，并创建版本库。
2. 配置用户和组，使得可以完成基本的 `checkout` 和 `commit` 操作。
3. 利用 SVN 钩子（Hooks），实现仓库代码更新后自动同步到 Web 目录下，免去手动操作的烦恼。

<!-- more -->

---

### 一、安装环境并创建 SVN 版本库

#### 1. 安装

安装 Subversion：

```bash
yum install -y subversion
```

查看是否安装成功：

```bash
svnserve --version
```

若安装成功，会显示类似如下信息：`svnserve, version 1.7.14 (r1542130)...`

#### 2. 创建版本库

接下来创建版本库：

- 建立目录：`mkdir -p /cloudlink/webapp`
- 创建版本库：`svnadmin create /cloudlink/webapp/project`

执行完毕后，可以在 `project` 目录下看到生成了如下版本库文件及目录：
`conf`、`db`、`format`、`hooks`、`locks`、`README.txt`。

进入 `conf` 目录，主要有以下三个核心配置文件：
- `authz`：权限配置文件
- `passwd`：用户名口令配置文件
- `svnserve.conf`：服务配置文件

---

### 二、配置用户和组

#### 1. 配置 authz

编辑 `authz` 文件，设置用户组及目录读写权限：

```ini
[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]
# 将用户 xiefayang 加入 cloudlink 用户组
cloudlink = xiefayang 

# 用户组 cloudlink 对版本库 project 具有读写 (rw) 权限
[project:/]
@cloudlink = rw
```

#### 2. 配置 passwd

编辑 `passwd` 文件，设置用户的密码：

```ini
[users]
# harry = harryssecret
# sally = sallyssecret
xiefayang = xiefayang
```

#### 3. 配置 svnserve.conf

编辑 `svnserve.conf` 文件，找到如下配置项，去掉前面的 `#` 注释，并做相应配置：

```ini
# 匿名用户访问权限：无
anon-access = none

# 普通用户访问权限：读、写
auth-access = write

# 指定密码文件
password-db = passwd

# 指定权限配置文件
authz-db = authz

# 版本库所在绝对路径
realm = /cloudlink/webapp/project
```

---

### 三、启动与测试

#### 1. 启动服务

启动 SVN 服务（注意是 `svnserve` 命令，而不是 `svn`）：

```bash
svnserve -d -r /cloudlink/webapp
```

> 如果不能启动成功，可以先查出进程并杀掉后再重启：
> `ps aux | grep svn`

#### 2. 测试本地 Checkout 与 Commit

切换到 `/home/www` 目录下测试 `checkout`：

```bash
svn co svn://localhost/project
```

查看是否检出成功。接着测试 `commit`：

```bash
vi test.py
svn add test.py
svn commit test.py -m "test commit file"
```

如果显示提交成功，则说明服务器本地环境搭建完成。

#### 3. 远程客户端测试

在另一台机器（如 Windows）上安装 SVN 客户端（推荐 TortoiseSVN）。
右键选择 **SVN Checkout**：填入对应的仓库地址（如 `svn://<服务器IP>/project`）、用户名和密码，点击 OK 即可检出。

![svn_checkout](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/svn_checkout.png)

> **注意**：如果检出失败（如网络超时），需要检查一下防火墙（如 `/etc/sysconfig/iptables`）是否开放了 3690 端口。可加入如下规则：
> `-A INPUT -p tcp -m state --state NEW -m tcp --dport 3690 -j ACCEPT`

---

### 四、利用钩子实现自动更新和同步

这一步主要是为了实现：当我们在本地修改完代码并提交到 SVN 服务器后，服务器上用于部署的 Web 目录能自动同步最新代码，无需人工干预。

在刚才版本库的 `hooks` 目录下新建 `post-commit` 文件：
（`post-commit` 意味着在提交完成后被调用执行，目录自带的 `post-commit.tmpl` 只是个模板，直接新建即可）

1. 创建并编辑脚本：`vi post-commit`

```bash
#!/bin/sh
export LANG=en_US.utf8
SVN_PATH=/usr/bin/svn   
WEB_PATH=/home/www/project  # web目录的路径
$SVN_PATH update $WEB_PATH --username 'xiefayang' --password 'xiefayang' --no-auth-cache
```

2. 检查 Web 目录的权限（假设 `/home/www` 的所有者和所属组都是 `thank`）。
3. 修改 `post-commit` 脚本的属主为对应的 Web 目录用户：`chown thank:thank post-commit`
4. 赋予脚本执行权限：`chmod 755 post-commit`

**总结**：
请务必区分【SVN 服务器端】和【Web 部署目录】这两个概念。
如果 SVN 服务器和 Web 目录都在**同一台机器**上，按照上述配置，在任意地点提交代码后，该机器的 Web 目录就会自动触发更新同步。

> **进阶探讨**：如果 SVN 服务器和 Web 目录**不在同一台机器上**该如何处理？
> 欢迎阅读我的另一篇解决方案：《[CentOS 中 SVN 服务器实现自动更新](./CentOS%20%E4%B8%AD%20SVN%20%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E6%9B%B4%E6%96%B0.md)》。
