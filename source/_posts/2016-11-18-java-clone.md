---
title: 关于 Java 克隆
date: 2016-11-18 00:00:00
categories:
  - Backend Engineering
tags:
  - Java
  - Archived
excerpt: "解释 Java 对象的浅拷贝与深拷贝区别，介绍 clone 方法的使用及 CloneNotSupportedException 处理。"
aiSummary: "本文探讨 Java 中的对象克隆问题。首先说明基本类型赋值实现的是值拷贝，而自定义对象赋值只是引用拷贝，无法真正独立。接着深入区分浅拷贝与深拷贝的核心区别：浅拷贝仅复制对象本身及其基本类型字段，深拷贝则递归复制所有成员变量直至基本类型。通过重写 clone() 方法并实现 Cloneable 接口可以完成对象拷贝，同时需处理 CloneNotSupportedException 异常。"
---

补充关于 Java 克隆的一点点。

<!-- more -->

### 一、开始

先来看一段代码：

```java
int a = 1;
int b = a;
b = 2;
System.out.println(a);  // 输出: 1
System.out.println(b);  // 输出: 2
```
`a` 的值并没有因为复制给 `b` 然后 `b` 被修改而发生改变。
这就实现了 `b` 对 `a` 的一份独立拷贝。对于 byte, short, int, char 等基本数据类型，直接赋值都适用这种行为（你可以将其理解为基本类型的深拷贝）。

但是，如果要复制一份自定义的“对象”，简单的赋值操作就无法达到这种独立性了，这涉及到浅拷贝与深拷贝的区别。

**何为深浅呢？**
一个重要的标识：**是否完全复制了一个新的对象，并在堆中为其申请了新的内存空间。**

在 Java 中，对象的直接赋值（`Student s2 = s1;`）只是将两个栈中的引用指向了堆内存中的同一块存储空间。所以一个引用的操作必定会影响原数据。

> **理论解释**：
> - 对于基本类型，变量直接存储在栈中，赋值操作是直接将值复制一份，互不影响；
> - 对于自定义对象，由于引用存储在栈中，而对象实际数据分配在堆中，默认的赋值或浅拷贝只拷贝了引用，并没有在堆中去申请新的对象空间。
> - 特殊的如 `String`，虽然是对象，但由于其不可变性（Immutable）及常量池机制，直接赋值时其行为表现得像基本类型一样安全。

---

### 二、浅拷贝 (Shallow Copy)

通过实现 `Cloneable` 接口并重写 `clone()` 方法，可以实现对象拷贝。默认的 `super.clone()` 提供的就是浅拷贝。

```java
package com.xie.core.base;

class School {
    private String name;
    // 省略 getter/setter
}

class Student implements Cloneable {
    private School school;
    private String name;
    // 省略 getter/setter

    /**
     * 浅拷贝实现
     */
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class CloneTest01 {
    public static void main(String[] args) throws Exception {
        School school = new School(); 
        school.setName("Thank School");
        
        Student student1 = new Student(); 
        student1.setName("Thank"); 
        student1.setSchool(school); 
        
        // 浅拷贝
        Student student2 = (Student) student1.clone();
        student2.setName("Chunhua");
        // 修改 student2 的 school 属性
        student2.getSchool().setName("Chunhua School");

        System.out.println(student1.getName() + ", " + student1.getSchool().getName());
        System.out.println(student2.getName() + ", " + student2.getSchool().getName());
    }
}
```

**输出结果为:**
```text
Thank, Chunhua School
Chunhua, Chunhua School
```

我分别打印了 `school` 的内存地址（`toString()`）：
- `student1 --> school: com.xie.core.base.School@762efe5d`
- `student2 --> school: com.xie.core.base.School@762efe5d`

可以看到，`student2` 的拷贝只是对 `String` 类型的 `name` 实现了独立拷贝，而对 `School` 对象类型的属性，两者依然指向同一个内存地址。这就叫浅拷贝。

---

### 三、深拷贝 (Deep Copy)

为了解决上面的问题，我们需要在拷贝 `Student` 的同时，连同它内部的 `School` 对象也一并拷贝。

