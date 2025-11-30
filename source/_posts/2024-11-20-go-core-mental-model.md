---
title: Go 核心心智模型
date: 2024-11-20 00:00:00
categories:
  - Backend Engineering
tags:
  - Go
excerpt: "Go 核心语言观、如何调度任务，以及如何通过 gRPC 和 Protobuf 在分布式边界上实现极低延迟的通信"
aiSummary: "语言观（值传递、函数一等公民、鸭子类型、闭包）、调度与并发（GMP 模型、sync 并发原语、Channel 通信、Select 多路复用、Context 生命周期管理）、突破进程边界（Protobuf 与 gRPC 契约驱动开发，message 对应结构体、service 对应 RPC 接口）。"
---




## 语言观

大多数现代语言都需要缴纳“运行税”。Python 缴纳解释器税，Java 缴纳 JVM 启动和 JIT 预热税。而 Go 直接砍掉了中间商：
- Go 编译型语言（静态语言），较 c++, Java 编译速度更快
- 直接编译为二进制可执行文件，部署简单，Cloud Native Friendly
- 并发编程简单且天然（goroutine）

```
python 源码  <---- 字节码 ---->  解释器  <---- 二进制 ---->  处理器
java   源码  <---- 字节码 ---->  解释器  <---- 二进制 ---->  处理器
go     源码  <---- 二进制 ---->  处理器
```

<!-- more -->

### 值传递 or 引用传递

🧠一句话结论：Go 中全部是值传递，没有例外！

Slice 引用传递的错觉
```go
func funSlice(data []string) {
	data[0] = "python"	
	data = append(data, "java", "javascript")
	fmt.Println("funSlice: ", data) // [python grpc gin java javascript]
}

func main() {
	courses := []string{"go", "grpc", "gin"}
	funSlice(courses)
	fmt.Println("main: ", courses) // python grpc gin -- 注意 go 变成了 python
}
```

当你在函数中传递 Slice 时，你并没有传递一个对象的引用，你传递的还是一个包含底层数组指针的结构体副本。

```go
type slice struct {
  array unsafe.Pointer  // 用来存储实际数据的数组指针，指向一块连续的内存
  len int               // 切片中元素的数量
  cap int               // Array 数组的容量，长度
}
```

slice 本质是一个 结构体，因为值传递所以 `funSlice` 中的 data 相当于一份 结构体的拷贝，但是 array 指向的是同一个数组，所以修改 `data[0]` 时原数组也发生了改变；

但如果你触发了扩容，你的 Slice 副本将指向一块全新的内存：例如在 `funSlice` 中执行 `data = append(data, "java", "javascript")` 触发了扩容后重新赋值，所以 array 指向了新的（扩容后的）


### 函数是一等公民

👉 "函数是一等公民"
- ❌ 函数很重要
- ✅ 函数在语言层面和 int, string 是一样档次的东西，而非一种只能 “被调用” 的语法

一等公民意味着它可以：
- 赋值给变量
- 作为参数传递进函数
- 作为返回值从函数出来
- 存在数据结构里（slice, map, struct 等）

```go

// 赋值 & 像普通值一样用
func add(a, b int) int { 
	return a + b 
}
f := add 
fmt.Println(f(1, 2))


// 作为参数传递
func apply(x int, fn func(int) int) int {
  retrun fn(x)
}

// 作为返回值（闭包）
func counter() func() int {
	n := 0
	return func() int {
		n ++
		return n
	}
}
```

从 Java8+ 的能力清单看（lambda, Function, 方法引用等），也有 “函数是一等公民” 的一些味道，但函数穿着接口外套的特殊对象，仍带着面向对象的枷锁。


### Ducking type（结构即接口）

> 如果它 ”走“ 起来像鸭子，“叫” 起来也像鸭子，那它就是鸭子（不看 “它是什么”，只看 “它能干什么”）

放在编程里：判断类型是否匹配，不靠显式声明和继承，而是看是否实现了相应方法。
- Java：你身份证显式写 Duck 了吗？
- Go：你会不会 Fly
- Python：你先飞一个我看看

非鸭子类型（Java， C#）
```java
class Duck implements Flyable {
	 public void fly() {}
}

class Duck2 { // 没显式实现 Flyable，不算鸭子
		public void fly() {}
}
```

鸭子类型（Go， Python， JS）
不用显式声明，编译器 / 运行时自己看你会不会飞
```go
type Flyer interface {
	 Fly()
}

type Bird struct {}

func (Bird) Fly() {}

func TakeOff(f Flyer) {
		f.Fly()
}
```

- Bird 不用显式 Implement Flyer, 只要有 `Fly()` 方法，就自动满足 Flyer 接口类型
- Go 属于编译期检查的鸭子类型
- Python 属于运行期鸭子类型

	```python
	''' 传进去的 f 对象如果没有 fly(), 运行时会报 AttributeError '''
	def take_off(f):
		f.fly()
	```


