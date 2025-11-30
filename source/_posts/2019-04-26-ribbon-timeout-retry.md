---
title: Ribbon - 超时与重试
date: 2019-04-26
categories:
  - 微服务
tags:
  - Ribbon
  - 分布式
excerpt: "梳理 Spring Cloud 中各组件（Ribbon、Feign、Zuul、Hystrix、RestTemplate）的超时与重试机制，分析其配置差异与联系。"
aiSummary: "本文梳理了 Spring Cloud 微服务体系中各组件的超时与重试机制。在 Netflix OSS 生态中，Eureka、Ribbon、Zuul、Feign 等组件各自独立但又相互协作，Spring Cloud 将它们组合以简化分布式开发。文章详细分析了 Ribbon 的重试机制，结合源码说明了重试策略的配置参数（ConnectTimeout、ReadTimeout、MaxAutoRetries 等），并对比了与 Feign、Zuul、Hystrix 在超时配置上的差异和联系。"
---

### 前言

在上篇《[源码分析——客户端负载 Netflix Ribbon](/posts/ribbon-source-analysis)》中提到了**重试**。在 Spring Cloud 中，各组件关于重试的概念其实很容易混淆。

你会发现在 Netflix OSS 中：Eureka, Ribbon, Zuul, Feign 等等，虽然每个功能组件都很清晰，能够单独使用。在 Spring Cloud 中看似也很清晰，但实际上 Spring Cloud 的整合中会把多个组件配套组合起来，以达到简化分布式开发的目的。

例如，在深入实践组件 Feign 时，你是不是需要了解 feign-hystrix、ribbon 相关的东西呢？
再例如超时机制，Zuul、Feign、Hystrix 还是 RestTemplate，它们都有 timeout 相关的概念，retry 机制也类似……

