在使用Lambda表达式的时候，实际上传递进去的代码就是一种解决方案：拿什么参数做什么操作。如果我们在Lambda中所指定的操作方案，已经有地方存在相同方案，没有必要再写重复逻辑。

当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用，方法引用可以看做是Lambda表达式深层次的表达。

换句话说，方法引用就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法，可以认为是Lambda表达式的一个语法糖。

看一个简单的函数式接口以应用Lambda表达式：

```java
@FunctionalInterface
public interface Printable { 
    void print(String str);
}
```

在`Printable`接口当中唯一的抽象方法`print`接收一个字符串参数，目的就是为了打印显示它。通过Lambda来使用它的代码很简单：

```java
public class Demo01PrintSimple {
    private static void printString(Printable data) { 
        data.print("Hello, World!");
    }
    public static void main(String[] args) {
        printString(s ‐> System.out.println(s));
    }
}
```

`printString`方法只管调用`Printable`接口的`print`方法，`main`方法通过Lambda表达式指定了函数式接口`Printable`的具体操作方案为：拿到String（类型可推导，所以可省略）数据后，在控制台中输出它。

这段代码的问题在于，对字符串进行控制台打印输出的操作方案，已经有了现成的实现，那就是`System.out`对象中的`println(String)` 方法。Lambda希望做的事情就是调用`println(String)`方法，不必自己手动调用。省去Lambda的语法格式，只要“引用”过去就好了：

```java
public class Demo02PrintRef {
    private static void printString(Printable data) { 
        data.print("Hello, World!");
    }
    public static void main(String[] args) {
        printString(System.out::println);
    }
}
```


## 方法引用符

双冒号 `::` 为引用运算符，而它所在的表达式被称为方法引用。如果Lambda要表达的函数方案已经存在于某个方法的实现中，那么则可以通过双冒号来引用该方法作为Lambda的替代者。

**要求：** 实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的方法的参数列表和返回值类型保持一致。

**格式：** 使用操作符 `::` 将类(或对象) 与 方法名分隔开来。

如下三种主要使用情况：

* `对象::实例方法名`
* `类::静态方法名`
* `类::实例方法名`



### 语义分析

上例中，`System.out`对象中有一个重载的`println(String)`方法恰好就是我们所需要的。对于`printString`方法的函数式接口参数，对比下面两种写法，完全等效：

* Lambda表达式写法： `s -> System.out.println(s);`
* 方法引用写法： `System.out::println`

> **第一种语义是指：** 拿到参数之后经Lambda之手，继而传递给`System.out.println`方法去处理。
> **第二种等效写法的语义是指：** 直接让`System.out`中的`println`方法来取代Lambda。
> 两种写法的执行效果完全一样，而第二种方法引用的写法复用了已有方案，更加简洁。



### 推导与省略

如果使用Lambda，那么根据“可推导就是可省略”的原则，无需指定参数类型，也无需指定的重载形式——**它们都将被自动推导**。而如果使用方法引用，也是同样可以根据上下文进行推导。

函数式接口是Lambda的基础，而方法引用是Lambda的孪生兄弟。

下面这段代码将会调用`println`方法的不同重载形式，将函数式接口改为int类型的参数：

```java
@FunctionalInterface
public interface PrintableInteger { 
    void print(int str);
}
```

由于上下文变了之后可以自动推导出唯一对应的匹配重载，所以方法引用没有任何变化：

```java
public class Demo03PrintOverload {
    private static void printInteger(PrintableInteger data) { 
        data.print(1024);
    }
    public static void main(String[] args) {
        printInteger(System.out::println);
    }
}
```

这次方法引用将会自动匹配到`println(int)`的重载形式。



## 对象名引用成员方法

这是最常见的一种用法，与上例相同。如果一个类中已经存在了一个成员方法：

```java
public class MethodRefObject {
    public void printUpperCase(String str) { 
        System.out.println(str.toUpperCase());
    }
}
```


函数式接口仍然定义为：

```java
@FunctionalInterface
public interface Printable { 
    void print(String str);
}
```


