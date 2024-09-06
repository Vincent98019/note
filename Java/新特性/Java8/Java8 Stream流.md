
在Java 8中，得益于Lambda所带来的函数式编程，引入了一个全新的Stream概念，用于解决已有集合类库既有的弊端。

传统集合的多步遍历代码几乎所有的集合（如 Collection 接口或 Map 接口等）都支持直接或间接的遍历操作。而当我们需要对集合中的元素进行操作的时候，除了必需的添加、删除、获取外，最典型的就是集合遍历。


**循环遍历的弊端：**

Java 8的Lambda让我们可以更加专注于做什么（What），而不是怎么做（How）。

* for循环的语法就是“**怎么做**”
* for循环的循环体是“**做什么**”

如果希望对集合中的元素进行筛选过滤：

1. 将集合A根据条件一过滤为子集B；
2. 然后再根据条件二过滤为子集C。

在Java 8之前的做法可能为：

```java
public class Demo02NormalFilter {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张无忌");
        list.add("周芷若");
        list.add("赵敏");
        list.add("张强");
        list.add("张三丰");
        List<String> zhangList = new ArrayList<>(); 
        for (String name : list) {
            if (name.startsWith("张")) {
                zhangList.add(name);
            }
        }
        List<String> shortList = new ArrayList<>();
        for (String name : zhangList) {
            if (name.length() == 3) {
                shortList.add(name);
            }
        }
        for (String name : shortList) {
            System.out.println(name);
        }
    }
}
```


这段代码中含有三个循环，每一个作用不同：

1. 首先筛选所有姓张的人；
2. 然后筛选名字有三个字的人；
3. 最后进行对结果进行打印输出。

每当我们需要对集合中的元素进行操作的时候，总是需要进行循环、循环、再循环。循环是做事情的方式，而不是目的。另一方面，使用线性循环就意味着只能遍历一次。如果希望再次遍历，只能再使用另一个循环从头开始。


**Stream的更优写法：**

```java
public class Demo03StreamFilter {
    public static void main(String[] args) { 
        List<String> list = new ArrayList<>(); 
        list.add("张无忌");
        list.add("周芷若");
        list.add("赵敏");
        list.add("张强");
        list.add("张三丰");
        list.stream()
            .filter(s ‐> s.startsWith("张"))
            .filter(s ‐> s.length() == 3)
            .forEach(System.out::println);
    }
}
```

> 直接阅读代码的字面意思即可完美展示无关逻辑方式的语义：获取流、过滤姓张、过滤长度为3、逐一打印。

> 代码中并没有体现使用线性循环或是其他任何算法进行遍历，我们真正要做的事情内容被更好地体现在代码中。



## 流式思想

整体来看，流式思想类似于工厂车间的“生产流水线”。当需要对多个元素进行操作（特别是多步操作）的时候，考虑到性能及便利性，应该首先拼好一个“模型”步骤方案，然后再按照方案去执行它。

![](assets/Java8%20Stream流/7782a2ae2ada9f2d612cced91d1b8ee0_MD5.png)


这张图中展示了过滤、映射、跳过、计数等多步操作，这是一种集合元素的处理方案，而方案就是一种“函数模型”。图中的每一个方框都是一个“流”，调用指定的方法，可以从一个流模型转换为另一个流模型。而最右侧的数字3是最终结果。  

这里的filter、map、skip都是在对函数模型进行操作，集合元素并没有真正被处理。只有当终结方法 count执行的时候，整个模型才会按照指定策略执行操作。而这得益于Lambda的延迟执行特性。

Stream是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。**“集合讲的是数据，Stream讲的是计算！”**

* Stream 自己不会存储元素。
* Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
* Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。



Stream（流）是一个来自数据源的元素队列

* 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。  
* **数据源** 流的来源。 可以是集合，数组等。  



和以前的Collection操作不同， Stream操作还有两个基础的特征：

* **Pipelining：** 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluentstyle）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。  
* **内部迭代：** 以前对集合遍历都是通过Iterator或者增强for的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式，流可以直接调用遍历方法。

当使用一个流的时候，通常包括三个基本步骤：获取一个数据源（source）→ 数据转换 → 执行操作获取想要的结果，每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道。

**Stream 的操作三个步骤：**

