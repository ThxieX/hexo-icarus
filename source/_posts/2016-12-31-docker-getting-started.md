---
title: Docker 初探
date: 2016-12-31 00:00:00
categories:
  - Cloud Native
tags:
  - Docker
excerpt: "在 CentOS 7 上搭建 Docker 环境，学习容器镜像、容器管理、镜像加速等核心操作和常用命令。"
aiSummary: "Docker 是云原生时代最重要的容器化技术。本文是 Docker 入门教程，在 CentOS 7 虚拟机上搭建 Docker 环境，记录了 JDK 安装、防火墙配置等前置准备，以及 Docker 的核心概念：镜像（Image）和容器（Container）。文章详细介绍了镜像的获取（pull）、查看（images）、删除（rmi）、导出（save）等操作，容器的创建（run）、启动（start）、停止（stop）、进入（exec）、删除（rm）等命令，还涉及镜像加速器的配置和使用。是学习 Docker 的实用入门指南。"
---

### 前言

本次学习环境是本地虚拟机，Linux 版本使用 CentOS 7。

由于使用了最小化（Minimal）安装方式，所以在正式学习 Docker 之前，顺带踩坑并总结了一下 JDK 的安装和防火墙相关配置。

<!-- more -->

---

### 一、JDK 安装

在 CentOS 下安装 JDK 有多种方式，例如手动下载 tar 文件解压、Yum 安装、RPM 安装等。这里我为了省事，直接使用 Yum 方式安装：

1. **查看系统中是否已经安装 JDK**
   ```bash
   yum list installed | grep java
   ```
   如果有安装且想卸载，直接 `yum remove xxx` 即可。

2. **查看 Yum 库中的 JDK 安装包**
   ```bash
   yum list java-1.8.0*
   # 或者
   yum search java | grep jdk
   ```

3. **安装 JDK 1.8**
   ```bash
   yum -y install java-1.8.0-openjdk 
   ```

4. **确认安装目录与验证**
   安装完成后的默认目录通常在 `/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64`（具体版本号可能略有差异）。直接输入 `java -version` 验证即可。

> **注意**：这种 Yum 方式安装的是 OpenJDK，并不是 Oracle Sun JDK。如果对 Sun JDK 有强需求，建议走手动下载解压的安装方式。

因为这种方式使用 `alternatives` 进行版本控制，所以在没有手动设置环境变量的情况下也可以直接执行 `java` 命令。但如果要在 Tomcat 或其他依赖 Java 的软件中使用，还是得老老实实配置环境变量。

**环境变量配置**：
修改 `/etc/profile` 文件，追加以下内容：

```bash
# set java environment
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

保存后，执行 `source /etc/profile` 让修改生效。

---

### 二、防火墙配置

CentOS 7 中默认使用 `firewalld` 作为防火墙，而 6.x 版本中是 `iptables`。
虽然 CentOS 7 下也支持 `iptables`（需要关闭禁止 `firewall` 然后安装 `iptables-services`），但这二者不能共用。

> > CentOS7下Firewall防火墙配置用法详解(推荐): https://yq.aliyun.com/ziliao/94786

我觉得既然 CentOS 7 已经升级到了 `firewall`，就没必要强行切换回 `iptables` 了，不如顺势熟悉一下新工具。

关于 `firewall` 的配置，首先明确两个核心目录：
- **系统配置目录**：`/usr/lib/firewalld/services`（存放定义好的网络服务和端口参数，系统默认，不建议修改）
- **用户配置目录**：`/etc/firewalld/`（自定义配置）

#### 1. 自定义添加端口

`firewall` 支持两种自定义添加的方式：命令行添加和配置文件添加。使用命令添加的内容，最终也会在配置文件中体现。

**方式 A：命令行添加**

```bash
# 默认添加到 public zone
firewall-cmd --permanent --add-port=8050/tcp 

