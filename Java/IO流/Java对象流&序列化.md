## 序列化

Java 提供了一种**对象序列化**的机制。用一个字节序列可以表示一个对象，该字节序列包含该**对象的数据、对象的类型和对象中存储的属性**等信息。字节序列写出到文件之后，相当于文件中持久保存了一个对象的信息。



反之，该字节序列还可以从文件中读取回来，重构对象，对它进行反序列化。对象的数据、对象的类型和对象中存储的数据信息，都可以用来在内存中创建对象。

* **序列化：**保存数据时，同时保存**数据的值**和**数据类型**
* **反序列化：**恢复数据时，恢复**数据的值**和**数据类型**

所以，为了使某个类是可序列化的，该类必须实现`Serializable` 或`Externalizable` 接口。

* `Serializable`：是一个标记接口，里面没有任何方法。不实现此接口的类将不会使任何状态序列化或反序列化，会抛出`NotSerializableException`。
* `Externalizable`：该接口继承了`Serializable`，需要实现方法，一般使用Serializable。



序列化具备可继承性，如果某类实现了序列化，它的所有子类也默认实现了序列化。

序列化对象中的属性的类型也需要实现序列化接口。



## transient关键字

实现上面两个接口后的所有属性**必须**是可序列化的。如果有一个属性不需要可序列化，则该属性必须注明是瞬态的，使用`transient`关键字修饰。

> 被static修饰的成员变量是不能被序列化的，序列化的都是对象。

```java
public class Employee implements java.io.Serializable {
    public String name;
    public String address;
    public transient int age; // transient瞬态修饰成员，不会被序列化

    public void addressCheck() {
        System.out.println("Address check : " + name + " -- " + address);
    }
}
```


## 版本号

`Serializable`接口给需要序列化的类，提供了一个序列版本号。

`serialVersionUID`该版本号的目的在于验证序列化的对象和对应类是否版本匹配。



```java
public class Employee implements java.io.Serializable {
    // 加入序列版本号
    private static final long serialVersionUID = 1L;
    public String name;
    public String address;
    // 添加新的属性，重新编译，可以反序列化，该属性赋为默认值
    public int eid; 
    public void addressCheck() {
        System.out.println(name + " -- " + address);
    }
}
```


## ObjectOutputStream类

`java.io.ObjectOutputStream`类，将Java对象的原始数据类型写出到文件，实现对象的持久存储。

**构造方法**

* `public ObjectOutputStream(OutputStream out)`：创建一个指定OutputStream的ObjectOutputStream。

```java
FileOutputStream fileOut = new FileOutputStream("employee.txt");
ObjectOutputStream out = new ObjectOutputStream(fileOut);
```


**常用方法**

* `public final void writeObject (Object obj)`：将指定的对象写出。



```java
public static void main(String [] args)   {
    Employee e = new Employee();
    e.name = "zhangsan";
    e.address = "beiqinglu";
    e.age = 20; 
    try {
        // 创建序列化流对象
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.txt"));
        // 写出对象
        out.writeObject(e);
        // 释放资源
        out.close();
        // 姓名，地址被序列化，年龄没有被序列化。
    } catch(IOException i)   {
        i.printStackTrace();
    }
}
```


## ObjectInputStream类

`ObjectInputStream`反序列化流，将之前使用`ObjectOutputStream`序列化的原始数据恢复为对象。

> 反序列化时，读的顺序要和写的顺序一致。

* 可以反序列化的对象，必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个`ClassNotFoundException`异常。
* 反序列化对象时，能找到class文件，但class文件在序列化对象之后发生了修改，那么反序列化操作也会失败，抛出一个`InvalidClassException`异常。



**反序列化出现异常的情况：**

* 该类的序列版本号与从流中读取的类描述符的版本号不匹配
* 该类包含未知数据类型
* 该类没有可访问的无参数构造方法






**构造方法**

* `public ObjectInputStream(InputStream in)`：创建一个指定InputStream的ObjectInputStream。



**常用方法**

* `public final Object readObject()`：读取一个对象。

```java
public static void main(String [] args)   {
    Employee e = null;
    // 创建反序列化流
    FileInputStream fileIn = new FileInputStream("employee.txt");
    ObjectInputStream in = new ObjectInputStream(fileIn);
    // 读取一个对象
    e = (Employee) in.readObject();
    // 释放资源
    fileIn.close();

    // 无异常,直接打印输出
    System.out.println("Name: " + e.name); // zhangsan
    System.out.println("Address: " + e.address); // beiqinglu
    System.out.println("age: " + e.age); // 0
}
```

> 对于JVM可以反序列化对象，它必须是能够找到class文件的类。如果找不到该类的class文件，则抛出一个 `ClassNotFoundException` 异常。
> 当JVM反序列化对象时，能找到class文件，但是class文件在序列化对象之后发生了修改，那么反序列化操作也会失败，抛出一个 `InvalidClassException` 异常。
> .
> 发生这个异常的原因如下：
• 该类的序列版本号与从流中读取的类描述符的版本号不匹配
• 该类包含未知数据类型
• 该类没有可访问的无参数构造方法