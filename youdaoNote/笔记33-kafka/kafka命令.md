

- 启动


```
# zk
bin/zookeeper-server-start.sh config/zookeeper.properties


bin/kafka-server-start.sh config/server.properties
```


- 创建topic

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```


- 查看 topic
```
 bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --list

```

- 查看topic 详情


```
# 列出了lx_test_topic的parition数量、replica因子以及每个partition的leader、replica信息

bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic test --describe

```

- 消费

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

```


- 生产


```
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

```


