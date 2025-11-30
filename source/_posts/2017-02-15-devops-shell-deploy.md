---
title: Dev 微服务部署简单 Shell 脚本
date: 2017-02-15 00:00:00
categories:
  - DevOps
tags:
  - Linux
  - Shell
excerpt: "使用 Shell 脚本自动化微服务的部署流程，包括从 Git 拉取代码、Maven 打包编译、服务启停等操作。"
aiSummary: "本文记录了作者为解决微服务部署繁琐问题而编写的 Shell 脚本经验。Shell 是 Linux 系统管理的重要工具，文章介绍了 Shell 脚本的基础知识和实际应用场景：实现从 Git 远端拉取指定分支代码、使用 Maven 自动打包编译、微服务的单个和批量启停操作。通过脚本实现了部署流程的半自动化，大幅提升了开发和测试环境的部署效率。"
---

### 序言

之前作为一个“开发狗”，其实并不太涉及 Linux 系统管理，对 Shell 脚本也一无所知。

但是由于后端服务的部署和打包经常需要在 Linux 环境下进行，这对于极度懒人的我来说过于繁琐。每次都要到处查询命令、复制粘贴然后再执行。

能让计算机做的，为什么要我们自己做呢哈哈哈哈哈哈哈哈！

我开始去研究 Shell。虽然没有系统完整地去学习，但目前在测试和开发环境中已经有十多个脚本来负责服务的批量和单个操作，用起来方便很多，基本能够实现：

- 从 Git 远端拉取指定分支的服务代码
- 使用 Maven 自动打包编译
- 微服务的单个 / 批量启停

<!-- more -->

---

### 关于 Shell

Shell 使用 C 语言编写，是 Linux 管理中必不可少的工具，也是系统与硬件交互的介质。

我们通过指令传给 Shell，Shell 再通过系统内核实现对计算机硬件的操作。它有自己的命令和程序设计语言，供我们编写自动化脚本，在我看来，有点像是在做 PL/SQL 编程。

Shell 脚本需要的运行环境超级简单：
- 文本编辑器
- 脚本解释器（默认即带）

所以，在 Linux 环境下我们可以轻而易举地编写 Shell 脚本。

---

### 部署脚本实践

接下来，我按照自己实际要解决的目的，选择性地记录一些踩坑和实用的命令。

#### 1. 杀死服务进程

要杀死某个服务的进程，通常需要先找到它的网络连接信息或进程 ID。

**常规做法：使用 `ps` 命令**
先显示程序的运行状态：

```bash
ps -aux | grep java
```

程序的标识、状态、ID、资源占用等信息就会显示出来。然后找到对应的进程 ID（PID），通过 `kill <PID>` 来杀死这个服务。

