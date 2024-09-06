
到目前为止，臭名昭著的空指针异常是导致Java应用程序失败的最常见原因。

以前，为了解决空指针异常，Google公司著名的Guava项目引入了Optional类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，Optional类已经成为Java 8类库的一部分。

`Optional<T>`类(`java.util.Optional`) 是一个容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null，表示这个值不存在。原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避免空指针异常。

Optional类的Javadoc描述如下：这是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。


创建Optional类对象的方法：

* `Optional.of(T t)` ：创建一个 Optional 实例，t必须非空；
* `Optional.empty()` ：创建一个空的 Optional 实例
* `Optional.ofNullable(T t)` ：t可以为null



判断Optional容器中是否包含对象：

* `boolean isPresent()` ：判断是否包含对象
* `void ifPresent(Consumer<? super T> consumer)` ：如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它。



获取Optional容器的对象：

* `T get()` ：如果调用对象包含值，返回该值，否则抛异常
* `T orElse(T other)` ：如果有值则将其返回，否则返回指定的other对象。
* `T orElseGet(Supplier<? extends T> other)` ：如果有值则将其返回，否则返回由Supplier接口实现提供的对象。
* `T orElseThrow(Supplier<? extends X> exceptionSupplier)` ：如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。



```java
@Test
public void test1() {
    Boy b = new Boy("张三");
    Optional<Girl> opt = Optional.ofNullable(b.getGrilFriend());
    // 如果女朋友存在就打印女朋友的信息
    opt.ifPresent(System.out::println);
}
```


```java
@Test
public void test2() {
    Boy b = new Boy("张三");
    Optional<Girl> opt = Optional.ofNullable(b.getGrilFriend());
    // 如果有女朋友就返回他的女朋友，否则只能欣赏“嫦娥”了
    Girl girl = opt.orElse(new Girl("嫦娥"));
    System.out.println("他的女朋友是：" + girl.getName());
}
```


```java
@Test
public void test3(){
    Optional<Employee> opt = Optional.of(new Employee("张三", 8888));
    //判断opt中员工对象是否满足条件，如果满足就保留，否则返回空
    Optional<Employee> emp = opt.filter(e -> e.getSalary()>10000);
    System.out.println(emp);
}
```


```java
@Test
public void test4(){
    Optional<Employee> opt = Optional.of(new Employee("张三", 8888));
    //如果opt中员工对象不为空，就涨薪10%
    Optional<Employee> emp = opt.map(e -> {
        e.setSalary(e.getSalary()%1.1);
        return e;
    });
    System.out.println(emp);
}
```