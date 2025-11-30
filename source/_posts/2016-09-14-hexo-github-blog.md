---
title: Hexo + GitHub 搭建独立博客
date: 2016-09-14 00:00:00
categories:
  - Personal
tags:
  - Hexo
  - GitHub
  - Blog
excerpt: "使用 Hexo 静态博客框架结合 GitHub Pages 搭建个人独立博客，告别 CSDN 和博客园的广告与兼容性问题。"
aiSummary: "本文记录了使用 Hexo + GitHub Pages 搭建独立博客的完整过程。Hexo 是一个简洁高效的静态博客框架，GitHub Pages 提供免费的静态页面托管服务。文章介绍了环境准备（Git、Node.js）、Hexo 安装初始化、主题选择与配置、撰写博客文章、部署到 GitHub Pages 等核心步骤，并推荐了相关参考教程。是技术人员搭建个人技术博客的入门指南。"
---

### 前言

之前一直看别人在 CSDN 和博客园上写博客，上面有很丰富的资料。但自己在 CSDN 上记录学习文章的过程中，发现了一些不太喜欢的地方：比如嵌入代码和图片时偶尔会出现兼容性问题（可能是我的操作姿势不对），而且会有广告，页面也缺乏个性。


在互联网上搜集了一些资料，参考了很多别人的个人博客后，我最终选择了 **Hexo + GitHub Pages** 的方案。

整体难度不大，但是过程还是比较繁琐的，需要 Git，很多操作都是命令行的配置。如果你有强迫症，想要把博客定制得比较符合心意，就需要阅读更多文档来做个性化配置。

因为网上有很多优秀的教程和官方文档，完全够用。本文仅简短记录我自己的搭建过程，不做累赘的配置描述。