1. 创建 Stream：一个数据源（如：集合、数组），获取一个流
2. 中间操作：一个中间操作链，对数据源的数据进行处理
3. 终止操作(终端操作)：一旦执行终止操作，就执行中间操作链，并产生结果。之后，不会再被使用

![](assets/Java8%20Stream流/35b726679399af4e5e89e38bdb158872_MD5.png)




## 获取流

`java.util.stream.Stream`是Java 8新加入的最常用的流接口。（不是函数式接口）

获取一个流非常简单，有以下几种常用的方式：

* 所有的Collection集合都可以通过stream默认方法获取流；
* Stream接口的静态方法of可以获取数组对应的流。
  * `public static  Stream of(T t)`：返回包含单个元素的序列 Stream 。
  * `public static  Stream of(T... values)`：返回其元素是指定值的顺序排序流。



### 通过集合

Java8 中的`Collection`接口被扩展，提供了两个获取流的方法：

* `default Stream stream()`：返回一个顺序流
* `default Stream parallelStream()`：返回一个并行流，可以理解为开多个线程同时操作流中的数据，但数据的顺序无法保证

```java
public class Demo04GetStream {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        // list.add...
        // 顺序流
        Stream<String> stream1 = list.stream();
        // 并行流
        Stream<String> stream2 = list.parallelStream();
    }
}
```


### 通过数组

Java8 中的`Arrays`的静态方法`stream()`可以获取数组流：

* `static <T> Stream<T> stream(T[] array)`：返回一个流

重载形式，能够处理对应基本类型的数组：

* `public static IntStream stream(int[] array)`
* `public static LongStream stream(long[] array)`
* `public static DoubleStream stream(double[] array)`

```java
public class Demo06GetStream {
    public static void main(String[] args) {
        String[] array = { "张无忌", "张翠山", "张三丰", "张一元" }; 
        Stream<String> stream1 = Arrays.stream(array);
    }
}
```


### 通过Stream的of()

可以调用Stream类静态方法`of()`，通过显示值创建一个流。它可以接收任意数量的参数。

* `public static Stream of(T... values)`：返回一个流

```java
public class Demo06GetStream {
    public static void main(String[] args) {
         Stream<String> stream = Stream.of("张无忌", "张翠山", "张三丰", "张一元");
    }
}
```

> **of** **方法的参数其实是一个可变参数，也支持数组。**



### 创建无限流

可以使用静态方法`Stream.iterate()`和`Stream.generate()`，创建无限流。

* 迭代：`public static Stream iterate(final T seed, final UnaryOperator f)`
* 生成：`public static Stream generate(Supplier s)`

```java
public void test4() {
    // 迭代
    // public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)
    Stream<Integer> stream = Stream.iterate(0, x -> x + 2);
    stream.limit(10).forEach(System.out::println);

    // 生成
    // public static<T> Stream<T> generate(Supplier<T> s)
    Stream<Double> stream1 = Stream.generate(Math::random);
    stream1.limit(10).forEach(System.out::println);
}
```


## Stream的中间操作

多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性全部处理，称为“惰性求值”。



### 筛选与切片

| **方法**              | **描述**                                                     |
| --------------------- | ------------------------------------------------------------ |
| `filter(Predicate p)` | 接收 Lambda ， 从流中排除某些元素                            |
| `distinct()`          | 筛选，通过流所生成元素的 hashCode() 和 equals() 去除重复元素 |
| `limit(long maxSize)` | 截断流，使其元素不超过给定数量                               |
| `skip(long n)`        | 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补 |



#### 过滤：filter

可以通过 filter 方法将一个流转换成另一个子集流。方法签名：

```java
Stream<T> filter(Predicate<? super T> predicate);
```

该接口接收一个 Predicate 函数式接口参数（可以是一个Lambda或方法引用）作为筛选条件。

![](assets/Java8%20Stream流/329fe4feac24d507893aa7b1da93b5b4_MD5.png)


`java.util.stream.Predicate`函数式接口，其中唯一的抽象方法为：

```java
boolean test(T t);
```

该方法将会产生一个boolean值结果，代表指定的条件是否满足。如果结果为true，那么Stream流的filter方法将会留用元素；如果结果为false，那么filter方法将会舍弃元素。

