---
title: Oracle 错误排查
date: 2013-12-22
categories:
  - Data
tags:
  - Oracle
  - Database
  - Archived
excerpt: "排查 Oracle 数据库 ORA-01034、ORA-27101 错误，通过启动监听服务、创建数据库实例、启动 SQL*Plus 解决 Oracle not available 问题。"
aiSummary: "ORA-01034 和 ORA-27101 是 Oracle 数据库连接时的典型错误，表明 Oracle 实例未正常启动。本文记录了完整的排查与解决流程：首先使用 lsnrctl start 启动监听服务；其次使用 dbca 或手工方式创建数据库实例（init.ora、create database）；最后通过 SQL*Plus 以 sysdba 身份 startup 启动实例。文章详细给出了每一步的命令和可能出现的问题点，是 Oracle 数据库故障排查的实用参考。"
---

> **前言**：这两天使用 Oracle 遇到了很多问题，大都比较容易解决。这里记录一个典型的启动连接报错及其排查过程，当个备忘录。

<!-- more -->

### 一、问题现象

在尝试连接 Oracle 数据库时，控制台抛出如下经典错误：

```text
ERROR: ORA-01034: ORACLE not available 
ORA-27101: shared memory realm does not exist
```

### 二、排查与解决步骤

出现该错误的核心原因是 Oracle 实例未正常启动或共享内存未分配。可按照以下步骤进行手动恢复：

**1. 检查并启动监听服务**
首先确认 Oracle 的监听和服务是否都已启动。打开 cmd 命令行窗口，输入以下命令启动监听：
```cmd
lsnrctl start
```

**2. 手工设置 Oracle SID**
查看你的 Oracle 实例 SID 叫什么（比如创建数据库时实例名叫 `abc`），在命令行中先手工设置一下环境变量：
```cmd
set ORACLE_SID=abc
```

**3. 以 DBA 身份登录 SQL*Plus**
继续在当前命令行窗口中，依次执行以下命令：
```cmd
sqlplus /nolog
conn / as sysdba
```

**4. 重启数据库实例**
连接成功后，输入启动命令：
```sql
startup
```
> **提示**：如果输入 `startup` 后系统提示“服务已经启动”，说明之前的状态可能是假死或挂起。此时可以先强制关闭再启动：
> ```sql
> shutdown immediate
> startup
> ```

**5. 验证连接**
等待几秒钟让命令运行完成。此时数据库应该已经恢复连接。可以输入一条测试 SQL 看看是否有查询结果：
```sql
select count(*) from user_tables;
```

---

### 三、总结分析

出现 `ORA-01034` 和 `ORA-27101` 的原因是多方面的，但根本原因通常是 **Oracle 当前的服务不可用**。

报错中提到的 `shared memory realm does not exist`，是因为 Oracle 实例没有启动（或没有正常启动），导致共享内存（SGA）并没有分配给当前实例。

因此，解决该问题的核心思路就是：通过手工设置实例名（SID），利用操作系统身份验证（`/ as sysdba`）绕过常规登录，然后强制重新启动数据库实例。这样数据库就能重新分配内存并正常启动，这两个异常也就随之消除了。