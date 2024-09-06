
## 打印流

平时在控制台的打印输出是调用`print`方法和`println`方法完成的，这两个方法都来自于`java.io.PrintStream`类，该类能够方便地打印各种数据类型的值，是一种便捷的输出方式。继承于`java.io.OutputStream` 。

### PrintStream类

**构造方法**

* `public PrintStream(String fileName)`：使用指定的文件名创建一个新的打印流。（输出的目的地是一个文件路径）
* `public PrintStream(File file)`：创建具有指定文件且不带自动行刷新的新打印流。（输出的目的地是一个文件）
* `public PrintStream(OutputStream out)`：创建新的打印流。（输出的目的地是一个字节输出流）

```java
PrintStream ps = new PrintStream("ps.txt")；
```

### 改变打印流向

`System.out`就是`PrintStream`类型的，只不过它的流向是系统规定的，打印在控制台上。因为是流对象，可以通过`System.setOut(PrintStream ps)`方法改变它的流向。

* `System.setOut(PrintStream ps)`：改变改变打印流的流向，将输出的数据打印到指定的位置。

```java
public static void main(String[] args) throws IOException {
    // 调用系统的打印流，控制台直接输出97
    System.out.println(97);
    // 创建打印流，指定文件的名称
    PrintStream ps = new PrintStream("ps.txt");
    // 设置系统的打印流流向，输出到ps.txt
    System.setOut(ps);
    // 调用系统的打印流，ps.txt中输出97
    System.out.println(97);
}
```


### 还原打印流向

> 还原打印流向可以先创建一个`PrintStream`对象保存原有的`System.Out`，还原时使用该对象还原。

```java
public static void main(String[] args) throws FileNotFoundException {
    //先创建一个PrintStream对象保存原有的流向
    PrintStream out = System.out;
    //此时打印输出语句还在系统中
    System.out.println("第一句话！");
    //创建有地址的PrintStream对象
    PrintStream ps = new PrintStream("E:\\123.txt");
    //将流向设置为文件
    System.setOut(ps);
    //此时打印的输出语句流向至文件中
    System.out.println("第二句话！");
    //还原流向
    System.setOut(out);
    //此时的输出语句打印在系统中，已经还原
    System.out.println("第三句话！");
}
```


> 如果使用继承自父类的`write`方法写数据，在查看数据的时候会查询编码表（97→a）。
> 如果使用自己特有方法`print`/`println`方法写数据，将原样输出（97→97）。



### PrintWriter类

**构造方法**

* `public PrintWriter(String fileName)`：使用指定的文件名创建一个新的打印流。（输出的目的地是一个文件路径）
* `public PrintWriter(File file)`：创建具有指定文件且不带自动行刷新的新打印流。（输出的目的地是一个文件）
* `public PrintWriter(OutputStream out)`：创建新的打印流。（输出的目的地是一个字符输出流）

```java
PrintWriter ps = new PrintWriter(new FileWriter("E:\\ps.txt"));
```


## 标准输入输出流 

`java.lang.System`类中提供了大量的静态方法，可以获取与系统相关的信息或系统级操作。

### 标准输入流

`System.in`表示的是标准输入流，从键盘输入。

在Java中，要想通过控制台进行输入，首先需要构造一个`Scanner`对象，并与标准输入流 `System.in`关联。

* `System.in`的编译类型是`InputStream`
* `System.in`的运行类型是`BufferedInputStream`


**Scanner类常用方法：**

- `Scanner (InputStream in)`：用给定的输入流创建一个 Scanner 对象。
- `String nextLine()`：读取输入的下一行内容。
- `String next()`：读取输入的下一个单词（以空格作为分隔符。)
- `int nextlnt()`：读取并转换下一个表示整数的字符序列。
- `double nextDouble()`：读取并转换下一个表示浮点数的字符序列。
- `boolean hasNext()`：检测输入中是否还有其他单词。
- `boolean hasNextInt()`：检测是否还有表示整数的下一个字符序列。
- `boolean hasNextDouble()`：检测是否还有表示浮点数的下一个字符序列。

```java
// 创建Scanner构造函数，并传入参数
Scanner in = new Scanner(System.in);
// nextLine 方法将输入一行，这一行中可以包含空格
System.out.print("What is your name? ");
String name = in.nextLine();
// 读取一个单词(以空白符作为分隔符)
String firstName = in.next()；
// 要想读取一个整数
System.out.print("How old are you? ")；
int age = in.nextlnt();
```



### 标准输出流

`System.out`表示的是标准输出流，将数据输出到显示器。

在Java中，可以使用`System.out.print(x)`将数值 x 输出到控制台上。如果希望显示美元、美分等符号， 则有可能会出现问题。

* `System.out`的编译类型是`OutputStream`
* `System.out`的运行类型是`BufferedOutputStream`


Java SE 5.0 沿用了C语言库函数中的`printf`方法。例如，调用 `System.out.printf("%8.2f", x); `可以用 8 个字符的宽度和小数点后两个字符的精度打印 x。比如，`10000.0 / 3.0`，则打印`3333.33`。

在`printf`中，可以使用多个参数， 例如：`System.out.printf("Hello, %s. Next year, you'll be %d", name, age);` 每一个以 ％ 字符开始的格式说明符都用相应的参数替换。

还可以给出控制格式化输出的各种标志。例如，逗号标志增加了分组的分隔符。`Systen.out.printf("%,.2f", 10000.0 / 3.0);`打印`3,333.33` 。

| 转换符 | 类型 |
|----|----|
| d | 十进制整数 |
| x | 十六进制整数 |
| o | 八进制整数 |
| f | 定点浮点数 |
| e | 指数浮点数 |
| g | 通用浮点数 |
| a | 十六进制浮点数 |
| s | 字符串 |
| c | 字符 |
| b | 布尔 |
| h | 散列码 |
| tx 或 Tx | 日期时间(T强制大写）已经过时，应当改为使用java.time 类 |
| % | 百分号 |
| n | 与平台有关的行分隔符 |


还可以给出控制格式化输出的各种标志。表 3-6 列出了所有的标志。例如，逗号标志增加了分组的分隔符。 即
```java
Systen.out.printf("%,.2f", 10000.0 / 3.0); // 打印 3,333.33
```

| 标志 | 描述 |
|----|----|
| + | 打印正数和负数的符号 |
| 空格 | 在正数之前添加空格 |
| 0 | 数字前面补 0 |
| -	 | 左对齐 |
| ( | 将负数括在括号内 |
| , | 添加分组分隔符 |
| # (对于 f 格式） | 包含小数点 |
| # (对于 0x 或 0 格式） | 添加前缀 0x 或 0 |
| $ | 给定被格式化的参数索引。例如 `%1$d`，`%1$x` 将以十进制和十六进制格式打印第 1 个参数 |
| < | 格式化前面说明的数值。 例如`%d%<x` 以十进制和十六进制打印同一个数值 |

可以使用静态的 `String.format` 方法创建一个格式化的字符串， 而不打印输出：
```java
String message = String.format("Hello, %s. Next year, you'll be %d", name , age);
```

![](assets/Java打印流&标准输入输出/542b9a171b910b7026a622edf0b967a0_MD5.jpg)

