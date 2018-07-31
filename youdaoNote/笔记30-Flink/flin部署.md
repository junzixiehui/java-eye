

### 本地部署
- 下载
- 解压

```
下载地址 https://archive.apache.org/dist/flink/flink-1.5.1/

flink-1.5.1-bin-scala_2.11.tgz  

tar xzf flink-1.5.1-bin-scala_2.11.tgz

bin/start-cluster.sh
```

- 访问

 > http://localhost:8081/#/overview
 
- 测试


```
另起一个tab执行 启动本地服务器
nc -l 9000

回来执行：
bin/flink run -c /Applications/soft/flink-1.5.1/examples/streaming/SocketWindowWordCount.jar

# 在nc -l 9000下输入字符


# 输出
tail -f flink-*-*.out

```


### 集群部署

- 参考 http://wuchong.me/blog/2016/02/26/flink-docs-setup-cluster/
- 设置属性

> conf 目录中找到文件 flink-conf.yaml。在这个文件中定义了 Flink 各个模块的基本属性，如 RPC 的端口，JobManager 和 TaskManager 堆的大小等。在不考虑 HA 的情况下，一般只需要修改属性 ***taskmanager.numberOfTaskSlots***，也就是每个 Task Manager 所拥有的 Slot 个数。这个属性，一般设置成机器 CPU 的 core 数，用来平衡机器之间的运算性能。




#### Yarn Cluster 模式
- 参考 https://www.ibm.com/developerworks/cn/opensource/os-cn-apache-flink/index.html

> 在一个企业中，为了最大化的利用集群资源，一般都会在一个集群中同时运行多种类型的 Workload。因此 Flink 也支持在 Yarn 上面运行。首先，让我们通过下图了解下 Yarn 和 Flink 的关系。
图 6. Flink 与 Yarn 的关系

![image](https://www.ibm.com/developerworks/cn/opensource/os-cn-apache-flink/img006.png)




#### Flink 的 HA

> 对于一个企业级的应用，稳定性是首要要考虑的问题，然后才是性能，因此 HA 机制是必不可少的。另外，对于已经了解 Flink 架构的读者，可能会更担心 Flink 架构背后的单点问题。和 Hadoop 一代一样，从架构中我们可以很明显的发现 **JobManager 有明显的单点问题**（SPOF，single point of failure）。 JobManager 肩负着任务调度以及资源分配，一旦 JobManager 出现意外，其后果可想而知。Flink 对 JobManager HA 的处理方式，原理上基本和 Hadoop 一样（一代和二代）。

> 首先，我们需要知道 Flink 有两种部署的模式，分别是 Standalone 以及 Yarn Cluster 模式。对于 Standalone 来说，Flink 必须依赖于 Zookeeper 来实现 JobManager 的 HA（Zookeeper 已经成为了大部分开源框架 HA 必不可少的模块）。**在 Zookeeper 的帮助下，一个 Standalone 的 Flink 集群会同时有多个活着的 JobManager，其中只有一个处于工作状态，其他处于 Standby 状态。当工作中的 JobManager 失去连接后（如宕机或 Crash），Zookeeper 会从 Standby 中选举新的 JobManager 来接管 Flink 集群**。

> 对于 Yarn Cluaster 模式来说，Flink 就要依靠 Yarn 本身来对 JobManager 做 HA 了。其实这里完全是 Yarn 的机制。对于 Yarn Cluster 模式来说，JobManager 和 TaskManager 都是被 Yarn 启动在 Yarn 的 Container 中。此时的 JobManager，其实应该称之为 Flink Application Master。也就说它的故障恢复，就完全依靠着 Yarn 中的 ResourceManager（和 MapReduce 的 AppMaster 一样）。由于完全依赖了 Yarn，因此不同版本的 Yarn 可能会有细微的差异。这里不再做深究



#### 云端部署