还好关于各组件中的超时和重试，已经有前人对其做了详细总结，附上原文链接：
- [Spring Cloud 各组件超时总结](http://www.itmuch.com/spring-cloud-sum/spring-cloud-timeout/)
- [Spring Cloud 各组件重试总结](http://www.itmuch.com/spring-cloud-sum/spring-cloud-retry/)

> ^_^ 感谢这些分享技术的人。

<!-- more -->

---

### 补充分析

由于原文中的总结比较概括，这里结合源码，再对重试机制做一些补充记录。

#### 1. spring-retry 依赖
在 Ribbon 的重试中，除了配置项，还需要在类路径中加入 `spring-retry` 包。

根据 [官方文档说明](http://cloud.spring.io/spring-cloud-static/Finchley.SR2/single/spring-cloud.html#retrying-failed-requests)：
> 当你在使用 Spring Cloud Netflix 提供的负载均衡（如 RestTemplate, Ribbon, Feign, Zuul）时，如果想要实现请求失败后的自动重试，必须在 classpath 中包含 `Spring Retry`。只有当它存在时，重试机制（假定配置允许）才会被自动触发。

**POM 依赖：**
```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

#### 2. 重试次数配置项 (int)

文中提及了三个核心配置项参数：
```yaml
ribbon:
  # 同一实例最大重试次数，不包括首次调用
  MaxAutoRetries: 1
  # 重试其他实例的最大重试次数，不包括首次所选的server
  MaxAutoRetriesNextServer: 2
  # 是否所有操作都进行重试
  OkToRetryOnAllOperations: true
```

在 `ribbon-core` 源码的 `DefaultClientConfigImpl` 中可以找到它们对应的默认值：
```java
public class DefaultClientConfigImpl implements IClientConfig {
    // 重试下一实例 (MaxAutoRetriesNextServer) 默认值：1
    public static final int DEFAULT_MAX_AUTO_RETRIES_NEXT_SERVER = 1; 

    // 重试相同实例 (MaxAutoRetries) 默认值：0
    public static final int DEFAULT_MAX_AUTO_RETRIES = 0;
    
    // OkToRetryOnAllOperations 默认值：false
    public static final Boolean DEFAULT_OK_TO_RETRY_ON_ALL_OPERATIONS = Boolean.FALSE;
}
```

这里要理解前两个参数的意义，可以通过以下公式推算出**总请求次数**（包含首次请求）：
> `Total Requests = (MaxAutoRetries + 1) * (MaxAutoRetriesNextServer + 1)`

- 当 `MaxAutoRetries=1`, `MaxAutoRetriesNextServer=2` 时：总请求次数 = `(1+1) * (2+1) = 6次`。
- 当 `MaxAutoRetries=0`, `MaxAutoRetriesNextServer=1` 时（默认情况）：总请求次数 = `(0+1) * (1+1) = 2次`。

> **技术严谨性提示**：在计算重试次数时，务必注意**总耗时**的叠加。如果单次请求的 Timeout 设为 2 秒，重试 6 次，极端情况下该请求会阻塞长达 12 秒。这对前端用户体验和后端的线程池资源都是灾难性的。

#### 3. 配置项细节 (bool)

上面三个参数中还有一个 bool 参数：`OkToRetryOnAllOperations`。
字面意思是“是否所有操作都进行重试”，但这到底指什么操作？第一眼很难看懂。

细节尽在源码中：
```java
@Override
public RequestSpecificRetryHandler getRequestSpecificRetryHandler(
        RibbonRequest request, IClientConfig requestConfig) {
    if (this.ribbon.isOkToRetryOnAllOperations()) {
        return new RequestSpecificRetryHandler(true, true, this.getRetryHandler(), requestConfig);
    }
    if (!request.toRequest().method().equals("GET")) {
        return new RequestSpecificRetryHandler(true, false, this.getRetryHandler(), requestConfig);
    }
    else {
        return new RequestSpecificRetryHandler(true, true, this.getRetryHandler(), requestConfig);
    }
}
```

可以看到，`isOkToRetryOnAllOperations` 在这里影响了后续的两个分支判断：
1. 当配置为 `true` 时，直接返回 `true, true`。
2. 当配置为 `false` 时，会去判断 HTTP Method：
   - 如果是 `GET` 请求：返回 `true, true`。
   - 如果不是 `GET` 请求（如 POST, PUT 等）：返回 `true, false`。

差异在于 `RequestSpecificRetryHandler` 构造函数中传入的第二个 bool 参数：
- 第一个参数 `okToRetryOnConnectErrors`：重试连接错误（始终为 true）。
- 第二个参数 `okToRetryOnAllErrors`：重试所有错误（非 GET 请求时被置为 false）。

**如何理解这两个参数？**
你只需要区分两种常见的 Exception：
- `java.net.ConnectException: Connection refused`：连接被拒绝（服务不可达），此时客户端连请求都没发出去。
- `java.net.SocketTimeoutException: Read timed out`：服务器**已经接收到请求并在处理**，但客户端等待响应超时了。

因此，这里的重试主要是指目标服务不可达或无法正常响应，而**不是**指返回了 4xx、5xx 业务错误码就去重试。
> **注意**：`OkToRetryOnAllOperations` 逻辑中判断的 HTTP Method，作用的是服务提供者的真实 HTTP 方法，并非调用方本身的代码声明。

#### 4. 关于 OpenFeign 的坑

文中提到了 Feign 的重试。虽然 Feign 和 Ribbon 重试是独立的，如果都启用可能会让重试次数指数级叠加。所以 Spring Cloud 在后来版本改为 `feign.Retryer#NEVER_RETRY`，即**默认不开启 Feign 的自身重试**，转而统一使用 Ribbon 的重试配置即可。

但是，如果你使用的是 Spring Cloud Finchley 或更高版本，引入的依赖变成了 `spring-cloud-starter-openfeign`。
这里有几个大坑需要注意：
- OpenFeign 内部有自己的重试机制实现。
- `spring-retry` 依赖对于 OpenFeign 是不起作用的！
- 但是，OpenFeign 在底层依然会读取 `ribbon.MaxAutoRetries` 和 `ribbon.MaxAutoRetriesNextServer` 这两个配置项并生效。

详细源码分析可参考：[Spring Cloud Finchley OpenFeign 的重试配置相关的坑](https://blog.csdn.net/zhxdick/article/details/89490339)

---

### 四、使用建议

重试机制确实能提高微服务调用的容错和可用性，但是**使用不当不如不要使用**！

**1. Hystrix 超时时间必须大于 Ribbon 重试总耗时**
这点很容易理解，如果 Hystrix 的熔断超时时间小于 Ribbon 的重试总耗时，那么请求还没来得及重试完，Hystrix 就直接把链路熔断降级了，重试机制形同虚设。

**2. 务必考虑服务幂等性**
这点从 `OkToRetryOnAllOperations` 的源码逻辑就能看出来。在启用重试时，一定要考虑下游服务接口的幂等性：
- 如果你的服务 API（尤其是 POST/PUT 插入或更新数据的接口）没有完全实现幂等性控制，**绝对不要**把 `OkToRetryOnAllOperations` 设置为 `true`，否则极易产生脏数据。
- 其次，即使你把它设置成了默认的 `false`（只对 GET 请求重试所有错误），但如果你下游服务的 GET 接口设计不规范（比如 GET 方法里竟然包含了资源写入操作），同样无法满足幂等要求，这种情况下强烈建议在业务层面完全规避或彻底关闭该链路的重试机制。