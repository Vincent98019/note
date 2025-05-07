
>`Properties`类继承自`HashTable`类并且实现了Map接口，也是使用键值对的形式来存储数据，来表示一个持久的属性集。
`Properties`的特点和`HashTable`类似。
`Properties`还可以用于从`xxx.properties`文件，加载数据到`Properties`类对象，并进行读取和修改。该类也被许多Java类使用，比如获取系统属性时，`System.getProperties`方法就是返回一个`Properties`对象。


**构造方法**

* `public Properties()`：创建一个空的属性列表。



**常用方法**

基本的存储方法

* `public Object setProperty(String key, String value)`：保存一对属性。
* `public String getProperty(String key)`：使用此属性列表中指定的键搜索属性值。
* `public Set<String> stringPropertyNames()`：所有键的名称的集合。
* `public void list(PrintStream out)`：将数据显示到指定设备

```java
public static void main(String[] args) throws FileNotFoundException {
    // 创建属性集对象
    Properties properties = new Properties();
    // 添加键值对元素
    properties.setProperty("filename", "a.txt");
    properties.setProperty("length", "209385038");
    properties.setProperty("location", "D:\\a.txt");
    // 打印属性集对象
    System.out.println(properties);
    // 通过键，获取属性值
    System.out.println(properties.getProperty("filename"));
    System.out.println(properties.getProperty("length"));
    System.out.println(properties.getProperty("location"));
    // 遍历属性集，获取所有键的集合
    Set<String> strings = properties.stringPropertyNames();
    // 打印键值对
    for (String key : strings ) {
        System.out.println(key+" -- "+properties.getProperty(key));
    }
    // list方法，把属性全部输出到控制台
    properties.list(System.out);
}
```


与流相关的方法

* `public void load(InputStream inStream)`：从字节输入流中读取键值对。
* `public void store(OutputStream out, String comments)`：将Properties中的键值对存储到配置文件，在idea中，如果有中文，会存储为Unicode码。

> 参数中使用了字节输入流，通过流对象，可以关联到某文件上，这样就能够加载文本中的数据了。

> **文本中的数据，必须是键值对形式，可以使用空格、等号、冒号等符号分隔。**

```java
public static void main(String[] args) throws FileNotFoundException {
    // 创建属性集对象
    Properties pro = new Properties();
    // 加载文本中信息到属性集
    pro.load(new FileInputStream("read.txt"));
    // 遍历集合并打印
    Set<String> strings = pro.stringPropertyNames();
    for (String key : strings ) {
        System.out.println(key+" -- "+pro.getProperty(key));
    }

    // 保存到新的文件， null表示的是注释
    pro.store(new FileOutputStream("src\\read2.txt"), null);
}
```


> ① Properties的Key和Value都不能为`null`，因为Properties继承了HashTable。
② 如果有相同的Key，那么Value会被替换。
