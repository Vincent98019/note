

**缓冲流，也叫高效流**，是对4个基本的FileXxx流的增强，所以也是4个流，按照数据类型分类：



* **字节缓冲流：** BufferedInputStream，BufferedOutputStream
* **字符缓冲流：** BufferedReader，BufferedWriter

缓冲流的基本原理，是在创建流对象时，会创建一个内置的默认大小的缓冲区数组，通过缓冲区读写，减少系统IO次数，从而提高读写的效率。

关闭时，只需要关闭最外层流即可。



## 字符缓冲流

**构造方法**

* `public BufferedReader(Reader in)`：创建一个新的缓冲输入流。
* `public BufferedWriter(Writer out)`： 创建一个新的缓冲输出流。

```java
// 创建字符缓冲输入流
BufferedReader br = new BufferedReader(new FileReader("br.txt"));
// 创建字符缓冲输出流
BufferedWriter bw = new BufferedWriter(new FileWriter("bw.txt"));
```


**常用方法**

**BufferedReader：**

* `public String readLine()`：读一行文字，如果达到末尾，则返回null。

**BufferedWriter：**

* `public void newLine()`：写一行行分隔符，由系统属性定义符号。



```java
public static void main(String[] args) throws IOException {
    // 创建流对象
    BufferedReader br = new BufferedReader(new FileReader("in.txt"));
    // 定义字符串,保存读取的一行文字
    String line  = null;
    // 循环读取, 读取到最后返回null
    while ((line = br.readLine()) != null) {
        System.out.print(line);
        System.out.println("------");
    }
    // 释放资源
    br.close();
}
```


```java
public static void main(String[] args) throws IOException {
    // 创建流对象
    BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"));
    // 写出数据
    bw.write("我是");
    // 写出换行
    bw.newLine();
    bw.write("程序");
    bw.newLine();
    bw.write("员");
    bw.newLine();
    // 释放资源
    bw.close();
}
```


## 字节缓冲流

**构造方法**

* `public BufferedInputStream(InputStream in)`：创建一个新的缓冲输入流。
* `public BufferedOutputStream(OutputStream out)`： 创建一个新的缓冲输出流。

```java
// 创建字节缓冲输入流
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("bis.txt"));
// 创建字节缓冲输出流
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("bos.txt"));
```
