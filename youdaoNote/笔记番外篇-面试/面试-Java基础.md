# # ![image](https://note.youdao.com/yws/api/personal/file/A07E0313B7E940588C9BE2356EFF7908?method=download&shareKey=2fadbb8f52fb31685423e85d990639a2)

有些知识点知道，但就是不知道为什么这样，深层知识是啥。
---
### JAVA基础
##### 1. hashCode相等两个类一定相等吗?
  
>   否;两个不同的对象，hashCode的结果可能是相同的，这就是哈希表中的冲突。
-  == : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不试同一个对象
- equals() : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况

```
情况1，类没有覆盖equals()方法。则通过equals()比较该类的两个对象时，等价于过“==”比较这两个对象。
 
情况2，类覆盖了equals()方法。一般，我们都覆盖equals()方法来两个对象的内容相
 等；若它们的内容相等，则返回true(即，认为这两个对象相等)。
```
 

-  hashCode()

> hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。


> hashCode() 和 equals()的关系”分2种情况来说明。
1. 第一种 不会创建“类对应的散列表”
这里所说的“不会创建类对应的散列表”是说：我们不会在HashSet,Hashtable,HashMap等这些本质是散列表的数据结构中，用到该类。例如，不会创建类的HashSet集合。在这种情况下，该类的“hashCode() 和 equals() ”没有半毛钱关系的！这种情况下，equals() 用来比较该类的两个对象是否相等。而hashCode()则根本没有任何作用，所以，不用理会hashCode()。


2. 第二种 会创建“类对应的散列表”

> 这里所说的“会创建类对应的散列表”是说：我们会在HashSet, Hashtable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，会创建该类的HashSet集合。

> 在这种情况下，该类的“hashCode() 和 equals() ”是有关系的：
1)、如果两个对象相等，那么它们的hashCode()值一定相同。这里的相等是指，通过equals()比较两个对象时返回true。
2)、如果两个对象hashCode()相等，它们并不一定相等。
因为在散列表中，hashCode()相等，即两个键值对的哈希值相等。然而哈希值相等，并不一定能得出键值对相等。补充说一句：“两个不同的键值对，哈希值相等”，这就是哈希冲突。

> 此外，在这种情况下。若要判断两个对象是否相等，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。



##### 2.集合框架

![image](https://github.com/junzixiehui/java-eye/blob/master/photo/%E9%9B%86%E5%90%88.png?raw=true)

- ArrayList
>   用数组存储元素的，这个数组可以动态创建，如果元素个数超过了数组的容量，那么就创建一个更大的新数组，并将当前数组中的所有元素都复制到新数组中。


- LinkedList
>  LinkedList是在一个链表中存储元素。如果除了在末尾外不能在其他位置插入或者删除元素，那么ArrayList效率更高，如果需要经常插入或者删除元素，就选择LinkedList

- 散列集HashSet


- 链式散列集LinkedHashSet
> LinkedHashSet是用一个链表实现来扩展HashSet类，它支持对规则集内的元素排序。HashSet中的元素是没有被排序的，而LinkedHashSet中的元素可以按照它们插入规则集的顺序提取。

- 树形集TreeSet

> 树形集是一个有序的Set，其底层是一颗树，这样就能从Set里面提取一个有序序列了。在实例化TreeSet时，我们可以给TreeSet指定一个比较器Comparator来指定树形集中的元素顺序。

- Queue
> 接口Deque，是一个扩展自Queue的双端队列，它支持在两端插入和删除元素，因为LinkedList类实现了Deque接口，所以通常我们可以使用LinkedList来创建一个队列。

- HashMap

> HashMap采用数组+链表实现，即使用链表处理冲突，同一hash值的链表都存储在一个链表里。但是当链表中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，HashMap采用数组+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。

- LinkedHashMap
> LinkedHashMap继承自HashMap，它主要是用链表实现来扩展HashMap类，HashMap中条目是没有顺序的，但是在LinkedHashMap中元素既可以按照它们插入图的顺序排序，也可以按它们最后一次被访问的顺序排序。

- TreeMap

> TreeMap基于红黑树数据结构的实现，键值可以使用Comparable或Comparator接口来排序。TreeMap继承自AbstractMap，同时实现了接口NavigableMap，而接口NavigableMap则继承自SortedMap。SortedMap是Map的子接口，使用它可以确保图中的条目是排好序的.
在实际使用中，如果==更新图时不需要保持图中元素的顺序，就使用HashMap，如果需要保持图中元素的插入顺序或者访问顺序，就使用LinkedHashMap，如果需要使图按照键值排序，就使用TreeMap==。

- HashTable
> HashTable中的函数都是同步的，这意味着它也是线程安全的，另外，HashTable中key和value都不可以为null。

- ConcurrentHashMap
> HashMap的线程安全
- CopyOnWriteArrayList
> 是一个线程安全的List接口的实现，它使用了ReentrantLock锁来保证在并发情况下提供高性能的并发读取。

##### 3.hashMap hashTable底层实现  hashTable与ConcurrentHashMap


> HashMap和Hashtable的底层实现都是数组+链表结构实现

> HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。

> HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。

> 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。

- ConcurrentHashMap

> ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。



##### 4. hashMap treeMap


[HashMap详解](https://tech.meituan.com/java-hashmap.html)

##### 5.线程池底层实现


```
  public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
```

- corePoolSize

> 线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。

- maximumPoolSize

> 线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；

- keepAliveTime

> 线程空闲时的存活时间，即当线程没有任务执行时，继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用；

- unit

> keepAliveTime的单位；

- BlockingQueue

> 用来保存等待被执行的任务的阻塞队列，且任务必须实现Runable接口，在JDK中提供了如下阻塞队列：


- threadFactory

> 创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。


- RejectedExecutionHandler

> 线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略：
当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务














