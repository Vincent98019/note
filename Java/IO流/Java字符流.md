

Java提供一些字符流类，以字符为单位读写数据，专门用于处理文本文件。

> 字符流，只能操作文本文件，不能操作图片，视频等非文本文件。

## 字符输入流

`java.io.Reader`抽象类是表示用于读取字符流的所有类的超类，可以读取字符信息到内存中。它定义了字符输入流的基本共性功能方法。


**常用方法**


* `public void close()`：关闭此流并释放与此流相关联的任何系统资源。
* `public int read()`： 从输入流读取一个字符。
* `public int read(char[] cbuf)`： 从输入流中读取一些字符，并将它们存储到字符数组cbuf中 。



### FileReader类

`java.io.FileReader`类是读取字符文件的便利类。构造时使用系统默认的字符编码和默认字节缓冲区。

> **字符编码：** 字节与字符的对应规则。Windows系统的中文编码默认是GBK编码表，而idea的默认码表是UTF-8。



**构造方法**

* `FileReader(File file)`： 创建一个新的 FileReader ，给定要读取的File对象。
* `FileReader(String fileName)`： 创建一个新的 FileReader ，给定要读取的文件的名称。

> 当创建一个流对象时，必须传入一个文件路径。类似于`FileInputStream` 。



```java
public static void main(String[] args) {
    // 使用File对象创建流对象
    File file = new File("a.txt");
    FileReader fr = new FileReader(file);
    // 使用文件名称创建流对象
    FileReader fr = new FileReader("b.txt");
}
```


#### 读取字符数据

1. 读取字符：`read()`方法，每次可以读取一个字符的数据，提升为int类型，读取到文件末尾，返回-1，循环读取。

```java
public static void main(String[] args) throws IOException {
    // 使用文件名称创建流对象
    FileReader fr = new FileReader("read.txt");
    // 定义变量，保存数据
    int b;
    // 循环读取
    while ((b = fr.read()) != -1) {
        System.out.println((char)b);
    }
    // 关闭资源
    fr.close();
}
```


2. 使用字符数组读取：`read(char[] cbuf)`，每次读取b的长度个字符到数组中，返回读取到的有效字符个数，读取到末尾时，返回-1 。

```java
public static void main(String[] args) throws IOException {
    // 使用文件名称创建流对象
    FileReader fr = new FileReader("read.txt");
    // 定义变量，保存有效字符个数
    int len;
    // 定义字符数组，作为装字符数据的容器
    char[] cbuf = new char[2];
    // 循环读取
    while ((len = fr.read(cbuf)) != -1) {
        System.out.println(new String(cbuf, 0, len));
    }
    // 关闭资源
    fr.close();
}
```


## 字符输出流

`java.io.Writer`抽象类是表示用于写出字符流的所有类的超类，将指定的字符信息写出到目的地。它定义了字节输出流的基本共性功能方法。



**常用方法**




* `public void write(int c)`：写入单个字符。
* `public void write(char[] cbuf)`：写入字符数组。
* `public abstract void write(char[] cbuf, int off, int len)`：写入字符数组的某一部分，off数组的开始索引，len写的字符个数。
* `public void write(String str)`：写入字符串。
* `public void write(String str, int off, int len)` ：写入字符串的某一部分，off字符串的开始索引，len写的字符个数。
* `public void flush()`：刷新该流的缓冲。
* `public void close()`：关闭此流，但要先刷新它。

> FileWrite使用后，必须要关闭(close)或刷新(flush)，否则写入不到指定的文件。



### FileWriter类

`java.io.FileWriter`类是写出字符到文件的便利类。构造时使用系统默认的字符编码和默认字节缓冲区。

**构造方法**

数据覆盖输出，清空这个文件的数据，从头开始添加数据：

* `FileWriter(File file)`： 创建一个新的 FileWriter，给定要读取的File对象。
* `FileWriter(String fileName)`： 创建一个新的 FileWriter，给定要读取的文件的名称。



