1.hashMap

JDK7-->数组+链表
JDK8--->数据+链表+红黑树

(1)JDK7 允许NULL键和NULL值。
查询速度快--因为key是通过hashCode计算hash值。

hashMap数组Entry[] 

loadFactor 加载因子，临界值threshold = 加载因子*容量，加载因子越大,能填满元素越多，冲突越高，查找效率低。   
 
#### 死循环
 http://www.cnblogs.com/xrq730/p/5037299.html
 
#### 手写hashMap
https://mp.weixin.qq.com/s/H-BkzeJ_gQo3fA7Zxaor-w