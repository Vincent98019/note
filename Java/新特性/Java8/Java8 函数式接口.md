Java从诞生日起就是一直倡导“一切皆对象”，在Java里面面向对象(OOP)编程是一切。但是随着python、scala等语言的兴起和新技术的挑战，Java不得不做出调整以便支持更加广泛的技术要求，也即Java不但可以支持OOP还可以支持OOF（面向函数编程）。

在将函数作为一等公民的编程语言中，Lambda表达式的类型是函数。但是在Java8中，Lambda表达式是对象，而不是函数，它们必须依附于一类特别的对象类型——函数式接口。

在Java8中，Lambda表达式就是一个函数式接口的实例。只要一个对象是函数式接口的实例，那么该对象就可以用Lambda表达式来表示。所以以前用匿名实现类表示的现在都可以用Lambda表达式来写。



函数式接口在Java中是指：**有且仅有一个抽象方法的接口**。

函数式接口适用于函数式编程场景的接口。可以通过 Lambda 表达式来创建该接口的对象。

> 若 Lambda 表达式抛出一个受检异常，即非运行时异常，那么该异常需要在目标接口的抽象方法上进行声明。



## 格式

只要确保接口中有且仅有一个抽象方法即可（可以有其他默认方法、静态方法、私有方法）：

```java
修饰符 interface 接口名称 {
    public abstract 返回值类型 方法名称(可选参数信息); 
    // 其他非抽象方法内容
}
```


由于接口当中抽象方法的`public abstract`是可以省略的，所以定义一个函数式接口很简单：

```java
public interface MyFunctionalInterface { 
    void myMethod();
}
```


## `@FunctionalInterface`注解

与`@Override`注解的作用类似，Java 8中专门为函数式接口引入了一个新的注解，可以在一个接口上使用`@FunctionalInterface`注解，这样做可以检查它是否是一个函数式接口。同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。

```java
// 函数式接口的定义：
@FunctionalInterface
public interface MyFunctionalInterface { 
    void myMethod();
}

// 函数式接口的使用：
public class DemoFunctionalInterface {
    // 使用自定义的函数式接口作为方法参数
    private static void doSomething(MyFunctionalInterface inter) { 
        inter.myMethod(); // 调用自定义的函数式接口方法
    }
    public static void main(String[] args) {
        // 调用使用函数式接口的方法
        doSomething(() ‐> System.out.println("Lambda执行啦！"));
    }
}
```

一旦使用该注解来定义接口，编译器将会强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错。即使不使用该注解，只要满足函数式接口的定义，这仍然是一个函数式接口，使用起来都一样。



## 四大核心函数式接口

JDK提供了大量常用的函数式接口以丰富Lambda的典型使用场景，它们主要在在`java.util.function`包中被提供。

### Supplier接口

`java.util.function.Supplier`接口仅包含一个无参的方法：`T get()`。用来获取一个泛型参数指定类型的对象数据。由于这是一个函数式接口，这也就意味着对应的Lambda表达式需要“对外提供”一个符合泛型类型的对象数据。

```java
import java.util.function.Supplier;
public class Demo08Supplier {
    private static String getString(Supplier<String> function) { 
        return function.get();
    }
    public static void main(String[] args) {
        String msgA = "Hello";
        String msgB = "World";
        System.out.println(getString(() ‐> msgA + msgB));
    }
}
```


### Consumer接口

`java.util.function.Consumer<T>`接口则与`Supplier`接口相反，它不是生产一个数据，而是消费一个数据，其数据类型由泛型决定。

#### 抽象方法accept

`Consumer`接口中包含抽象方法`void accept(T t)`，意为消费一个指定泛型的数据。基本使用如：

```java
import java.util.function.Consumer;
public class Demo09Consumer {
    private static void method(String name, Consumer<String> con) { 
        con.accept(name);
    }
    public static void main(String[] args) {
        method("乔木先生", name -> System.out.println("我的名字是" + name));
    }
}
```


#### 默认方法andThen

如果一个方法的参数和返回值全都是`Consumer`类型，那么就可以实现效果：消费数据的时候，首先做一个操作，然后再做一个操作，实现组合。而这个方法就是`Consumer`接口中的default方法`andThen`。要想实现组合，需要两个或多个Lambda表达式即可，而`andThen`的语义正是“一步接一步”操作。例如两个步骤组合的情况：

```java
import java.util.function.Consumer;
public class Demo10ConsumerAndThen {
    private static void consumeString(Consumer<String> one, Consumer<String> two) { 
        one.andThen(two).accept("Hello");
    }
    public static void main(String[] args) {
        consumeString(
        s ‐> System.out.println(s.toUpperCase()),
        s ‐> System.out.println(s.toLowerCase()));
    }
}
```

运行结果将会首先打印完全大写的HELLO，然后打印完全小写的hello。当然，通过链式写法可以实现更多步骤的组合。



### Predicate接口

有时候我们需要对某种类型的数据进行判断，从而得到一个boolean值结果。这时可以使用`java.util.function.Predicate`接口。

#### 抽象方法test

`Predicate`接口中包含一个抽象方法：`boolean test(T t)`。用于条件判断的场景：

