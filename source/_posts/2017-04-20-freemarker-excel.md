---
title: Freemarker 导出复杂 Excel
date: 2017-04-20 00:00:00
categories:
  - Backend Engineering
tags:
  - Java
  - Excel
excerpt: "使用 Freemarker 模板引擎导出复杂格式 Excel，包括多行合并、跨列合并、复杂表头等高级布局。"
aiSummary: "Freemarker 可作为 Excel 导出模板引擎，但复杂格式（如多行合并、跨列合并）的处理需要深入了解其 XML 配置规则。本文记录了使用 Freemarker 导出复杂 Excel 的完整流程：首先将 Excel 模板另存为 XML 格式，理解 sheet.xml 的结构（如单元格合并用 <mergeCell>、行样式用 <row>），然后编写 FTL 模板语法进行动态数据绑定。文章详细演示了多行合并、跨列合并、复杂表头、金额格式化等实际项目中常见需求的解决方案。"
---

### 序言

用 Freemarker 做简单的 Excel 导出确实很容易，但是要导出带有特殊格式的复杂 Excel（例如多行合并），还是费了我一天的时间来研究排版规则。

> 关于Freemarker的语法, 可以参考<http://freemarker.foofun.cn/index.html>

<!-- more -->

---

### 一、基础步骤

首先是引入 POM 依赖，构建工具使用 Maven：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

#### 1. 准备模板

接下来，制作 `.ftl` 模板文件。

先在 Office Excel 中编辑好一个表格，调整好你想要的样式，这就是最后要生成的模板长相。

![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/freemark1.png)

> **注意**：必须在 Excel 中选择“另存为” -> **XML 表格 (*.xml)** 格式，**千万不要直接去改后缀名**！

将保存好的 xml 文件拷贝到项目下（比如 `resources/templates` 目录），然后将其后缀名修改为 `.ftl`。

用文本编辑器（XML 方式）打开这个 `.ftl` 文件，找到类似下面这段代码：

```xml
<Table ss:ExpandedColumnCount="4" ss:ExpandedRowCount="3" x:FullColumns="1"
       x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
    <Row>
        <Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
    </Row>
    <Row>
        <Cell ss:StyleID="s67"/>
        <Cell ss:StyleID="s67"><Data ss:Type="String">name</Data></Cell>
        <Cell ss:StyleID="s67"><Data ss:Type="String">age</Data></Cell>
        <Cell ss:StyleID="s67"><Data ss:Type="String">address</Data></Cell>
    </Row>
    <Row>
        <Cell ss:StyleID="s67"><Data ss:Type="Number">1</Data></Cell>
        <Cell ss:StyleID="s67"><Data ss:Type="String">zhangsan</Data></Cell>
        <Cell ss:StyleID="s67"><Data ss:Type="Number">22</Data></Cell>
        <Cell ss:StyleID="s67"><Data ss:Type="String">BeiJing</Data></Cell>
    </Row>
</Table>
```

对比你的 Excel 很容易看出对应关系：每个 `<Row>` 就是一行，每个 `<Cell>` 就是一个单元格。

找到你要循环渲染的那一行，添加 Freemarker 表达式和标签（关于 Freemarker 语法可参考官方文档）。

**加上遍历标签后的代码如下：**

```xml
<Table ss:ExpandedColumnCount="4" ss:ExpandedRowCount="${userListSize}" x:FullColumns="1"
       x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
    <Row>
        <Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
    </Row>
    <Row>
        <Cell ss:StyleID="s67"/>
        <Cell ss:StyleID="s67"><Data ss:Type="String">姓名</Data></Cell>
        <Cell ss:StyleID="s67"><Data ss:Type="String">年龄</Data></Cell>
        <Cell ss:StyleID="s67"><Data ss:Type="String">地址</Data></Cell>
    </Row>

    <#assign index = 0 >
    <#list userList as u>
        <#assign index = index + 1 >
        <Row>
            <Cell ss:StyleID="s67"><Data ss:Type="Number">${index}</Data></Cell>
            <Cell ss:StyleID="s67"><Data ss:Type="String">${u.name}</Data></Cell>
            <Cell ss:StyleID="s67"><Data ss:Type="Number">${u.age}</Data></Cell>
            <Cell ss:StyleID="s67"><Data ss:Type="String">${u.address}</Data></Cell>
        </Row>
    </#list>
</Table>
```

