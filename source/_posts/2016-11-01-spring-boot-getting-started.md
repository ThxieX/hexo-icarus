---
title: Spring Boot 初探
date: 2016-11-01
categories:
  - Backend Engineering
tags:
  - Java
  - Spring
  - Spring-Boot
  - 源码
excerpt: "Spring Boot 入门指南，介绍自动配置、启动类机制、常用注解及 SSM 项目快速重构为 Spring Boot 的方法。"
aiSummary: "Spring Boot 是 Spring 生态的颠覆性框架，通过自动配置彻底告别了传统的 XML 配置时代。本文记录了 Spring Boot 的核心特性：@SpringBootApplication 启动类的秘密（隐含了 @Configuration、@EnableAutoConfiguration、@ComponentScan 三大注解）、嵌入式 Web 服务器（Tomcat/Jetty）、starter 依赖简化理念、以及将传统 SSM 项目重构为 Spring Boot 的实践步骤。是学习 Spring Boot 和理解 Spring Boot 自动配置原理的基础入门文章。"
---

### 前言

工作中要用到 Spring Boot 和 Spring Cloud 了！

之前一直在用传统的 Spring MVC 搭建项目，各种 XML 配置繁琐不堪。下面对初学 Spring Boot 中几个比较核心的特性进行归纳总结。

<!-- more -->

---

### 一、启动类机制

一个最基础的 Spring Boot 启动类通常长这样：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        // 负责启动引导应用程序
        SpringApplication.run(Application.class, args);
    }
}
```

其中核心是 `@SpringBootApplication`，它用于开启组件扫描和自动配置。它实际上是一个复合注解，主要包含以下三个核心注解：

1. `@Configuration`：表明该类是一个基于 Java 的配置类。
2. `@ComponentScan`：开启组件扫描，自动发现控制器类和其他组件，并注册为 Spring 应用程序上下文的 Bean。
3. `@EnableAutoConfiguration`：开启 Spring Boot 的自动配置功能。

所以它一般加在需要启动引导和自动配置的主类上。

---

### 二、起步依赖 (Starter)

Gradle 和 Maven 都是 Spring Boot 为应用程序提供的构建工具。以 Maven 为例：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- 其他依赖... -->
</dependencies>
```

**父起步依赖**：从 `spring-boot-starter-parent` 中继承版本号，把它作为上一级，利用 Maven 的 dependencyManagement 机制统一管理依赖版本。所以在下面的 `dependency` 中就不用再写版本号了。

> **核心价值**：除了手工添加的一些特定包以外，其他以 `spring-boot-starter-*` 打头的都是 Spring Boot 的起步依赖。如果没有这些依赖，你可能需要考虑实现一个 Web 功能需要引入哪些核心包，它们之间是否兼容等问题。事实上我们需要关注的，仅仅是“功能”而已。通过引入一个 starter，传递依赖会自动帮你加一大堆互相兼容的库。

此外，你依然可以使用 `<exclusions>` 来排除和覆盖某个起步依赖中的特定包。

---

### 三、自动配置 (Auto-Configuration)

Spring Boot 的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程。

每当应用程序启动的时候，Spring Boot 的自动配置都要做将近数百个这样的决定，涵盖安全、集成、持久化、Web 开发等诸多方面。所有这些自动配置就是为了尽量不让你自己写冗长的 XML 或 Java 配置。

简而言之：**自动配置让我们专注于应用程序代码本身**。

---

### 四、条件化配置 (Conditional)

在 `spring-boot-autoconfigure` 包中，提供了用于 Spring MVC、Spring Data JPA、Thymeleaf 等等各种框架的自动配置类。

这些配置虽然存在，但可以根据需要在满足某个条件时才生效（或被忽略）。

例如，我们要对 `JdbcTemplate` 进行自动配置，假设条件为：**只有在 Classpath 里存在 `JdbcTemplate` 类时才会生效**。