> **推荐两篇比较好的参考博文：**
> - [如何搭建一个独立博客——简明Github Pages与Hexo教程](http://blog.sina.com.cn/s/blog_617ccc0c0101h84p.html)
> - [一步步在GitHub上创建博客主页(7个系列)](http://www.pchou.info/ssgithubPage/2013-01-03-build-github-blog-page-01.html)

<!-- more -->

---

### 一、方案介绍

#### 1. 关于 Hexo
Hexo 是一个简单、轻便的静态博客框架。
关于 Hexo 的详细配置，官方文档已经非常完善：[Hexo 中文文档](https://hexo.io/zh-cn/docs/)

#### 2. 关于 GitHub Pages
我有自己的阿里云ECS, 但是不想放在上面, 因为是借同学在校证明买的优惠机, 他毕业机器就没了...

> GitHub Pages 是 Github 提供的静态页面托管服务。它设计的初衷是为了让用户能够直接通过 Github 仓库来托管个人、组织或是项目的专属页面，也就是一个由 Git 管理的静态服务器。

相比 WordPress，它的搭建部署稍微复杂一点，需要安装环境，也会因为粗心出现一些 Bug。但它**非常适合爱折腾的 Coder**。

**GitHub Pages 的限制与优势：**
- 仓库存储的所有文件不能超过 1 GB。
- 页面的带宽限制是每月 100 GB 或每月 100,000 次请求。
- 每小时最多只能部署 10 个静态网站。
- **优势**：有 300M 免费空间，依托 GitHub，能与大神为邻，拥抱开源，分享知识。

*(注：在搭建之前，需要懂一点 Git 相关知识。可以参考我之前写的《[Git 初探](./Git%20%E5%88%9D%E6%8E%A2.md)》。)*

---

### 二、环境准备

这两个环境是必须的，软件较小，近乎傻瓜式安装：
- **Git**: [http://git-scm.com/](http://git-scm.com/)
- **Node.js**: [http://nodejs.org/](http://nodejs.org/)

#### 1. 注册 GitHub
进入 [GitHub](http://www.github.com/) Sign up 注册账号。牢记注册信息（特别是 Email），需要到邮箱验证一下。
登录之后，点击头像下的 `Settings` 选项，我们主要需要设置 **SSH keys** 和 **Repositories (仓库)**。

![GitHub Settings](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%8D%9A%E5%AE%A21.png)

#### 2. 配置 SSH keys
秘钥的作用是让你的本地电脑和远程 GitHub 建立安全联系。
打开 Git Bash，输入以下命令检查本机的 SSH 秘钥：

```bash
cd ~/.ssh
```

如果提示 `No Such file or directory`，说明需要我们自己生成：

```bash
# 替换为你的注册邮箱
ssh-keygen -t rsa -C "your_email@example.com"
```

按照提示一路回车即可（如果需要设置密码可以自己输入）。当提示 `your identification has been saved in...` 并出现一段字符画时，表示生成成功。

去你生成的那个目录（如 `C:\Users\你的用户名\.ssh`）找到 `id_rsa.pub` 文件，用文本编辑器打开并把里面的内容全选复制。

回到 GitHub 页面的 `Settings` -> `SSH and GPG keys`，点击 `New SSH key`，将复制的内容粘贴到 key 框中并保存。

![添加 SSH Key](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%8D%9A%E5%AE%A22.png)

**验证连接：**
在 Git Bash 中输入：
```bash
ssh -T git@github.com
```
如果提示 `Hi dog! You've successfully authenticated...`，就表示连接成功了！

顺便配置一下以后提交的用户信息（填你自己的）：
```bash
git config --global user.name "dog"
git config --global user.email "coderthank@163.com"
```

#### 3. 建立仓库 (Repositories)
在 GitHub 点击 `New repository`。

![创建仓库](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%8D%9A%E5%AE%A23.png)

> **核心注意**：这里的 Repository name 必须是 `你的用户名.github.io`。如果要用 GitHub Pages 建立个人博客，格式必须严格遵守，不能乱写。个人博客的这类专属仓库每个账号只能建立一个。

点击 `Create repository`，以后所有的博客内容、模板、文章等都会存放在这里。

---

### 三、部署 Hexo 搭建工具

Hexo 基于 Node.js，使用前可以先检查下环境：`node -v`。

**1. 全局安装 Hexo：**
```bash
npm install -g hexo
```

**2. 初始化博客目录：**
在电脑上随便创建一个文件夹（比如叫 `Hexo`），将 Git Bash 切换到该目录下，输入：
```bash
hexo init
```
它会在这个文件夹中初始化静态网站需要的所有文件。安装好的目录结构如图：

![Hexo 目录结构](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%8D%9A%E5%AE%A24.png)

`themes` 目录中自带了一个 `landscape`，它是默认主题, 你也可以启动服务来看一下。

#### 选主题

挑选自己喜欢的主题（可以去知乎搜搜推荐）。我选择的是 **[NexT](https://github.com/iissnan/hexo-theme-next)**。这个主题以黑白两色搭配，做了大量留白，看起来就是：简单！简单！！简单！！！

在 Hexo 根目录下执行克隆：
```bash
git clone https://github.com/iissnan/hexo-theme-next.git themes/hexo-theme-next
```

克隆完毕后，打开 Hexo 根目录下的 `_config.yml`（这是**主站配置文件**），将 `theme` 属性修改为你的主题名字：
```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: hexo-theme-next
```

同时，在 `themes/hexo-theme-next` 目录下也有一个 `_config.yml`，我们把它称作**主题配置文件**，主要用于定制外观和功能。

![主题配置](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%8D%9A%E5%AE%A25.png)

#### 本地预览

在 Git Bash 中输入：
```bash
hexo g   # generate，生成静态网页
hexo s   # server，启动本地服务
```

看到 `Hexo is running at http://localhost:4000/` 的提示后，在浏览器中访问 `http://localhost:4000/` 就可以看到你的博客了。

到此，博客骨架已经搭建完毕。

之后就需要花些时间来研究和定制 NexT 主题，主题的设置很多很多, 我配置了一部分，懒得写了～

最好参考 [NexT 官方文档](http://theme-next.iissnan.com/getting-started.html)）。

---

### 四、日常操作与踩坑记录

主题配置完啦，你也满意了。接下来的就是日常的写文章和部署操作。常用命令总结如下：

```bash
hexo new page "pageName"   # 新建独立页面
hexo new "blogName"        # 新建文章 (简写: hexo n)
hexo p                     #  简写 hexo publish
hexo generate              # 生成静态网页 (简写: hexo g)
hexo server                # 启动本地服务 (简写: hexo s)
hexo deploy                # 部署到远程仓库 (简写: hexo d)
```

> hexo命令详细的可以参考: <https://segmentfault.com/a/1190000002632530>

新建的页面默认是 `.md`（Markdown）格式，这也是我非常喜欢的书写方式。你只需要关注内容，而不用过分关心排版，花几分钟就能掌握基础语法。如果觉得局限，它也完全兼容 HTML 标签。

#### 问题记录：部署报错
在执行 `hexo deploy` 部署到 GitHub 时，可能会遇到如下错误：
```text
ERROR Deployer not found: git
```
**解决办法**：
1. 安装 git 部署插件：
   ```bash
   npm install hexo-deployer-git --save
   ```
2. 检查主站 `_config.yml` 文件中的 `deploy` 配置，确保 `type` 和冒号后面有空格：
   ```yaml
   deploy:
     type: git
     repo: git@github.com:你的用户名/你的用户名.github.io.git
     branch: master
   ```

---

### 五、个性化域名绑定

当调试好并通过 `hexo deploy` 部署后，你就可以通过 `thxiex.github.io` 这样的二级域名访问博客了。
如果你想用一个专属域名（如 `www.xxx.com`）来访问，就需要自己购买域名。

我是在 [GoDaddy](https://sg.godaddy.com/) 上购买的，支持支付宝。大致步骤就是搜索域名、选择购买项、填个人信息，验证。

购买完毕后登录godaddy, 进入my product页面. 可以看到你购买的域名信息.
![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%8D%9A%E5%AE%A26.png)

购买完毕后进入 GoDaddy 的 DNS 管理页面，配置解析记录：

![DNS 解析](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E5%8D%9A%E5%AE%A27.png)

1. **A 记录配置**：
   指向 GitHub Pages 的服务器 IP： `192.30.252.153` 和 `192.30.252.154`。
   
   > **提示 (时效性更新)**：GitHub Pages 的 A 记录 IP 随时间可能会发生变更（比如目前常见的有 `185.199.108.153` 等 4 个 IP）。建议在配置前查阅 [GitHub Pages 官方最新文档](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)，或者直接使用 `CNAME` 记录将你的域名解析到 `用户名.github.io`，这样更为稳妥。

2. **绑定仓库**：
   在你的 GitHub 博客仓库的根目录下，新建一个名为 `CNAME` 的文件（没有任何后缀），里面只写一行你购买的域名：
   ```text
   www.xxx.com
   ```

等 DNS 解析生效后（可能需要几十分钟），访问你的域名，就能看到你的独立博客了！

虽然静态博客托管在 GitHub 上的访问速度可能不如国内的 CSDN，但现在很多平台支持智能 DNS 路由，速度已经有了不小的提升。折腾完毕，享受写作吧！