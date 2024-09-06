
XML的全称是EXtensible Markup Language，可扩展标记语言编写，XML就是编写标签，与HTML非常类似，扩展名.xml。拥有良好的人机可读性。

**用途：**
- Java程序的配置描述文件
- 用于保存程序产生的数据
- 网络间的数据传输


### XML和HTML的区别

- XML与HTML非常相似，都是编写标签，XML的语法严格，HTML语法松散
- XML没有预定义标签，HTML存在大量预定义标签
- XML重在保存与传输数据，HTML用于显示信息


### XML文档结构

- 第一行必须是XML声明
- 有且只有一个根节点
- XML标签的书写规则与HTML相同


### XML声明

XML声明说明XML文档的基本信息，包括版本号与字符集，写在XML第一行。
```xml
<?xml version="1.0" encoding="UTF-8"?> 
```
- `version`：代表版本号1.0/1.1 
- `encoding`：UTF-8设置字符集，用于支持中文
- `standalone`：是否独立
	- yes：不依赖其他文件
	- no：依赖其他文件


### XML标签书写规则

**合法的标签名**
- 标签名要有意义
- 建议使用英文小写字母，单词之间使用 `-` 分割
- 建议多级标签之间不要存在重名的情况

**适当的注释与缩进**
- 适当的注释与缩进可以让XML文档更容易阅读

**合理使用属性**
- 标签属性用于描述标签不可或缺的信息
- 对标签分组或为标签设置ID时常用属性表示

**有序的子元素**
- 在XML多层嵌套的子元素中，标签前后顺序应保持一致

**特殊字符与CDATA标签**
- 标签体中，出现 `<` 、`>` 特殊字符，会破坏文档结构
- 使用实体引用或者CDATA标签

| 实体引用 | 对应符号 | 说明 |
|----|----|----|
| `&lt;` | `<` | 小于 |
| `&gt;` | `>` | 大于 |
| `&amp;` | `&` | 和号 |
| `&apos;` | `'` | 单引号 |
| `&quot;` | `"` | 双引号 |


CDATA 指的是不应由 XML 解析器进行解析的文本数据，从 `<![CDATA[` 开始，到`]]>`结束

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--人力资源管理系统-->
<hr>
    <employee no="3301">
        <name>张三</name>
        <age>30</age>
        <salary>4000</salary>
        <department>
            <dname>市场部</dname>
            <address>xx大厦-B1003</address>
        </department>
    </employee>

    <employee no="3303">
        <name>李四</name>
        <age>28</age>
        <salary>5000</salary>
        <department>
            <dname>财务部</dname>
            <address>xx大厦-B1004</address>
        </department>
    </employee>
</hr>
```

## XML语义约束

XML文档结构正确，但可能不是有效的。XML语义约束有两种定义方式：DTD与XML Schema。

### DTD约束

DTD(Document Type Definition，文档类型定义)是一种简单易用的语义约束方式，DTD文件的扩展名为`.dtd`。

#### DTD定义节点

利用DTD中的`<!ELEMENT>`标签，可以定义XML文档中允许出现的节点及数量：
- `<!ELEMENT hr (employee)> employee>`：定义hr节点下只允许出现1个employee子节点
- `<!ELEMENT employee (name,age,salary,department)>`：节点下必须包含以下四个节点，且按顺序出现
- `<!ELEMENT name (#PCDATA)>`：定义name标签体只能是文本，`#PCDATA`代表文本元素


如某个子节点需要多次重复出现，则需要在子节点后增加相应的描述符：
- `<!ELEMENT hr (employee+)>`：hr节点下最少出现1个employee子节点
- `<!ELEMENT hr (employee*)>`：hr节点下可出现0..n个employee子节点
- `<!ELEMENT hr (employee?)>`：hr节点下最多出现1个employee子节点

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!ELEMENT hr (employee+)>
<!ELEMENT employee (name,age,salary,department)>
<!ATTLIST employee no CDATA "">
<!ELEMENT name (#PCDATA)>
<!ELEMENT age (#PCDATA)>
<!ELEMENT salary (#PCDATA)>
<!ELEMENT department (dname,address)>
<!ELEMENT dname (#PCDATA)>
<!ELEMENT address (#PCDATA)>
```


#### XML引用DTD

在XML中使用`<!DOCTYPE>`标签来引用DTD文件：
- `<!DOCTYPE 根节点 SYSTEM "dtd文件路径">`：本地引用
- `<!DOCTYPE 根节点 PUBLIC "dtd文件名字" "dtd文件的位置URL">`：网络引用

```xml
<!DOCTYPE hr SYSTEM "hr.dtd">
```


### Schema约束
XML Schema比DTD更为复杂，提供了数据类型、格式限定、数据范围等特性，是 W3C 标准。

- `<根节点 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="XML地址" xsi:schemaLocation="XSD地址">`：引入

```xml
<hr xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemalocation="hr.xsd">
</hr>
```

**定义约束：**
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<schema xmlns="http://www.w3.org/2001/XMLSchema">
    <element name="hr">
        <!--complexType标签代表是复杂节点，包含子节点时必须使用此标签-->
        <complexType>
            <!--sequence标签代表必须按顺序进行严格书写-->
            <sequence>
                <!--minOccurs="1" maxOccurs="10"  这个标签最少出现一次，最多出现10次-->
                <element name="employee" minOccurs="1" maxOccurs="10">
                    <complexType>
                        <sequence>
                            <!--type="string" 指定类型-->
                            <element name="name" type="string"/>
                            <!---->
                            <element name="age">
                                <!--简单类型约束-->
                                <simpleType>
                                    <!--限定-->
                                    <restriction base="integer">
                                        <!--最小限定-->
                                        <minInclusive value="18"/>
                                        <!--最大限定-->
                                        <maxInclusive value="60"/>
                                    </restriction>
                                </simpleType>
                            </element>
                            <element name="salary" type="integer"/>
                            <element name="department">
                                <complexType>
                                    <sequence>
                                        <element name="dname" type="string"/>
                                        <element name="address" type="string"/>
                                    </sequence>
                                </complexType>
                            </element>
                        </sequence>
                        <!--use="required"表示必须存在-->
                        <attribute name="no" type="string" use="required"/>
                    </complexType>
                </element>
            </sequence>
        </complexType>
    </element>
</schema>
```

