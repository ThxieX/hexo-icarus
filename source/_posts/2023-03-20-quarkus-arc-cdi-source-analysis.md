---
title: Quarkus ArC 原理 - CDI 构建时注入
date: 2023-03-20
categories:
  - Backend Engineering
tags:
  - Java
  - Quarkus
  - 源码
excerpt: "分析 Quarkus 的依赖注入核心 ArC，对比 Spring 的运行时反射注入。"
aiSummary: "ArC（Annotation + CDI）是 Quarkus 的依赖注入核心实现，基于 CDI Lite 规范。本文分析了 ArC 与 Spring 运行时反射注入的本质区别：Spring 在启动时通过反射扫描并创建 Bean，而 ArC 在编译时通过注解处理器生成注入代码，运行时直接调用。文章还探讨了 Quarkus 构建时处理的具体机制，以及这种设计对启动性能和内存占用的影响。"
---



## TL;DR

- Spring 的 IoC 是**运行时反射**：启动时扫 classpath、解析注解、创建代理
- ArC 的 IoC 是**构建时代码生成**：编译时搞定注入，运行时直接调用
- 👉 理解了这个本质区别，就理解了 Quarkus 为什么这么快

<!-- more -->

---

## 回顾 Spring 的运行时注入

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

这套流程背后发生的事情：

1. **启动阶段**：Spring 扫描 classpath，找到所有 `@Component`（包括 `@Service`）
2. **Bean 创建**：通过反射调用构造函数创建 bean 实例
3. **依赖注入**：通过 `@Autowired` 找到依赖关系，再次通过反射把 `userRepository` 注入进去
4. **代理创建**：如果用了 `@Transactional` 等切面，会创建 JDK 动态代理或 CGLIB 代理



之前也从源码角度分析了`@ConfigurationProperties` 绑定流程的核心逻辑（ [Spring Boot 2.0 源码解析 - 配置绑定](/posts/spring-boot-config-binding)）

1. `ConfigurationPropertiesBindingPostProcessor` 在 bean 初始化时扫描所有带 `@ConfigurationProperties` 的类
2. 通过反射拿到注解信息，再通过 `Binder` 把配置属性绑定到 bean 上
3. 这一套全是在**运行时**发生的



👉 Spring 把能留到运行时做的事，都留到了运行时。



它在追求**灵活性**和**动态性**。但如果你的场景追求的是**启动快、内存省**，这套机制就成了瓶颈。



**问题在哪？**：每一步都离不开反射：

- 扫描 classpath：ClassPathScanningCandidateComponentProvider
- 创建实例：Constructor.newInstance()
- 注入依赖：Field.setAccessible() + Field.set()
- 创建代理：Proxy.newProxyInstance()

**反射的成本不低**：每次调用都要经过安全检查、权限校验。



---



## CDI 构建时注入 VS Spring 运行时注入



### ArC 是什么？

ArC = Annotation + CDI，是 Quarkus 的依赖注入核心实现。

> 官方文档的定义：ArC is a CDI-based dependency injection solution optimized for Quarkus build-time processing.

关键词：

1. **CDI-based**：基于 Jakarta CDI（Context and Dependency Injection）规范
2. **build-time processing**：构建时处理



### CDI vs Spring DI

CDI 是 Java EE 的标准依赖注入规范，Spring 的 `@Autowired` 其实就是对标 CDI 的 `@Inject`。

```java
// Spring
@Autowired
private UserRepository userRepository;

// CDI (Quarkus)
@Inject
private UserRepository userRepository;
```

写法几乎一样，但底层实现完全不同。



### ArC vs Spring

| 对比维度 | Spring | ArC (Quarkus) |
| :--- | :--- | :--- |
| **注入时机** | 运行时 | 构建时 |
| **实现方式** | 反射 | 代码生成 |
| **启动扫描** | 有 | 无 |
| **延迟加载** | 支持 | 基本不支持 |
| **动态特性** | 强 | 弱 |

**ArC 几乎不支持延迟加载**，因为所有注入代码都在编译时生成好了，不存在「运行时再去找」的概念。



---



## ArC 构建时注入原理



### 编译时注解处理器

ArC 的核心是一个**编译时注解处理器**（Annotation Processor）。

当你在 IDE 里保存 Java 文件，或者执行 `mvn compile` 时，Quarkus 的注解处理器会：

1. 扫描所有 `@Inject` 注解
2. 分析依赖关系图
3. 生成直接的注入代码

没有反射、没有权限检查、没有运行时扫描。



生成的代码大概是这个样子：

```java
// 你写的代码
@Inject
UserService userService;

// 生成的代码（简化版）
public class UserServiceInjectionBean {
    private final UserService userService;

    public UserServiceInjectionBean() {
        this.userService = new UserService(); // 直接 new，不需要反射
    }

    public UserService getUserService() {
        return this.userService;
    }
}
```

**注意**：`new UserService()` —— 这是**直接调用构造函数**，不是 `Class.newInstance()`，更不是 `Constructor.newInstance()`。



