


#### 问题
- redis单线程，怎么保证高性能
- 高可用
- 高并发
- redis五种数据结构及其实现原理 为了性能怎么设计



#### redis为什么那么快

- 1）绝大部分请求是纯粹的内存操作（非常快速） 
- 2）采用单线程,避免了不必要的上下文切换和竞争条件 
- 3）非阻塞IO 
> 内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间
- pipeline  不等上一次请求返回就发出下一请求的模式就是pipeline模式。一次批量请求，等批量返回结果。例如MULIT  EXEC命令
- 

#### 数据库

- redis中所有键值对都保存在一个dict字典里面。
- 平级还有一个expires过期字典，保存的值是键的过期时间。
- 删除策略:redis用的是惰性删除和定期删除。惰性删除是每次获取值的时候判断该键是否过期，过期则删除。定期删除是一个任务每次去扫描库中一定量的键，直至扫描完毕。
- 还有一种删除策略是定时删除，监控在到期的时候，实时删除。
- 
#### 客户端
- redisclient客户端对象，每一个连接redis服务器的系统都会对应一个redisclient对象，里面包括一个字典的监控键。