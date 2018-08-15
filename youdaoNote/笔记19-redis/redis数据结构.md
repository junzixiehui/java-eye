

### 值对象数据
- 查看对象类型用 TYPE命令
- 每一个对象redisObject结构表示:
```
typedef struct redisObject{
    unsigned type:4;//类型
    unsigned encoding:4;//编码底层数据结构
    void *ptr;//指向底层数据结构的指针
}
```
- encodeing编码包括
 * REDIS_ENCODING_INT long类型整数
 * EMBSTR embstr编码简单动态字符串 小于32字节用
 * RAW 简单动态字符串 大于32字节用
 * HT 字典
 * LINKEDLIST 双端链表
 * ZIPLIST 压缩列表
 * INTSET整数集合
 * SKIPLIST 跳跃表和字典

#### string
- 重写c的string结构可以保存整数和字符串。底层用了三种数据结构
- 为了更好扩容，初始化的容量时候会预留空间。
- long double类型浮点数也是字符串保存的。

#### list 列表对象

- 底部是双端链表linkedlist
- 压缩列表ziplist
- 快速列表quicklist
- index(x)获取索引的时候很慢。
- 列表对象包含元素比较少使用压缩列表作为底层实现，因为压缩列表占用更少内存，在内存中以连续块保存在压缩列表比双端列表更快载入缓冲。都是为了速度和内存啊。
- 使用ziplist条件 (值可以修改)     
* 所有字符串长度小于64字节
* 元素数量小于512个  

#### hash

- 字典 数组+链表
- 压缩列表实现 原理是键和值紧密挨着都在表尾
- 当键值对多的时候，rehash的时候是渐进式rehash.此时有两个字典h0 h1，获取数据的时候同时从两个字典获取数据。

#### set 集合对象
- 底层是值为null的字典和整数集合实现
- intset编码使用条件
* 元素都是整数值
* 元素数量不超过512个

#### zset
- 底层跳跃列表字典结构或压缩列表
- skiplist编码的有序集合使用zset结构同时包含一个字典和一个跳跃表。
```
typedef struct zset{
    zskiplist *zsl;
    dict *dict;
}zset;
```
- zsl按分值从小到大保存所有集合元素。每个跳跃表节点都保存了一个集合元素:节点object保存元素成员，节点score元素保存了分值。通过跳跃表可以进行范伟查找。
- 同时dict字典保存元素的成员和分值。获取某个成员的分值就可以做到O(1)例如ZSCORE命令就根据这一个特性。

#### 总结
- redis可以根据不同场景为每个对象设置不同数据结构，从而优化效率。
