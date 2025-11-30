---
title: Activiti 初探
date: 2017-11-15
categories:
  - Backend Engineering
tags:
  - Java
  - 工作流
excerpt: "Activiti 是基于 Java 的开源工作流引擎，实现 BPMN 2.0 规范，介绍环境搭建、流程部署、启动和查询等核心操作。"
aiSummary: "Activiti 是 Apache 旗下的开源工作流引擎，实现了 BPMN 2.0 规范。本文是 Activiti 入门指南，介绍了工作流的基本概念、与 BPMN 2.0 规范的关系，以及 Activiti 的下载安装和开发环境配置。通过一个完整的请假流程示例，详细演示了流程定义文件（BPMN XML）的编写、流程引擎 API 的使用（流程部署、启动、查询、审批等），是 Java 开发者入门工作流技术的实用参考。"
---

### 前言

Activiti 是一个基于 Java 的开源工作流引擎，实现了 BPMN 2.0 规范，提供了流程的定义、部署和调度功能。

工作流（Workflow）的概念最初听起来可能有些抽象。如果接触过 Git Workflow，可以类比理解：其应用的本质是使用 Git 来进行有效的代码流程管理和高效的开发协同约定。

关于工作流与 BPMN 2.0 规范的理论基础，可以参考以下资料：

