---
title: Linux 远程无密传输实现脚本
date: 2017-05-15
categories:
  - DevOps
tags:
  - Linux
  - Shell
excerpt: "编写 Shell 脚本实现 Linux 服务器间远程无密码文件传输，通过 SSH Key 认证和 expect 自动化交互完成一键部署。"
aiSummary: "本文记录了通过 SSH Key 实现 Linux 服务器间无密码远程传输文件的脚本编写过程。核心原理是使用 ssh-keygen 生成公私钥对，将公钥追加到目标服务器的 ~/.ssh/authorized_keys 实现免密登录。对于 scp/ssh 命令的自动化交互，使用 expect 工具编写脚本自动应答密码提示。文章给出了完整的脚本实现，实现了开发、测试、预上线等多环境间的一键文件传输和部署，将原本需要手动完成的重复性工作自动化。"
---

### 序言

我在开发过程中，除了要部署后台的微服务，还要对前端的 WEB 目录进行部署。
虽然每次只需要 2 分钟左右，但却是机械的重复性劳动。我认为很 xx！

听说过俄罗斯有一位程序员，认为超过 90 秒以上的事情，就必须要做成脚本，比如煮咖啡、骗老婆说要加班等等……

所以我决定花几个小时时间，把这种事情写成脚本。

目前公司的前端 WEB 目录部署其实就是文件的上传下载，不需要启动什么进程。既然前端人员不想用 Linux 也不想用 FTP。那么……我在 Linux 下编写成脚本后，让前端人员直接在 Windows 下通过一个按钮点击，就完成 WEB 目录的一键部署。

这样，我就完全可以从这件事中脱身了。

<!-- more -->

---

### 一、场景描述

首先梳理一下预上线环境部署 WEB 目录的流程：
- **开发环境**: xxx.xxx.x.213
- **测试环境**: xxx.xxx.x.212
- **预上线环境**: xxx.xxx.x.110

前两个环境是我来手动维护，因为之前已经写了很多脚本（自动 Git 拉取、Maven 打包、一键杀死/启动微服务等），所以维护起来很轻松。

而 110 环境是通过 Jenkins 来构建的，每次构建完成都会生成**全新的虚拟机镜像**（服务器、数据库、Redis 都会是新的，只有 IP 不变）。

所以 WEB 目录的部署逻辑是：在 212 上拉取测试通过的目录，传输到预上线的 WEB 目录。但是经常会有 bug 修正，每次修正完又要重新拉一遍。

看起来这个脚本应该很简单，就是把主机 A 的 WEB 目录 copy 到主机 B 嘛。

---

### 二、建立通信机制

但是涉及到两台机器通信，就会遇到密码验证的问题。总不能执行脚本之后再卡在终端让人工输入密码吧？

解决这个问题主要有以下两种方式：
1. **SSH 免密**：建立互信关系。
2. **Expect 脚本**：自动模拟人工输入密码。

#### 方式一：SSH 免密登录

关于什么是 SSH、公钥、密钥原理什么的可以 Google，这里简单介绍下操作方法。

假设现在有两台 Linux (CentOS 7) 机器：
- 机器 A：`192.168.100.212`
- 机器 B：`192.168.100.213`

要实现从 A 用 SSH 无密码远程登录到 B：

**1. 首先在 A 上生成公钥**
```bash
ssh-keygen -t rsa
```
这里一直点回车即可（不要输入密码，就是空密码）。
执行完成后，在 `~/.ssh/` 下就会生成 `id_rsa` 和 `id_rsa.pub` 两个文件。

**2. 把 A 机器刚才生成的 `id_rsa.pub` 文件复制到 B 上**
```bash
scp ~/.ssh/id_rsa.pub root@192.168.100.213:/root/
```
然后在 B 的 root 目录下就能看到 `id_rsa.pub` 了（这里放在 root 下只是临时放一下）。

**3. 在 B 机器中，将公钥内容追加到 `authorized_keys` 文件**
```bash
cat /root/id_rsa.pub >> ~/.ssh/authorized_keys
```
> *如果 `~/` 下没有 `.ssh` 目录或者没有 `authorized_keys` 文件，就自己手动创建。*

**4. 在 B 上重启 sshd 服务**
```bash
service ssh restart
```

配置完成！现在在 A 上试试登录 B：`ssh root@192.168.100.213`，应该就不需要输入密码了。

> **踩坑注意事项**：
> - 要用哪个用户远程登录，就把 `id_rsa.pub` 复制到该用户对应的路径下。如：root 用户就复制到 `/root/` 下；普通用户就复制到 `/home/xxx/` 下，不要混了。
> - B 机器上的文件权限非常严格：`.ssh` 文件夹必须是 `700`；`authorized_keys` 必须是 `600`。

#### 方式二：使用 Expect 自动输入

在我写这个脚本的时候，并**没有**使用上面那种 SSH 免密的方法。
原因是上面提到过，每次预上线环境（110）构建完都会是一台全新的虚拟机，不可能每次构建完都去人工配一次 SSH 信任关系。

