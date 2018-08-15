
推荐: Flink 靠什么征服饿了么工程师 https://mp.weixin.qq.com/s/jCsUJ4HGLcFI1JRhAmeDcg


- 是什么
- 怎么用 https://lunatictwo.gitbooks.io/flink/content/chapter1.html
- 用作什么
- 与替代性产品有什么区别 例如storm
- 场景
- 原理
- 文档 

#### 问题

- 任务是怎么调度的
- Flink 如何更高效的管理内存，如何进一步的避免用户程序的 OOM
- 系统部署成功后各个节点都启动了哪些服务，各个服务之间又是怎么交互和协调的
- 

#### 概述
-  实时
-  有状态 可容错
-  流处理 批处理框架
-  高的吞吐量和低事件延迟
-  支持恰好一次语义
-  灵活的时间窗口 计数窗口
-  与YARN，HDFS，HBase和Apache Hadoop生态系统的其他组件集成
-  内存管理 与 数据库
-  可伸缩


> 其所要处理的主要场景就是流数据，批数据只是流数据的一个极限特例而已。再换句话说，Flink 会把所有任务当成流来处理，这也是其最大的特点。

#### 源码

- 地址： https://github.com/apache/flink.git
- 安装 Scala插件 [下载1](https://plugins.jetbrains.com/plugin/1347-scala)       | [下载2](https://github.com/JetBrains/intellij-scala/releases)

> 导入idea 参考：https://github.com/apache/flink/blob/master/docs/internals/ide_setup.md#intellij-idea
- Alternatively, **mvn clean package -DskipTests** also creates the necessary files for the IDE to work with but without installing libraries.
- Build the Project (Build -> Make Project)


#### 使用

> 使用Flink不需要安装Apache Hadoop。 如果您计划在YARN中运行Flink或处理存储在HDFS中的数据，请选择与您安装的Hadoop版本匹配的版本。


##### 环境准备

- java8
- 下载页面 http://flink.apache.org/downloads.html
- Linux

```
$ cd ~/Downloads        # Go to download directory
$ tar xzf flink-*.tgz   # Unpack the downloaded archive
$ cd flink-1.5.0

$ ./bin/start-cluster.sh  # Start Flink
```
- MAC

```
For MacOS X users, Flink can be installed through Homebrew.

$ brew install apache-flink
...
$ flink --version
Version: 1.2.0, Commit ID: 1c659cf
```

- http://localhost:8081 
- 查看log 
```
$ tail log/flink-*-standalonesession-*.log
```
- To stop Flink when you’re done type:

```
$ ./bin/stop-cluster.sh
```


##### 测试用例


- First of all, we use netcat to start local server via

> $ nc -l 9000

- Submit the Flink program:
```
$ ./bin/flink run examples/streaming/SocketWindowWordCount.jar --port 9000
Starting execution of program
```

```
$ nc -l 9000
lorem ipsum
ipsum ipsum ipsum
bye
```
- The .out file will print the counts at the end of each time window as long as words are floating in, e.g.:

> $ tail -f log/flink-*-taskexecutor-*.out
lorem : 1
bye : 1
ipsum : 4

#### 怎么用

 - 获取一个执行环节
 - 加载 创建 初始化环境
 - 制定操作系统的transaction算子
 - 制定把计算好的数据放在哪
 - 调用 execute（）触发执行程序
 - 
 


![image](https://images2015.cnblogs.com/blog/820234/201512/820234-20151216023858381-417460984.jpg)


#### 文档

- 中文文档 http://doc.flink-china.org/1.1.0/quickstart/setup_quickstart.html
- 



#### 测试

````
设置容灾和exactly once语义支持
当打开Flink的checkpointing功能时，Flink log consumer会周期性的将每个shard的消费进度保存起来，当作业失败时，flink会恢复log consumer，并从保存的最新的checkpoint开始消费。

写checkpoint的周期定义了当发生失败时，最多多少的数据会被回溯，即重新消费，使用代码如下：

final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
// 开启flink exactly once语义
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
// 每5s保存一次checkpoint
env.enableCheckpointing(5000);


```


