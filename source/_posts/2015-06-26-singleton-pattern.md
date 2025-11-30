---
title: 设计模式 — 单例模式
date: 2015-06-26
categories:
  - Backend Engineering
tags:
  - 设计模式
excerpt: "介绍单例模式的定义、目的及常见实现方式：饿汉式、懒汉式、线程安全双重检查锁、静态内部类、枚举等。"
aiSummary: "单例模式是设计模式中最基础的一种，其核心目标是保证某个类在整个应用中只有一个实例。文章详细介绍了单例模式的多种实现方式：饿汉式（类加载时创建，线程安全）、懒汉式（延迟加载，线程不安全）、懒汉式双重检查锁（线程安全且高效）、静态内部类方式（利用类加载机制保证线程安全）、以及枚举方式（Effective Java 作者推荐，最简洁防反射）。分析了各种实现方式的优缺点和适用场景，是理解单例模式的全面指南。"
---





> 序言: 计算机老师让每个人准备一个PPT对其演讲, 内容不限 
>
> 看到同学都在讲Java实现xxx系统, 感觉很厉害. 所以我想来看下设计模式. 从最简单的看起


<!-- more -->

### 介绍

设计模式(Design pattern): 是一套被反复使用, 多数人知晓的, 经过分类编目的,代码设计经验的总结.

目的: 使用设计模式是为了可重用代码, 让代码更容易被他人理解, 保证代码的可靠性

设计模式（Design pattern）是一套被反复使用、多数人知晓、经过分类编目的代码设计经验总结。

目的：可复用、易理解、可靠。

- 保证整个应用中某个实例有且只有一个

- 有些对象我们只需要一个: 配置文件, 工具类, 线程池, 缓存, 日志对象等.

- 保证整个应用中某个实例有且只有一个。

- 有些对象我们只需要一个：配置、工具类、线程池、缓存、日志对象等。

- 如果创建出多个实例可能导致问题：占用过多资源、状态不一致等。

- 静态的私有的本类型的变量
- 构造方法私有化
- 提供一个公共的静态的入口点方法

- 静态的、私有的本类型变量
- 构造方法私有化
- 提供一个公共的静态入口方法

第一种: 饿汉式  

## 饿汉式（Eager Initialization）
public class SingletonParrent01 {  
特点：类加载阶段创建对象，并且只创建一次。
        Singleton s1 = Singleton.getInstance();  
        Singleton s2 = Singleton.getInstance();  
public final class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
不好处: 无论这个类是否被使用,都会创建. 所以很多创建过程是无用的.

缺点：无论这个类是否被使用，都会创建实例（可能造成不必要的初始化开销）。

第二种: 懒汉式, 或者说lazy loaded  

## 懒汉式（Lazy Initialization）
 public class SingletonParrent2 {  
特点：第一次调用时才初始化实例。
        Singleton s1 = Singleton.getInstance();  
        Singleton s2 = Singleton.getInstance();  
public final class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

### 线程不安全

上面懒汉式的单例模式代码很清楚，也很简单。

## 线程不安全问题

单线程下，这段代码没有什么问题，可是如果是多线程，麻烦就来了。

我们来分析一下：线程A希望使用Singleton，调用getInstance()方法。

因为是第一次调用，A就发现instance是null的，于是它开始创建实例，就在这个时候，CPU发生时间片切换，线程B开始执行，它要使用Singleton，调用getInstance()方法，同样检测到instance是null——注意，这是在A检测完之后切换的，也就是说A并没有来得及创建对象——因此B开始创建。B创建完成后，切换到A继续执行，因为它已经检测完了，所以A不会再检测一遍，它会直接创建对象。

这样，线程A和B各自拥有一个Singleton的对象——单例失败！

**所以懒汉式是线程不安全的!**



#### 尝试加锁

所以,我们用加锁来试一下.

## 尝试加锁（同步方法）
public class Singleton {

	private static Singleton instance = null;

public final class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public synchronized static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

但是不管是否为Null, 只要调用getInstance()方法, 都会进入synchronized同步代码块, 速度会很慢. 

性能很低. 让我们来分析一下，究竟是整个方法都必须加锁，还是仅仅其中某一句加锁就足够了？

我们为什么要加锁呢？分析一下出现lazy loaded的那种情形的原因。

原因就是检测null的操作和创建对象的操作分离了。如果这两个操作能够原子地进行，那么单例就已经保证了。



#### Double checked

于是，我们开始修改代码：

## Double-Checked Locking（DCL）
//double-checked locking  
public class Singleton {   
  
