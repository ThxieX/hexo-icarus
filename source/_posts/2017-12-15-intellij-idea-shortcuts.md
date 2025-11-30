---
title: IDEA 快捷键 & 插件
date: 2017-12-15 00:00:00
thumbnail: /thumbnails/idea.png
categories:
  - 工具
tags:
  - IDEA
  - IDE
excerpt: "记录 IDEA 常用快捷键（切换导航、查找、重构、代码生成）及常用插件推荐（MyBatis Log Plugin、Lombok、Maven Helper 等）。"
aiSummary: "IDEA（IntelliJ IDEA）是 Java 开发最主流的 IDE，相比 Eclipse 拥有更智能的代码提示和更强大的重构能力。本文总结了 IDEA 的常用快捷键：切换与导航（Ctrl+PgUp/PgDn、Alt+数字）、查找（Ctrl+Shift+A、Ctrl+N）、重构（Ctrl+Alt+Shift+T）、代码生成（Alt+Insert）等。此外还推荐了常用插件：MyBatis Log Plugin（格式化 MyBatis 日志）、Lombok（简化 Java 代码）、Maven Helper（依赖分析）、Grep Console（日志高亮）、Alibaba Java Coding Guidelines（代码规范检查）等。"
---

### 前言

用了一段时间IDEA, 不想再用Eclipse了, 记录一些快捷键和好用插件

<!-- more -->

---

### 一、常用快捷键

#### 1. 切换与导航

- `Ctrl + PgUp / PgDn`：切换打开的文件（Tab 页）。
- `Ctrl + Alt + [ / ]`：在多个打开的项目之间切换。
- `Ctrl + Shift + ↑ / ↓`：在当前文件中，跳转到上一个/下一个方法。
- `Alt + 数字`：快速切换 IDE 边栏窗口（例如 `Alt + 1` 打开 Project 视图，每个窗口边上都有数字标号）。
- `Shift + ESC`：关闭当前焦点所在的边栏窗口。
- `ESC`：从其他窗口迅速跳回代码编辑区。

#### 2. 查找功能

- `Ctrl + Shift + A`：查找 IDE 内部的所有选项/动作（**极其好用，忘记快捷键时直接搜**）。
- `Ctrl + Shift + R`：查找文件（如果要搜索本项目依赖包外的文件，连按 2 下 `R`）。
- `Ctrl + Shift + T`：查找 Class 类（如果要搜索本项目依赖包外的类，连按 2 下 `T`）。
- `Ctrl + H`：全文查找指定字符串。
- `Ctrl + O`：查看当前类的方法列表。

#### 3. 编码与其他

- `Alt + Enter`：万能建议键（报错修复、导包、代码优化等）。
- `Alt + Insert`：快速生成代码（例如 Getter/Setter、构造方法、Override 方法等）。
- `Ctrl + Shift + U`：大小写转换。
- `Ctrl + Alt + S`：打开全局设置（Settings）。
- `Ctrl + Alt + Shift + S`：打开项目设置（Project Structure，包括 Module、SDK 配置等）。

#### 4. 个人自定义快捷键

> **提示**：以下快捷键是我根据个人习惯在 Keymap 中自定义修改的，并非 IDEA 默认快捷键。

- `Ctrl + Q`：跳转到上一个修改的地方。
- `Alt + Q`：跳转到下一个修改的地方。
- `F10`：添加到收藏（Favorites，方法和类都能添加）。
- `F11 / F12`：添加当前代码行到书签（Bookmarks）。
- `Alt + 2`：调出 Favorites 窗口（里面包含书签、收藏列表、断点等）。
- `Alt + J`：配合 AceJump 插件使用的光标快速跳转。

---

### 二、好用插件推荐

IDEA 旗舰版（Ultimate）本身就已经集成了非常多强大的插件，我不想安装太多导致 IDE 卡顿，但以下这几个真的是心头好：

- **Alibaba Java Coding Guidelines**：阿里巴巴代码规范检查插件，强制治好你的代码洁癖。
- **EJS**：因为我的博客是用 Hexo (Node.js) 搭建的，需要这个插件来高亮和格式化 `.ejs` 模板文件。
- **emacsIDEAs**：Emacs 神器，不用多说，键盘流脱离鼠标的利器。
- **Grep Console**：控制台日志插件，可以根据日志级别（Info, Warn, Error）变色输出，很好看也很实用。
- **Lombok Plugin**：如果你在代码里使用了 Lombok 注解（如 `@Data`），IDEA 必须安装这个插件才能正常识别 Getter/Setter，否则会满屏报错。
- **Maven Helper**：解决 Maven 依赖冲突的绝对神器，查看依赖树非常直观。
- **Rainbow Brackets**：彩虹括号。代码括号配对高亮，再也不会被多层嵌套的括号看瞎眼，好看又有趣。
- **Translation**：翻译插件，阅读源码注释和官方文档时的好帮手，支持多种翻译引擎。