# 指定添加到某个 zone
firewall-cmd --zone=public --permanent --add-port=8050/tcp
```
*参数说明：*
- `firewall-cmd`：Linux 提供的操作 firewall 的 CLI 工具。
- `--permanent`：设置为持久化规则（重启不失效）。
- `--add-port`：标识要添加的端口。
- `--zone=public`：指定生效的作用域（zone）。如果指定 `--zone=dmz`，则会在 `dmz.xml` 文件中新增一条规则。

**方式 B：配置文件添加**

直接在 `/etc/firewalld/zones` 目录下修改 XML 配置，如 `public.xml`：

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>xxx</description>
  <service name="dhcpv6-client"/>
  <service name="ssh"/>
  <port protocol="tcp" port="8050"/>
  <port protocol="tcp" port="8761"/>
</zone>
```

#### 2. 常用命令备忘

**Firewall 常用操作：**

```bash
# 查看 firewall 服务状态
systemctl status firewalld

# 查看 firewall 的运行状态 (running or not running)
firewall-cmd --state

# 查看防火墙所有规则
firewall-cmd --list-all 

# 查看已开放的端口 (--zone 可以不写，默认为 public)
firewall-cmd --zone=public --list-ports

# 重启 / 开启 / 关闭 防火墙
systemctl restart firewalld
systemctl start firewalld
systemctl stop firewalld
```

*(顺便补充下 iptables 的基本使用，以备不时之需)*

```bash
# 查看 / 关闭 / 启动
service iptables status
service iptables stop
service iptables start

# 查看规则列表
iptables --list-rules

# 编辑规则
vim /etc/sysconfig/iptables
# 在文件中加入一行：-A INPUT -p tcp -m state --state NEW -m tcp --dport 25 -j ACCEPT

# 保存并重启防火墙
service iptables restart
```

---

### 三、Docker 初探

#### 1. 安装 Docker

**安装方式一：一键安装（适用于测试）**

网上有说 CentOS 7 以上直接用 `docker` 包，我这里用 `docker-io` 测试也没问题：

```bash
yum install -y docker-io

# 启动并设置开机自启
systemctl start docker
systemctl enable docker

# 查看信息与版本
docker info 
docker version
```

**安装方式二：官方推荐安装方式（推荐）**

参考官方文档：[Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)

1. **卸载旧版本**（如果存在）：
```bash
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-selinux \
                docker-engine-selinux \
                docker-engine
                
sudo rm -rf /var/lib/docker
```

2. **安装依赖并配置仓库**：
```bash
# 安装所需依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 配置官方 Docker 仓库
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

> **踩坑记录**：在国内配置官方仓库时，极大概率会遇到 `TCP connection reset by peer` 的网络报错。
> **解决方案**：使用阿里云的源替换官方源：
> ```bash
> sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
> ```

3. **安装并启动**：
```bash
sudo yum install docker-ce
systemctl start docker
```

#### 2. 镜像管理 (Image)

- **在线拉取**：可以在 [Docker Hub](http://hub.docker.com) 中查找需要的镜像，或者使用 `docker search xxx` 命令。
  - 例如拉取 MySQL：`docker pull mysql`（默认从官方源拉取，国内太慢建议配置镜像加速器，如阿里云加速器）。
- **离线安装**：将 tar 包拷贝至服务器，执行 `docker load < ./xxx.tar`。
- **列出本地镜像**：`docker images`
- **打包镜像导出**：`docker save -o ./mysql123.tar mysql:latest`

#### 3. 容器管理 (Container)

镜像（Image）是只读的，在镜像之上加一层读写层，就成了容器（Container）。

- **创建容器**：`docker create`（创建后处于 Created 状态）。
- **启动容器**：`docker start [container id]`。

**最常用的二合一命令：`docker run`**
该命令实际包含了 `create` 和 `start` 两个动作。

```bash
# 启动两个独立的 MySQL 容器
docker run -d -p 3306:3306 --name mysql1 -e MYSQL_ROOT_PASSWORD=root mysql:5.7.14
docker run -d -p 3308:3306 --name mysql2 -e MYSQL_ROOT_PASSWORD=root mysql:5.7.14
```
*核心参数解释：*
- `-d`：后台运行（Detached mode）。
- `-p`：端口映射，格式为 `宿主机端口:容器内端口`。
- `-e`：设置环境变量（如 MySQL 的 root 密码）。
- `--name`：为容器指定一个容易辨识的名字（不指定的话 Docker 会随机分配一个奇葩名字）。

**生命周期管理**：
```bash
# 查看容器（-a 表示包括已停止的所有容器）
docker ps -a

