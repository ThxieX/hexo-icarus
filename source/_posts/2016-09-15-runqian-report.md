---
title: 润乾报表的使用
date: 2016-09-15 00:00:00
categories:
  - 工具
tags:
  - 报表
  - Java
excerpt: "使用润乾报表进行 Java Web 报表开发，介绍报表设计器的安装、配置及与 Spring MVC 框架的集成方法。"
aiSummary: "润乾报表是一款纯 Java 的报表开发工具，采用设计与集成分离的开发模式。本文记录了润乾报表的使用方法：报表设计器安装配置（内置 Demo 和 Tomcat 预览）、报表设计界面介绍（类似 Excel 的表格设计）、数据源配置、表达式编写、图表绑定、报表发布，以及与 Spring MVC 框架集成的 Java API 调用。支持 HTML、Excel、PDF 等多种导出格式，是 Java Web 项目中报表模块开发的实用参考。"
---


## 前言

前段时间项目用到了报表开发, 选择润乾报表来做. 它是一款纯 Java 的报表开发工具, 采用设计和集成分离的方式, 开发过程也相对简单.


<!-- more -->

## 介绍

设计时需要安装一个报表设计器, 设计过程类似于 Excel 的表格设计. 支持 HTML、Excel、PDF 等多种展现方式, 也支持导出 DOC、Excel、PDF 和打印.

内置一个数据库 demo, 提供了很多报表类型的例子与较丰富的教学文档, 并且内置 Tomcat, 方便在设计过程中直接发布到网页预览.

## 安装

安装过程也较为简单, 不再赘述.

需要注意的是: 

- JDK: 安装中可以选择内置的运行环境（例如 1.5）, 也可以自定义选择本机安装的 JDK. 考虑到设计过程中运行环境差异带来的兼容问题, 可以先使用内置运行环境进行设计与预览, 最终部署以项目运行时的 JDK 为准.

> 授权: 在设计器打开和发布预览中通常都需要授权. 建议通过官方渠道购买/申请试用, 避免合规与稳定性风险.

安装完成打开设计器如图:
![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E6%B6%A6%E4%B9%BE%E6%8A%A5%E8%A1%A81.png)

## 设计

设计过程一般分为: 绘制表图、填写字段和表达式、配置数据源、定义数据集与参数、集成项目等步骤.

#### 1. 绘制表图

为了使最后展现给用户的报表比较满意, 我们可能需要进行显示格式, 段落, 缩进, 字体的设置. 以及最后的打印, 分页都可以在报表属性中设置.

#### 2. 填写字段和表达式

前两个过程较为简单, 我用程序提供的小例子. 效果如下:
![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E6%B6%A6%E4%B9%BE%E6%8A%A5%E8%A1%A82.png)

在A2单元格中, 相当于一个游标, 来纵向扩展数据. 当然也支持横向扩展, 支持自己选择.

在做分组报表和卡式报表时候, 有一个左/上主格和附属格(子格)的概念:

- 主格: 原来的扩展单元格.
- 附属格: 跟随主格扩展的单元格
- 左主格: 只有纵向扩展才有的概念, 右边的单元格跟随左边的单元格进行纵向扩展, 这个左边被跟随的单元格就是左主格. 其他跟随的单元格叫做附属格.
- 上主格: 和左主格概念类似, 只不过是在横向扩展中, 称下边附属格上面的即为上主格.



#### 3. 配置数据源

默认的数据源是润乾自带的demo数据库, 可供我们学习和参考. 

在实际设计中, 我们通常需要配置自己的数据源, 在新建中选择驱动, 填写数据源URL, 口令和密码即可.



#### 4. 定义数据集和参数

这是比较关键的一步. 在配置中, 可以进行数据集的设置. 数据集支持SQL检索, 存储过程, 复杂SQL等多种类型.

其中SQL检索支持可以检索到你的数据库, 可以用选择的方式进行数据库中 表, 列, 条件, 排序, 参数的设置.

复杂SQL是比较常用的一种数据集创建方式. 直接在里面编写sql即可.

如果传参的话, 常见有普通传参和宏传参两种方式.

- 普通传参(数据集中):
  定义:
  ```sql
  select name, age from student where xxx = ?
  ```
  参数: 参数表达式: id , 结果类型: 字符串.
  配置->参数: 也要配置一下.
- 宏传参:
  定义:
  ```sql
  select name, age from student where ${maceventid}
  ```
  配置->宏定义: 也要配置一下.

以上步骤做完, 点击发布报表, 会生成一个raq文件. 这个就是最终要集成到项目的文件.



#### 5. 集成项目

关于报表在项目中的配置文件: `reportConfig.xml`

配置属性很多, 就不一一列举, 就举几个我配置过的属性:

- jdbc-ds-configs: 配置数据库信息
- maxCellNum: 在我集成过程中遇到一个错误, `cell num exceeds limit`. 意思是单元格数超过限制, 但是我的单元格数据量并不大, 为什么会出现这个错误. 查阅资料, 发现xml文件中有个设置:maxCellNum. 它是一个并发控制的选项, 取决于内存大小. 内存越大可以设置的越大, 不超过2000000.

```xml
<config>
     <name>maxCellNum</name>
     <value>100000</value>
</config>
```

再设计器设计过程中 → 报表属性中有个’报表格数’ 属性, 可以设置它来规定单元格数的限制. 

如果不设置. 默认会按defaultCellNum来. 如果没有defaultCellNum, 则会按20000来算, 我的xml中并没有设置它, 所以发生这个错误. 

> 但是在后面做项目过程中, 其他人没有设置居然没有报这个错. 也很无解

```xml
<config>
     <name>defaultCellNum</name>
     <value>9999</value>
</config>
```

