---
title: PowerDesigner 简单使用
date: 2015-04-15 00:00:00
categories:
  - 工具
tags:
  - Database
  - Archived
excerpt: "介绍 PowerDesigner 数据库建模工具的使用，包括 CDM（概念数据模型）和 PDM（物理数据模型）的创建与转换。"
aiSummary: "PowerDesigner 是一款强大的数据库建模工具，与 Rational Rose 并称业界最著名建模软件。本文介绍了 CDM（概念数据模型）和 PDM（物理数据模型）的核心概念：CDM 关注实体、属性和关系，不涉及具体数据库；PDM 则与具体数据库系统绑定，将实体转为表、属性转为列并指定字段类型。文章还涵盖了如何通过 PowerDesigner 设计数据库表结构、生成 DDL 脚本和导出数据字典的实操流程。"
---

> **声明**：部分内容引言转自 [博客园 - langtianya 的博客](http://www.cnblogs.com/langtianya/archive/2013/03/08/2949118.html)。

### 简介

PowerDesigner 是一款功能非常强大的建模工具软件，足以与 Rational Rose 比肩，同样是当今最著名的建模软件之一。

Rose 是专攻 UML 对象模型的建模工具，之后才向数据库建模发展；而 PowerDesigner 则与其正好相反，它是以数据库建模起家，后来才发展为一款综合全面的 CASE 工具。对于后端开发者来说，它几乎是设计数据库表结构、生成 DDL 脚本和导出数据字典的必备神器。

<!-- more -->

---

### 核心概念模型

在使用 PowerDesigner 设计数据库时，我们最常打交道的是以下两种模型：

1. **CDM (Conceptual Data Model) - 概念数据模型**
   主要用于数据库的逻辑设计。在 CDM 中，我们只关心实体（Entity）、属性（Attribute）以及实体之间的关系（如一对多、多对多），而**不需要**考虑具体使用的是哪种数据库（MySQL、Oracle 还是 SQL Server）。

2. **PDM (Physical Data Model) - 物理数据模型**
   这是与具体数据库系统强绑定的模型。在 PDM 中，实体会变成表（Table），属性会变成列（Column），并且需要指定具体数据库支持的字段类型（如 `VARCHAR2`, `INT` 等）。

---

### 简单使用步骤（基础流）

日常开发中最经典的使用流就是：**建立 CDM -> 转换为 PDM -> 导出 SQL 脚本**。

#### 1. 创建概念数据模型 (CDM)
1. 打开 PowerDesigner，点击 `File > New Model`。
2. 在弹出的对话框中，选择 `Model types > Conceptual Data Model`，命名后点击 OK。
3. 在右侧的工具面板（Palette）中，点击 `Entity`（实体）图标，然后在空白画布上点击，即可创建实体。
4. 双击实体，可以设置实体的 `Name`（中文名称，用于显示）和 `Code`（英文代号，最终会变成数据库表名）。
5. 切换到 `Attributes` 选项卡，添加属性，并勾选 `P` (Primary Key)、`M` (Mandatory/必填) 等约束。

#### 2. 将 CDM 转换为 PDM
当我们把所有实体和关联关系画好后，就可以将其转换为物理模型了。
1. 在顶部菜单栏点击 `Tools > Generate Physical Data Model...`（快捷键：`Ctrl + Shift + P`）。
2. 在弹出的窗口中，选择你要生成的目标数据库类型（DBMS），例如 `MySQL 5.0` 或 `Oracle 11g`。
3. 点击 OK，PowerDesigner 会自动生成一个对应的 PDM 文件。此时你可以看到，所有的实体变成了表结构，数据类型也自动映射为了对应数据库的类型。

#### 3. 导出 SQL 建表脚本
这是最后也是最爽的一步。
1. 在生成的 PDM 视图下，点击顶部菜单 `Database > Generate Database...`（快捷键：`Ctrl + G`）。
2. 选择你要导出 SQL 文件的保存路径。
3. 可以在 `Format` 和 `Options` 选项卡中调整一些导出细节（例如是否导出 Drop Table 语句，是否导出外键等）。
4. 点击 OK，一份完美的 DDL 建表脚本就生成了，直接拿到数据库里执行即可。

> **小技巧**：在团队协作中，我们经常需要把设计好的表结构导出为 Excel 或 Word 格式的“数据字典”。PowerDesigner 提供了强大的 `Report` 功能，可以通过自定义模板一键生成非常专业的数据库设计文档。