### 闭包

👉 Go 中的闭包 = 函数 + 它所 "记住" 的外部变量环境

先来看

```go
func makeCounter() func() int {
	count := 0
	return func() int {
		count++
		return count
	}
}

func main() {
	c := makeCounter()
	fmt.Println(c()) // 1
	fmt.Println(c()) // 2
	fmt.Println(c()) // 3
}
```

- `makeCounter()` 已经返回了，栈帧按理该没了，但是 `c()` 多次调用 `count` 中的值也是在累加的
- 看起来 GO 编译器把 `count` 提升到了 堆 上
- 一句话：函数活着，变量就活着

再来看

```go
func main() {
	a := 0
	f := func() { a++ }

	f()
	f()
	fmt.Println(a) // 2
}
```

- `a` 明明是 `main` 里的变量，但是 `f` 却没有通过参数，而直接用到并能修改 `a`

  


👉 这就是闭包：函数拿着外面的变量在用，而且拿的是 “原件”，不是 “复印件”

Go 里的闭包 = 函数在 ”偷用“ 外面的变量，而且是同一个变量，不是副本。从而产生我们映像中 ”Go 中是值拷贝“ 不一样的结果。闭包不是魔法，它只是干了两件事：
1. 函数能用外层作用域的变量
2. 这些变量会一直活着，而且是共享的


## 调度 & 并发生命

### GMP

> 现代操作系统中：分配资源的基本单位是**进程**，独立运行和调度的基本单位是**线程**。

传统的 OS 线程太重了（兆字节级别的栈、昂贵的上下文切换）。Go 实现了一个极其强悍的用户态调度器：GMP 模型。

> Go 中执行和调度的基本单位是**协程**，所以 Go 中实现了自己的**协程调度模型（GMP）** 
> - G：Goroutine 任务执行单元（初始栈仅 2KB）
> - M：Machine OS 内核态线程
> - P：Processor 调度器/处理器

Go 中的并发并不是 “开很多线程”，而是依靠 调度器/处理器（**P**），用少量的线程（**M**），高效轮转执行大量协程任务（**G**）

**怎么做到 “并发而不乱” ？** 
1. 抢占式调度：G 不让出，谁也跑不了。所以运行太久应该被强制打断
2. 工作队列窃取机制：偷活，P0 忙 P1 闲，P1 可以从 P0 中偷一些 G

**发生阻塞怎么办？** 
假如此时某个 G 正在做 IO `go func() { http.Get(...) }`，阻塞了很久
- P 会立刻解绑这个 M
- 在找一个新 M
- 继续跑比的 G

**必须要有 P 吗？**
P 并不是理论洁癖，而是性能工程的产物：本地队列 + 少锁 + 机制调度  --> 高吞吐 


### Sync

Go 中的 `sync` 包用来解决并发安全和同步协作问题，在多个 goroutine 同时访问一块内存时，保证一致性和可见性。

虽然 Go 的并发哲学是 *“不要通过共享内存来通信， 而要通过通信来共享内存（Channels）”*，但实际开发中直接操作共享变量有时在性能和逻辑简单性上更有优势，所以躲不过去。

**核心并发原语**
sync 包中提供了集中常用的工具，我们可以根据使用场景来选择：

1. `sync.Mutex`（互斥锁）
   这是最基础的排他性锁，一次只允许一个 Gorouine 进入临界区，即一个 Goroutine 获得了锁其他 Goroutine 必须等待它释放后才能继续
	 - 场景：保护一个共享变量，确保同一时刻只有一个协程在读写
	 - 注意：一定记得 `Unlock()` ，通常配合`defer` 使用
2. `sync.RWMutex`（读写锁）
   它是 `Mutex` 的进阶版，遵循 “多读单写” 原则
   - 场景：读多写少的场景（比如配置信息， 缓存等），性能远高于普通 Mutext
   - 特点
	   - 多个 Goroutine 可以同时获取读锁 - RLock
	   - 同一时刻只有一个 Goroutine 获取写锁 - Lock
	   - 当有人在写时，别人既不能读也不能写
3. `sync.WaitGroup`（等待组）
   用来等待一组 Goroutine 执行结束
	- 场景：并发执行多个任务，需要全部执行完成后进行汇总
	- 核心方法：
		- `Add(delta)`：计数器 + N
		- `Done()`：计数器 - 1（通常在协程内部 `defer` 调用）
		- `Wait`：主协程阻塞，直到计数器归零
4. `sync.Once`（只执行一次）
	- 场景：确保某个操作在程序运行期间只执行一次，哪怕它同时被多个协程同时调用（例如单例模式的初始化， 配置文件的加载等），即保证并发下仅执行一次（对其他 goroutine 可见）
