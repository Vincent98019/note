Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set可以**自动排重**，当需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的**复杂度都是O(1)**。

一个算法，随着数据的增加，执行时间的长短，如果是O(1)，数据增加，查找数据的时间不变。



#### 常用命令

* `sadd <key> <value1> <value2> .....` ：将一个或多个元素加入到集合key中，已经存在的元素将被忽略

 ---

* `smembers <key>` ：取出该集合的所有值
* `sismember <key> <value>` ：判断集合`<key>`是否为含有该`<value>`值，有1，没有0
* `scard <key>` ：返回该集合的元素个数
* `spop <key>`：**随机从该集合中吐出一个值。**
* `srandmember <key> <n>` ：随机从该集合中取出n个值，不会从集合中删除 

 ---

* `srem <key> <value1> <value2> ....` ：删除集合中的某个元素

 ---

* `smove <source><destination>` ：value把集合中一个值从一个集合移动到另一个集合
* `sinter <key1><key2>` ：返回两个集合的交集元素。
* `sunion <key1><key2>` ：返回两个集合的并集元素。
* `sdiff <key1><key2>` ：返回两个集合的**差集**元素(key1中的，不包含key2中的)



#### 数据结构

Set数据结构是dict字典，字典是用哈希表实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。