    private static Singleton instance = null;   
public final class Singleton {
    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

**从源头检查:**

下面我们开始说编译原理。所谓编译，就是把源代码“翻译”成目标代码——大多数是指机器代码——的过程。

### 从源头检查：指令重排序与可见性

编译原理里面有一个很重要的内容是编译器优化。所谓编译器优化是指，在不改变原来语义的情况下，通过调整语句顺序，来让程序运行的更快。

这个过程成为reorder。

要知道，JVM只是一个标准，并不是实现。JVM中并没有规定有关编译器优化的内容，也就是说，JVM实现可以自由的进行编译器优化。

这个过程称为重排序（reorder）。

- 一个是申请一块内存，调用构造方法进行初始化操作
- 另一个是分配一个指针指向这块内存。

这两个操作谁在前谁在后呢？JVM规范并没有规定。

那么就存在这么一种情况，JVM是先开辟出一块内存，然后把指针指向这块内存，最后调用构造方法进行初始化。

下面我们来考虑这么一种情况：线程A开始创建SingletonClass的实例，此时线程B调用了getInstance()方法，首先判断instance是否为null。

按照我们上面所说的内存模型，A已经把instance指向了那块内存，只是还没有调用构造方法，因此B检测到instance不为null，于是直接把instance返回了——问题出现了，尽管instance不为null，但它并没有构造完成，就像一套房子已经给了你钥匙，但你并不能住进去，因为里面还没有收拾。

此时，如果B在A将instance构造完成之前就是用了这个实例，程序就会出现错误了！

#### 解决方案: volatile

此时，如果 B 在 A 将 instance 构造完成之前就使用了这个实例，程序就可能出现错误！

### 解决方案：volatile
volatile关键字有了明确的语义——在JDK1.5之前，volatile是个关键字，但是并没有明确的规定其用途——被volatile修饰的写变量不能和之前的读写代码调整，读变量不能和之后的读写代码调整！

因此，只要我们简单的把instance加上volatile关键字就可以了。
`volatile` 关键字在 JDK 5 之后有了更明确的内存语义：对 `volatile` 变量的写入与后续读取之间会建立 happens-before 关系，并且会限制相关的重排序。
```java
/*
 * volatile关键字
 */
public class Singleton {

    private volatile static Singleton instance = null;

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    private Singleton() {

    }

}
```
而，这只是JDK1.5之后的Java的解决方案，那之前版本呢？其实，还有另外的一种解决方案，并不会受到Java版本的影响：

```java
/* 这是一种常见的写法。但还有另外一种更简洁的方案，不依赖 `volatile`，也不需要显式加锁：
 * 静态内部类方式
 */
public final class Singleton {
    private Singleton() {
    }

    private static final class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

在这一版本的单例模式实现代码中，我们使用了Java的静态内部类。

这一技术是被JVM明确说明了的，因此不存在任何二义性。

在这段代码中，我们使用了静态内部类 Holder。
在这段代码中，因为SingletonClass没有static的属性，因此并不会被初始化。直到调用getInstance()的时候，会首先加载SingletonClassInstance类，这个类有一个static的SingletonClass实例，因此需要调用SingletonClass的构造方法，然后getInstance()将把这个内部类的instance返回给使用者。由于这个instance是static的，因此并不会构造多次。
当 `Singleton` 类被加载时，Holder 不会立刻初始化；只有在第一次调用 `getInstance()` 时，Holder 才会被加载并触发类初始化，从而完成一次性初始化。Java 的类初始化过程本身是线程安全的。
由于SingletonClassInstance是私有静态内部类，所以不会被其他类知道，同样，static语义也要求不会有多个实例存在。并且，JSL规范定义，类的构造必须是原子性的，非并发的，因此不需要加同步块。同样，由于这个构造是并发的，所以getInstance()也并不需要加同步。
由于 `INSTANCE` 是 `static final`，因此也不会构造多次。
至此，我们完整的了解了单例模式在Java语言中的时候，提出了两种解决方案。个人偏向于第二种，并且Effiective Java也推荐的这种方式。
由于 Holder 是私有静态内部类，不会被其他类直接访问，并且类初始化只会发生一次，因此不需要同步块，`getInstance()` 也不需要加 `synchronized`。

## 结论速查

- **饿汉式**：实现简单、天然线程安全；缺点是可能带来不必要的初始化开销。
- **懒汉式（不加锁）**：实现简单；多线程不安全。
- **同步方法**：线程安全；每次调用都有同步开销。
- **DCL + volatile**：线程安全且减少同步开销；实现复杂度更高，且依赖 JDK 5+ 的内存模型语义。
- **静态内部类 Holder**：线程安全、延迟初始化、实现简洁；多数场景下是更推荐的写法之一。

个人偏向于静态内部类方式，Effective Java 也推荐过类似思路。
