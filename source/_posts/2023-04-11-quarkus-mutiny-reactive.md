---
title: Quarkus 响应式编程 - Mutiny 
date: 2023-04-11 00:00:00
categories:
  - Backend Engineering
tags:
  - Java
  - Quarkus
  - Reactive
excerpt: "Mutiny 是 Quarkus 推荐的响应式编程库，对比理解 Spring WebFlux Reactor。"
aiSummary: "Mutiny 是 Quarkus 生态的响应式编程库，提供 Uni（单值异步）和 Multi（多值异步）两种类型。本文从响应式编程的核心概念讲起，对比了 Mutiny 与 Spring WebFlux（Reactor）的区别，详细介绍了 Uni、Multi 的基本用法、转换操作符、组合操作符，以及在 RESTEasy Reactive 和响应式消息。"
---



## TL;DR

- Mutiny = Quarkus 的响应式编程库，提供 `Uni` 和 `Multi` 两种核心类型
- `Uni<T>` = 异步单值，类似 RxJava 的 `Maybe`
- `Multi<T>` = 异步多值，类似 RxJava 的 `Flowable`
- 👉 对比 Spring 的 `Mono` / `Flux`，Mutiny 更好上手
- 👉 结合 Quarkus 的 RESTEasy Reactive，写响应式 REST 接口非常舒服

<!-- more -->

---

## Intro

看过 Go 并发模型的一定看到过一句话

> 不要通过共享内存来通信，而是通过通信来共享内存。

响应式编程其实是同样的哲学，只是表达方式不同。
- Spring 生态：**Reactor**（`Mono` / `Flux`）
- Quarkus 生态：**Mutiny**。



**为什么需要响应式编程？**

场景出发：假设有个接口要调用三个服务，然后把结果拼在一起：

```java
// 阻塞写法
public UserVO getUser(Long id) {
    User user = userService.getUser(id);           // 等 100ms
    List<Order> orders = orderService.getOrders(id); // 等 200ms
    List<Address> addresses = addressService.getAddresses(id); // 等 150ms
    return new UserVO(user, orders, addresses);    // 组装
}
```

三个请求是串行的，总耗时 = 100 + 200 + 150 = 450ms。

```java
// 优化：异步写法
public CompletableFuture<UserVO> getUser(Long id) {
    CompletableFuture<User> userFuture = userService.getUserAsync(id);
    CompletableFuture<List<Order>> ordersFuture = orderService.getOrdersAsync(id);
    CompletableFuture<List<Address>> addressesFuture = addressService.getAddressesAsync(id);

    return CompletableFuture.allOf(userFuture, ordersFuture, addressesFuture)
        .thenApply(v -> new UserVO(
            userFuture.join(),
            ordersFuture.join(),
            addressesFuture.join()
        ));
}
```

三个请求并行了，总耗时 = max(100, 200, 150) = 200ms。

看起来更快了，但是如果订单服务返回空、地址服务超时了呢？错误处理怎么写？重试逻辑怎么加？

抛开这些复杂度问题，还有一个更本质问题 --> **线程模型**
- 阻塞模型意味着每个请求占用一个线程，IO 等待期间线程被浪费。
- 并发上涨场景：线程数暴涨，上下文切换开销变大 --> 整个系统吞吐受限


👉 **Mutiny 就是来解决这些问题的**。
- 用更少的线程，处理更多的请求，并且让异步逻辑更容易组合


---


## Mutiny 核心类型

Mutiny 的两种核心类型：

| 类型 | 含义 | 比喻 | 类似 |
| :--- | :--- | :--- | :--- |
| `Uni<T>` | 异步单值 | 未来的某个 T | `Optional<T>`, `CompletionStage<T>` |
| `Multi<T>` | 异步多值 | 未来的某个列表 | `Flux<T>` |




### Uni<T>：异步单值

一个 Uni 代表**一个**异步结果，有也行，没有也行。

```java
Uni<User> userUni = userService.findById(1L);
```

**创建方式**：

```java
// 从已存在的值
Uni<User> fromValue = Uni.createFrom().item(user);

// 从 null
Uni<User> fromNull = Uni.createFrom().nullItem();

// 从异常
Uni<User> fromError = Uni.createFrom().failure(new RuntimeException("oops"));

// 从异步任务
Uni<User> fromCallable = Uni.createFrom().call(() -> {
    return CompletableFuture.supplyAsync(() -> findUser());
});

// 手动完成
Uni<User> manual = Uni.createFrom().publisher(subscriber -> {
    subscriber.emit(user);
    subscriber.complete();
});
```