所以这里介绍一种更优雅的方式：在脚本里自动输入密码（当屏幕交互需要输入时，由脚本捕获并自动发送输入）。
这里用到的工具就是 **Expect**。

**1. 安装 Expect**
```bash
sudo yum install expect
```

**2. 编写 `scp.exp` 脚本**
```expect
#!/usr/bin/expect  
set timeout 20  

if { [llength $argv] < 2} {  
    puts "Usage:"  
    puts "$argv0 local_file remote_path"  
    exit 1  
}  

set local_file [lindex $argv 0]  
set remote_path [lindex $argv 1]  

# 注意：在现代安全规范中，极其不推荐在脚本中硬编码明文密码！
# 生产环境中建议结合 CI/CD 工具的安全环境变量（Secret）来传递。
set passwd your_passwd  

set passwderror 0  

spawn scp $local_file $remote_path  

expect {  
    "*assword:*" {  
        if { $passwderror == 1 } {  
            puts "passwd is error"  
            exit 2  
        }  
        set timeout 1000  
        set passwderror 1  
        send "$passwd\r"  
        exp_continue  
    }  
    "*es/no)?*" {  
        send "yes\r"  
        exp_continue  
    }  
    timeout {  
        puts "connect is timeout"  
        exit 3  
    }  
}
```

这里我找到了一个比较通用的模板（除了能自动输入密码之外，还支持首次连接时 RSA 验证的 `yes/no` 自动选择）。

> **语法说明**：
> 可以看到第一行并不是 `#!/bin/bash`，所以这段脚本是由 `expect` 解释执行的，语法也和 Shell 不一样。
> - `[lindex $argv 0]`：获取传入的第一个参数（本地要拷贝的文件）。
> - `[lindex $argv 1]`：获取传入的第二个参数（远程目标地址）。
> - `spawn scp ...`：派生出一个子进程来执行实际的 scp 拷贝命令。

**执行示例**：
例如我要把本机（213）的某个文件复制到主机 110 上：
```bash
./scp.exp /hahah/hello.py root@192.168.120.110:/xx/yy/
```

#### 踩坑记录：Expect 中执行 scp 目录的坑

在使用上述 Expect 脚本进行 scp 拷贝时，如果涉及到通配符 `*`，会有一个坑：

- `./scp.exp /hahah/hello.py root@ip:/xx/yy/`：拷贝单个文件，没问题。
- `./scp.exp /hahah/dir1 root@ip:/xx/yy/`：拷贝整个目录，没问题。
- `./scp.exp /hahah/dir1/* root@ip:/xx/yy/`：按理说是把 `dir1` 目录下的所有文件（不包括 dir1 目录本身）拷贝过去。这在原生的 Shell scp 命令下是没问题的，但由于这里执行的是 `scp.exp`，它会直接把 `*` 当成特殊字符处理导致报错。

**解决办法**：需要用反斜杠 `\` 来转义星号。
```bash
./scp.exp /hahah/dir1/\* root@192.168.120.110:/xx/yy/
```

---

### 三、整合调度脚本

两种通信方式介绍完了。因为我要通过一个前端的 HTTP 请求去调用拷贝动作，为了方便，我又写了一个 Bash 脚本去统筹调用刚才的 `scp.exp`。

`update_pre_online.sh` 内容如下：

```bash
#!/bin/bash

# 为了让前端调用时能正常返回文本信息
echo "Content-Type: text/html"
echo ""

# 源路径与目标路径配置
SRC_PATH=/home/www/update_pre_online/xxws-web-120/
TAG_PATH=/sysroot/tmp/xxws/
TAG_ADDRESS=192.168.120.110

# 1. 先把测试环境(212)的 WEB 目录拉取到当前执行脚本的机器(213)上
scp -r root@192.168.100.212:/home/www/xxws-web/* ${SRC_PATH}

# 2. 赋予访问权限
chmod -R 755 ${SRC_PATH}
echo "============================"
echo "递归修改目录访问权限(755)..."
echo "============================"

sleep 1

# 3. 调用 expect 脚本，将本地拉取好的文件推送到预上线环境(110)
# 注意这里的星号转义：${SRC_PATH}\*
/home/www/update_pre_online/scp.exp ${SRC_PATH}\* root@${TAG_ADDRESS}:${TAG_PATH}

# 4. 输出日志与结果
cat ~/update_pre_online.log
echo "--------------------------------------"
echo " 更新 212WEB目录 到 110WEB目录 成功!  "
echo "--------------------------------------"

sleep 2
exit 0
```

---

### 总结

完成后，在后台管理系统加一个小页面按钮，前端开发同学点一下就行了，效果如下：

![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/shell%E9%83%A8%E7%BD%B2%E6%95%88%E6%9E%9C%E5%9B%BE.png)

虽然以前熟练的话，手动敲命令部署每次只需要三分钟，但我编写脚本加测试却用了好几个小时。

**成果就是：现在每次部署只需要点击按钮后等待 5 秒即可。最重要的是！！！不需要我来操作了！**

**The End**