**我更喜欢使用 `lsof` 命令**
> 如果你还不了解，可以看看这篇：[很好的 lsof 命令介绍](https://linux.cn/article-4099-1.html)

在查找进程时，我通常只用到以下几个参数：
- `-i:[port]`：显示与指定端口相关的网络信息
- `-t`：仅获取进程的 ID（这在写脚本时非常有用）
- `-sTCP`：仅显示 TCP 连接（UDP 同理）

找出处于监听状态的端口：

```bash
lsof -i -sTCP:LISTEN
```
*(或者使用 `grep` 命令也可以获得：`lsof -i | grep -i LISTEN`)*

**一键杀死进程**
结合反引号执行命令，要杀死某个启动了具体端口的服务进程，只需要一行：

```bash
kill -15 `lsof -i:[端口号] -t -sTCP:LISTEN`
```

为了确保稳妥，我们可以再通过一个简单的 `if` 判断来确定进程是否已经被真正杀死：

```bash
kill -15 `lsof -i:[端口号] -t -sTCP:LISTEN`
sleep 1
if [ -z "$(lsof -i:[端口号] -t -s TCP:LISTEN)" ]; then  
  echo " port killed! "
fi
```

**实战脚本：按端口传参杀进程**
这里我写了一个通过传参来杀死对应服务进程的脚本 `killed_single_port.sh`：

```bash
#!/bin/bash

printf " Please input killed port: "
read port

# 定义允许操作的安全端口列表
port_array=(8901 8902 8903 8905 8906 8907 8908 8909 8761 8050)

# 判断输入的端口是否在允许的列表中
result=`echo "${port_array[@]}" | grep -wq "$port" && echo "y" || echo "n"`

if [ "n" == "${result}" ]; then
   echo "port:\"${port}\" not exits ! "
   exit 5
fi

kill -15 `lsof -i:${port} -t -sTCP:LISTEN`
sleep 1

# 检查是否清理成功
if [ -z "$(lsof -i:${port} -t -s TCP:LISTEN)" ]; then
  echo " port \"${port}\" is killed! "
fi

exit 0
```

---

#### 2. 启动服务

通常，我们使用 `nohup` 命令来“不挂断”地运行程序，以此来启动我们的微服务。

**`nohup` 命令解析**
在 Linux/Unix 启动程序中，我们一般希望服务在后台运行，只需在命令末尾加一个 `&` 即可：

```bash
nohup java -jar /aa/bb/xxx.jar &
```

**日志重定向**
如果需要记录程序的启动信息，可以通过 `>` 将输出重定向到日志文件中。系统预留了几个标准输出符：
- `0`：标准输入信息
- `1`：标准输出信息（正常日志）
- `2`：标准错误输出信息
- `/dev/null`：黑洞，输出到这里的内容将不显示也不保存

例如，启动服务并分离正常日志与错误日志：

```bash
nohup java -jar /aa/bb/xxx.jar  \
  --server.port=8050 \
  --eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8761/eureka/ \
  > /my_log/zuul.log 2> /my_log/zuul_error.log &
```

**实战脚本：存在依赖关系的批量启动**
如果要批量启动几个服务，而且这几个服务之间存在先后依赖关系（比如网关或注册中心必须先起），如果仅仅把启动命令堆到一个脚本里顺序执行，那肯定会出问题。

所以，这里我们再次利用 `lsof` 命令来循环探测前置程序的状态，进而来判断是否继续执行后续的启动命令。

以下脚本实现了：判断 `x` 服务（端口 `9000`）启动成功后，再顺序执行 `a`、`b` 程序的启动，最后再判断 `a`、`b` 是否都已成功上线。

```bash
#!/bin/bash
echo "x server booting..."
nohup java -jar /aa/bb/x.jar &>/dev/null &

# 循环探测 9000 端口的 x 服务是否启动
# 提示：在生产环境中，为防止死循环，建议在此处增加超时跳出逻辑（例如设定尝试次数上限）
while [ -z "$(lsof -i:9000 -t -s TCP:LISTEN)" ] 
do
  sleep 3
done

if [ -n "$(lsof -i:9000 -t -s TCP:LISTEN)" ]; then
  echo "x server start success!"
else
  echo "ERROR: x server boot failure !!!! "
  exit 5
fi

# 只有当 9000 的 x 服务启动成功后，继续执行下面步骤

# 启动 a 服务
nohup java -jar /aa/bb/a_server.jar  \
  --server.port=8901 \
  > /my_log/a.log 2> /my_log/a_error.log &

# 启动 b 服务
nohup java -jar /aa/bb/b_server.jar  \
  --server.port=8902 \
  > /my_log/b.log 2> /my_log/b_error.log &

# 判断 a(port:8901), b(port:8902) 服务是否启动成功
i=2
a_server=1
b_server=1

# 同样，在严谨的脚本中，这里也应考虑超时跳出以防止无限阻塞
while [ 1 -gt 0 ] && [ ${i} -gt 0 ]
do
  if [ -n "$(lsof -i:8901 -t -s TCP:LISTEN)" ]; then
    echo "a server start success !!!"
    i=$[ ${i} - 1 ]
  fi
  if [ -n "$(lsof -i:8902 -t -s TCP:LISTEN)" ]; then
    echo "b server start success !!!"
    i=$[ ${i} - 1 ]
  fi
  sleep 3
done

echo "All servers are successfully booted!"
```