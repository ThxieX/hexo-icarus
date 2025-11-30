---
title: 明日黄花 Struts2
date: 2015-10-19 00:00:00
categories:
  - Backend Engineering
tags:
  - Java
  - Struts2
  - Archived
excerpt: "搭建 Struts2 开发环境，通过登录示例快速跑通 Struts2 MVC 最小示例，了解其核心工作原理。"
aiSummary: "本文记录了 Struts2 框架的学习过程，通过一个登录小示例帮助读者快速跑通 Struts2 的最小开发环境。文章涵盖了 Java Web 项目创建、Struts2 依赖包引入（commons-logging、freemarker、ognl、struts2-core 等）、web.xml 配置过滤器、Action/Service/DAO 三层编写、Struts.xml 配置及 JSP 页面实现的完整流程，并简要分析了 Struts2 的核心工作原理。"
---


虽然 Struts2 已成为明日黄花，但还是决定来学习一下，体会设计思想，实用为辅。


<!-- more -->


## Struts2 登录（跑通最小示例）

> 需要注意：本文以 Struts2 2.1.x 的依赖版本为例，该版本通常要求 JRE 1.5+。更高版本可能要求更高的 JDK。

### 1. 创建 Java Web 项目

### 2. 引入 Struts2 依赖包

将依赖包放在 `WEB-INF/lib` 目录下，例如：

- commons-logging-1.0.4.jar
- freemarker-2.3.15.jar
- ognl-2.7.3.jar
- struts2-core-2.1.8.1.jar
- xwork-core-2.1.6.jar
- commons-fileupload-1.2.1.jar

### 3. 配置 Struts2 Filter

在 `web.xml` 配置 `StrutsPrepareAndExecuteFilter`。

> `FilterDispatcher` 属于早期写法，后续版本中已不推荐使用。