**Stream流中的 filter 方法基本使用的代码如：**

```java
public class Demo07StreamFilter {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若"); 
        Stream<String> result = original.filter(s ‐> s.startsWith("张"));
    }
}
```

在这里通过Lambda表达式来指定了筛选的条件：必须姓张。

 

#### 取用前几个：limit

limit 方法可以对流进行截取，只取用前n个。方法签名：

```java
Stream<T> limit(long maxSize);
```

参数是一个long型，如果集合当前长度大于参数则进行截取；否则不进行操作。

```java
public class Demo10StreamLimit {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若"); 
        Stream<String> result = original.limit(2);
        System.out.println(result.count()); // 2
    }
}
```


#### 跳过前几个：skip

如果希望跳过前几个元素，可以使用 skip 方法获取一个截取之后的新流：

```java
Stream<T> skip(long n);
```

如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流。

```java
public class Demo11StreamSkip {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若"); 
        Stream<String> result = original.skip(2);
        System.out.println(result.count()); // 1
    }
}
```




### 映射

| **方法**                          | **描述**                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| `map(Function f)`                 | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。 |
| `mapToDouble(ToDoubleFunction f)` | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 DoubleStream。 |
| `mapToInt(ToIntFunction f)`       | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 IntStream。 |
| `mapToLong(ToLongFunction f)`     | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 LongStream。 |
| `flatMap(Function f)`             | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |



#### 映射：map

如果需要将流中的元素映射到另一个流中，可以使用map方法。方法签名：

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

该接口需要一个 Function 函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。

![](assets/Java8%20Stream流/efdc43d9d834299ac4aff7c2b0070e5a_MD5.png)


`java.util.stream.Function`函数式接口，其中唯一的抽象方法为：

```java
R apply(T t);
```

这可以将一种T类型转换成为R类型，而这种转换的动作，就称为“映射”。

**Stream流中的 map 方法基本使用的代码如：**

```java
public class Demo08StreamMap {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("10", "12", "18");
        Stream<Integer> result = original.map(str‐>Integer.parseInt(str)); 
    }
}
```

这段代码中， map 方法的参数通过方法引用，将字符串类型转换成为了int类型（并自动装箱为 Integer 类对象）。



#### 连接：flatMap

```java
@Test
public void test2(){
    List<String> list = Arrays.asList("aa", "bb", "cc", "dd");
    // flatMap(Function f)——接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。
    Stream<Character> characterStream = list.stream().flatMap(this::fromStringToStream);
    characterStream.forEach(System.out::println);
}

private Stream<Character> fromStringToStream(String str){
    ArrayList<Character> list = new ArrayList<>();
    for(Character c : str.toCharArray()){
        list.add(c);
    }
    return list.stream();
}
```


### 排序

| **方法**                 | **描述**                           |
| ------------------------ | ---------------------------------- |
| `sorted()`               | 产生一个新流，其中按自然顺序排序   |
| `sorted(Comparator com)` | 产生一个新流，其中按比较器顺序排序 |



```java
@Test
public void test4(){
    List<Integer> list = Arrays.asList(12, 43, 65, 34, 87, 0, -98, 7);
    list.stream().sorted().forEach(System.out::println);
    // 抛异常，原因:Employee没有实现Comparable接口
    List<Employee> employees = EmployeeData.getEmployees();
    employees.stream().sorted().forEach(System.out::println);


    // 定制排序
    List<Employee> employees = EmployeeData.getEmployees();
    employees.stream().sorted((e1,e2) -> {
       int ageValue = Integer.compare(e1.getAge(),e2.getAge());
       if(ageValue != 0){
           return ageValue;
       }else{
           return -Double.compare(e1.getSalary(),e2.getSalary());
       }
    }).forEach(System.out::println);
}
```


### 组合

#### 组合：concat

如果有两个流，希望合并成为一个流，那么可以使用Stream接口的静态方法concat：

```java
static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
```

> 这是一个静态方法，与java.lang.String当中的concat方法不同。



```java
public class Demo12StreamConcat {
    public static void main(String[] args) {
        Stream<String> streamA = Stream.of("张无忌");
        Stream<String> streamB = Stream.of("张翠山");
        Stream<String> result = Stream.concat(streamA, streamB); 
    }
}
```


