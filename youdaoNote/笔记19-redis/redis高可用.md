

Redis 高可用架构最佳实践
https://dbarobin.com/2017/05/27/ha-of-redis/

### 目录

- 哨兵sentinel
- sentinel监控流程
  * 监控主服务器
  * 监控从服务器
  * 监控其他sentinel
- 命令连接
- 订阅连接

#### 流程

- sentinel会按频率向其他实例发送ping命令，如果没有回复，则判定其为主观下线，会询问其他sentinel对其的判定，若多数判定其下线，则会判定其客观下线，对主服务器进行故障转移。

- sentinel知道主服务器的存在 通过用户配置文件。
- 知道从服务器的存在通过向主服务器发送INFO命令。
- 知道其他sentinel的存在 会向主服务器的_sentinel_:hello频道发送命令，宣告自己的存在。同时会从该频道订阅信息，知道其他sentinel的信息。
- sentinel和主服务器和从服务器都会建立两个连接，命令连接和订阅连接，只会和其他sentinel建立命令连接。


#### 命令连接 订阅连接


