---
title: EasyUI 下拉选择树
date: 2016-09-04 00:00:00
categories:
  - Web
tags:
  - EasyUI
  - 前端
excerpt: "使用 EasyUI 的 combotree 组件实现下拉树形选择框，继承自 tree 和 combobox，支持异步加载和节点操作。"
aiSummary: "本文介绍在 EasyUI 中使用 combotree（下拉选择树）组件的实现方法，该组件继承自 tree 和 combobox，既具有下拉框的便捷性，又支持树形层级结构的展示。文章详细说明了前台 HTML 和 JS 的配置方式、数据的异步加载、节点的选择取值等操作，是管理后台开发的实用参考。"
---

### 前言

很多时候在开发管理后台时都会用到下拉框。有时候为了显示更好的层级效果，会用到树型下拉框。它实际就是下拉框中带了一个树型的控件，所以在 EasyUI 中，它自然继承自 `tree` 和 `combobox` 组件。

> 后台数据处理其实没什么问题，但是 EasyUI 我是真的不会啊……。

<!-- more -->

---

### 一、前台实现

首先从前台来讲，先把 HTML 元素写好：

```html
<select id="cc" value="01" style="width:200px;"></select>
<a href="#" id="btn" class="easyui-linkbutton" onclick="getValue()">获取选中值</a>
```

这里使用 JavaScript 的方式进行加载：

```javascript
$('#cc').combotree({
    url: 'tree_data.json'
});
```

这里的 `url` 是跟 HTML 处于同一文件夹下的一个 JSON 格式文件。为了测试，我们可以准备一个模拟的 JSON 数据。它的基础结构必须包含 `id`、`text` 和 `children` 这几个关键属性，大致结构如下：

```json
[{	
    "id": 1,	
    "text": "Languages",	
    "children": [{		
        "id": 11,		
        "text": "Java"	
    },{		
        "id": 12,		
        "text": "C++"	
    }]
}]
```

这样，一个简单基础的下拉树就形成了：

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui1.png)

我们在下拉树旁边写了一个按钮，这个按钮绑定了一个普通的 `onclick` 事件，目的是获取下拉树当前选中的值。代码如下：

```javascript
function getValue(){
    var value = $('#cc').combotree('getValue');
    alert(value);
}
```

#### 实现多选功能

下拉树如果需要实现多选，实际上就是加一个 `multiple` 属性，代码如下：

```javascript
$('#ccm').combotree({
    url: 'tree_data.json',
    multiple: true
    // cascadeCheck: false
    // onlyLeafCheck: true
});
```

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui2.png)

如果是多选模式，获取选中值只需要把前面的 `getValue` 换成 `getValues` 即可：

```javascript
var values = $('#ccm').combotree('getValues');
```
这会返回选中 `id` 值的数组。

**多选模式下的两个重要属性**：
- `cascadeCheck`：定义是否层叠选中状态。默认是 `true`（级联选中），也就是勾选子节点，它的父节点也可能被选中；如果设为 `false` 就是不级联（但用户体验可能会有点怪）。
- `onlyLeafCheck`：定义是否只在末级节点（叶子节点）前显示复选框。如果设为 `true`，就只有最底层的子节点才会有复选框，自然也就没什么级联关系了。

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui3.png)

---

### 二、数据库结构设计

假如下拉树的业务场景是：选择“班级”及其关联的“学生信息”，我们可以随便造点数据结构。

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui4.png)

*（图示为两张表：班级表和学生表，通过班级 ID 进行关联）*

---

### 三、后台接口实现

前台的 `url` 接收的是一个 JSON 格式的数组，并且我们知道，节点对象中 `id`、`text`、`children` 这三个属性缺一不可。

数据库中的两张表在持久层（Persistence）通常对应着两个实体类：`ClassEntity` 和 `StudentEntity`。除此之外，为了适配 EasyUI 的数据结构，我们还需要专门定义一个视图模型（ViewModel）：

```java
public class ComboTreeModel {
    private String id;
    private String text;
    private List<ComboTreeModel> children;
    
    // 省略 getter 和 setter
}
```

接下来就是在 Controller 层通过业务关联，将查询出的实体数据组装到 `ComboTreeModel` 模型中。


```java
@RequestMapping(value = "/getSelectData")
@ResponseBody
public List<ComboTreeModel> getTreeData(HttpServletRequest request, HttpServletResponse response) {
    // 查询所有班级列表
    // 假设返回：[{eventid:"c001", classname:'奥赛班'}, {eventid:"c002", classname:'直播班'}]
    List classList = this.studentService.queryClassList();
    
    List<ComboTreeModel> list = new ArrayList<ComboTreeModel>();
    
    for (int i = 0; i < classList.size(); i++) {
        ComboTreeModel model = new ComboTreeModel();
        Map classMap = (Map) classList.get(i);
        
        String classId = (String) classMap.get("eventid");
        String className = (String) classMap.get("classname");
        
        model.setId(classId); 
        model.setText(className);
        
        // 警告：循环内查询数据库（N+1 问题），仅做早期功能验证示例
        List studentList = this.studentService.queryByClassId(classId);
        
        if (studentList.size() > 0) {
            List<ComboTreeModel> childrenList = new ArrayList<ComboTreeModel>();
            for (int j = 0; j < studentList.size(); j++) {
                ComboTreeModel model2 = new ComboTreeModel();
                Map studentMap = (Map) studentList.get(j);
                
                String studentEventId = (String) studentMap.get("eventid");
                String studentName = (String) studentMap.get("studentname");
                
                model2.setId(studentEventId);
                model2.setText(studentName);
                childrenList.add(model2);
            }
            model.setChildren(childrenList);
        }
        list.add(model); 
    }
    
    return list;
}
```

最后，由于使用了 Spring MVC 的 `@ResponseBody` 注解，该接口会自动将 `List<ComboTreeModel>` 转换为前台 EasyUI 所需的 JSON 格式数据。Over！