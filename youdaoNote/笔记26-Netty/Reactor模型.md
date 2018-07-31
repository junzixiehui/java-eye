- 是什么
- 缺点
- 逐渐优化
- 

### 原理

>  复杂的 Reactor 模型
#### 单线程模型

> Java NIO 框架比较原始，目前主流的 Java 网络程序都在其上设计实现了 Reactor 模型，隐藏了 NIO 底层的复杂细节，大大简化了 NIO 编程，其原理和架构如下图所示，Acceptor 负责接收客户端 Socket 发起的新建连接请求，并把该 Socket 绑定到一个 Reactor 线程上，于是这个Socket 随后的读写事件都交给此 Reactor 线程来处理。Reactor 线程读取数据后，交给用户程序中的具体 Handler 实现类来完成特定的业务逻辑处理。为了不影响 Reactor 线程，我们通常使用一个单独的线程池来异步执行 Handler 的接口方法。



> 如果仅仅到此为止，则 NIO 里的 Reactor 模型还不算是很复杂，但实际上，我们的服务器是多核心的，而且需要高速并发处理大量的客户端连接，单线程的 Reactor 模型就满足不了需求了，因此我们需要多线程的 Reactor。一般原则是 Reactor（线程）的数量与 CPU 核心数（逻辑CPU）保持一致，即每个 CPU 执行一个 Reactor 线程，而客户端的 Socket 连接则随机均分到这些 Reactor 线程上去处理，如果有 8000 个连接，而 CPU 核心数为 8，则平均每个 CPU 核心承担 1000 个连接。

#### 多线程 Reactor 

> 模型下可能带来另外一个问题，即负载不均衡的问题，虽然每个 Reactor 线程服务的 Socket 数量是均衡的，但每个 Socket 的 I/O 事件可能是不均衡的，某些 Socket 的 I/O事件可能大大多于其他 Socket，从而导致某些 Reactor 线程负载更高，此时是否需要重新分配Socket 到不同的 Reactor 线程呢？这的确是一个问题，因为如果要切换 Socket 到另外的 Reactor 线程，则意味着 Socket 相关的 Connection 对象、Session 对象等必须是线程安全的，这本身就带来一定的性能损耗，另外需要对 I/O 事件做统计分析，启动额外的定时线程在合适的时机完成 Socket 重分配，这本身就是很复杂的事情。