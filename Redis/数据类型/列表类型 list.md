**单键多值**

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

![](assets/列表类型%20list/8081fa1d3326c570b947c9460a7c418c_MD5.png)


#### 常用命令

* `lpush <key> <value1> <value2> <value3> ...` ：将一个或多个元素加入列表左边
* `rpush <key> <value1> <value2> <value3> ...` ：将一个或多个元素加入列表右边

 ---

* `lpop <key>` ：从左边吐出一个值。值在键在，值光键亡。
* `rpop <key>` ：从右边吐出一个值。值在键在，值光键亡。

 ---

* `rpoplpush  <key1> <key2>` ：从`<key1>`列表右边吐出一个值，插到`<key2>`列表左边。
* `lrange <key> <start> <stop>` ：按照索引下标获得元素(从左到右)
* `lrange mylist 0 -1` ： 0左边第一个，-1右边第一个，（0-1表示获取所有）
* `lindex <key> <index>` ：按照索引下标获得元素(从左到右)
* `llen <key>` ：获得列表长度 

 ---

* `linsert <key> before <value> <newvalue>` ：在`<value>`的后面插入`<newvalue>`插入值
* `lrem <key> <n> <value>` ：从左边删除n个value(从左到右)
* `lset<key> <index> <value>` ：将列表key下标为index的值替换成value



#### 数据结构

List的数据结构为快速链表quickList。

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。

它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

当数据量比较多的时候才会改成quicklist。

因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

![](assets/列表类型%20list/2abc605057974afd7455d63d9a54c38d_MD5.png)


Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