```java
package com.xie.core.base;

class School implements Cloneable {
    private String name;
    // 省略 getter/setter

    @Override
    protected Object clone() {
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
            return null;
        }
    }
}

class Student implements Cloneable {
    private School school;
    private String name;
    // 省略 getter/setter

    /**
     * 深拷贝实现
     */
    @Override
    protected Object clone() throws CloneNotSupportedException {
        Student o = (Student) super.clone();
        // 手动调用内部对象 School 的 clone 方法实现深拷贝
        o.school = (School) this.school.clone();
        return o;
    }
}

public class CloneTest01 {
    public static void main(String[] args) throws Exception {
        School school = new School(); 
        school.setName("Thank School");
        
        Student student1 = new Student(); 
        student1.setName("Thank"); 
        student1.setSchool(school); 
        
        Student student2 = (Student) student1.clone();
        student2.setName("Chunhua");
        student2.getSchool().setName("Chunhua School");

        System.out.println(student1.getName() + ", " + student1.getSchool().getName());
        System.out.println(student2.getName() + ", " + student2.getSchool().getName());
    }
}
```

**输出结果为:**
```text
Thank, Thank School
Chunhua, Chunhua School
```

再次打印 `school` 的内存地址：
- `student1 --> school: com.xie.core.base.School@762efe5d`
- `student2 --> school: com.xie.core.base.School@41a4555e`

此时它们是两个不同的对象。符合“完全复制一个新的对象，并申请新的内存空间”的定义，这就是深拷贝。

---

### 四、利用序列化实现深拷贝

如果一个对象内部嵌套了多层其他对象，一层层去重写 `clone()` 方法将是一场噩梦。此时可以利用 Java 的序列化（Serialization）来实现深拷贝。

> 把对象写到流里的过程是串行化（Serialization），把对象从流中读出来的过程叫反序列化（Deserialization）。
> 应当指出的是，写在流里的是对象的一个拷贝，而原对象仍然存在于 JVM 里面。因此，先使对象实现 `Serializable` 接口，然后把对象写到一个流里，再从流里读出来便可以完美重建出一个全新的对象。

这种方式不需要实现 `Cloneable` 接口，但是所有涉及的类**必须实现 `Serializable` 接口**。

```java
package com.xie.core.base;

import java.io.*;

class School implements Serializable { // 实现序列化接口
    private String name;
    // 省略 getter/setter
}

class Student implements Serializable { // 实现序列化接口
    private School school;
    private String name;
    // 省略 getter/setter

    /**
     * 串行化深复制
     */
    public Object deepClone() throws IOException, ClassNotFoundException {
        // 写入字节流
        ByteArrayOutputStream bo = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bo);
        oos.writeObject(this);
        
        // 从字节流读出
        ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
        ObjectInputStream osi = new ObjectInputStream(bi);
        
        return osi.readObject();
    }
}


public class CloneTest01 {

    public static void main(String[] args) throws CloneNotSupportedException, ClassNotFoundException, IOException {

        School school = new School();
        school.setName("Thank School");

        Student student1 = new Student();
        student1.setName("Thank");
        student1.setSchool(school);

        Student student2 = (Student) student1.deepClone();
        student2.setName("Chunhua");
        student2.getSchool().setName("Chunhua School");

        System.out.println(student1.getName() + ", " + student1.getSchool().getName());
        System.out.println(student2.getName() + ", " + student2.getSchool().getName());

    }
}
```

### 总结

在内存中通过字节流的拷贝是比较容易实现且稳妥的。把母对象写入到一个字节流中，再从字节流中将其读出来，就可以创建一个新的对象，该新对象与母对象之间并不存在引用共享的问题，真正实现对象的深拷贝。

> **【现代开发避坑与进阶提示】**：
> 虽然利用 Java 原生序列化可以实现深拷贝，但其性能较低且代码繁琐。在现代企业级开发中，我们很少再手写原生的 `clone()` 或序列化流。
> 
> **常用的替代方案有**：
> 1. **JSON 序列化方式**：利用 `Fastjson`、`Jackson` 或 `Gson`，将对象转为 JSON 字符串，再反序列化为对象（代码极简，推荐）。
> 2. **工具类**：如果是单纯的属性拷贝（VO/DTO 转换），直接使用 `Spring BeanUtils` 或 `MapStruct`。
> 3. **Apache Commons Lang**：直接调用 `SerializationUtils.clone(object)` 即可一行代码完成基于序列化的深拷贝。