关于它, 还有更加详细的并发控制, 附上链接:
<http://wenku.baidu.com/link?url=J6iqHMVKd2FDgnBAFsWfdUoHkaEdLeqFlhp4nOB0AumdWQ72-EInuncQ71jpMvjLOcCsZbt7muhxH6kj2AdS3PuF6LFIlfdxnbYWyTb7qsq>



**errorPage**

这个属性来指定异常页, 可以先把myErrorPage.jsp拷贝到项目里. 用来查看异常信息. 比直接看错误页面好多了.

```xml
<config>
     <name>errorPage</name>
     <value>/myErrorPage.jsp</value>
</config>
```



除此之外, 还需要拷贝一些必要的文件.
![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/%E6%B6%A6%E4%B9%BE%E6%8A%A5%E8%A1%A83.png)
web.xml中report的配置

```xml
<!-- report -->
	<servlet>
		<servlet-name>SetContextServlet</servlet-name>
		<servlet-class>com.runqian.util.webutil.SetContextServlet</servlet-class>
		<load-on-startup>2</load-on-startup>
	</servlet>
	<servlet>
		<servlet-name>reportServlet</servlet-name>
		<servlet-class>com.runqian.report4.view.ReportServlet</servlet-class>
		<init-param>
			<param-name>configFile</param-name>
			<param-value>/WEB-INF/report/reportConfig.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>reportServlet</servlet-name>
		<url-pattern>/reportServlet</url-pattern>
	</servlet-mapping>

	<jsp-config>
		<taglib>
			<taglib-uri>/WEB-INF/report/runqianReport4.tld</taglib-uri>
			<taglib-location>/WEB-INF/report/runqianReport4.tld</taglib-location>
		</taglib>
	</jsp-config>
```

下面看看最主要的文件吧.

首先, 我们设计出来的raq文件怎么在html中展现呢?

只需要在你的请求url上, 映射到showReport.jsp上, url上一般还附带着请求的传参, 和raq文件名.

例如: `showReport.jsp?raq=xxx.raq&params=...`

在 `showReport.jsp` 中:

```jsp
<%@ page contentType="text/html;charset=GBK" %>
<%@ taglib uri="/WEB-INF/runqianReport4.tld" prefix="report" %>
<%@ page import="java.io.*"%>
<%@ page import="java.util.*"%>
<%@ page import="com.runqian.report4.usermodel.Context"%>

<html>
<head><link type="text/css" href="css/style.css" rel="stylesheet"/></head>
<body topmargin=0 leftmargin=0 rightmargin=0 bottomMargin=0 style="overflow:hidden;">
<%
	request.setCharacterEncoding( "GBK" );
	String report = request.getParameter( "raq" );
	String reportFileHome=Context.getInitCtx().getMainDir();
	StringBuffer param=new StringBuffer();
	
	//保证报表名称的完整性
	int iTmp = 0;
	if( (iTmp = report.lastIndexOf(".raq")) <= 0 ){
		report = report + ".raq";
		iTmp = 0;
	}
	
	Enumeration paramNames = request.getParameterNames();
	if(paramNames!=null){
		while(paramNames.hasMoreElements()){
			String paramName = (String) paramNames.nextElement();
			String paramValue=request.getParameter(paramName);
			if(paramValue!=null){
				//把参数拼成name=value;name2=value2;.....的形式
				param.append(paramName).append("=").append(paramValue).append(";");
			}
		}
	}

	//以下代码是检测这个报表是否有相应的参数模板
	String paramFile = report.substring(0,iTmp)+"_arg.raq";
	File f=new File(application.getRealPath(reportFileHome+ File.separator +paramFile));

%>
<jsp:include page="toolbar.jsp" flush="false" />
<div style="overflow:auto;height:expression(parent.document.body.clientHeight-33+'px');">
<table id="rpt" align="left" ><tr><td>
<%	//如果参数模板存在，则显示参数模板
	if( f.exists() ) {
	%>
	<table id="param_tbl" width="100%" height="100%"><tr><td>
 
		<report:param name="form1" paramFileName="<%=paramFile%>"
			needSubmit="no"
			params="<%=param.toString()%>"
			
		/>
	</td>
	<td><a href="javascript:_submit( form1 )"><img src="../images/search.gif" border=no style="vertical-align:middle"></a></td>
	</tr></table>
	<% }
%>

<table align="center" width="100%" height="100%" >
	<tr><td>
		<report:html name="report1" reportFileName="<%=report%>"
			funcBarLocation="top"
			needPageMark="yes"
			generateParamForm="no"
			params="<%=param.toString()%>"
			needPivot="yes"
			pivotLabel=""
			exceptionPage="/reportJsp/myError2.jsp"
			appletJarName="runqianReport4Applet.jar,dmGraphApplet.jar"
			pageMarkLabel=""
			firstPageLabel=""
            prevPageLabel=""
            nextPageLabel=""
            lastPageLabel=""

		/>
	</td></tr>
</table>
</div>
<script language="javascript">
	//设置分页显示值
	document.getElementById( "t_page_span" ).innerHTML=report1_getTotalPage();
	document.getElementById( "c_page_span" ).innerHTML=report1_getCurrPage();
</script>
</body>
</html>
```

其中最关键的两个参数是:

- raqName: 接收的是raq的全文件名.
- param: 接收的是:` “eventid=” + eventid`

除此之外, 还需要把该jsp需要引入的jsp和静态资源加进来. 如`toolbar.jsp`, `style.css`等.



### 总结

润乾报表是一个国产收费的报表工具, 工作中遇到了, 承认东西很强.

但是不喜欢, 所以不想做过多研究.