那么当需要使用这个`printUpperCase`成员方法来替代`Printable`接口的Lambda的时候，已经具有了`MethodRefObject`类的对象实例，则可以通过对象名引用成员方法，代码为：

```java
public class Demo04MethodRef {
    private static void printString(Printable lambda) { 
        lambda.print("Hello");
    }
    public static void main(String[] args) {
        MethodRefObject obj = new MethodRefObject(); 
        printString(obj::printUpperCase);
    }
}
```


## 通过类名称引用静态方法

在`java.lang.Math`类中已经存在静态方法`abs`，当我们需要通过Lambda来调用该方法时，有两种写法。

首先是函数式接口：

```java
@FunctionalInterface
public interface Calcable { 
    int calc(int num);
}
```


对比：

```java
public class Demo05Lambda {
    private static void method(int num, Calcable lambda) { 
        System.out.println(lambda.calc(num));
    }
    public static void main(String[] args) {
        // Lambda写法：
        method((‐10, n) ‐> Math.abs(n));
        // 方法引用：
        method(‐10, Math::abs);
    }
}
```


## 通过super引用成员方法

如果存在继承关系，当Lambda中需要出现`super`调用时，也可以使用方法引用进行替代。

函数式接口：

```java
@FunctionalInterface
public interface Greetable { 
    void greet();
}
```


父类 Human ：

```java
public class Human {
    public void sayHello() {
        System.out.println("Hello!");
    }
}
```


子类 Man ，其中使用了Lambda调用父类的`sayHello`方法：

```java
public class Man extends Human {
    @Override
    public void sayHello() {
        System.out.println("大家好，我是Man!");
    }
    // 定义方法method，参数传递Greetable接口 
    public void method(Greetable g){
        g.greet();
    }
    public void show(){
        // Lambda写法：
        method(() ‐> super.sayHello());
        // 方法引用：
        method(super::sayHello);
    }
}
```


## 通过this引用成员方法

`this`代表当前对象，如果需要引用的方法就是当前类中的成员方法，那么可以使用`this::成员方法`的格式来使用方法引用。

函数式接口：

```java
@FunctionalInterface
public interface Richable { 
    void buy();
}
```


丈夫 Husband 类：

方法`beHappy`调用了方法`marry`，后者的参数为函数式接口`Richable`，所以需要一个Lambda表达式。

```java
public class Husband {
    private void buyHouse() {
        System.out.println("买套房子");
    }
    private void marry(Richable lambda) { 
        lambda.buy();
    }
    public void beHappy() {
        // Lambda写法：
        marry(() ‐> this.buyHouse());
        // 方法引用：
        marry(this::buyHouse);
    }
}
```


## 类的构造器引用

由于构造器的名称与类名完全一样，并不固定。所以构造器引用使用`类名称::new`的格式表示。

Person 类：

```java
public class Person {
    private String name;
    public Person(String name) {
        this.name = name;
    }
    // get方法...
    // set方法...
}
```


创建 Person 对象的函数式接口：

```java
public interface PersonBuilder {
    Person buildPerson(String name); 
}
```


对比：

```java
public class Demo09Lambda {
    public static void printName(String name, PersonBuilder builder) { 
        System.out.println(builder.buildPerson(name).getName());
    }
    public static void main(String[] args) {
        // Lambda写法：
        printName("赵丽颖", name ‐> new Person(name));
        // 方法引用：
        printName("赵丽颖", Person::new);
    }
}
```


## 数组的构造器引用

数组也是 Object 的子类对象，所以同样具有构造器，只是语法稍有不同。

如果对应到Lambda的使用场景中时，需要一个函数式接口：

```java
@FunctionalInterface
public interface ArrayBuilder { 
    int[] buildArray(int length);
}
```


对比：

```java
public class Demo11ArrayInitRef {
    private static int[] initArray(int length, ArrayBuilder builder) { 
        return builder.buildArray(length);
    }
    public static void main(String[] args) {
        // Lambda写法：
        int[] array = initArray(10, length ‐> new int[length]);
        // 方法引用：
        int[] array = initArray(10, int[]::new);
    }
}
```