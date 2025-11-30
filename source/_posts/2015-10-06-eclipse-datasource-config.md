---
title: 问题排查在 eclipse 下配置 dbcp
date: 2015-10-06 00:00:00
categories:
  - 工具
tags:
  - Eclipse
  - Java
  - Archived
excerpt: "记录在 Eclipse 中通过 JNDI 方式配置 Tomcat DBCP 数据库连接池的步骤及问题排查经验。"
aiSummary: "DBCP（Database Connection Pool）是 Apache 提供的数据库连接池实现。本文记录了使用 JNDI 方式在 Eclipse + Tomcat 环境下配置 DBCP 数据源的完整步骤：Oracle 驱动拷贝到 Tomcat lib 目录、web.xml 资源引用配置、context.xml 数据源配置（DriverClassName、Url、Username、Password、连接池参数）、以及在代码中通过 InitialContext 查找数据源的方法。同时总结了配置过程中可能遇到的问题及排查思路。"
---


## 配置

**使用 JNDI 的方式配置 Tomcat 数据源。**


我的步骤如下: 

<!-- more -->

1. 将 Oracle 的驱动拷贝到 Tomcat 的 `lib` 目录下

2. 配置 `web.xml` 文件（这一步尝试了一下不写也暂未出现异常，原因未知…）

   ```xml
   <resource-ref>  
     <description>dbcp_drp</description>  
     <!--数据源名称, 要和context.xml中的数据源名称一致-->  
     <res-ref-name>jdbc/drp</res-ref-name>    
     <res-type>javax.sql.DataSource</res-type>  
     <res-auth>Container</res-auth>    
   </resource-ref>
   ```

3. 配置 `context.xml` 文件

   修改了该文件的内容, Tomcat 就会自动装载该应用.
   为了满足每个应用的单独配置, 所以我是在项目的 `META-INF` 中配置的

   配置内容如下:

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>  
   <Context>  
   <Resource  
         name="jdbc/drp"  
         type="javax.sql.DataSource"  
         password="***"  
         driverClassName="oracle.jdbc.driver.OracleDriver"  
         maxIdle="2"  
         maxWait="5000"  
         username="***"  
         url="jdbc:oracle:thin:@localhost:1521:orcl"  
         maxActive="4"/>  
   </Context>
   ```

   可以使用 Tomcat 管理页面配置, 但是个人感觉太麻烦…

4. 在 JSP 中的 Java 代码测试

   ```java
   Connection conn = null;  
   Context ctx = new InitialContext();  
   //通过JNDI查找DataSource  
   DataSource ds = (DataSource)ctx.lookup("java:comp/env/jdbc/drp");  
   conn = ds.getConnection();
   ```

按理说, 这样就得到连接啦…

但是在 Eclipse 下启动 Tomcat 时, 却出现了几个问题.. 花了几小时去找原因.



## 遇到问题

### 问题一：Tomcat 无法启动

```text
[SetPropertiesRule]{Server/Service/Engine/Host/Context} Setting property ‘source’ to ‘org.eclipse.jst.jee.server:aa’ did not find a matching property
……………
……………
```

这导致 Tomcat 根本没法启动

大致阅读, 发现有可能是 `server.xml` 配置中的 `source` 属性的问题

于是我进入 `tomcat_home/conf/server.xml` 查看, 确实自动加了一段代码标签

然后我尝试删除这段代码, 重新启动tomcat.

发现能启动了, 但却出现另一个严重问题:

### 问题二：JNDI DataSource 读取失败

```text
Cannot create JDBC driver of class ‘’ for connect URL ‘null’
………
………
```

为什么 JDBC 的 DriverClass 和 url 都为空, 明明在 `META-INF` 中都配置好了啊, 为什么没取到?

还是 JNDI 数据源读取出现错误?

于是我怀疑他读到的是 `tomcat_home/conf/context.xml` 这个, 而不是我应用中的 `context.xml`

于是将 `tomcat_home/conf/` 下的 `context.xml` 删除, 发现还是不行

显然, 这种解决方式很愚蠢.

于是, 我去寻找问题的原因.

最后找到原因如下:

在 Eclipse 下, 启动 Tomcat 时, Tomcat 的配置文件 `conf/server.xml` 中会自动生成一个关于该 Web 工程的配置项信息. 类似:

```xml
docBase="" path="" reloadable="" source=""/>
```

而默认情况下, server.xml的Context元素不支持名称为source的属性, 所以产生了警告.



## 解决办法

点开 Eclipse 下配置好的 Tomcat, 在打开的页面找到 server option 选项

选中 `Publish module contexts to separate XML files` 选项即可, 就是将 Context 部分放到一个单独的文件中.

此时, 再修改 `META-INF` 中的 `context.xml` 配置

`tomcat_home/conf` 下的 `server.xml` 中就不会再出现什么 source 属性了.

警告也消失了!
