

- 连接

```
redis-cli -h 10.161.186.xxx -p 6379
```
- CLIENT LIST
> 命令用于返回所有连接到服务器的客户端信息和统计数据。
- INFO
> 命令以一种易于理解和阅读的格式，返回关于Redis服务器的各种信息和统计数值。 

- DBSIZE 
 > 返回当前数据里面keys的数量。

- MONITOR 
> 是一个调试命令，返回服务器处理的每一个命令，它能帮助我们了解在数据库上发生了什么操作，可以通过redis-cli和telnet命令使用.


```
$ redis-cli monitor
1339518083.107412 [0 127.0.0.1:60866] "keys" "*"
1339518087.877697 [0 127.0.0.1:60866] "dbsize"
1339518100.544926 [0 127.0.0.1:60866] "get" "x"
```

使用SIGINT (Ctrl-C)来停止 通过redis-cli使用MONITOR命令返回的输出.


```
$ telnet localhost 6379
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
MONITOR
+OK
+1339518083.107412 [0 127.0.0.1:60866] "keys" "*"
+1339518099.363765 [0 127.0.0.1:60866] "del" "x"
QUIT
+OK
Connection closed by foreign host.
```

使用QUIT命令来停止通过telnet使用MONITOR返回的输出.

##MONITOR 性能消耗

由于MONITOR命令返回 服务器处理的所有的 命令, 所以在性能上会有一些消耗.

在不运行MONITOR命令的情况下，benchmark的测试结果:

$ src/redis-benchmark -c 10 -n 100000 -q
PING_INLINE: 101936.80 requests per second
PING_BULK: 102880.66 requests per second
SET: 95419.85 requests per second
GET: 104275.29 requests per second
INCR: 93283.58 requests per second
在运行MONITOR命令的情况下，benchmark的测试结果: (redis-cli monitor > /dev/null):

$ src/redis-benchmark -c 10 -n 100000 -q
PING_INLINE: 58479.53 requests per second
PING_BULK: 59136.61 requests per second
SET: 41823.50 requests per second
GET: 45330.91 requests per second
INCR: 41771.09 requests per second
在这种特定的情况下，运行一个MONITOR命令能够降低50%的吞吐量，运行多个MONITOR命令 降低的吞吐量更多.

> monitor的模型是这样的，它会将所有在Redis服务器执行的命令进行输出，通常来讲Redis服务器的QPS是很高的，也就是如果执行了monitor命令，Redis服务器在Monitor这个客户端的输出缓冲区又会有大量“存货”，也就占用了大量Redis内存。
 
-  紧急处理和解决方法
> 进行主从切换（主从内存使用量不一致），也就是redis-cluster的fa-over操作，继续观察新的Master是否有异常，通过观察未出现异常。
查找到真正的原因后，也就是monitor，关闭掉monitor命令的进程后，内存很快就降下来了



- 上面的client_longest_output_list看，应该是输出缓冲区占用内存较大，也就是有大量的数据从Redis服务器向某些客户端输出。
于是使用client list命令（类似于mysql processlist） redis-cli -h host -p port client list | grep -v "omem=0"，来查询输出缓冲区不为0的客户端连接，