数据追加续写
，保留目标文件中数据，继续添加新数据：

* `FileWriter(File file, boolean append)`： 创建一个新的 FileWriter，给定要读取的File对象。
* `FileWriter(String fileName, boolean append)`： 创建一个新的 FileWriter，给定要读取的文件的名称。

> 这两个构造方法，参数中都需要传入一个boolean类型的值，`true`表示追加数据，`false`表示清空原有数据。这样创建的输出流对象，就可以指定是否追加续写了。


> 当创建一个流对象时，必须传入一个文件路径，类似于FileOutputStream。

```java
public static void main(String[] args) throws IOException {
    // 使用File对象创建流对象
    File file = new File("a.txt");
    FileWriter fw = new FileWriter(file);
    // 使用文件名称创建流对象
    FileWriter fw = new FileWriter("b.txt");
}
```


#### 写出数据

1. 写出字符：`write()`方法，每次可以写出一个字符数据。

```java
public static void main(String[] args) throws IOException {
    // 使用文件名称创建流对象
    FileWriter fw = new FileWriter("fw.txt");     
    // 写出数据
    fw.write(97); // 写出第1个字符
    fw.write('b'); // 写出第2个字符
    fw.write('C'); // 写出第3个字符
    fw.write(30000); // 写出第4个字符，中文编码表中30000对应一个汉字。
    /*
    【注意】关闭资源时,与FileOutputStream不同。
    如果不关闭,数据只是保存到缓冲区，并未保存到文件。
    */
    fw.close();
}
```

> 虽然参数为int类型四个字节，但是只会保留一个字符的信息写出。未调用close方法，数据只是保存到了缓冲区，并未写出到文件中。



2. 写出字符数组：`write(char[] cbuf)`和`write(char[] cbuf, int off, int len)`，每次可以写出字符数组中的数据，用法类似FileOutputStream。

```java
public static void main(String[] args) throws IOException {
    // 使用文件名称创建流对象
    FileWriter fw = new FileWriter("fw.txt");     
    // 字符串转换为字节数组
    char[] chars = "逆龙程序员".toCharArray();
    // 写出字符数组
    fw.write(chars); // 逆龙程序员    
    // 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
    fw.write(b,2,2); // 程序  
    // 关闭资源
    fos.close();
}
```


3. 写出字符串：`write(String str)`和`write(String str, int off, int len)`，每次可以写出字符串中的数据，更为方便。

```java
public static void main(String[] args) throws IOException {
    // 使用文件名称创建流对象
    FileWriter fw = new FileWriter("fw.txt");     
    // 字符串
    String msg = "逆龙程序员";
    // 写出字符数组
    fw.write(msg); //逆龙程序员
    // 写出从索引2开始，2个字节。索引2是'程'，两个字节，也就是'程序'。
    fw.write(msg, 2, 2); // 程序
    // 关闭资源
    fos.close();
}
```


#### 关闭和刷新

因为内置缓冲区的原因，如果不关闭输出流，无法写出字符到文件中。但是关闭的流对象，是无法继续写出数据的。如果既想写出数据，又想继续使用流，就需要`flush`方法。



* `flush`：刷新缓冲区，流对象可以继续使用。
* `close`：先刷新缓冲区，后系统释放资源，流对象不可以再被使用。

> 即便是flush方法写出了数据，操作的最后还是要调用close方法，释放系统资源。

```java
public static void main(String[] args) throws IOException {
    // 使用文件名称创建流对象
    FileWriter fw = new FileWriter("fw.txt");
    // 写出数据，通过flush
    fw.write('刷'); // 写出第1个字符
    fw.flush();
    fw.write('新'); // 继续写出第2个字符，写出成功
    fw.flush();
    // 写出数据，通过close
    fw.write('关'); // 写出第1个字符
    fw.close();
    fw.write('闭'); 
    // 继续写出第2个字符,【报错】java.io.IOException: Stream closed
    fw.close();
}
```