**订阅和处理**：

```java
userUni
    .onItem().transform(user -> user.getName().toUpperCase())
    .onItem().invoke(name -> System.out.println("Name: " + name))
    .onFailure().invoke(ex -> System.err.println("Error: " + ex.getMessage()))
    .subscribe().with(
        name -> System.out.println("Final: " + name),
        error -> System.err.println("Failed: " + error)
    );
```

**链式调用**：可以看到 Mutiny 大量使用链式调用，每个操作符都返回新的 Uni，这种风格非常流畅。



### Multi<T>：异步多值

一个 Multi 代表**多个**异步结果，可以是零个、一个或多个。

```java
Multi<User> userMulti = userService.findAll();
```

**使用场景**：

- 分页查询
- 实时数据流
- SSE（Server-Sent Events）

**创建方式**：

```java
// 从集合
Multi<Integer> numbers = Multi.createFrom().items(1, 2, 3, 4, 5);

// 从 publisher
Multi<String> fromPublisher = Multi.createFrom().publisher(Flux.just("a", "b", "c"));

// 手动控制
Multi<Integer> manual = Multi.createFrom().producer(subscriber -> {
    subscriber.emit(1);
    subscriber.emit(2);
    subscriber.emit(3);
    subscriber.onCompletion();
});
```

**订阅**：

```java
userMulti
    .onItem().transform(user -> user.getName())
    .filter(name -> name.startsWith("A"))
    .subscribe().with(
        name -> System.out.println("User: " + name),
        error -> System.err.println("Error: " + error),
        () -> System.out.println("Completed!")
    );
```


---


## 基本操作符

Mutiny 提供了丰富的操作符，跟 RxJava 类似。



### 转换操作符

```java
// transform: 同步转换
userUni.onItem().transform(user -> user.getName());

// transformToUni: 异步转换（链式调用另一个异步操作）
userUni.onItem().transformToUni(user -> userService.getDetail(user.getId()));

// flatMap: 拍平（类似 JS 的 Promise chain）
userUni.onItem().flatMap(user -> detailService.getDetail(user.getId()));
```



### 过滤操作符

```java
// filter: 过滤
userMulti.filter(user -> user.getAge() > 18);

// take: 取前 N 个
userMulti.take().atMost(10);

// distinct: 去重
userMulti.distinct();
```



### 组合操作符

```java
// merge: 合并多个 Multi
Multi<User> merged = Multi.createBy().merging()
    .streams(userMulti1, userMulti2, userMulti3);

// combine: 组合（配对）- Uni 版本
Uni.combine().all()
    .unis(userUni, orderUni)
    .asTuple()
    .onItem().transform(tuple -> new UserOrder(tuple.getItem1(), tuple.getItem2()));

// zip: 拉链式组合
Uni<UserOrder> zipped = Uni.zip().all()
    .unis(userUni, orderUni)
    .combinedWith(UserOrder::new);
```



### 错误处理

```java
// onFailure: 捕获异常
userUni.onFailure().invoke(ex -> log.error("Error", ex));

// recoverWithItem: 异常时返回默认值
userUni.onFailure().recoverWithItem(fallbackUser);

// recoverWithUni: 异常时执行另一个异步操作
userUni.onFailure().recoverWithUni(ex -> fallbackService.getDefault());

// retry: 重试
userUni.retry().withBackOff(Duration.ofSeconds(1)).atMost(3);
```


---


## 能力地图 - REST

### 环境准备

添加依赖：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest</artifactId>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-jackson</artifactId>
</dependency>
```

### 返回 Uni

```java
@Path("/users")
public class UserResource {

    @Inject
    UserService userService;