* [工作流（Workflow）基础](http://blog.csdn.net/u014682573/article/details/29922093)
* [BPMN 2.0 规范](http://www.mossle.com/docs/jbpm4devguide/html/bpmn2.html)

<!-- more -->

### 准备

**下载 Activiti**

官网：http://activiti.org/download.html

下载解压后的目录结构：

* `database`：提供一些数据库脚本。
* `libs`：Activiti 所需要的 jar 包和源码包。
* `wars`：官方提供的一些 Demo，可以部署到 Tomcat 下运行体验。

**安装 Activiti Eclipse 设计器**

在 Eclipse 中依次点击 `Help -> Install New Software -> Add`：

> Name: Activiti BPMN 2.0 designer
> 
> Location: http://activiti.org/designer/update/
> 也支持离线安装方式

此外，还需要 JDK 6+ 的环境，Activiti 支持多种主流数据库环境。

### 了解 ProcessEngine

ProcessEngine 是流程引擎对象，所有的核心操作都依赖它。

了解 ProcessEngine 最好的方式是在项目中建立 Activiti Project，引入相关 jar 包，然后编写 JUnit 测试来验证其工作方式。

#### 初始化数据库

整个流程引擎需要数据库表的支持，不同版本可能表数量不同（例如 Activiti 6.0.0 版本需要 28 张表）。

初始化数据库的方式主要有以下几种：

1. **执行脚本方式**：可在 `activiti-6.0.0/database` 目录下手动执行初始化脚本。其中包括了建立数据库表和约束。
2. **代码方式**：使用 ProcessEngine 来创建，代码如下：

```java
@Test
public void createTable1() {
    ProcessEngineConfiguration processEngineConfiguration = ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
    processEngineConfiguration.setJdbcDriver("com.mysql.jdbc.Driver");
    processEngineConfiguration.setJdbcUrl("jdbc:mysql://192.168.118.128:3306/activiti_test?useUnicode=true&characterEncoding=utf8");
    processEngineConfiguration.setJdbcUsername("root");
    processEngineConfiguration.setJdbcPassword("root");
    processEngineConfiguration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
    
    ProcessEngine processEngine = processEngineConfiguration.buildProcessEngine();
    System.out.println("processEngine1: " + processEngine);
}
```

工作流引擎配置也可以通过配置文件 `activiti.cfg.xml` 读取：

```java
@Test
public void createTable2() {
    System.out.println("init table");
    ProcessEngineConfiguration processEngineConfiguration = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("activiti.cfg.xml");
    ProcessEngine processEngine = processEngineConfiguration.buildProcessEngine();
    System.out.println("processEngine2: " + processEngine);
}
```

或者使用更简单的默认方式获取 ProcessEngine：

```java
/**
 * getDefaultProcessEngine() 会调用 init 方法，
 * 自动读取 classpath 下的 activiti.cfg.xml 文件来初始化数据库
 */
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
```

对应的 `activiti.cfg.xml` 配置文件示例：

```xml
<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    <property name="jdbcDriver" value="com.mysql.jdbc.Driver" />
    <property name="jdbcUrl" value="jdbc:mysql://192.168.118.128:3306/activiti_test?useUnicode=true&amp;characterEncoding=utf8" />
    <property name="jdbcUsername" value="root" />
    <property name="jdbcPassword" value="root" />
    <property name="databaseSchemaUpdate" value="true" />
</bean>  
```

初始化完成后，查看数据库会看到以 `ACT_` 开头的表。

#### 核心数据库表解析

表名的开头标识了该表的业务含义：

* **RE (Repository)**：该类表包含流程定义和流程静态资源。
* **RU (Runtime)**：该类表包含流程实例、任务、变量、异步任务等运行中的数据。
* **ID (Identity)**：该类表包含身份信息，比如用户、组。
* **HI (History)**：该类表包含历史数据，比如历史流程实例、变量、任务等。
* **GE (General)**：表示通用数据，用于不同场景下，如存放资源文件。

```sql
-- 部署对象与流程定义相关
SELECT * FROM act_re_deployment;  -- 部署对象表
SELECT * FROM act_re_procdef;     -- 流程定义表
SELECT * FROM act_ge_bytearray;   -- 资源文件表
SELECT * FROM act_ge_property;    -- 主键生成策略表

-- 流程实例、执行对象与任务
SELECT * FROM act_ru_execution;   -- 正在执行的执行对象表
SELECT * FROM act_hi_procinst;    -- 流程实例的历史表
SELECT * FROM act_ru_task;        -- 正在执行的任务表（只有节点是 UserTask 时，该表中存在数据）
SELECT * FROM act_hi_taskinst;    -- 任务历史表（只有节点是 UserTask 时，该表中存在数据）
SELECT * FROM act_hi_actinst;     -- 所有活动节点的历史表

-- 流程变量
SELECT * FROM act_ru_variable;    -- 正在执行的流程变量表
SELECT * FROM act_hi_varinst;     -- 历史的流程变量表
```

#### 核心对象模型

* **ProcessDefinition**：流程定义对象，可以从中获取到资源文件。
* **ProcessInstance**：流程实例对象，代表流程定义的执行实例。一个流程实例包括所有运行节点，可以利用该对象了解当前流程实例的进度等信息。它表示一个流程从开始到结束的最大流程分支（一个流程中的流程实例只有一个）。
* **Execution**：执行对象。表示按照流程定义规则执行一次的过程。Activiti 用这个对象去描述流程执行的每一个节点。在没有并发的情况下，Execution 就等同于 ProcessInstance。
  * 对应表：`ACT_RU_EXECUTION`, `ACT_HI_PROCINST`, `ACT_HI_ACTINST`
  * 关系：`ProcessInstance extends Execution`

**总结**：
* 一个流程中，执行对象可以存在多个，但是流程实例只能有一个。
* 当流程按照规则只执行一次的时候，流程实例就是执行对象。

* **Task**：任务。执行到某任务环节时生成的任务信息。
  * 对应表：`ACT_RU_TASK`, `ACT_HI_TASKINST`

#### 核心 Service 组件

* **RepositoryService**：管理流程定义及部署。
* **RuntimeService**：执行管理，包括启动、推进、删除流程实例等操作。
* **TaskService**：任务管理。
* **HistoryService**：历史管理（执行完的数据的管理与查询）。
* **IdentityService**：组织机构管理（用户与组）。
* **FormService**：可选服务，用于任务表单管理。
* **ManagementService**：引擎管理服务，主要用于读取数据库配置、表状态及 Job 管理。
