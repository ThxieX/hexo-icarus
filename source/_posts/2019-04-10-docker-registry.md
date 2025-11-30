---
title: Docker 镜像仓库
thumbnail: /thumbnails/harbor_logo.png
date: 2019-04-10 00:00:00
categories:
  - Cloud Native
tags:
  - Docker
  - Harbor
excerpt: "搭建私有 Docker 镜像仓库，对比 Docker Hub、阿里云镜像加速器、Registry 和 Harbor 的使用方法。"
aiSummary: "本文从实际场景出发，介绍了 Docker 镜像仓库的完整体系：公共仓库（Docker Hub、阿里云容器镜像服务加速器）和私有仓库（Docker Registry、Harbor）。详细记录了镜像的推送（docker push）、拉取（docker pull）操作，Harbor 的安装配置（Docker Compose、Nginx 反向代理、HTTPS 配置），以及 Web UI 管理、镜像同步、权限管理等企业级功能。是搭建私有镜像仓库的实用指南。"
---

本文仅从一个实际场景出发：**自己制作的镜像该如何分享与管理？** 
重点记录从公共仓库（Docker Hub、阿里云）到私有镜像仓库（Registry、Harbor）的搭建与使用过程。

<!-- more -->

---

## 一、Docker Hub

首先介绍官方的 [Docker Hub](http://hub.docker.com)。

### 推送与拉取镜像

推送自己的镜像到 Docker Hub 上。因为网速原因……这里我用一个超小的镜像 `busybox` 来做实验。

```bash
[root@bogon ~]# docker images | grep busy
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
busybox                      latest              af2f74c517aa        6 days ago          1.2MB
```

**1. 打标签 (Tag)**

```bash
docker tag busybox:latest thxiex/busybox:1.0
```

```bash
[root@bogon ~]# docker images | grep busy
busybox                      latest              af2f74c517aa        6 days ago          1.2MB
thxiex/busybox             latest              af2f74c517aa        6 days ago          1.2MB
```

> **注意**：这里的 `thxiex` 就是我在 Docker Hub 中的 username。必须对应，否则推不上去。

**2. 推送 (Push)**

```bash
docker push thxiex/busybox:1.0
```

去自己的镜像仓库看下吧，已经有了！

> **踩坑提示**：如果推送过程中提示 `denied: requested access to the resource is denied`，说明未授权，先执行 `docker login` 登录一下即可。

拉取镜像（Pull）大家都很熟悉了：

```bash
docker pull thxiex/busybox:1.0
```

---

## 二、国内镜像仓库

国内也有很多镜像托管服务，例如网易云、DaoCloud、阿里云等，上传和拉取镜像的“姿势”基本和 Docker Hub 一样。

> 吐槽：网易云的仓库这两天个人认证居然无法使用了，必须要企业认证。所以我这里改用阿里云的容器镜像服务来做实验。

首先需要在`阿里云 - 容器镜像服务`中创建一个命名空间，并设置仓库类型（公有或私有）：

![创建命名空间](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF/docker/%E9%98%BF%E9%87%8C%E4%BA%91-%E5%88%9B%E5%BB%BA%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4.jpg)

### 1. 推送和拉取

这里省略自己制作镜像的过程，以一个我已经拉取好的官方镜像 `redis:latest` 为例。

**首先登录阿里云 Registry**：
```bash
$ sudo docker login --username=xxx registry.cn-hangzhou.aliyuncs.com
```

**Tag & Push & Pull**：
```bash
# 打标签
docker tag redis:latest registry.cn-hangzhou.aliyuncs.com/thank/redis:1.0

# 推送
docker push registry.cn-hangzhou.aliyuncs.com/thank/redis:1.0

# 拉取
docker pull registry.cn-hangzhou.aliyuncs.com/thank/redis:1.0
```

去阿里云控制台的镜像仓库看看吧，成功上传！

![镜像仓库](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF/docker/%E9%98%BF%E9%87%8C%E4%BA%91-%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93.jpg)

### 2. 基于代码源自动构建

上面的方式是自己在 Docker 环境中制作好镜像再手动 Push。这里介绍一种更极客的**基于代码源自动构建**方式。

以 GitHub 为例，代码源为：`https://github.com/thxiex/spring-cloud-study2`

1. **创建镜像仓库**：填写仓库名称，选择对应的命名空间。
2. **设置代码源**：选择 GitHub，第一次使用需要 OAuth 绑定账号。
3. **构建设置**：
   - 开启代码变更自动构建镜像功能。
   - **构建规则设置**：设置代码源的分支/tag、Dockerfile 的相对路径及名称、需要构建的镜像版本标签。
4. **开始构建**：点击规则上的“立即构建”即可。

每次的构建过程会实时输出到构建日志中，方便排错查看。

![构建设置](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF/docker/%E9%98%BF%E9%87%8C%E4%BA%91-%E6%9E%84%E5%BB%BA%E8%AE%BE%E7%BD%AE.jpg)

---

## 三、私有仓库搭建

出于安全性、合规性以及内网传输速度的考虑，很多企业会选择搭建内部的私有镜像仓库。

### 1. 官方原生 Registry

这是一个官方提供的基础镜像仓库，安装极其简单！

```bash
# 拉取镜像并启动
docker pull registry:2
docker run -d -p 5000:5000 registry:2
```

推送和拉取过程跟推送到公有仓库差不多：

```bash
# 打标签
docker tag busybox:latest localhost:5000/busybox:1.0

# 推送
docker push localhost:5000/busybox:1.0

# 拉取
docker pull localhost:5000/busybox:1.0
```

> **注意避坑（HTTP Insecure 限制）**：
> 如果镜像仓库和 Docker 客户端不在同一台机器上，推送时大概率会报 HTTPS 安全错误。
> 解决办法：在 Client 端的 `/etc/docker/daemon.json` 中加入免密白名单：
> ```json
> { "insecure-registries": ["<Registry_IP>:5000"] }
> ```
> 然后重启 Docker：`systemctl restart docker`。

除了推送和拉取，Registry 还提供了一些 RESTful API，用于查看镜像信息等。例如：
```bash
[root@bogon ~]# curl localhost:5000/v2/_catalog
{"repositories":["busybox"]}
```

**Registry 的缺点**：
虽然它很轻便，具备了基础的镜像管理功能，但有两个硬伤：
- 不具备授权认证功能，需要自己去折腾一层认证方案。
- 只有干巴巴的 API，没有好看且易用的 Web 界面。

---

### 2. 企业级私有仓库：Harbor

Harbor 意为“港湾”，非常贴合它的作用。

> **背景**：Harbor 最初由 VMware 中国研发团队开源，后来捐赠给了 CNCF（云原生计算基金会），所以它的 GitHub 仓库地址现在也转到了 [goharbor](https://github.com/goharbor/harbor) 下。

官方将 Harbor 定义为**企业级私有 Registry 服务器**。它底层实际也是依赖 Docker Registry，但在这之上封装了大量企业级特性：
- 提供直观友好的 Web UI 界面。
- 完善的 RBAC 安全机制（包含角色、项目权限控制、审计日志等）。
- 支持多实例水平扩展和镜像复制同步。
- 镜像分层传输优化，提升内网分发速度。

#### 安装 Harbor

**环境准备**：
安装前需要宿主机已安装 Docker 和 Docker-Compose，这里不赘述了。

**下载与解压**：
在 [GitHub Releases](https://github.com/goharbor/harbor/releases) 中查看需要的版本（这里我用的是 `v1.7.5`）。
官方提供 `offline`（离线）和 `online`（在线）两种安装包。考虑到服务器网络和代理问题，我果断选择了 `offline` 离线包（约 500MB，内含所有依赖镜像）。

解压后大致看下目录结构：
```bash
[root@localhost opt]# tree harbor -CL 2
harbor                                 
├── common                             
│   ├── config                         
│   └── templates                      
├── docker-compose.yml                 
├── harbor.cfg                         
├── install.sh                         
...
```

**修改配置 `harbor.cfg`**：
编辑核心配置文件，按需修改：

```properties
# 强烈建议：不要设置成 localhost 或 127.0.0.1，换成本机 IP 或域名
hostname = hub.thank.com

# 邮箱相关的设置（按需）
email_server = smtp.163.com
email_server_port = 25
email_username = coderthank@163.com
email_password = ******
email_from = thank <coderthank@163.com>

# 修改 admin 默认的登录密码
harbor_admin_password = 123123
```

**执行安装**：
运行 `./install.sh` 即可一键傻瓜式安装：

```text
[Step 0]: checking installation environment ...
Note: docker version: 1.13.1
Note: docker-compose version: 1.16.1

[Step 1]: loading Harbor images ...
...略

[Step 4]: starting Harbor ...
...略

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://hub.thank.com.
```

通过 `docker ps` 可以看到，Docker-Compose 已经在后台帮我们拉起了一整套 Harbor 相关的微服务集群（包含 nginx、core、db、redis、jobservice 等）。

#### 界面使用

因为刚才在配置文件配了一个假域名，这里我在本地修改下 hosts：
```properties
192.168.118.143 hub.thank.com
```

浏览器访问 `http://hub.thank.com`（默认 80 端口），即可看到 Harbor 的登录页面。

![harbor主页](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF/docker/harbor%E4%B8%BB%E9%A1%B5.jpg)

**基础测试**：
登录 Admin 账号 -> 创建用户 `thank` -> 创建私有项目 `cloudlink-base` -> 在项目中添加用户 `thank`，赋予项目管理员角色。
其它功能就不做过多介绍了，中文简体界面，自己点一点摸索下就好啦！

#### 镜像推送测试

这里测试一下如何往搭建好的 Harbor 中推送镜像。方法跟其它仓库基本一致，不啰嗦了：

```bash
# 登录
docker login hub.thank.com

# 重新打标签，注意命名空间对应项目名
docker tag redis:latest hub.thank.com/cloudlink-base/thank-redis:latest

# 推送
docker push hub.thank.com/cloudlink-base/thank-redis:latest

# 拉取测试
docker pull hub.thank.com/cloudlink-base/thank-redis:latest
```

#### 运维管理与 HTTPS 避坑

我们可以直接使用 Docker-Compose 来统一完成 Harbor 服务的运维启停：
- 停止：`docker-compose stop`
- 启动：`docker-compose start`

**HTTPS 避坑指南**：
刚才在配置文件 `harbor.cfg` 中默认使用的是 HTTP 协议。如果从其它主机上传镜像，Docker 客户端默认会强制使用 HTTPS 访问 Harbor，这会导致报 `connection refused` 或证书错误。

**两种解决办法**：
1. **简单粗暴法（开发测试用）**：
   在所有 Client 端的 `/etc/docker/daemon.json` 中加入非安全信任白名单：
   ```json
   {
     "insecure-registries" : ["hub.thank.com"]
   }
   ```
2. **正规配置法（生产环境）**：
   修改 Harbor 配置文件 `ui_url_protocol = https`，并配置合法的 TLS 证书。配置不算很复杂，具体可参考网上的相关教程（如：[Harbor 配置 TLS 证书](https://www.cnblogs.com/pangguoping/p/7650014.html)）。