---
title: Quarkus VS Spring Boot
date: 2022-12-23
categories:
  - Backend Engineering
tags:
  - Java
  - Quarkus
  - Spring-Boot
excerpt: "从'习惯'到'理解'框架本质: 用 Spring Boot 的经验去看 Quarkus，从设计理念、依赖注入、响应式编程、启动性能四个维度做对比。"
aiSummary: "Quarkus 是 Red Hat 推出的云原生 Java 框架，定位为「Supersonic Subatomic Java」。本文从 Spring Boot 开发者的视角出发，对比两者的设计理念、依赖注入机制、响应式编程支持和启动性能表现。Quarkus 通过构建时元数据处理和 GraalVM 原生镜像实现毫秒级启动和极低内存消耗，适合 Kubernetes 和 Serverless 场景；Spring Boot 则以生态丰富和社区成熟见长。两者并非替代关系，而是针对不同场景的互补选择。"
---



## TL;DR

- Quarkus：编译时 IoC + GraalVM 原生镜像 = 毫秒启动 + 超低内存
- Spring Boot：运行时反射 + 动态代理 = 灵活 + 启动慢
- 服务要跑在 K8s 里、或者做 Serverless：👉 Quarkus 值得尝试
- 追求生态丰富、文档完善、社区成熟： 👉 Spring Boot 依旧稳如老狗

<!-- more -->

---

## Intro

**Quarkus**：Supersonic Subatomic Java，「超音速亚原子 Java」🚀 Cool！

👉 来自官方的介绍：专为快速启动、高吞吐量和低资源消耗而设计。
- Container First：轻量级 Java 应用程序，最适合在容器中运行。
- Cloud Native：Quarkus 是为 Kubernetes 建立的，使其能够轻松部署应用程序，而无需了解该平台的所有复杂性。
- Versatile：从小型微服务到大型单体应用。
- Fast startup：在构建时完成更多工作，启动速度快。
- Unify imperative and reactive：统一命令式和响应式, 将非阻塞和命令式开发风格统一在一个编程模型下。
- Standards-based：基于你所常用和喜爱的标准与框架（RESTEasy and JAX-RS, Hibernate ORM and JPA, Netty, Eclipse Vert.x, Eclipse MicroProfile, Apache Camel...）



📦 All under ONE framework.





🔍 **从自己熟悉的视角出发，理解 Quarkus 在解决什么问题 ？**

我们知道 Spring Boot 通过自动配置，内嵌容器等方式已经做到了开箱即用，是 Spring 生态顺应云原生趋势的进步。

Quarkus 不是 Spring Boot 的替代品，而是一个针对特定场景的补充 👉 让你的服务在云上跑得更舒服。



它解决的核心问题是：**传统 Java 框架在云原生场景下太重了**。
- Spring Boot：灵活、成熟、生态好，适合大多数场景
- Quarkus：快、轻、省，适合 K8s / Serverless / 容器优先的场景



---

## 设计理念



两条不同的路：Spring Boot 把能做的都留到运行时做，Quarkus 把能做的都提前到编译时做。


| 对比维度 | Spring Boot | Quarkus |
| :--- | :--- | :--- |
| **核心理念** | 「约定大于配置」，简化 Spring 开发 | 「容器优先」，为云原生而生 |
| **设计哲学** | 运行时灵活，依赖注入、反射、AOP 全上 | 编译时搞定一切，运行时要快要轻 |
| **启动时间** | 慢（秒级） | 极快（毫秒级，GraalVM 原生更是变态） |
| **内存占用** | 高（至少几百 MB） | 低（原生镜像可以 <100MB） |
| **生态** | 极其丰富 | 相对年轻但覆盖核心场景 |




---



## 依赖注入



**ArC vs Spring DI** 




Spring 的依赖注入依赖 **运行时反射**：

1. 启动时扫 classpath，找到所有 `@Component` / `@Service`
2. 通过反射拿到构造函数，字段，方法参数等
3. 动态创建 bean 实例
4. 根据 `@Autowired` 找到依赖关系，注入

