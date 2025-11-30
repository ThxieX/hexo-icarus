---
title: Eclipse 几个常用设置
date: 2015-06-15 00:00:00
categories:
  - 工具
tags:
  - Eclipse
  - IDE
  - Archived
excerpt: "记录 Eclipse 常用设置：关闭拼写检查、语法提示调整、代码字体修改、护眼背景色（豆沙绿）配置等。"
aiSummary: "本文总结了 Eclipse IDE 的常用设置，包括：关闭红色波浪线拼写检查、去掉语法错误提示、调整代码字体大小和样式、配置护眼背景色（豆沙绿方案）、设置文件编码为 UTF-8、配置代码模板、更改控制台输出编码等。是提升 Eclipse 使用体验的实用配置指南。"
---

总结一些 Eclipse 常用的设置，不设置不好用

<!-- more -->



### 常用设置清单

- **去掉红色波浪线的拼写检查**：
  路径：`Windows > Preferences > General > Editors > Text Editors > Spelling`
  操作：将 **Enable spell checking** 取消勾选即可。

- **去掉恼人的语法提示**：
  路径：`Windows > Preferences > General > Editors > Text Editors > Annotations > Errors`
  操作：将 **Show in** 下面的三个选项都去掉。

- **调整代码字体**：
  路径：`Windows > Preferences > General > Appearance > Colors and Fonts > Basic > Text Font`

- **调整护眼背景色（豆沙绿）**：
  路径：`Windows > Preferences > General > Editors > Text Editors`
  操作：找到底部的 **Background color**，取消系统默认，自定义颜色选择：色调 85，饱和度 90，亮度 205（经典的护眼豆沙绿）。

- **去掉自动添加括号**：
  路径：`Windows > Preferences > Java > Editor > Content Assist`
  操作：把 **Insert single proposals automatically** 前面的勾去掉。

- **修改 Tomcat 部署位置**：
  在 Servers 面板双击 Tomcat 服务器，打开配置页找到 **Server Locations**：
  操作：选择第二项 **Use Tomcat installation (takes control of Tomcat installation)**。
  下面的 Deploy path 可以改为 Tomcat 原生的：`webapps`（如果不改，Eclipse 会默认发布到一个叫 `wtpwebapps` 的深层目录下，很难找）。

- **修改 Tomcat 启动 Timeouts**：
  默认的启动超时时间（45 秒）太小了！如果你的项目在这个时间内还没跑起来，Tomcat 就会直接报错强制停止。
  操作：在 Servers 配置页找到 **Timeouts** 面板，把 Start 的 seconds 改大点（比如 120 或更高）。