```xml
<filter>  
    <filter-name>struts2</filter-name>  
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>  
</filter>  
  
<filter-mapping>  
    <filter-name>struts2</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

### 4. 提供 Struts2 配置文件

提供 `struts.xml`，放在 `src` 下（使其在 classpath 中可加载）。

### 5. 建立 JSP

例如：`login.jsp`、`login_success.jsp`、`login_error.jsp`。

### 6. 创建 Action

Struts2 的 Action 可以不用继承框架中的任意类，也不用实现框架中的任何接口，因此 Action 可以是一个 POJO（纯粹的 Java 对象），测试更容易。

Struts2 缺省方法名称：

```java
public String execute() throws Exception
```

### 7. 通过 getter/setter 收集数据

在 Action 中提供 getter 和 setter 便于收集数据（一般称为属性驱动模式）。

### 8. 在 struts.xml 中配置 Action 与 Result

### 9. 了解默认配置

了解 `struts-default.xml` 与 `default.properties`。Struts2 的默认后缀是 `.action`。

- struts2-core-2.1.8.1.jar/struts-default.xml
- struts2-core-2.1.8.1.jar/org.apache.struts2/default.properties



## Struts2 常用配置
#### Struts2的常用配置
1. `result` 标签的 `name` 属性，如果不配置，缺省为 `success`。
2. Struts2 提供了一个 `Action` 接口，在接口中定义了一些常量和 `execute()` 方法，实现它可以让开发更规范（但不是必须）。
3. 常用配置参数（可在 `default.properties` 中找到）：

- `struts.configuration.xml.reload=true`：当 `struts.xml` 发生更改时立刻加载（开发环境可用，生产环境不建议开启）。
- `struts.devMode=true`：提供更友好的错误提示（开发环境可用，生产环境不建议开启）。

配置方式通常有两种：

- 在 `struts.properties` 中配置
- 在 `struts.xml` 中通过 `<constant />` 配置（更常见）


#### Struts2对团队开发的支持(多配置文件)
## 团队开发：多配置文件
1 可以为某个模块建立单独的配置文件, 该配置文件的格式需要和struts.xml文件的格式一样
1. 可以为某个模块建立单独的配置文件，格式与 `struts.xml` 一致。
2. 在 `struts.xml` 中通过 `<include />` 引入。


#### Struts2对ModelDriven模式的支持
## ModelDriven 模式
Struts2可以采用类似于Struts1中的ActionForm方式收集数据, 这样的方式叫做ModelDriven模式
Struts2 可以采用类似于 Struts1 中 ActionForm 的方式收集数据，这种方式叫做 ModelDriven 模式。
**如何实现模型驱动模式?**

- 创建User
- Action需要实现ModelDriven接口
- 实现getModel()方法, 返回Bean对象



#### Struts2中直接对Action中的对象进行赋值(不new)
## 直接对 Action 中的对象赋值（不 new）
在html中可以通过如下方式命名输入域:
在 HTML 中可以通过如下方式命名输入域：
```html
<form action="login.action">  
用户：<input type="text" name="user.username"><br>  
密码：<input type="password" name="user.password"><br>  
<input type="submit" value="登录">  
</form>
```



#### Struts2的Action访问Servlet API
## Action 访问 Servlet API
**方式一:**
### 方式一：ActionContext（相对无侵入）

可以通过 `ActionContext` 访问 Servlet API，此种方式侵入性较低。
2 在Struts2中默认为转发，也就是标签中的type=”dispatcher”，type的属性可以修改为重定向. Struts的重定向有两种：
在 Struts2 中默认是转发，即 `result` 标签中的 `type="dispatcher"`。`type` 可以修改为重定向，常见有两种：
- type=”redirect”,可以重定向到任何一个web资源，如：jsp或Action 如果要重定向到Action，需要写上后缀：xxxx.action
- `type="redirect"`：可以重定向到任何 web 资源（JSP 或 Action）。如果重定向到 Action，需要写后缀：`xxxx.action`。
- `type="redirectAction"`：重定向到 Action，不需要写后缀。后缀改变时不影响配置，相对更通用。
3 关于Struts2的type类型，也就是Result类型，他们都实现了共同的接口Result，都实现了execute方法
关于 Struts2 的 `type`（也就是 Result 类型），它们都实现了共同的 `Result` 接口，并实现 `execute()` 方法。
他们体现了策略模式，具体Result类型参见：struts-default.xml文件：
这体现了策略模式。具体 Result 类型可参见 `struts-default.xml`：
```xml
<result-types>  
   <result-type name="chain" class="com.opensymphony.xwork2.ActionChainResult"/>  
   <result-type name="dispatcher" class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>  
   <result-type name="freemarker" class="org.apache.struts2.views.freemarker.FreemarkerResult"/>  
   <result-type name="freemarker" class="org.apache.struts2.views.freemarker.FreemarkerResult"/>  
   <result-type name="httpheader" class="org.apache.struts2.dispatcher.HttpHeaderResult"/>  
   <result-type name="redirect" class="org.apache.struts2.dispatcher.ServletRedirectResult"/>  
   <result-type name="redirectAction" class="org.apache.struts2.dispatcher.ServletActionRedirectResult"/>  
   <result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>  
   <result-type name="velocity" class="org.apache.struts2.dispatcher.VelocityResult"/>  
   <result-type name="xslt" class="org.apache.struts2.views.xslt.XSLTResult"/>  
   <result-type name="plainText" class="org.apache.struts2.dispatcher.PlainTextResult" />  