5. `sync.Pool`（对象池）
   用来保存和复用临时对象，以减少内存分配和 GC 的压力
	- 场景：高并发下的 `fmt.Printf` 内部缓冲区，数据库连接池中的对象复用

**sync.Map：线程安全的 Map**
Go 原生 map 不是并发安全的，比如多个协程中同时读写一个 map，可能会 panic, 数据不正确。



### Channel

> 不要通过共享内存来通信，而是通过通信来共享内存 - Go 设计哲学


channel 有缓冲和无缓冲：
- 无缓冲 channel：不是在 “传数据”，而是 “约定一个触发/时间点”，适用于通知，例如 B 要第一时间知道 A 怎么样了，从而保证顺序语义。
	```go
	ch := make(chan struct{})
	
	go func() {
		prepare()
		ch <- struct{}{} // 告诉你，我准备好了！
	}
	
	<- ch
	use()
	```
	除此之外，也适用于 并发 -> 串行 的场景
- 有缓冲 channel：典型场景是生产者和消费者通信场景，此外还有弹性缓冲等作用

**生产 - 消费 Demo**（ 有缓冲区 + 单向（约束方向控制） ）
```go
func producer(out chan<- int) { // chan <- ：只进不出的单向 channel
	for i := 0; i < 10; i++ {
		out <- i
	}
	close(out)
}

func consumer(in <-chan int) { // <- chan ：只出不进的单向 channel
	for data := range in {
		fmt.Print(data)
	}
	fmt.Println("consumer done")
}

func main() {
	ch := make(chan int)
	go producer(ch) // 这里双向 ch 会自动转换为单向的 chan <- 入参
	go consumer(ch) // 这里双向 ch 会自动转换为单向的 <- chan 入参

	time.Sleep(10 * time.Second)
}
```

**两个 goroutine 交叉打印数字和字母**
e.g.  `12AB34CD56xxxx` （2数字 2字母 2数字 ...）

```go
// 2 个无缓冲 channel
var number, letter = make(chan bool), make(chan bool)

func printNum() {
	i := 1
	for {
		<-number
		fmt.Printf("%d%d", i, i+1)
		i += 2
		letter <- true
	}
}

func printLetter() {
	i := 0
	str := "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
	for {
		if i >= len(str) {
			return
		}
		<-letter
		fmt.Print(str[i : i+2])
		i += 2
		number <- true
	}
}

func main() {
	go printNum()
	go printLetter()
	number <- true // number 先打印
	time.Sleep(10 * time.Second)
}
```


### Select

> Channel 是路，Select 是红绿灯，Goroutine 是车

Select 类似并发版本的 Switch，但它选的不是值，而是选的 “谁先准备好”；
Select 会同时兼听多个 Channel 通道，哪个操作先不阻塞（就绪），执行哪个 Case
```go
select {
	// ch1 有数据 ？ 走 ch1
	// ch2 有数据 ？ 走 ch2
	// 都没准备好 ？卡住
	// 同时有数据？随机走（避免饥饿）
	case v := <- ch1: 
		fmt.Println("from ch1: ", v)
	case v := <- ch2: 
		fmt.Println("from ch2: ", v)
}
```

**经典用法 - select + timeout**

```go
select {
	case v := <- ch1:
		handle(v)
	case <- time.After(time.Second) // time.After 返回的就是一个 channel
		fmt.Println("timeout")
}
```

**经典用法 - select + for**

```go
// Go 里很常见的 goroutine 生命周期模版
for {
	select {
		case v := <- ch:
			handle(v)
		case <- done:
			return 
	}
}
```


### Context

context 可以被看作是一个在 Gorouine 之间传递信号和数据的 “信使”，用来把 “取消信号，超时，请求级数据” 沿着调用链路进行传递。
不是控制流程，而是控制生命周期。

context 是树状结构组织的，跟节点通常是 `context.Background()`，当你基于一个父 context 创建子 context 时，它们就形成了父子关系。如果父 context 被取消，所有由他衍生的 子 context 也会自动被取消，反之子 context 的取消不会影响父 context。

**context 核心能力**（就三件事）
1. cancel （取消）：当一个任务不再需要时，通知所有 Gorouine 停止
2. timeout / deadline (超时 / 截止时间)：时间一到，自动取消
3. request-scoped values ：trace id, user id 这种，不是业务参数

**context 核心函数**
1. `WithCancel(parent)`
	 - 作用：手动取消
	 - 返回：一个新的 Context 和一个 `cancel` 函数
	 - 场景：当你手动判断任务需要取消时（比如用户关闭了浏览器连接），调用 `cancel()`
