

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分（score）**，这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。**集合的成员是唯一的，但评分可以重复。**

因为元素是有序的，所以可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的，因此能够使用有序集合作为一个没有重复成员的智能列表。



#### 常用命令

* `zadd <key> <score1> <value1> <score2> <value2>…` ：将一个或多个元素及score值加入到有序集key中
* `zrange <key> <start> <stop> [withscores]` ：返回有序集key中下标在`<start>` `<stop>`之间的元素，带`withscores`，可以让分数一起和值返回到结果集
* `zrangebyscore key min max [withscores] [limit offset count]` ：返回有序集key中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员，有序集成员按 score 值递增(从小到大)次序排列
* `zrevrangebyscore key max min [withscores] [limit offset count]`  ： 同上，改为从大到小排列。
* `zincrby <key> <increment> <value>` ：为元素的score加上增量
* `zrem <key> <value>` ：删除该集合下指定值的元素 
* `zcount <key> <min> <max>` ：统计该集合，分数区间内的元素个数 
* `zrank <key> <value>` ：返回该值在集合中的排名，从0开始



#### 数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构`Map<String, Double>`，可以给每一个元素value赋予一个权重score，另一方面它又类似于`TreeSet`，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

**zset底层使用了两个数据结构：**

1. hash：hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。
2. 跳跃表：跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。



##### **跳跃表（跳表）**

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。

Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。



**有序链表：**

![](assets/有序集合类型%20sortedset/e5b31aaff46dd5d6993deabeb6212954_MD5.png)


要查找值为51的元素，需要从第一个元素开始依次查找、比较才能找到，共需要6次比较。



**跳跃表：**

![](assets/有序集合类型%20sortedset/35ab4f396d5f4bcff8121fd0441f5cdc_MD5.png)


1. 从第2层开始，1节点比51节点小，向后比较。
2. 21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层。
3. 在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下
4. 在第0层，51节点为要查找的节点，节点被找到，共查找4次。



从此可以看出跳跃表比有序链表效率要高。