</result-types>
```

我们也可以根据需求扩展 Result 类型。

### 方式二：实现 Aware 接口（装配注入）

可以通过实现装配接口完成对 Servlet API 的访问：

- `ServletRequestAware`：取得 `HttpServletRequest` 对象
- `ServletResponseAware`：取得 `HttpServletResponse` 对象
- `ServletContextAware`：取得 `ServletContext` 对象

### 方式三：ServletActionContext（静态方法）

可以通过 `ServletActionContext` 提供的静态方法取得 Servlet API：

- `getPageContext()`
- `getRequest()`
- `getResponse()`
- `getServletContext()`



## 命名空间（namespace）

- 采用命名空间, 可以区分不同包下的相同Action名称.
- 如果package的namespace属性没有设定, 使用默认的命名空间为””.
- Struts2中Action的完整路径为: namespace + Action的名称.
- 寻找原则: 首先在指定的命名空间下寻找Action,　如果找到了就使用此Action, 如果没有找到, 就在上层目录中查找, 一直到根(缺省命名空间), 找到则使用此Action, 没找到就抛出Action没找到异常.



## 字符集设置

配置方式通常有几种：

1. `struts.properties`：`struts.i18n.encoding=...`
2. `struts.xml`：

```xml
<constant name="struts.i18n.encoding" value="GB18030"/>
```

3. 在 `StrutsPrepareAndExecuteFilter` 中配置：

```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <init-param>
        <param-name>struts.i18n.encoding</param-name>
        <param-value>GB18030</param-value>
    </init-param>
</filter>
```

注：这个配置通常只针对表单提交为 POST 的场景。如果提交方式为 GET，需要在 `tomcat_home/conf/server.xml` 中 8080 端口处添加 `URIEncoding="GB18030"`。



## Action 包含多个方法时如何调用（动态方法调用）

具体的调用方式：
1. 方法的动态调用
2. 在 `action` 标签中配置 `method` 属性
3. 使用通配符

### 方式一：方法的动态调用

```html
[action名称] + ! + 方法名称 + 后缀  
如: <a href="user!add.action">添加用户</a><br>  
<a href="user!del.action">删除用户</a><br>  
<a href="user!update.action">修改用户</a><br>  
<a href="user!list.action">查询用户</a><br>
```

Action 不用实现 `Action` 接口，只需继承 `ActionSupport`（它实现了 `Action` 接口）。

动态调用的参数配置，默认允许；设为 `false` 表示禁用：

```xml
<constant name="struts.enable.DynamicMethodInvocation" value="false" />
```

Action 中的方法最好和 `execute()` 一致（参数、返回值、异常尽量保持一致）。

### 方式二：配置 method 属性

添加 `method` 属性。JSP 页面要写 `name` 名，而不是 `method` 名。

```xml
     <package name="user-package" extends="struts-default">  
        <action name="addUser" class="com.xie.struts2.UserAction" method="add">  
            <result>/success.jsp</result>  
        </action>  
        <action name="deleteUser" class="com.xie.struts2.UserAction" method="delete">  
            <result>/success.jsp</result>  
        </action>  
        <action name="updateUser" class="com.xie.struts2.UserAction" method="update">  
            <result>/success.jsp</result>  
        </action>  
        <action name="listUser" class="com.xie.struts2.UserAction" method="list">  
            <result>/success.jsp</result>  
        </action>  
    </package>
