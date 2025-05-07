

## 集合选型规则

根据业务分析：

① 先判断存储对象的类型(一组对象或一组键值对)

② 一组对象：`Conllection`接口

**允许重复：`List`**
>**增删多**：`LinkedList` 【底层是双向链表】
>**改查多**：`ArrayList`  【底层是`Object`类型的可变数组】

**不允许重复：`Set`**
> **无序**：`HashSet`  【底层是`HashMap`，是数组+链表+红黑树】
> **排序**：`TreeSet` 【底层是`TreeMap`，是红黑树，默认排序或者自定义排序】
> **插入和取出顺序一致**：`LinkedHashSet`  【底层是`LinkedHashMap`，是数组+双向链表】

③ 一组键值对：`Map`接口
> **键无序**：`HashMap`  【底层是哈希表，JDK7：数组+链表；**JDK8：数组+链表+红黑树**】
> **键排序**：`TreeMap` 【底层是红黑树，默认排序或者自定义排序】
> **键插入和取出顺序一致**：`LinkedHashMap`  【底层是数组+双向链表】
> **读取文件**：`Properties`


## Collections工具类

`java.utils.Collections`是集合工具类，用来对集合进行操作。

**常用方法**

`reverse(List<?> list)` ：反转集合中元素的顺序。
`shuffle(List<?> list)`：打乱集合顺序。

`sort(List<T> list)`：将集合中元素按照默认规则排序。
`sort(List<T> list，Comparator<? super T>)`：将集合中元素按照指定规则排序。
>涉及到了Comparator接口，位于位于java.util包下，排序是comparator能实现的功能之一，该接口代表一个比较器，比较器具有可比性。
>.
>`int compare(String o1, String o2)`：比较其两个参数的顺序。
两个对象比较的结果有三种：大于，等于，小于
如果要按照升序排序，则o1小于o2，返回负数，相等返回0，01大于02返回正数
如果要按照降序排序，则o1小于o2，返回正数，相等返回0，01大于02返回负数

`swap(List<?> list, int i, int j)` ：交换集合中指定位置的元素。 

`max(Collection<? extends T> coll) `：根据其元素的自然顺序返回给定集合的最大元素。 
`max(Collection<? extends T> coll, Comparator<? super T> comp) `：根据指定的比较器引发的顺序返回给定集合的最大元素。 

`min(Collection<? extends T> coll) `：根据其元素的自然顺序返回给定集合的最小元素。 
`min(Collection<? extends T> coll, Comparator<? super T> comp) `：根据指定的比较器引发的顺序返回给定集合的最小元素。

`frequency(Collection<?> c, Object o)` ：返回集合中元素出现的次数。

`copy(List<? super T> dest, List<? extends T> src) `：将所有元素从一个列表复制到另一个列表中。 
> dest的长度需要大于等于src的长度。否则抛出索引越界异常。

`replaceAll(List<T> list, T oldVal, T newVal) `：将列表中一个指定值的所有出现替换为另一个。

`addAll(Collection<T> c, T... elements)`：往集合中添加一些元素。

