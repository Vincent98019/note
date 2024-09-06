Lambda 是一个匿名函数，是在Java 8 语言中引入的一种新的语法元素和操作符。可以理解为是一段可以传递的代码（将代码像数据一样进行传递）。可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升。



## 冗余的代码

当需要启动一个线程去完成任务时，通常使用`java.lang.Runnable`接口来定义任务内容，并使用`java.lang.Thread`类来启动该线程。代码如下：

```java
public class Demo01Runnable {  
     public static void main(String[] args) {  
        // 匿名内部类  
         Runnable task = new Runnable() {  
             @Override  
             public void run() { // 覆盖重写抽象方法  
                 System.out.println("多线程任务执行！");  
             }  
         };  
        new Thread(task).start(); // 启动线程  
    }  
 }
```

对于 Runnable 的匿名内部类用法，**可以分析出几点内容：**

1. Thread 类需要 Runnable 接口作为参数，其中的抽象 run 方法是用来指定线程任务内容的核心；
2. 为了指定 run 的方法体，**不得不**需要 Runnable 接口的实现类；
3. 为了省去定义一个 RunnableImpl 实现类的麻烦，**不得不**使用匿名内部类；
4. 必须覆盖重写抽象 run 方法，所以方法名称、方法参数、方法返回值**不得不**再写一遍，且不能写错；
5. 而实际上，**似乎只有方法体才是关键所在**。



## Lambda的更优写法

借助Java 8的全新语法，上述 Runnable 接口的匿名内部类写法可以通过更简单的Lambda表达式达到等效：

```java
public class Demo02LambdaRunnable {
     public static void main(String[] args) {
        // 启动线程 
        new Thread(() -> System.out.println("多线程任务执行！")).start(); 
     }  
 }
```

这段代码和刚才的执行效果是完全一样的，可以在1.8或更高的编译级别下通过。



## Lambda的标准格式

Lambda省去面向对象的条条框框，格式由3个部分组成：

1. 一些参数
2. 一个箭头，被称为 Lambda 操作符或箭头操作符。
3. 一段代码

**Lambda表达式的标准格式为：**

```java
(参数类型 参数名称) -> { 代码语句 }
```

**格式说明：**

1. 小括号内的语法与传统方法参数列表一致：无参数则留空；多个参数则用逗号分隔。
2. `->`是新引入的语法格式，代表指向动作。
3. 大括号内的语法与传统方法体要求基本一致。



**省略规则：**

**在Lambda标准格式的基础上，使用省略写法的规则为：**

1. 小括号内参数的类型可以省略；
2. 如果小括号内有且仅有一个参，则小括号可以省略；
3. 如果大括号内有且仅有一个语句，则无论是否有返回值，都可以省略大括号、return关键字及语句分号。



## Lambda的使用前提

Lambda的语法非常简洁，完全没有面向对象复杂的束缚。但是使用时有几个问题需要特别注意：

1. 使用Lambda必须具有接口，且要求接口中有且仅有一个抽象方法。 无论是JDK内置的Runnable、Comparator接口还是自定义的接口，只有当接口中的抽象方法存在且唯一时，才可以使用Lambda。
2. 使用Lambda必须具有上下文推断。 也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用Lambda作为该接口的实例。

> 有且仅有一个抽象方法的接口，称为“函数式接口”。



## Lambda的使用

**语法格式一：** 无参，无返回值

```java
Runnable r1 = () -> {System.out.println("Hello");}
```


**语法格式二：** 需要一个参数，但是没有返回值

```java
Consumer<String> con = (String str) -> {System.out.println("Hello, " + str);}
```


**语法格式三：数据类型可以省略，** 因为可由编译器推断得出，称为“类型推断”

```java
Consumer<String> con = (str) -> {System.out.println("Hello, " + str);}
```


**语法格式四：** 若只需要一个参数时，**参数的小括号可以省略**

```java
Consumer<String> con = str -> {System.out.println("Hello, " + str);}
```


**语法格式五：** 需要两个或以上的参数，多条执行语句，并且可以有返回值

```java
Comparator<Integer> com = (x, y) -> {
    System.out.println("实现函数式接口方法");
    return Integer.compare(x, y);
};
```


**语法格式六：** 当Lambda体只有一条语句时，**return 与大括号若有，都可以省略**

```java
Comparator<Integer> com = (x, y) -> Integer.compare(x, y);
```


## 类型推断

上述 Lambda 表达式中的参数类型都是由编译器推断得出的。Lambda表达式中无需指定类型，程序依然可以编译，这是因为 javac 根据程序的上下文，在后台推断出了参数的类型。Lambda 表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的“类型推断”。

![](assets/Java8%20Lambda表达式/7483b2fb61ee25cc9f591e86a53e7950_MD5.png)