2. `WithTimeout(parent, duration)`
	- 作用：超时自动发取消
	- 场景：最常用比如请求 DB 或调用三方 API，如果 3 秒内没有返回就自动放弃
3. `WithDeadline(parent, duration)`
	- 作用：在指定时间点取消
	- 场景：类似于 `WithTimeout` ，但它是指定一个具体时间
4. `WithValue(parent, key, value)`
	- 作用：传递请求作用域数据
	- 场景：传递 Trace ID，用户识别信息等


**Best Practice**
- 作为第一个参数传入： `ctx`
- 不要存储在结构体中：Context 应该随函数调用链路传递，不应该持久化在对象里（生命周期可能不匹配）
- 显式调用 cancel：凡是使用了 `WithCancel` 或 `WithTimeout` 返回的 `cancel` 函数，一定要在函数结束时调用（通常用 `defer cancel()`），避免 Context 泄漏
- WithValue 仅传递元数据：不要把 Context 当做一个万能参数包，业务参数应该通过函数行参正常传递。

**Sample - 超时控制**
```go
func main() {
	// 创建一个 5 秒超时的 context
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	go doSomething(ctx)

	// 阻塞等待
	select {
	case <-ctx.Done():
		fmt.Println("Main: 任务超时或已被手动取消", ctx.Err())
	}
}

func doSomething(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Workder: 收到推出信号，停止工作")
			return
		default:
			fmt.Println("Workder：正常工作")
			time.Sleep(500 * time.Millisecond)
		}
	}
}
```





## 突破进程边界 Protobuf & gRPC

> 🧠Protobuf 解决了 “数据该长什么样”，gRPC 解决了 “如何调用远程服务”



🤔 职责分离的设计哲学
- 👉 gRPC 是一套完整的 RPC 传输框架，它负责 RPC 过程中的多个环节：连接，HTTP/2，多路复用，流控，超时，重试，状态码等。但本质问题还是 “如何把远程服务调用伪装成本地函数调用”。
- 👉 Protobuf 是一套 IDL（接口定义语言） + 序列化协议，它负责数据结构定义，字段编号，编码规则，代码生成，前后向兼容等。但本质问题还是 ”数据如何被稳定，紧凑高效的表示和传输“。

- 两者的关系：一般所看到都是成双成对（gRPC 默认使用 Protobuf 作为它的 IDL 和 消息交换格式）。
- 事实理论设计中 gRPC 也可以使用非 Protobuf （例如 JSON），但是现实工程中 PB 几乎是必须。



### 契约即代码 `.proto`

👉 工程实践中，我们往往是“契约优先”。

在 Go 中使用 gRPC 和 Protobuf 时：编写 `.proto` 文件 -> 通过 `protoc` 配合插件生成 Go 代码文件。

执行 protoc 编译后，Go 会自动生成两块核心资产:
- message -> struct：负责数据结构（序列化），定义数据的内存形态
- service -> interface: 负责 RPC 服务接口调用（网络通信），定义服务的编译期契约



### Part1：由 message 生成的

> 一句话：message = Go struct = gRPC 的 “载体对象”
- 数据结构（pb.go）
- 作用只有一个，这是 Protobuf 在 Go 中的内存表示
```go

// proto message 
message User {
	int64 id = 1;
	string name = 2;
}


// 生成的 Go struct
type User struct {
	Id   int64  `protobuf:"varint,1,opt,name=id,proto3" json:"id,omitempty"`
	Bane string `protobuf:"bytes,2,opt,name=name,proto3" json:"name,omitempty"` 
}
```

### Part2: 由 service 生成的

假设 proto 文件
```protobuf
service UserService {
	rpc GetUser(GetUserRequest) return (User);
}
```

#### Client 接口（客户端用）

> 一句话：它把 “远程调用” 伪装成 “本地函数调用”
- gRPC 通信骨架（x_grpc.pd.go)
- 作用是客户端的调用入口（`client.GetUser(ctx, req)`）
- 内部会序列化 request，走 HTTP/2，等 response，反序列化
```go
// 生成的 Go

// 接口定义：调用者（客户端）看到的逻辑
type UserServiceClient interface {
	GetUser(ctx context.Context, in *GetUserRequest, opts ...grpc.CallOption) (*User, error);
}

// 接口实现：内部封装了 grpc.ClientConn
type userServiceClient struct {
	cc grpc.ClientConnInterface
}
```

#### Server 接口（你必须实现）
- gRPC 强迫你 “契约优先”：你业务代码必须实现的接口
```
type UserServiceServer interface {
	GetUser(ctx contextg.Context, *GetUserRequest) (*User, error)
}
```



### Q&A

> protobuf 和 JSON 是一个层面的东西？

是，也不是。
- 是：都定义了数据序列化的格式
- 不是：protobuf = 协议 + schema + 编译产物