## Stream的终止操作

终端操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是 void 。

> 流进行了终止操作后，不能再次使用。



### 匹配与查找

| **方法**               | **描述**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| allMatch(Predicate p)  | 检查是否匹配所有元素                                         |
| anyMatch(Predicate p)  | 检查是否至少匹配一个元素                                     |
| noneMatch(Predicate p) | 检查是否没有匹配所有元素                                     |
| findFirst()            | 返回第一个元素                                               |
| findAny()              | 返回当前流中的任意元素                                       |
| count()                | 返回流中元素总数                                             |
| max(Comparator c)      | 返回流中最大值                                               |
| min(Comparator c)      | 返回流中最小值                                               |
| forEach(Consumer c)    | 内部迭代(使用 Collection 接口需要用户去做迭代，称为外部迭代。相反，Stream API 使用内部迭代——它帮你把迭代做了) |



#### 逐一处理：forEach

虽然方法名字叫 forEach ，但是与for循环中的“for-each”昵称不同。

```java
void forEach(Consumer<? super T> action);
```

该方法接收一个Consumer接口函数，会将每一个流元素交给该函数进行处理。

`java.util.function.Consumer<T>`接口是一个消费型接口。

Consumer接口中包含抽象方法`void accept(T t)`，意为消费一个指定泛型的数据。

```java
public class Demo12StreamForEach {
    public static void main(String[] args) {
        Stream<String> stream = Stream.of("张无忌", "张三丰", "周芷若"); 
        stream.forEach(name‐> System.out.println(name));
    }
}
```


#### 统计个数：count

正如旧集合Collection当中的size方法一样，流提供count方法来数一数其中的元素个数：

```java
long count();
```

该方法返回一个long值代表元素个数（不再像旧集合那样是int值）。

```java
public class Demo09StreamCount {
    public static void main(String[] args) {
        Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若"); 
        Stream<String> result = original.filter(s ‐> s.startsWith("张"));
        System.out.println(result.count()); // 2
    }
}
```


### 归约

| **方法**                           | **描述**                                              |
| ---------------------------------- | ----------------------------------------------------- |
| `reduce(T iden, BinaryOperator b)` | 可以将流中元素反复结合起来，得到一个值。返回 T        |
| `reduce(BinaryOperator b)`         | 可以将流中元素反复结合起来，得到一个值。返回 Optional |

> map 和 reduce 的连接通常称为 map-reduce 模式，因 Google用它来进行网络搜索而出名。

```java
@Test
public void test3(){
    // reduce(T identity, BinaryOperator)——可以将流中元素反复结合起来，得到一个值。返回 T
    // 计算1-10的自然数的和
    List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
    Integer sum = list.stream().reduce(0, Integer::sum);
    System.out.println(sum);


     // reduce(BinaryOperator) ——可以将流中元素反复结合起来，得到一个值。返回 Optional<T>
     // 计算公司所有员工工资的总和
    List<Employee> employees = EmployeeData.getEmployees();
    Stream<Double> salaryStream = employees.stream().map(Employee::getSalary);
    // Optional<Double> sumMoney = salaryStream.reduce(Double::sum);
    Optional<Double> sumMoney = salaryStream.reduce((d1,d2) -> d1 + d2);
    System.out.println(sumMoney.get());

}
```


### 收集

| **方法**               | **描述**                                                     |
| ---------------------- | ------------------------------------------------------------ |
| `collect(Collector c)` | 将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法 |

Collector 接口中方法的实现决定了如何对流执行收集的操作(如收集到 List、Set、Map)。

> Collectors 实用类提供了很多静态方法，可以方便地创建常见收集器实例。

```java
@Test
public void test4(){
    // collect(Collector c)——将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法
    // 查找工资大于6000的员工，结果返回为一个List或Set
    List<Employee> employees = EmployeeData.getEmployees();
    List<Employee> employeeList = employees.stream().filter(e -> e.getSalary() > 6000).collect(Collectors.toList());
    employeeList.forEach(System.out::println);

    System.out.println();
    Set<Employee> employeeSet = employees.stream().filter(e -> e.getSalary() > 6000).collect(Collectors.toSet());
    employeeSet.forEach(System.out::println);
}
```