### 构建时 vs 运行时

来看个更具体的例子，对比两种方式启动时发生了什么：

**Spring Boot 启动**：

```
启动 JVM
  → Spring 容器初始化
    → 扫描 classpath（扫所有 @Component）
    → 创建 BeanDefinition
    → 遍历创建所有 Bean
      → 反射调用构造函数
      → 反射注入依赖
      → 如果有切面，创建代理
    → 运行时织入（AspectJ）
  → 应用启动完成
```

**Quarkus 启动**：

```
启动 JVM
  → 加载预生成的注入代码（已经编译进 JAR）
  → 直接调用生成好的方法
  → 应用启动完成
```

没有扫描，没有反射，没有运行时判断。



### 如何验证？

Quarkus 提供了一个 Dev UI，可以直观地看到构建时生成了什么。

```bash
# 启动开发模式
quarkus dev
```

然后访问 `http://localhost:8080/q/dev`，可以看到：

- 已注册的 beans
- 依赖注入关系
- 构建时生成的信息



---



## ArC 的实现细节

### @Inject 的处理流程

ArC 处理 `@Inject` 大致分三步：

**1. 收集阶段（Build Time）**

```
扫描所有带 @Inject 的字段/构造函数
  → 分析依赖类型
  → 构建依赖图
  → 生成注入代码
```

**2. 验证阶段（Build Time）**

```
检查循环依赖
  → 检查可选依赖（@Inject Optional<T>）
  → 检查producer方法
```

**3. 注入阶段（Runtime）**

```
直接调用生成好的注入代码
  → 无反射
  → 无运行时判断
```



### 循环依赖检测

Spring 处理循环依赖有一套（三级缓存），ArC 更简单直接：**构建时就检测，发现循环就编译失败**。

```java
@Service
public class A {
    @Inject
    B b;
}

@Service
public class B {
    @Inject
    A a; // 循环依赖！
}
```

这段代码在编译时就会报错：

```
Build failure: Circular dependency detected:
  - A depends on B
  - B depends on A
```



### 作用域（Scope）

ArC 支持标准 CDI 作用域：

- `@Singleton`
- `@ApplicationScoped`
- `@RequestScoped`

其中 `@ApplicationScoped` 是最常用的，等价于 Spring 的单例。

```java
@ApplicationScoped
public class UserService {
    // 整个应用只有一个实例
}
```



---



## ArC 的局限



**1）不支持动态注册**

因为 ArC 的 Bean 元数据在**编译时**就已固化，无法在应用运行时新增 Bean 定义。



**2）不支持延迟注入**

ArC 基本不支持延迟注入，因为没有这个必要（启动时就已经注入好了，没有额外成本）。

```java
@Lazy
@Inject
private UserService userService;
```



**3）切面代理的限制**

Spring 的 AOP 很灵活，可以对任意方法做切面。

ArC 的切面主要靠拦截器（Interceptor），而且也是编译时生成，灵活性不如 Spring。



---



## 自定义 ArC 扩展

自定义一个简单的 ArC 扩展



**1）创建扩展项目**

```bash
quarkus create extension my-extension
```



**2）编写注解**

```java
@Inherited
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface MyQualifier {
    String value();
}
```



**3）编写处理器**

```java
@BuildStep
void generatedBean(BuildProducer<GeneratedBeanBuildItem> generatedBeans) {
    ClassOutput beansClassOutput = new GeneratedBeanGizmoAdaptor(generatedBeans);
    ClassCreator beanClassCreator = ClassCreator.builder()
        .classOutput(beansClassOutput)
        .className("org.acme.MyBean")
        .build();
    beanClassCreator.addAnnotation(Singleton.class);
    beanClassCreator.close();
}
```

这个 `BuildStep` 是 Quarkus 扩展的核心概念，每个 `@BuildStep` 都会在构建时执行，生成对应的代码或配置。



---



## Reference

| 对比维度 | Spring DI | ArC (Quarkus) |
| :--- | :--- | :--- |
| **注入时机** | 运行时 | 构建时 |
| **实现机制** | 反射 + 动态代理 | 代码生成 |
| **启动性能** | 慢 | 快 |
| **内存占用** | 高 | 低 |
| **灵活性** | 高 | 中 |
| **动态特性** | 支持 | 基本不支持 |

**一句话总结**：Spring DI 追求「灵活」，ArC 追求「高效」。

- 如果你在云原生、K8s、Serverless 场景，追求的是启动快、内存省，ArC 这种构建时注入的设计非常适合。

- 如果你追求的是框架的灵活性和动态特性，Spring 依旧是更好的选择。

两者不是替代关系，而是针对不同场景的互补方案。

---

## Refrence

- [Quarkus CDI Reference](https://quarkus.io/guides/cdi-reference)
- [CDI 4.0 Specification](https://jakarta.ee/specifications/cdi/4.0/)
- [Quarkus Build Time Processing](https://quarkus.io/guides/building-native-image)
