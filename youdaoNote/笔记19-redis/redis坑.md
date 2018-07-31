
参考 https://blog.csdn.net/chenleixing/article/details/50530419

####  1.经常出现connect timeout
-  网络故障
-  慢查询 redis单线程查询
-  大value
-  AOF同步
解决步骤:
- 监控超时出现的时间是否周期性
- 监控超时出现时做的操作
- 分析TCP连接 握手
- I/O 多路复用程序通过队列向文件事件分派器传送套接字的过程
- 和redis有什么关系呢?
由于Redis的单线程模型（对命令的处理和连接的处理都是在一个线程中），如果存在慢查询的话，会出现上面的这种情况，造成新的accept的连接进不了队列。
- 监控慢查询