# 停止容器
docker stop [id]

# 删除容器（-f 强制删除正在运行的容器）
docker rm -f [id]  

# 删除镜像（注意：必须先删除依赖该镜像的容器，才能删除镜像）
docker rmi [image_id]

# 进入正在运行的容器内部
docker exec -it [container_name_or_id] /bin/bash
```

#### 4. 数据卷管理 (Data Volumes)

容器中管理数据主要有两种方式：数据卷（Data Volumes）和数据卷容器（Data Volumes Containers）。数据卷的作用主要是为了**数据持久化**。

**方式 A：挂载本地目录作为数据卷**

```bash
docker run -d -it --name=centos1 -v /root/dbdata:/test/dbdata centos:7.2.1511
```
*参数说明：*
- `-v /xx:/yy`：将宿主机中的绝对路径 `/xx` 挂载到容器内的 `/yy` 目录中。

启动容器后，在宿主机 `/root/dbdata` 和容器 `/test/dbdata` 中的任何操作都会双向同步。关闭或删除容器后，宿主机的数据依然保留。

> **踩坑记录**：挂载目录时可能会遇到 `Permission denied` 问题，导致容器中映射的目录无权限打开，或者启动容器后直接变成 `exited` 状态。
> **原因排查**：通常是 SELinux 导致的。这货的访问控制策略相当复杂，在非严苛的安全环境下，很多运维的建议都是直接把它关闭。
> **解决方案**：
> - 查看状态：`sestatus` 或 `getenforce`
> - 永久关闭：`vi /etc/selinux/config`，将 `SELINUX=enforcing` 改为 `disabled`，然后重启服务器。

**方式 B：数据卷容器**

如果有一批持续更新的数据需要在多个容器之间共享，创建一个专门的数据卷容器是更好的选择。

1. **创建一个专用的数据卷容器**（起名叫 `data_container`）：
   ```bash
   docker run -it -d -v /dbdata --name data_container centos:7.2.1511
   ```

2. **创建其他容器并挂载该数据卷容器**：
   ```bash
   docker run -it -d --volumes-from data_container --name centos1 centos:7.2.1511
   docker run -it -d --volumes-from data_container --name centos2 centos:7.2.1511
   ```
此时，`centos1` 和 `centos2` 都可以共享并读写 `/dbdata` 目录中的数据。

> **关于 `--volumes-from`**：
> 1. 提供数据卷的父容器（`data_container`）自身甚至不需要处于运行状态！
> 2. 可以多次使用该参数，从多个容器中挂载多个数据卷。

**利用数据卷容器进行数据迁移与备份**

- **备份**：将数据卷打包到宿主机
  ```bash
  docker run --volumes-from data_container -v $(pwd):/backup --name worker centos:7.2.1511 tar cvf /backup/backup.tar /dbdata 
  ```
  *(原理解析：启动一个临时的 worker 容器，挂载数据卷容器的数据，同时把宿主机当前目录挂载到容器的 `/backup`，然后执行 `tar` 压缩命令，最终在宿主机当前目录生成 `backup.tar`)*

- **恢复**：
  ```bash
  # 1. 创建一个带有目标数据卷的容器
  docker run -d -it -v /dbdata --name db centos:7.2.1511
  
  # 2. 启动临时容器，解压备份文件到目标数据卷
  docker run --volumes-from db -v $(pwd):/backup centos:7.2.1511 tar xvf /backup/backup.tar
  ```

#### 5. Dockerfile 构建镜像

如何使用 `Dockerfile` 将一个 Java 应用程序打包成镜像。
假设将应用程序 `jpa.jar` 和 `Dockerfile` 文件都放在 `/root/training` 目录下。

`Dockerfile` 内容如下：

```dockerfile
# 1. 指定基础镜像（必须放在第一行）
FROM java:8u102-jdk

# 2. 指定镜像维护者信息
MAINTAINER thank