运行时反射赋予了 Spring 强大的动态能力，但这套魔法的问题在于：**反射是要成本的**



Quarkus 的 ArC（Arc = Annotation + CDI），基于 CDI Lite 标准。核心区别在于：

- **编译时**：Quarkus 的构建工具会扫描你的代码，生成直接的依赖注入代码
- **运行时**：不再需要反射，直接调用生成好的代码

简单理解就是：Quarkus 把「边跑边组装」换成「跑之前就组装好」




例如我们最熟悉的注入，写法上仅是一个注解的区别，但 ArC 在编译时就把 `@Inject` 的目标解析好了，不再是运行时反射。

```java
@Service
public class UserService {

		// Spring Boot
    @Autowired
    private UserRepository userRepository;
}


@Service
public class UserService {

    // Quarkus
    @Inject
    private UserRepository userRepository;
}
```





---


## 响应式编程


Spring Boot 中 WebFlux 用的是 Project Reactor（`Mono` / `Flux`），Quarkus 中的响应式编程用的是 **Mutiny**。

相比之下，Reactor 的概念有点绕，而 Mutiny 似乎更好上手。

例如 Flux 可以是 0..N 也可以是 1..N，很有认知负担，而 Mutiny 看起来更加直观：

- `Uni<T>`：异步结果，有且只有一个结果
- `Multi<T>`：多个异步结果，类似于 `Flux`，可以有零个或多个


简单对比示例代码


```java
// Spring WebFlux
@Service
public class UserService {
    public Flux<User> findAll() {
        return userRepository.findAll();
    }
}

// Quarkus Mutiny
@Service
public class UserService {
    public Multi<User> findAll() {
        return userRepository.findAll();
    }
}
```


阻塞 vs 非阻塞示例代码


```java
// Spring WebFlux
@GetMapping("/{id}")
public Mono<User> getUser(@PathVariable Long id) {
    return userRepository.findById(id); // 看起来像同步，但底层是非阻塞
}

// Quarkus RESTEasy Reactive + Mutiny
@GetMapping("/{id}")
public Uni<User> getUser(@PathVariable Long id) {
    return userRepository.findById(id);
}
```

两者都是非阻塞的，但 Quarkus 的区别在于：如果你不加 `Uni`，直接返回 `User`，它默认就是阻塞的。

是不是看起来 Quarkus 更符合直觉？ 你用 `Uni` 就是非阻塞，用普通类型就是阻塞，没有魔法。


---


## 启动性能

其实无论是理念不同还是 XX 写法的不同，并不足以对 Quarkus 心动 ❤️。

我们更加关注：启动性能作为 Quarkus 的 Slogan，实际跑起来差多少？

> 官方给出的参考数据（不同硬件环境有差异）：

| 指标 | Spring Boot (JVM) | Quarkus (JVM) | Quarkus (Native) |
| :--- | :---: | :---: | :---: |
| **启动时间** | 秒级 | 亚秒级 ~1s | **毫秒级 <0.1s** |
| **内存占用** | 几百 MB | 约 100-200MB | **< 50MB** |
| **JIT 预热** | 需要 | 需要 | **不需要** |

这些数字在不同机器上会有差异，但 **趋势是确定的**：

- Quarkus 在 JVM 模式下已经比 Spring Boot 快很多
- Quarkus + GraalVM 原生镜像 = 开挂级别的性能



🚀 **为什么快？**：核心就三点：

1. **构建时元数据处理**：不需要在启动时扫 classpath、解析注解、创建反射代理
2. **静态代码生成**：把原本运行时做的事，提前到编译时做好
3. **GraalVM 原生镜像**：直接编译成机器码，不需要 JVM，进一步减少内存和启动时间


---


## Reference

- [Quarkus Official Site](https://quarkus.io/)
- [Quarkus Guides](https://quarkus.io/guides/)
