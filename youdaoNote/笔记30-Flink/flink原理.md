

 http://wuchong.me/categories/Flink/
 
 

 https://blog.csdn.net/lmalds/article/details/60575205
 
 
 #### 框架
 
 - JobManager
   * 任务调度
   * 检查点管理
   * 失败回复
   * Actor系统


  > 调度任务 Flink 集群中，计算资源被定义为 Task Slot。每个 TaskManager 会拥有一个或多个 Slots。JobManager 会以 Slot 为单位调度 Task。但是这里的 Task 跟我们在 Hadoop 中的理解是有区别的。对 Flink 的 JobManager 来说，其调度的是一个 Pipeline 的 Task，而不是一个点。

- TaskManager

> Task Managers是具体执行tasks的worker节点，执行发生在一个JVM中的一个或多个线程中。Task的并行度是由运行在Task Manager中的task slots的数量决定。如果一个Task Manager有4个slots，那么JVM的内存将分配给每个task slot 25%的内存。一个Task slot中可以运行1个或多个线程，同一个slot中的线程又可以共享相同的JVM。在相同的JVM中的tasks，会共享TCP连接和心跳消息

- Job Client

Job Client并不是Flink程序执行中的内部组件，而是程序执行的入口。Job Client负责接收用户提交的程序，并创建一个data flow，然后将生成的data flow提交给Job Manager。一旦执行完成，Job Client将返回给用户结果。 

- Actor系统
  
> Flink内部使用Akka模型作为JobManager和TaskManager之间的通信机制。

> Actor系统是个容器，包含许多不同的Actor，这些Actor扮演者不同的角色。Actor系统提供类似于调度、配置、日志等服务，同时包含了所有actors初始化时的线程池。

> 所有的Actors存在着层级的关系。新加入的Actor会被分配一个父类的Actor。Actors之间的通信采用一个消息系统，每个Actor都有一个“邮箱”，用于读取消息。如果Actors是本地的，则消息在共享内存中共享；如果Actors是远程的，则消息通过RPC远程调用。

> 每个父类的Actor都负责监控其子类Actor，当子类Actor出现错误时，自己先尝试重启并修复错误；如果子类Actor不能修复，则将问题升级并由父类Actor处理。在Flink中，actor是一个有状态和行为的容器。Actor的线程持续的处理从“邮箱”中接收到的消息。Actor中的状态和行为则由收到的消息决定。


- 检查点

> Flink的检查点机制是保证其一致性容错功能的骨架。它持续的为分布式的数据流和有状态的operator生成一致性的快照。其改良自Chandy-Lamport算法，叫做ABS（轻量级异步Barrier快照），具体参见论文：Lightweight Asynchronous Snapshots for Distributed Dataflows

> Flink的容错机制持续的构建轻量级的分布式快照，因此负载非常低。通常这些有状态的快照都被放在HDFS中存储（state backend）。程序一旦失败，Flink将停止executor并从最近的完成了的检查点开始恢复（依赖可重发的数据源+快照）。

> Barrier作为一种Event，是Flink快照中最主要的元素。它会随着data record一起被注入到流数据中，而且不会超越data record。每个barrier都有一个唯一的ID，将data record分到不同的检查点的范围中。下图展示了barrier是如何被注入到data record中的：

> 每个快照中的状态都会报告给Job Manager的检查点协调器；快照发生时，flink会在某些有状态的operator上对data record进行对齐操作（alignment），目的是避免失败恢复时重复消费数据。这个过程也是exactly once的保证。通常对齐操作的时间仅是毫秒级的。但是对于某些极端的应用，在每个operator上产生的毫秒级延迟也不能允许的话，则可以选择降级到at least once，即跳过对齐操作，当失败恢复时可能发生重复消费数据的情况。Flink默认采用exactly once意义的处理。


- Event Time语义

> Flink支持Event Time语义的处理，这有助于处理流计算中的乱序问题，有些数据也许会迟到，我们可以通过基于event time、count、session的窗口来处于这样的场景。



- [ ]  什么是EventTime

> 简单来说就是：事件发生于产生设备上的本地时间。作为对比，需要再引出另一个概念：ProcessingTime，它指的是事件被处理时所在处理节点的本地时间。

> 关于EventTime和ProcessingTime，有个很好的类比比较贴切。那就是经典的《星球大战》系列电影的发布顺序跟观影顺序的问题。

    这个不错的类比思路来自于Flink社区的Kostas Kloudas。