> **避坑提示**：注意 `<Table>` 标签中的 `ss:ExpandedRowCount` 属性，它的值可以通过变量 `${userListSize}` 动态传进来。如果你偷懒直接写死一个很大的固定值（比如 99999），在打开 Excel 时可能会报“文件损坏”警告，或者导致空行占用大量内存加载变慢。建议老老实实在后台计算好行数传进来。

#### 2. 准备数据源

对于动态生成的 List，里面可以放 `Map`，也可以放具体的 Java 对象（POJO）。这里我先用 `Map` 演示。

**模拟获取数据：**

```java
/**
 * 构造 user 数据 List<Map<String, Object>>
 */
private static List<Map<String, Object>> getUserList() {
    List<Map<String, Object>> returnList = new ArrayList<>();
    
    Map<String, Object> map1 = new HashMap<>();
    map1.put("name", "张三");
    map1.put("age", "18");
    map1.put("address", "广东");
    
    Map<String, Object> map2 = new HashMap<>();
    map2.put("name", "王五");
    map2.put("age", "22");
    map2.put("address", "北京");
    
    returnList.add(map1);
    returnList.add(map2);
    
    return returnList;
}
```

**控制器 (Controller) 导出逻辑：**

```java
@RequestMapping(value="/exportExcel", method=RequestMethod.GET)
public void exportExcelByFreeMarker(HttpServletRequest request, HttpServletResponse response) {
    try {
        List<Map<String, Object>> userList = this.getUserList();
        
        configuration.setDefaultEncoding("UTF-8");  
        configuration.setTemplateUpdateDelayMilliseconds(0);  
        configuration.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);  
        
        // 获取模板
        Template template = configuration.getTemplate("userlist.ftl");
        Map<String, Object> root = new HashMap<>();
        root.put("userList", userList);
        // 注意：计算行数时要加上表头占据的行数（这里是标题行+列名行=2行）
        root.put("userListSize", String.valueOf(userList.size() + 2));
        
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmss");
        String fileName = "人员列表_" + sdf.format(new Date()); 
        
        response.setContentType("application/msexcel;charset=UTF-8");
        response.setHeader("Content-disposition", "attachment;filename=\"" + 
            new String((fileName + ".xls").getBytes("GBK"), "ISO8859-1") + "\"");
            
        // response 字符流转换成字节流，template 需要字节流作为输出
        OutputStream outputStream = response.getOutputStream();
        OutputStreamWriter outputWriter = new OutputStreamWriter(outputStream, "UTF-8");
        Writer writer = new BufferedWriter(outputWriter);
        
        template.process(root, writer);
        
        writer.flush();  
        writer.close();  
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

现在，在浏览器中请求该接口地址，就可以成功下载导出的 Excel 文件了。

---

### 二、进阶：导出带有合并单元格的复杂 Excel

生成 Excel 还有很多复杂的业务场景，比如带图片、跨行合并等。这里以**多行向下合并**为例，效果如下：

![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/freemark2.png)

手动在 Excel 里合并好样式并另存为 XML 后，观察生成的关键代码：

```xml
<Table ss:ExpandedColumnCount="5" ss:ExpandedRowCount="99999" x:FullColumns="1"
       x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
   <Row>
      <Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
      <Cell ss:StyleID="s62"/>
   </Row>
   <Row>
      <Cell ss:StyleID="s67"/>
      <Cell ss:StyleID="s67"><Data ss:Type="String">姓名</Data></Cell>
      <Cell ss:StyleID="s67"><Data ss:Type="String">年龄</Data></Cell>
      <Cell ss:StyleID="s67"><Data ss:Type="String">孩子</Data></Cell>
      <Cell ss:StyleID="s69"><Data ss:Type="String">地址</Data></Cell>
   </Row>
   <Row>
      <Cell ss:MergeDown="1" ss:StyleID="m87387632"><Data ss:Type="Number">1</Data></Cell>
      <Cell ss:MergeDown="1" ss:StyleID="m87387612"><Data ss:Type="String">张三</Data></Cell>
      <Cell ss:MergeDown="1" ss:StyleID="m87387592"><Data ss:Type="Number">22</Data></Cell>
      <Cell ss:StyleID="s67"><Data ss:Type="String">小张</Data></Cell>
      <Cell ss:MergeDown="1" ss:StyleID="m87387572"><Data ss:Type="String">北京</Data></Cell>
   </Row>
   <Row>
      <Cell ss:Index="4" ss:StyleID="s67"><Data ss:Type="String">二张</Data></Cell>
   </Row>
