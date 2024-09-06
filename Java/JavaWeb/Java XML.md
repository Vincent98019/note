

## 解析xml的方式

### DOM

将标记语言文档一次性加载进内存，在内存中形成一颗dom树
- **优点：** 操作方便，可以对文档进行CRUD的所有操作
- **缺点：** 占内存


![](assets/Java%20XML/c43841a0ab261c9a6b524a9dc25c4a5b_MD5.jpg)


### SAX

逐行读取，基于事件驱动的

- **优点：** 不占内存
- **缺点：** 只能读取，不能增删改


### xml常见的解析器

**JAXP：**
sun公司提供的解析器，支持dom和sax两种思想。

**DOM4J：**
是一个易用的、开源的库，用于解析XML。它应用于Java平台，具有性能优异、功能强大和极其易使用的特点。Dom4j将XML视为Document对象，XML标签被Dom4j定义为Element对象。

**Jsoup：**
是一款Java 的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。

**PULL：**
Android操作系统内置的解析器，sax方式的。


## DOM4J

是一个易用的、开源的库，用于解析XML。它应用于Java平台，具有性能优异、功能强大和极其易使用的特点。Dom4j将XML视为Document对象，XML标签被Dom4j定义为Element对象。

> 需要JDK1.8以上

下载地址：[https://dom4j.github.io/](https://dom4j.github.io/)

### 读取XML文档

```java
/**
 * 描述：通过DOM4J读取xml文件
 */
public class HrReader {
    public void readXML() {
        String file = "E:\\IdeaProjects\\study\\xml-study\\src\\hr.xml";
        // SAXReader类是读取XML文件的核心类，用于将XML解析后以树的形式保存在内存中。
        SAXReader reader = new SAXReader();
        try {
            Document document = reader.read(file);
            // 获取XML文档的根节点，就是<hr>标签
            Element rootElement = document.getRootElement();
            // 用于获取指定标签的集合
            List<Element> elementList = rootElement.elements("employee");

            for (Element employee : elementList) {
                // 用于获取element唯一子节点
                Element name = employee.element("name");
                // 用于获取标签中的文本
                String nameText = name.getText();
                System.out.print("姓名：" + nameText);
                System.out.print("，年龄：" + employee.elementText("age"));
                System.out.print("，工资：" + employee.elementText("salary"));

                // 获取节点
                Element department = employee.element("department");
                String dname = department.element("dname").getText();
                System.out.print("，部门：" + dname);
                System.out.print("，工作地点：" + department.elementText("address"));

                // 获取属性
                Attribute att = employee.attribute("no");
                String no = att.getText();
                System.out.println("，工号：" + no);
            }
        } catch (DocumentException e) {
            e.printStackTrace();
        }
    }
    
    public static void main(String[] args) {
        HrReader hrReader = new HrReader();
        hrReader.readXML();
    }
}
```

### 更新XML文档

```java
/**
 * 描述：通过DOM4J更新xml文件
 */
public class HrWriter {
    public void writeXML(){
        String file = "E:\\IdeaProjects\\study\\xml-study\\src\\hr.xml";
        SAXReader reader = new SAXReader();
        try {
            Document document = reader.read(file);
            Element rootElement = document.getRootElement();
            // 创建一个属于rootElement对象的employee节点
            Element employee = rootElement.addElement("employee");
            // 创建no属性
            employee.addAttribute("no", "3311");
            // 创建一个属于employee对象的name节点
            Element name = employee.addElement("name");
            // 设置name节点的值
            name.setText("王五");

            employee.addElement("age").setText("22");
            employee.addElement("salary").setText("3500");

            Element department = employee.addElement("department");
            department.addElement("dname").setText("市场部");
            department.addElement("address").setText("xx大厦-B1003");

            // 创建字符输出流
            Writer writer = new OutputStreamWriter(new FileOutputStream(file), "UTF-8");
            // 将创建的节点写入文件中
            document.write(writer);
            writer.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        HrWriter hrWriter = new HrWriter();
        hrWriter.writeXML();
    }
}
```



## Jsoup

Jsoup是一款Java 的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。

1. 导入jar包
2. 获取Document对象
3. 获取对应的标签Element对象
4. 获取数据

```java
//2.1获取student.xml的path
String path = JsoupDemo1.class.getClassLoader().getResource("student.xml").getPath();
//2.2解析xml文档，加载文档进内存，获取dom树--->Document
Document document = Jsoup.parse(new File(path), "utf-8");
//3.获取元素对象 Element
Elements elements = document.getElementsByTag("name");
System.out.println(elements.size());
//3.1获取第一个name的Element对象
Element element = elements.get(0);
//3.2获取数据
String name = element.text();
System.out.println(name);
```


### Jsoup类

工具类，可以解析html或xml文档，返回Document

- parse：解析html或xml文档，返回Document
- parse(File in, String charsetName)：解析xml或html文件的。
- parse(String html)：解析xml或html字符串
- parse(URL url, int timeoutMillis)：通过网络路径获取指定的html或xml的文档对象

### Document类

文档对象。代表内存中的dom树。

**获取Element对象：**
- getElementById(String id)：根据id属性值获取唯一的element对象
- getElementsByTag(String tagName)：根据标签名称获取元素对象集合
- getElementsByAttribute(String key)：根据属性名称获取元素对象集合
- getElementsByAttributeValue(String key, String value)：根据对应的属性名和属性值获取元素对象集合

### Elements类

元素Element对象的集合。可以当做 `ArrayList<Element>` 来使用

### Element类

元素对象

**获取子元素对象：**
- getElementById(String id)：根据id属性值获取唯一的element对象
- getElementsByTag(String tagName)：根据标签名称获取元素对象集合
- getElementsByAttribute(String key)：根据属性名称获取元素对象集合
- getElementsByAttributeValue(String key, String value)：根据对应的属性名和属性值获取元素对象集合


**获取属性值：**
- String attr(String key)：根据属性名称获取属性值

**获取文本内容：**
- String text():获取文本内容
- String html():获取标签体的所有内容(包括字标签的字符串内容)

### Node类

节点对象，是Document和Element的父类


### 快捷查询方式

**selector：** 选择器
- 使用的方法：Elements select(String cssQuery)
- 语法：参考Selector类中定义的语法

**XPath：** XPath即为XML路径语言，它是一种用来确定XML（标准通用标记语言的子集）文档中某部分位置的语言

- 使用Jsoup的Xpath需要额外导入jar包。
- 查询w3cshool参考手册，使用xpath的语法完成查询

```java
    	  //1.获取student.xml的path
          String path = JsoupDemo6.class.getClassLoader().getResource("student.xml").getPath();
          //2.获取Document对象
          Document document = Jsoup.parse(new File(path), "utf-8");
  
          //3.根据document对象，创建JXDocument对象
          JXDocument jxDocument = new JXDocument(document);
  
          //4.结合xpath语法查询
          //4.1查询所有student标签
          List<JXNode> jxNodes = jxDocument.selN("//student");
          for (JXNode jxNode : jxNodes) {
              System.out.println(jxNode);
          }
  
          System.out.println("--------------------");
  
          //4.2查询所有student标签下的name标签
          List<JXNode> jxNodes2 = jxDocument.selN("//student/name");
          for (JXNode jxNode : jxNodes2) {
              System.out.println(jxNode);
          }
  
          System.out.println("--------------------");
  
          //4.3查询student标签下带有id属性的name标签
          List<JXNode> jxNodes3 = jxDocument.selN("//student/name[@id]");
          for (JXNode jxNode : jxNodes3) {
              System.out.println(jxNode);
          }
          System.out.println("--------------------");
          //4.4查询student标签下带有id属性的name标签 并且id属性值为itcast
  
          List<JXNode> jxNodes4 = jxDocument.selN("//student/name[@id='itcast']");
          for (JXNode jxNode : jxNodes4) {
              System.out.println(jxNode);
          }
```

## XPath 查找XML数据

XPath路径表达式是XML文档中查找数据的语言，可以极大的提高在提取数据时的开发效率。

参考：[https://www.w3school.com.cn/xpath/index.asp](https://www.w3school.com.cn/xpath/index.asp)