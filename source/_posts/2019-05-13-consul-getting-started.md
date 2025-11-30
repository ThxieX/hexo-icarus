---
title: Consul 基本使用
date: 2019-05-13
categories:
  - 微服务
tags:
  - Consul
  - 分布式
excerpt: "介绍 Consul 服务网格解决方案，实现服务发现和配置管理的分布式系统能力，包括单机部署和 Spring Cloud 集成。"
aiSummary: "Consul 是分布式服务网格（Service Mesh）解决方案，核心提供**服务发现**和**配置管理**能力。本文记录了 Consul 单机开发环境（dev 模式）的部署启动过程，以及 Spring Cloud Consul 的集成使用，包括服务注册与发现、配置中心（KV 存储）的实践操作。"
---

## 前言

官方将 Consul 定义为一个分布式服务网格（Service Mesh）解决方案。在微服务架构中，Consul 主要提供了分布式系统中的**服务发现**和**配置管理**能力。Consul 基于 Go 语言实现，其轻量、高性能的特性使其在微服务生态中广受欢迎（例如在 GitHub 上的 Star 数曾远超 Netflix Eureka）。

关于 Consul 的架构、功能及其与竞品的对比，建议参考官方文档：
- [What is Consul?](https://www.consul.io/intro/index.html)
- [Consul vs. Other Software](https://www.consul.io/intro/vs/index.html)

**本文主要目的：**
- 部署并启动 Consul 单机开发环境（dev 模式）
- 掌握 Spring Cloud Consul 的基本使用
- 实践 Consul 中的**服务发现**机制
- 实践 Consul 中的**配置中心**功能
- 实践 Consul 集群部署 (下节)~~

<!-- more -->

---

## 下载与启动

访问[官方下载地址](https://www.consul.io/downloads.html)下载对应的版本，解压后即可得到可执行文件（本文测试版本为 `1.4.4`）。

进入命令行环境：

1. **验证版本**：
```bash
$ consul --version
Consul v1.4.4
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

2. **启动开发环境**：
```bash
$ consul agent -dev
```

正常启动后，浏览器访问 `http://127.0.0.1:8500` 即可看到 Consul 提供的 Web UI 界面（界面效果可参考 [官方 Demo](https://demo.consul.io/)）。

> **提示**：上述 `-dev` 模式重启后不会保存数据。如果需要数据持久化，可以使用以下命令启动：
> ```bash
> $ consul agent -server -bootstrap -advertise 127.0.0.1 -data-dir ./data -ui
> ```

Consul 的日常操作主要有两种方式：CLI（命令行）和 Web UI。下文将结合这两种方式进行演示。

---

## Spring Cloud Consul 实践

Spring Cloud Consul 为 Spring Boot 应用提供了基于 Consul 的服务治理与配置管理集成方案。

### 1. 依赖配置

可以在项目中直接引入 `all` 依赖以包含所有核心功能：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-all</artifactId>
</dependency>
```

上述依赖实际上是一个聚合包，包含了以下三个核心模块，也可以根据实际需求按需引入：
- `consul-discovery`：服务注册和发现功能
- `consul-bus`：消息总线，提供配置实时刷新功能（Consul 自身支持，不再强依赖外部 MQ）
- `consul-config`：分布式配置中心

### 2. 健康检查

Spring Cloud Consul 默认会自动调用应用端点 `/actuator/health` 进行健康检查。

建议直接集成 `spring-boot-starter-actuator`，或者在应用中自定义该端点：

```java
@GetMapping("/actuator/health")
public String health() {
    return "OK";
}
```

启动后，在 Consul UI 的服务详情页中可以看到类似如下的检查结果：
```text
HTTP GET http://thank-pc:8801/actuator/health: 200  Output: OK
```

### 3. 服务间调用示例

准备两个微服务模块：`consul-provider`（服务提供者）与 `consul-consumer`（服务消费者）。

#### 服务提供者 (consul-provider)

1. **引入依赖**：参考上述 POM 配置。
2. **配置文件**：`bootstrap.yml`
```yaml
spring:
  application:
    name: consul-provider
  cloud:
    consul:
      host: 127.0.0.1
      port: 8500
```
3. **提供测试接口**：
```java
@RestController
public class ProviderController {
    @Value("${server.port}")
    private String port;

    @GetMapping("/getHello")
    public String getHello(@RequestParam String name) throws Exception {
        return String.format("[%s:%s] Hello %s", InetAddress.getLocalHost().getHostName(), this.port, name);
    }
}
```

#### 服务消费者 (consul-consumer)

1. **引入依赖**：参考上述 POM 配置。
2. **配置文件**：`bootstrap.yml`
```yaml
spring:
  application:
    name: consul-consumer
  cloud:
    consul:
      host: 127.0.0.1
      port: 8500
      discovery:
        register: true # 默认为 true，如果不希望 consumer 被发现也可设为 false
```
3. **调用接口**：
可以通过 `RestTemplate` 或 `Feign` 两种常用方式进行调用。

**方式一：RestTemplate**
```java
@RestController
@RequestMapping("/test")
public class ConsumerController {
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/restTemplateInvoke")
    public ResponseEntity<String> restTemplateInvoke(@RequestParam String name) {
        String url = "http://consul-provider/getHello?name=" + name;
        String result = this.restTemplate.getForObject(url, String.class);
        return new ResponseEntity<>(result, HttpStatus.OK);
    }
}
```

**方式二：Feign**
```java
@FeignClient(name = "consul-provider")
public interface HelloClient {
    @GetMapping("/getHello")
    String getHello(@RequestParam("name") String name);
}

@RestController
@RequestMapping("/test")
public class FeignConsumerController {
    @Autowired
    private HelloClient helloClient;

    @GetMapping("/feignInvoke")
    public ResponseEntity<String> feignInvoke(@RequestParam String name) {
        String result = this.helloClient.getHello(name);
        return new ResponseEntity<>(result, HttpStatus.OK);
    }
}
```

#### 测试服务发现与负载均衡

启动 Consul 后，分别启动两个 `consul-provider` 实例和一个 `consul-consumer` 实例。

打开 [Consul UI](http://localhost:8500)，确保所有服务注册成功并处于健康状态。

![Consul UI service](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E4%B8%93%E9%A2%98/%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0/consul_service.png)

请求 consumer 的测试接口：
- `GET http://localhost:8801/test/feignInvoke?name=thank`
- `GET http://localhost:8801/test/restTemplateInvoke?name=thank`

多次访问后可以观察到，Consumer 成功从 Consul 获取到了 Provider 的实例列表，并实现了客户端的负载均衡路由。

---

## 配置中心机制

Consul 不仅支持服务发现，还内置了 Key/Value 存储功能，这使其能够作为分布式配置中心使用。

### 1. 基础 KV 规则

首先通过 CLI 演示基础的读写操作（也可使用 UI 面板）：

```bash
# 写入配置
$ consul kv put name thank
Success! Data written to: name

# 读取配置
$ consul kv get name
thank
```

为了区分不同服务和环境（Profile），Consul 的 Key 支持使用 `/` 进行目录层级划分。例如：

```bash
$ consul kv put config/consul-provider/custom.address 北京

$ consul kv get config/consul-provider/custom.address
北京
```

在 Consul UI 的 [KV 面板](http://localhost:8500/ui/dc1/kv) 中，这种层级结构会以直观的文件夹形式展现。

---

### 2. Spring Cloud Consul 配置实践

与 Spring Cloud Config 相比，基于 Consul 的配置中心接入更加简易。在实现动态刷新时，无需额外部署消息中间件（如 RabbitMQ）并集成 Spring Cloud Bus。

#### 简单动态刷新测试

以 `consul-provider` 为例，测试属性的读取与动态刷新：

1. **本地兜底配置**（`bootstrap.yml`）：
```yaml
custom:
  address: defaultAddress
```

2. **代码读取**（加上 `@RefreshScope` 以支持动态刷新）：
```java
@RestController
@RefreshScope
public class ConfigTestController {
    @Value("${custom.address}")
    public String address;
    
    // 省略相应的 GET 接口代码...
}
```

3. **在 Consul 中添加配置**：
在 UI 面板或命令行中添加 KV：
- **Key**: `config/consul-provider/custom.address`
- **Value**: `北京`

启动服务并访问对应接口，此时读取到的值为 Consul 中的配置（`北京`）。在 Consul UI 中修改该值，无需重启服务，再次访问接口即可看到配置已动态刷新。

**核心注意点**：
- **配置文件优先级**：所有涉及从配置中心加载逻辑的配置项，必须放在 `bootstrap.yml` 中，而非 `application.yml`。
- **默认 Key 前缀解析**：`config/consul-provider/custom.address` 的结构拆解为：
  - `config`：Spring Cloud Consul 的默认根目录（可通过 `spring.cloud.consul.config.prefix` 修改）。
  - `consul-provider`：对应的微服务名称（可通过 `default-context` 修改）。
  - `custom.address`：映射到 Spring 环境中的具体属性。

#### 多环境 (Profile) 配置管理

在实际项目中，通常需要区分 `default`、`dev`、`test` 等多环境。

我们可以在 Consul 中建立三个不同的配置文件（以 YAML 格式存储整个文件的内容，而非单条属性）：

- **Default 环境** (`consul-provider/data`)：
```yaml
custom:
  env: default
  common: some common properties
```

- **Dev 环境** (`consul-provider,dev/data`)：
```yaml
custom:
  env: dev
```

- **Test 环境** (`consul-provider,test/data`)：
```yaml
custom:
  env: test
```

在应用的 `bootstrap.yml` 中开启 YAML 解析并指定激活的环境：
```yaml
spring:
  cloud:
    consul:
      config:
        format: YAML
  profiles:
    active: dev
```

通过指定不同的 `spring.profiles.active`，应用将自动读取并合并对应环境的配置。

---

### 3. 配置规则与原理总结

#### 存储格式 (Format)

Consul 支持两种主要的键值存储映射格式：
- **KEY_VALUE**（默认）：每个 Key 对应一个具体的属性，结构清晰但当配置较多时管理繁琐。
- **YAML / PROPERTIES**：将整个配置文件的内容作为一个 Value 存储，Key 以 `data` 结尾（如上文多环境演示），更符合传统开发习惯。

#### 常用配置项参考

以读取目标 `config/consul-provider,test/data` 为例，涉及的 Spring Boot 配置属性（前缀 `spring.cloud.consul.config.*`）如下：

| 配置项 | 含义说明 | 默认值 |
| :--- | :--- | :--- |
| `default-context` | 默认应用级上下文名称（全局配置共享时使用） | `application` |
| `prefix` | Consul 中的根目录前缀 | `config` |
| `profile-separator` | 服务名与环境名之间的分隔符 | `,` |
| `format` | 配置内容解析格式（支持 KEY_VALUE, PROPERTIES, YAML, FILES） | `KEY_VALUE` |
| `data-key` | 当格式为 YAML/PROPERTIES 时，指定的叶子节点 Key | `data` |

#### 读取与覆盖优先级

在 Spring Cloud Consul 中，环境配置会进行级联和合并。如果激活了 `dev` 环境，配置加载的优先级从高到低依次为：

1. `config/{appname},dev/data` （当前服务 Dev 环境配置）
2. `config/{appname}/data` （当前服务 Default 环境配置）
3. `config/application,dev/data` （全局 Dev 环境配置）
4. `config/application/data` （全局 Default 环境配置）

> **最佳实践**：将各个环境通用的配置提取到 Default 层级，将差异化配置放到对应的 Profile 层级，利用高优先级覆盖低优先级的特性，可以有效减少冗余配置。

#### 动态刷新原理解析

- **Spring Cloud Config** 的动态刷新依赖于 `Spring Cloud Bus` 和外部消息队列（MQ）。当配置变更时，通过 MQ 广播通知各个微服务节点执行批量刷新。
- **Spring Cloud Consul** 则更加轻量：客户端会启动一个长轮询（Watch 机制）定期检测 Consul 服务端配置节点的变化。一旦检测到版本号更新，客户端便会主动触发 Spring 的 `RefreshEvent` 事件，进而刷新本地应用上下文。

---

## 参考文档
- [Spring Cloud Consul Config 官方文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-consul/2.1.0.RELEASE/single/spring-cloud-consul.html#spring-cloud-consul-config)