</Table>
```

**仔细观察并分析上述代码，与普通 XML(不带合并) 的区别在于：**

1. 每个需要向下合并的 `<Cell>` 都带有一个 `ss:MergeDown` 属性，它的值为 **(实际需要合并的格数 - 1)**。
2. 合并的格数，通常需要通过子对象的集合大小来动态计算（比如这里的“孩子”集合长度）。
3. 被合并挤占后的扩展行（下一行），其有效单元格会带有一个 `ss:Index` 属性。这个值代表它在当前行属于第几列（例如上面的“二张”在第 4 列，所以是 `Index="4"`）。

**分析好逻辑后，我们开始在 `.ftl` 模板中加入判断标签：**

```xml
  <Table ss:ExpandedColumnCount="5" ss:ExpandedRowCount="4" x:FullColumns="1"
   x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
   <Row>
    <Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
    <Cell ss:StyleID="s62"/>
   </Row>
   <Row>
    <Cell ss:StyleID="s67"/>
    <Cell ss:StyleID="s67"><Data ss:Type="String">姓名</Data></Cell>
    <Cell ss:StyleID="s67"><Data ss:Type="String">年龄</Data></Cell>
    <Cell ss:StyleID="s67"><Data ss:Type="String">孩子</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="String">地址</Data></Cell>
   </Row>
   
<#assign index = 0 >
<#list userBoList as u>
<#assign index = index + 1 >
<#assign num = u.children?size-1>
<#if num lt 0>
<#assign num = 0>
</#if>
   <Row>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387632"><Data ss:Type="Number">${index}</Data></Cell>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387612"><Data ss:Type="String">${u.name}</Data></Cell>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387592"><Data ss:Type="Number">${u.age}</Data></Cell>
	<!-- 在children的List中取出第一个 -->
	<#list u.children as firstChildren>
	  <#if firstChildren_index == 0>
		<Cell ss:StyleID="s67"><Data ss:Type="String">${firstChildren.name}</Data></Cell>
	  </#if>
	</#list>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387572"><Data ss:Type="String">${u.address}</Data></Cell>
   </Row>
   
  <!-- 从第一个之后开始取 -->
  <#list u.children as c>
    <#if (u.children?size > 1 && c_index > 0 )>  
     <Row>
      <Cell ss:Index="4" ss:StyleID="s67"><Data ss:Type="String">${c.name}</Data></Cell>
     </Row>
    </#if>
  </#list>
  
</#list>

  </Table>
```

#### 关键总结

1. **防止负数报错**：`MergeDown` 属性中一定要判断 `<#if num lt 0>`。亲测在 Microsoft Office 中，如果该值为负数会导致 Excel 直接报错打不开（WPS 容错强不会报错），所以最好做兜底判断。
2. **数据结构设计**：在这个复杂 Excel 场景中，后台组装的数据结构需要是包含一对多嵌套关系的实体。例如：
   - 主对象：`List<UserBo>`
   - 子对象：在 `UserBo` 内部包含 `List<ChildrenBo>`
   - 两者关系：`User -> Children : 1 -> N`
   - 当然，使用嵌套的 `List<Map<String, Object>>` 也是完全可以的。