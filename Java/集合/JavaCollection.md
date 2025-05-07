
集合是java中提供的一种容器，可以用来存储多个数据。

**集合和数组的区别：**

> 数组的长度是固定的，集合的长度是可变的。
> 数组中存储的是同一类型的元素，可以存储基本数据类型值。集合存储的都是对象，而且对象的类型可以不一致。在开发中一般当对象多的时候，使用集合进行存储。
> 使用数组进行增加/删除元素比较麻烦，集合增加/删除更简洁了。

集合按照其存储结构可以分为两大类，分别是单列集合`java.util.Collection`和双列集合`java.util.Map`。

**单列集合：**

![](assets/JavaCollection/1d64b866630ad376f797481cc7372406_MD5.png)

**双列集合：**

![](assets/JavaCollection/9680c6feee8684716b2c783ee2f73b80_MD5.png)

## Collection接口

Collection：单列集合类的根接口，用于存储一系列符合某种规则的元素。

```java
public interface Collection<E> extends Iterable<E>
```

> Collection实现子类可以存放多个元素，每个元素可以是Object。 
> Collection有些实现类可以存放重复的元素，有些不可以。
> Collection接口没有直接实现的子类，是通过子接口List和Set接口来实现的。


> List的特点是元素有序、元素可重复。
> List接口的主要实现类有`java.util.ArrayList`和`java.util.LinkedList`。

> Set的特点是元素无序，而且不可重复。 Set接口的主要实现类有`java.util.HashSet`和`java.util.TreeSet`。

**常用方法**

Collection是所有单列集合的父接口，因此在Collection中定义了单列集合(List和Set)通用的一些方法，这些方法可用于操作所有的单列集合。

`boolean add(E e)`： 把给定的对象添加到当前集合中。
`boolean addAll(Collection<? extends E> c)`：将指定集合中的所有元素添加到此集合。

`void clear()`：清空集合中所有的元素。
`boolean remove(E e)`：把给定的对象在当前集合中删除。
`boolean removeAll(Collection<? extends E> c)`：把指定的集合中的所有元素在当前集合中删除。

`boolean contains(E e)`：判断当前集合中是否包含给定的对象。
`boolean containsAll(Collection<? extends E> c)`：判断当前集合中是否包含给定的集合中的所有对象。
`boolean isEmpty()`：判断当前集合是否为空。

`int size()`：返回集合中元素的个数。

`Object[] toArray()`：把集合中的元素，存储到数组中。

```java
import java.util.ArrayList;
import java.util.Collection;

public class Demo1Collection {
    public static void main(String[] args) {
		// 创建集合对象 
    	// 使用多态形式
    	Collection coll = new ArrayList();
    	// 使用方法
    	// 添加功能 boolean add(Object s)
    	coll.add("小李广");
    	coll.add(10);	// 这句话相当于 coll.add(new Interger(10));
    	coll.add("石破天");
    	System.out.println(coll);

    	// boolean contains(E e) 判断o是否在集合中存在
    	System.out.println("判断  扫地僧 是否在集合中" + coll.contains("扫地僧"));

    	// boolean remove(E e) 删除在集合中的o元素
    	System.out.println("删除石破天：" + coll.remove("石破天"));
    	System.out.println("操作之后集合中元素:" + coll);
    	
    	// size() 集合中有几个元素
		System.out.println("集合中有" + coll.size() + "个元素");

		// Object[] toArray()转换成一个Object数组
    	Object[] objects = coll.toArray();
    	// 遍历数组
    	for (int i = 0; i < objects.length; i++) {
			System.out.println(objects[i]);
		}

		// void clear() 清空集合
		coll.clear();
		System.out.println("集合中内容为：" + coll);
		// boolean isEmpty()  判断是否为空
		System.out.println(coll.isEmpty());  	
	}
}
```

## 迭代器

在程序开发中，经常需要遍历集合中的所有元素。针对这种需求，JDK专门提供了一个接口`java.util.Iterator`。Iterator接口也是Java集合中的一员，但它与Collection、Map接口有所不同，Collection接口与Map接口主要用于存储元素，而Iterator主要用于迭代访问（即遍历）Collection中的元素，因此Iterator对象也被称为迭代器。

> **迭代：** 即Collection集合元素的通用获取方式。在取元素之前先要判断集合中有没有元素，如果有，就把这个元素取出来，继续在判断，如果还有就再取出出来。一直把集合中的所有元素全部取出。这种取出方式专业术语称为迭代。

**常用方法**

`E next()`：返回迭代的下一个元素。
`boolean hasNext()`：如果仍有元素可以迭代，则返回 true。

```java
public class IteratorDemo {
    public static void main(String[] args) {
        // 使用多态方式 创建对象
        Collection<String> coll = new ArrayList<String>();

        // 添加元素到集合
        coll.add("串串星人");
        coll.add("吐槽星人");
        coll.add("汪星人");
        //遍历
        //使用迭代器 遍历   每个集合对象都有自己的迭代器
        Iterator<String> it = coll.iterator();
        //  泛型指的是 迭代出 元素的数据类型
        while(it.hasNext()){ //判断是否有迭代元素
            String s = it.next();//获取迭代出的元素
            System.out.println(s);
        }
    }
}
```

在进行集合元素取出时，如果集合中已经没有元素了，还继续使用迭代器的next方法，将会发生`java.util.NoSuchElementException`没有集合元素的异常。

## 增强for

增强for循环(也称for each循环)是JDK1.5以后出来的一个高级for循环，专门用来遍历数组和集合的。它的内部原理其实是个Iterator迭代器，所以在遍历的过程中，不能对集合中的元素进行增删操作。

```java
for(元素的数据类型 变量 : Collection集合or数组){ 
   //写操作代码
}
```

它用于遍历Collection和数组。通常只进行遍历元素，不要在遍历的过程中对集合元素进行增删操作。
