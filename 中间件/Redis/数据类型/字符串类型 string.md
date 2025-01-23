
string是Redis最基本的类型，一个key对应一个value。 

string类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

string类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M。



#### 常用命令

* **`set <key> <value>` ：添加键值对**

![](assets/字符串类型%20string/557dd1387d6adde098f3cf9c004087be_MD5.png)


> * NX：当数据库中key不存在时，添加
> * XX：当数据库中key存在时，添加，与NX参数互斥
> * EX：key的超时秒数
> * PX：key的超时毫秒数，与EX互斥


* `setnx <key> <value>`：只有在key不存在时，添加键值对
* **`get <key>`：查询对应键值**
* `strlen <key>`：获得值的长度
* **`del <key>`：删除键值对**

 ---

* `getrange <key> <起始位置> <结束位置>` ：获得值的范围，类似java中的substring，**前包后包**
* `setrange <key> <起始位置> <value>` ：用`<value>`覆写`<key>`所储存的字符串值，从`<起始位置>`开始(**索引从0开始**)。

 ---

* `incr <key>`：将key中储存的数字值增1，只能对数字值操作，如果为空，新增值为1
* `decr <key>`：将key中储存的数字值减1，只能对数字值操作，如果为空，新增值为-1
* `incrby/decrby <key> <步长>`：将 key 中储存的数字值增减，自定义步长
* `append <key> <value>`：将给定的`<value>`追加到原值的末尾

 ---

* `mset <key1> <value1> <key2> <value2> ....`：同时设置一个或多个 key-value对（要么全成功，要么全失败）
* `mget <key1> <key2> <key3> .....`：同时获取一个或多个value（要么全成功，要么全失败）
* `msetnx <key1> <value1> <key2> <value2> .....`：同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在（要么全成功，要么全失败）

 ---

* **`setex <key> <过期时间> <value>`：设置键值的同时，设置过期时间，单位秒。**
* `getset <key> <value>`：以新换旧，设置了新值同时获得旧值。



#### 数据结构

string的数据结构为简单动态字符串(Simple Dynamic String，缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

![](assets/字符串类型%20string/a04da7d6262041507b9608f2103c4bcd_MD5.png)

如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。**字符串最大长度为512M。**