    @GET
    @Path("/{id}")
    public Uni<User> getUser(@PathParam Long id) {
        return userService.findById(id);
    }
}
```

**注意**：返回值是 `Uni<User>`，不是 `User`。

Quarkus 会自动订阅这个 Uni，等结果返回后再序列化成 JSON。

### 返回 Multi（流式响应）

```java
@GET
@Path("/users")
public Multi<User> getAllUsers() {
    return userService.findAll();
}
```

这种方式适合做**流式响应**，比如 SSE。

```java
@GET
@Path("/stream")
public Multi<String> streamMessages() {
    return messageService.getMessageStream();
}
```

### 接收 Uni 参数

```java
@POST
@Path("/users")
public Uni<Response> createUser(User user) {
    return userService.create(user)
        .onItem().transform(saved -> Response.status(201).entity(saved).build());
}
```


---


## 能力地图 - 响应式消息

Mutiny 不仅能用在 HTTP 层，还能用在消息通信上。

### Kafka 响应式消费

添加依赖：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-messaging-kafka</artifactId>
</dependency>
```

**消费者**：

```java
@Incoming("orders")
@Outgoing("processed-orders")
public Uni<Order> processOrder(Order order) {
    return orderService.process(order);
}
```

消息配置（application.properties）：
```properties
mp.messaging.incoming.orders.connector=smallrye-kafka
mp.messaging.incoming.orders.topic=order-requests
mp.messaging.incoming.orders.group.id=order-processor

mp.messaging.outgoing.processed-orders.connector=smallrye-kafka
mp.messaging.outgoing.processed-orders.topic=order-results
```

**生产者**：

```java
@Inject
@Channel("order-events")
Emitter<Order> orderEmitter;

public void createOrder(Order order) {
    doCreate(order)
        .onItem().invoke(created -> orderEmitter.send(created))
        .subscribe().with(result -> {});
}
```

注意：`Emitter.send()` 是 fire-and-forget 的，不需要返回值。




### AMQP 消息

```java
@Incoming("prices")
public Uni<Void> processPrice(Message<Double> msg) {
    Double price = msg.getPayload();
    return priceService.update(price)
        .onItem().invoke(() -> msg.ack())
        .onFailure().invoke(e -> msg.nack(e))
        .replaceWithVoid();
}
```

---

## 最佳实践

### 什么时候用 Uni，什么时候用 Multi？

```java
// 查询单个 -> Uni
Uni<User> findById(Long id);

// 查询列表 -> Multi
Multi<User> findAll();

// 分页 -> Multi（流式返回每一页）
Multi<User> findAll(Paging paging);
```

### 避免嵌套 Uni

```java
// ❌ 不好：嵌套的 Uni
userUni.onItem().transformToUni(user -> {
    return orderService.findByUser(user.getId())
        .onItem().transformToUni(order -> {
            return addressService.findByUser(user.getId())
                .onItem().transform(address -> new UserDetail(user, order, address));
        });
});

// ✅ 好：链式扁平化
userUni
    .flatMap(user -> Multi.createFrom().item(user)
        .onItem().transformToMulti(u -> orderService.findByUser(u.getId()))
        .onItem().transformToUni(order -> addressService.findByUser(user.getId())
            .onItem().transform(address -> new UserDetail(user, order, address))))
```

### 善用备用值和默认值

```java
// 异常时返回默认值
userService.findById(id)
    .onFailure().recoverWithItem(User.DEFAULT);

// 异常时执行替代逻辑
userService.findById(id)
    .onFailure().recoverWithUni(ex -> fallbackService.getDefault());
```

---

## 总结


Mutiny 是 Quarkus 生态中非常优秀的响应式编程库：

- **概念简单**：`Uni` = 单值，`Multi` = 多值，一目了然
- **API 友好**：链式调用，操作符命名符合直觉
- **集成完善**：RESTEasy Reactive、Messaging（Kafka、AMQP）都有原生支持
- **性能优异**：底层基于 Vert.x，非阻塞、事件驱动



Mutiny VS Spring WebFlux（Reactor）

| 对比维度 | Reactor (Spring) | Mutiny (Quarkus) |
| :--- | :--- | :--- |
| **核心类型** | `Mono<T>`, `Flux<T>` | `Uni<T>`, `Multi<T>` |
| **概念复杂度** | 较高（冷/热流、背压） | 较低（概念直观） |
| **学习曲线** | 陡峭 | 平缓 |
| **文档质量** | 完善 | 清晰 |
| **调试友好度** | 一般 | 更好 |





---

## Reference

- [Mutiny Official Documentation](https://smallrye.io/smallrye-mutiny/)
- [Quarkus Reactive Guide](https://quarkus.io/guides/getting-started-reactive)
- [RESTEasy Reactive Guide](https://quarkus.io/guides/resteasy-reactive)
