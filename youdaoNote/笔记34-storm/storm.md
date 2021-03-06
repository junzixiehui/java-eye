

#### 问题
- 如果使用的是自己的消息队列，需要加入消息队列做数据的来源和产出的代码
- 需要考虑如何做故障处理：如何记录消息队列处理的进度，应对Storm重启，挂掉的场景
- 需要考虑如何做消息的回退：如果某些消息处理一直失败怎么办？

#### 简介

- 分布式
- 实时
- 大数据
- 可靠-容错 保证数据会处理
- 消息不丢失
- 可扩展 水平扩展


- 背景

> 在Storm之前，进行实时处理是非常痛苦的事情: 需要维护一堆消息队列和消费者，他们构成了非常复杂的图结构。消费者进程从队列里取消息，处理完成后，去更新数据库，或者给其他队列发新消息。

> 这样进行实时处理是非常痛苦的。我们主要的时间都花在关注往哪里发消息，从哪里接收消息，消息如何序列化，真正的业务逻辑只占了源代码的一小部分。一个应用程序的逻辑运行在很多worker上，但这些worker需要各自单独部署，还需要部署消息队列。最大问题是系统很脆弱，而且不是容错的：需要自己保证消息队列和worker进程工作正常。

> Storm完整地解决了这些问题。它是为分布式场景而生的，抽象了消息传递，会自动地在集群机器上并发地处理流式计算，让你专注于实时处理的业务逻辑。

#### 框架

- 拓扑(Topologies)

 > 一个Storm拓扑打包了一个实时处理程序的逻辑。而拓扑会一直运行（当然直到你杀死它)。一个拓扑是一个通过流分组(stream grouping)把Spout和Bolt连接到一起的拓扑结构。。一个拓扑就是一个复杂的多阶段的流计算。TopologyBuilder: 使用这个类来在Java中创建拓扑.

- 元组(Tuple)

> 元组是Storm提供的一个轻量级的数据格式，可以用来包装你需要实际处理的数据。元组是一次消息传递的基本单元。一个元组是一个命名的值列表，其中的每个值都可以是任意类型的。元组是动态地进行类型转化的--字段的类型不需要事先声明。在Storm中编程时，就是在操作和转换由元组组成的流。通常，元组包含整数，字节，字符串，浮点数，布尔值和字节数组等类型。要想在元组中使用自定义类型，就需要实现自己的序列化方式。

- 流(Streams)

> 流是Storm中的核心抽象。一个流由无限的元组序列组成，这些元组会被分布式并行地创建和处理。通过流中元组包含的字段名称来定义这个流。
每个流声明时都被赋予了一个ID。只有一个流的Spout和Bolt非常常见，所以OutputFieldsDeclarer提供了不需要指定ID来声明一个流的函数(Spout和Bolt都需要声明输出的流)。这种情况下，流的ID是默认的“default”。


- Spouts

> Spout(喷嘴，这个名字很形象)是Storm中流的来源。通常Spout从外部数据源，如消息队列中读取元组数据并吐到拓扑里。Spout可以是可靠的(reliable)或者不可靠(unreliable)的。可靠的Spout能够在一个元组被Storm处理失败时重新进行处理，而非可靠的Spout只是吐数据到拓扑里，不关心处理成功还是失败了。

> Spout可以一次给多个流吐数据。此时需要通过OutputFieldsDeclarer的declareStream函数来声明多个流并在调用SpoutOutputCollector提供的emit方法时指定元组吐给哪个流。

> Spout中最主要的函数是nextTuple，Storm框架会不断调用它去做元组的轮询。如果没有新的元组过来，就直接返回，否则把新元组吐到拓扑里。nextTuple必须是非阻塞的，因为Storm在同一个线程里执行Spout的函数。

> Spout中另外两个主要的函数是ack和fail。当Storm检测到一个从Spout吐出的元组在拓扑中成功处理完时调用ack,没有成功处理完时调用fail。只有可靠型的Spout会调用ack和fail函数。更多细节可以查看Storm Java文档和我的另外一篇文章：[Storm如何保证可靠的消息处理](http://www.cnblogs.com/Jack47/p/guaranteeing-message-processing-in-storm.html)

- Bolts

> 在拓扑中所有的计算逻辑都是在Bolt中实现的。一个Bolt可以处理任意数量的输入流，产生任意数量新的输出流。Bolt可以做函数处理，过滤，流的合并，聚合，存储到数据库等操作。Bolt就是流水线上的一个处理单元，把数据的计算处理过程合理的拆分到多个Bolt、合理设置Bolt的task数量，能够提高Bolt的处理能力，提升流水线的并发度。

> Storm可以沿着元组追踪到始发spout。collector.ack(tuple)和collector.fail(tuple)会告知spout每条消息都发生了什么。


#### 拓扑组成

- Worker进程
- Executor
- Tasks

![image](http://img1.tbcdn.cn/L1/461/1/1794fa8481b50868904ed46c63f23f66efe69bd2)

    Storm中worker进程，executor(线程)和任务的关系