**1. 首先实现 Condition 接口，覆盖 matches 方法：**

```java
public class JdbcTemplateCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        try {
            context.getClassLoader().loadClass("org.springframework.jdbc.core.JdbcTemplate");
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```

**2. 在声明 Bean 时应用该条件：**

```java
@Conditional(JdbcTemplateCondition.class)
public MyService myService() {
    // ...
}
```

这就表示：只有当 `JdbcTemplateCondition` 类中的条件成立（即 classpath 中确实引入了 JDBC 相关的 jar 包），才会创建 `myService` 这个 Bean。

> `@Conditional` 只是 Spring 框架中最基础的条件注解。Spring Boot 在此之上封装了大量丰富的条件注解（如 `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty` 等），承担起了配置 Spring 的重任。

---

### 五、自定义与覆盖配置

虽然自动配置很强大，但我们经常需要覆盖它。

#### 5.1 覆盖自动配置类

例如，我们要覆盖 Spring Security 的默认安全配置，首先要创建自己的安全配置类：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // 自己的安全拦截逻辑...
}
```

此时，Spring Boot 会跳过自动的安全配置，转而使用我们的 `SecurityConfig`。这是怎么做到的呢？

看个更具体的例子，Spring Boot 在 `DataSourceAutoConfiguration` 中定义 `JdbcTemplate` Bean 的源码片段：

```java
@Bean
@ConditionalOnMissingBean(JdbcOperations.class)
public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(this.dataSource);
}
```

这里的 `@ConditionalOnMissingBean` 注解就是允许覆盖自动配置的关键——它要求当前上下文中**不存在** `JdbcOperations` 类型（`JdbcTemplate` 实现了该接口）的 Bean 时才生效。

**问题来了：什么情况下会已经存在一个了呢？**

> **【技术原理解析】**：
> Spring Boot 的设计原则是：**先加载用户自定义的应用级配置，然后再加载框架的自动配置类**（这通常是通过 `@AutoConfigureAfter`、`@AutoConfigureBefore` 以及 Spring 的 Bean 加载顺序机制实现的）。
> 
> 因此，如果你已经在自己的 `@Configuration` 类中手动 `@Bean` 配置了一个 `JdbcTemplate`，那么在执行自动配置类时，容器中就已经存在一个 `JdbcOperations` 类型的 Bean 了，于是自动配置的 `jdbcTemplate()` 方法就会被忽略。

#### 5.2 自动配置微调 (Properties)

如果不想彻底覆盖 Bean，只想改点参数怎么办？
Spring Boot 自动配置的 Bean 提供了 300 多个用于微调的属性，能够从多种属性源获得值。最常用的就是通过 `application.properties` 或 `application.yml` 进行微调。

#### 5.3 Profile 多环境配置

在实际开发中，我们通常需要区分 `dev`（开发）、`test`（测试）、`prod`（生产）等环境。

**场景 1：某个配置类只在特定环境生效**
如果一个自定义的安全验证只用在生产环境，怎么做？
- Step 1：在你自定义的配置类上加入 `@Profile(value="production")`
- Step 2：在 `application.properties` 中激活该环境：`spring.profiles.active=production`
*(注意：如果 active 没有配置为 production，这个自定义的安全配置类在启动时就会被完全忽略！)*

**场景 2：不同环境需要不同的属性值**
例如：
- 生产环境中，想把日志写到文件里，且只记录 WARN 或更高级别。
- 开发环境中，只想把日志输出到控制台，且记录 DEBUG 级别方便排错。

此时可以创建额外的 properties 文件，遵守 `application-{profile}.properties` 的命名规范：

```properties
# application-development.properties
logging.level.root=DEBUG

# application-production.properties
logging.path=/var/logs/
logging.file=BookWorm.log
logging.level.root=WARN
```

启动时通过指定不同的 active profile，Spring Boot 就会自动去合并加载对应环境的配置文件。