# 3. RUN 命令：执行任何被基础镜像支持的 Linux 命令
RUN mkdir /app

# 4. ADD 命令：将宿主机当前路径下的文件拷贝到容器中的 /app 目录
ADD . /app

# 5. EXPOSE：声明容器打算暴露的端口（仅声明，实际映射需在 run 时指定 -p）
EXPOSE 8888

# 6. ENTRYPOINT：设置容器启动时默认执行的操作
ENTRYPOINT ["java", "-jar", "/app/jpa.jar", "--spring.profiles.active=prod"]
```

**构建镜像**：
在 `/root/training` 目录下执行构建命令（注意末尾的点 `.` 代表当前上下文路径）：

```bash
docker build -t jpa:1.0 .
```

执行命令后, 会看到以下输出, 每一层都对应Dockerfile文件中定义的指令。

```
Sending build context to Docker daemon  27.7 MB
Step 1 : FROM java:8u102-jdk
 ---> 69a777edb6dc
Step 2 : RUN mkdir /app
 ---> Running in 6779b11de804
 ---> 44fc7d80f5f6
Removing intermediate container 6779b11de804
Step 3 : ADD . /app
 ---> c419c4d34889
Removing intermediate container 00fa44dcbc4e
Step 4 : EXPOSE 8888
 ---> Running in bb4e70a56e04
 ---> 24e66dd52ee1
Removing intermediate container bb4e70a56e04
Step 5 : ENTRYPOINT java -jar /app/jpa.jar --spring.profiles.active=prod
 ---> Running in b5cbcfce01b1
 ---> 45e74f2c2173
Removing intermediate container b5cbcfce01b1
Successfully built 45e74f2c2173

```

构建成功后，启动容器验证：

```bash
docker run -d --name myapp --link mysql:mysql -p 8901:8888 jpa:1.0  
```
*(说明：`--link mysql:mysql` 用于容器间的网络互通，前面的 mysql 是被链接的容器名，后面是当前容器内的别名，这在较新的 Docker 网络模型中已逐渐被 User-defined Bridge Network 替代。)*

#### 6. Docker-compose 简介

Compose 是 Docker 的服务编排工具，主要用来构建基于 Docker 的复杂应用。通过一个 `docker-compose.yml` 配置文件即可管理多个容器的启动和交互，非常适合微服务开发场景。

安装方式（拉取二进制文件并赋予执行权限）：

```bash
curl -L "https://github.com/docker/compose/releases/download/1.8.0-rc1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

*(后续深入用法待续...)*

---

### 四、疑难杂症记录

#### 问题一：启动 Docker 报错 SELinux 不支持

执行 `systemctl start docker` 时报错（在部分内核较老的机器上会出现）：
```text
Error starting daemon: SELinux is not supported with the overlay2 graph driver on this kernel. 
Either boot into a newer kernel or disable selinux in docker (--selinux-enabled=false)
```
**原因**：当前 Linux 内核中的 SELinux 不支持 `overlay2` 存储驱动。
**解决办法**：修改 Docker 配置文件，在启动参数中禁用 SELinux 支持。
```bash
vi /etc/sysconfig/docker
# 将 --selinux-enabled 改为 --selinux-enabled=false
```

#### 问题二：容器启动报 IPv4 forwarding is disabled 警告

在虚拟机中启动容器时抛出警告：
`WARNING: IPv4 forwarding is disabled. Networking will not work.`
虽然容器勉强起来了，但由于网络转发被禁用，宿主机外部根本无法访问容器映射的端口。

**解决办法**：
编辑系统网络配置开启 IPv4 转发：
```bash
vi /etc/sysctl.conf
# 或者修改 /usr/lib/sysctl.d/00-system.conf
# 添加如下配置：
net.ipv4.ip_forward=1
```
重启网络服务并验证：
```bash
systemctl restart network
sysctl net.ipv4.ip_forward
# 如果返回 net.ipv4.ip_forward = 1 则表示修改成功。
```

> 推荐一个解决 Docker 学习中疑难杂症文章：[Docker 问答录](https://blog.lab99.org/post/docker-2016-07-14-faq.html)。