```
《星球大战》系列电影发布时间点可以看作是EventTime：
《星球大战4·新希望》(1977)
《星球大战5·帝国反击战》(1980)
《星球大战6·绝地大反攻》(1983)
《星战前传1·幽灵的威胁》(1999)
《星战前传2·克隆人进攻》(2002)
《星战前传3·西斯的复仇》(2005)
《星球大战7》(2015年)
```


> 但是对于一个不是很熟悉连贯剧情的人而言，他可能会按照1,2,3,4,5…的自然顺序观看（这里只是举个例子进行假设，其实关于观影顺序，网上也有各种讨论）。假设每一部都是一个事件，你观看一部类似于处理一个事件，那么你的ProcessingTime跟它原先的EventTime是不一致的。

- [ ] 为什么需要EventTime

> 假设你有个需求：需要计算一个网站的QPS然后绘制出变化曲线图，假设访问请求被记录到日志文件，然后最终通过流处理系统来计算、统计。因为某些原因，你的流处理系统出现故障，导致其将会下线一段时间，比如10分钟。那么在下线的这段时间内的事件可能堆积在持久存储里，比如文件系统或者消息系统（假如采集agent仍然继续工作）。当你的流处理系统恢复并重新上线后。如果你不以日志的EventTime作为时间基准，而是基于ProcessingTime，那么这中断的10分钟的请求日志就仿佛是突然到来的请求一样。因此，绘制的曲线图将会呈现一个非常短区间的尖锐的脉冲，而中断的那段时间则几乎是零，但这个图表显然是不符合事实的。这里时间戳的不一致就是由于选择的时间参照体系的不同。

> 上面的这个场景其实是流处理系统处理历史数据的场景，在实时事件流场景中，理论上EventTime跟ProcessingTime应该是一致的（也就是发生时间即处理时间），但现实也不是一致的，原因也有很多（比如，资源竞争，网络拥塞等等），这两个时间的偏差被称之为时间歪斜（skew）。

- [ ] 关于乱序

> 事件流可能是乱序（out-of-order）的，这跟EventTime和ProcessingTime无关，这是客观事实，关于乱序，不同的人有不同的理解。常容易因为理解不一致而导致沟通偏差。因为看起来它本身就有点模糊，或者说产生事件乱序的情况较多。（为了方便讨论，从流处理的角度来看我们将其称之为一个物理流），它本身并不能保证是有序的。这里涉及到你发送事件的机制（单线程？同步发送？Ack、网络）等。

> 而有些人认为的乱序是，假设上面这种情况有序的前提下，事件可能有多个源（多个生产者处于不同的设备），但它们仍可被视为一个逻辑流，那么它们也有可能因为网络延迟或者抖动，或者机器失败等造成延迟或者失序。

> 还有人谈及的无序会涉及到具体的技术细节里，比如类似于Kafka里的topic、partition如何设计&存储数据，甚至流的source并行等…

> **每一种都可能是某个人所理解的乱序，所以更高效的沟通方式是，确保沟通双方就乱序这一看法达成一致后再讨论。**

> 其实，大部分你看到的跟EventTime一起出现的“乱序”这个词，通常所描述的乱序的原因是：“网络”或者“机器”出现异常的情况，也就是说如果没有这些状况，那么它应该是“有序”的。所以，你也就应该知道以上哪些并不是EventTime所讨论的上下文中的“乱序”了。

摘自：https://blog.csdn.net/yanghua_kobe/article/details/72953921


#### 任务执行流程

> DataStream API和DataSetAPI是流处理和批处理的应用程序接口，当程序在编译时，生成JobGraph。编译完成后，根据API的不同，优化器（批或流）会生成不同的执行计划。根据部署方式的不同，优化后的JobGraph被提交给了executors去执行。



![image](https://img-blog.csdn.net/20170306132333243?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1hbGRz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> 首先，Flink程序提交给JobClient，JobClient再提交到JobManager，JobManager负责资源的协调和Job的执行。一旦资源分配完成，task就会分配到不同的TaskManager，TaskManager会初始化线程去执行task，并根据程序的执行状态向JobManager反馈，执行的状态包括starting、in progress、finished以及canceled和failing等。当Job执行完成，结果会返回给客户端。