```java
import java.util.function.Predicate;
public class Demo15PredicateTest {
    private static void method(Predicate<String> predicate) { 
        boolean veryLong = predicate.test("HelloWorld");
        System.out.println("字符串很长吗：" + veryLong);
    }
    public static void main(String[] args) {
        method(s ‐> s.length() > 5);
    }
}
```

条件判断的标准是传入的Lambda表达式逻辑，只要字符串长度大于5则认为很长。



#### 默认方法and

既然是条件判断，就会存在与、或、非三种常见的逻辑关系。其中将两个`Predicate`条件使用“与”逻辑连接起来实现“并且”的效果时，可以使用default方法`and`。

如果要判断一个字符串既要包含大写“H”，又要包含大写“W”，那么：

```java
import java.util.function.Predicate;
public class Demo16PredicateAnd {
    private static void method(Predicate<String> one, Predicate<String> two) { 
        boolean isValid = one.and(two).test("Helloworld");
        System.out.println("字符串符合要求吗：" + isValid);
    }
    public static void main(String[] args) {
        method(s ‐> s.contains("H"), s ‐> s.contains("W"));
    }
}
```


#### 默认方法or

与`and`的“与”类似，默认方法`or`实现逻辑关系中的“或”。

如果希望实现逻辑“字符串包含大写H或者包含大写W”，那么代码只需要将`and`修改为`or`名称即可，其他都不变：

```java
import java.util.function.Predicate;
public class Demo16PredicateAnd {
    private static void method(Predicate<String> one, Predicate<String> two) { 
        boolean isValid = one.or(two).test("Helloworld");
        System.out.println("字符串符合要求吗：" + isValid);
    }
    public static void main(String[] args) {
        method(s ‐> s.contains("H"), s ‐> s.contains("W"));
    }
}
```


#### 默认方法negate

从实现中很容易看出，它是执行了`test`方法之后，对结果boolean值进行`!`取反而已。一定要在`test`方法调用之前调用`negate`方法，正如`and`和`or`方法一样：

```java
import java.util.function.Predicate;
public class Demo17PredicateNegate {
    private static void method(Predicate<String> predicate) {
        boolean veryLong = predicate.negate().test("HelloWorld"); 
        System.out.println("字符串很长吗：" + veryLong);
    }
    public static void main(String[] args) {
        method(s ‐> s.length() < 5);
    }
}
```


### Function接口

`java.util.function.Function<T,R>`接口用来根据一个类型的数据得到另一个类型的数据，前者称为前置条件，后者称为后置条件。

#### 抽象方法apply

Function 接口中最主要的抽象方法为：`R apply(T t)`，根据类型`T`的参数获取类型`R`的结果。

使用的场景例如：将`String`类型转换为`Integer`类型。

```java
import java.util.function.Function;
public class Demo11FunctionApply {
    private static void method(Function<String, Integer> function) { 
        int num = function.apply("10");
        System.out.println(num + 20);
    }
    public static void main(String[] args) {
        method(s ‐> Integer.parseInt(s));
    }
}
```

当然，最好是通过方法引用的写法。



#### 默认方法andThen

`Function`接口中有一个默认的`andThen`方法，用来进行组合操作。

```java
import java.util.function.Function;
public class Demo12FunctionAndThen {
    private static void method(Function<String, Integer> one, Function<Integer, Integer> two) { 
        int num = one.andThen(two).apply("10");
        System.out.println(num + 20);
    }
    public static void main(String[] args) {
        method(str‐>Integer.parseInt(str)+10, i ‐> i *= 10);
    }
}
```

第一个操作是将字符串解析成为`int`数字，第二个操作是乘以10。两个操作通过`andThen`按照前后顺序组合到了一起。

> Function的前置条件泛型和后置条件泛型可以相同。



## 其他接口

| **函数式接口**                                               | **参数类型**                | **返回类型**                | **用途**                                                     |
| ------------------------------------------------------------ | --------------------------- | --------------------------- | ------------------------------------------------------------ |
| `BiFunction<T, U, R>`                                        | `T, U`                      | `R`                         | 对类型为`T, U`参数应用操作，返回`R`类型的结果。<br>包含方法为：`R apply(T t, U u);` |
| `UnaryOperator`<br>(`Function`子接口)                        | `T`                         | `T`                         | 对类型为`T`的对象进行一元运算，并返回`T`类型的结果。<br>包含方法为：`T apply(T t);` |
| `BinaryOperator`<br>(`BiFunction`子接口)                     | `T, T`                      | `T`                         | 对类型为`T`的对象进行二元运算，并返回`T`类型的结果。<br>包含方法为：`T apply(T t1, T t2);` |
| `BiConsumer<T, U>`                                           | `T, U`                      | `void`                      | 对类型为`T, U`参数应用操作。<br>包含方法为：`void accept(T t, U u)` |
| `BiPredicate<T,U>`                                           | `T,U`                       | `boolean`                   | 包含方法为：`boolean test(T t,U u)`                          |
| `ToIntFunction<T>`<br>`ToLongFunction<T>`<br>`ToDoubleFunction<T>` | `T`                         | `int`<br>`long`<br>`double` | 分别计算`int`、`long`、`double`值的函数                      |
| `IntFunction<R>`<br>`LongFunction<R>`<br>`DoubleFunction<R>` | `int`<br>`long`<br>`double` | `R`                         | 参数分别为`int`、`long`、`double`类型的函数                  |