```

### 方式三：使用通配符

具体配置：

```xml
<action name="*User" class="com.xie.struts2.UserAction" method="{1}">  
<result>/{1}Success.jsp</result>  
</action>
```

在 Struts2 的 `action` 标签中的 `class`、`name`、`method` 和 `result` 等都可以使用通配符。通配符的作用是减少配置量，但需要建立在约定的基础上。



## 上传

1. Struts2 默认采用 Apache Commons FileUpload。
2. Struts2 支持三种类型的上传组件。
3. 需要引入 Commons FileUpload 相关依赖包：

- commons-io-1.3.2.jar
- commons-fileupload-1.2.1.jar

4. 表单中需要采用 POST 提交方式，编码方式需要是 `multipart/form-data`。
5. Struts2 的 Action：

- 取得文件名称: 输入域的名称 + 固定字符串”FileName”
- 取得文件数据: File输入域的名称
- 取得内容类型: 输入域的名称 + 固定字符串”ContentType”

6 得到输入流, 通过输出流写出文件



## 类型转换

**如何实现Struts2的类型转换器?**
1 继承StrutsTypeConverter
2 覆盖convertFromString和convertToString方法

**注册类型转换器:**
这里我们手动编写一个转换日期的转换器.

**局部类型转换器:**
只对当前Action起作用, 需要提供如下配置文件:

> [MyActionName]-conversion.properties
> –[MyActionName]指需要使用转换器的Action名称(不是转换器的名称)

–conversion.properties”固定字符串, 不能修改.

例如: 我们addUserAction类型转换器的配置文件名称应该为: AddUserAction-conversion.properties.
该配置文件一定要和该Action放在同一个目录中, 该配置文件的格式为:Action中的属性名称=转换器的完整路径
如: birthday=com.xie.struts2.UtilDateConverter

**全局类型转换器:**
可以对所有的Action起作用(同Struts1的类型转换器), 需要提供如下配置文件:
xwork-conversion.properties, 该配置文件需要放在src目录下
该配置文件的格式: 需要转换的类型的完整路径=转换器的完整路径.
如: java.util.Date=com.xie.struts2.UtilDateConverter

如果全局类型转换器和局部类型转换器同时存在, 则局部优先.

Struts2 的标签库类似于 JSTL，默认以 `s` 开头。

```jsp
引入: <%@ taglib uri="/struts-tags" prefix="s"%>
读取: <s:property value="birthday" />
```

采用标签读取属性, 才能调用转换器的convertToString方法, 采用JSTL就不会.



## 国际化支持

1 Locale是由国家和语言代码构成的
2 国际化资源文件是由: baseName + locale.properties
3 国际化的内容:

- 在页面中的硬编码
- 动态文本(提示或错误)

4 配置baseName:

```xml
<constant name="struts.custom.i18n.resources" value="Message"/>
```

5 提供国际化资源文件:

- Message.properties
- Message_zh_CN.properties
- Message_en_US.properties

6 在开发阶段, 可以进行如下配置:

当国际化资源文件发生修改, 则立刻加载, 在生产环境下一般不要配置

```xml
<constant name="struts.i18n.reload" value="true"/>
```



## 异常支持（声明式异常 / 自动处理）

局部异常 配置:

```xml
<exception-mapping result="error" exception="com.xie.struts2.ApplicationException" />  
<result name="error">/login_error.jsp</result>
```

全局异常 配置:

```xml
<global-results>  
<result name="global-error">/global_error.jsp</result>  
</global-results>  
<global-exception-mappings>  
<exception-mapping result="global-error" exception="com.xie.struts2.ApplicationException"></exception-mapping>  
</global-exception-mappings>
```

在页面可以使用EL取得异常信息

```
${exception.message} --自定义错误信息
${exceptionStack}    --异常信息堆栈
```



## 拦截器

1 Struts2的拦截器只能拦截Action，拦截器是AOP的一种思路，可以使我们的系统架构
更松散（耦合度低），可以插拔，容易互换，代码不改变的情况下很容易满足客户需求，其实体现了OCP

2 如何实现拦截器？（整个拦截器体现了责任链模式，Filter也体现了责任链模式）

- 继承AbstractInterceptor（体现了缺省适配器模式）
- 实现Interceptor

3 如果自定了拦截器，缺省拦截器会失效，必须显示引用Struts2默认的拦截器

4 拦截器栈，多个拦截器的和

5 定义缺省拦截器,所有的Action都会使用

6 拦截器的执行原理，在ActionInvocation中有一个成员变量Iterator，这个Iterator中保存了所有拦截器，
每次都会取得Iterator进行next,如果找到了拦截器就会执行，否则就执行Action，都执行完了
拦截器出栈（其实出栈就是拦截器的intercept方法弹出栈）
