

## 变量

变量表示内存中的一个存储区域（不同的变量，类型不同，占用的空间大小不同）。

在Java中，每一个变量都有一个类型（type），变量的类型位于变量名之前。**变量名必须是以一个字母开头并由字母或数字构成的序列，包括“\_”、“\$”。变量名中所有的字符都是有意义的，并且大小写敏感，长度基本上没有限制。不能使用 Java 保留字作为变量名。**变量在同一个作用域内不能重名。

变量必须先声明，后使用，即有顺序。声明一个变量后，必须用赋值语句对变量进行显示初始化。变量的声明尽可能的靠近变量第一次使用的地方。

该区域的数据/值可以在同一类型范围内不断变化。

```java
public class Var01{
    public static void main(String[] args) {
        // 声明变量
        int a;
        a = 100;
        int b = 800;
        System.out.println(a);  // 输出100
        System.out.println(b);  // 输出800
        // 该区域的数据/值可以在同一类型范围内不断变化
        a = 88;
        System.out.println(a);  // 输出88
        // 变量在同一个作用域内不能重名
        // int a = 60; // 报错
    }
}
class Dog {
    public static void main(String[] args) {
        // 不在同一个域中，不会报错
        int a = 77;
        System.out.println(a);
    }
}
```


## 常量

在Java 中，利用关键字 `final` 指示常量，关键字 **final** 表示这个变量只能被赋值一次，赋值后不能再更改，常量名使用全大写。



## 标识符

Java对各种变量、方法和类等命名时使用的字符序列成为标识符(凡是可以自己起名字的地方都叫标识符)。

**标识符的命名规范：**

1. 由26个英文字母大小写，0-9，\_或\$组成
2. 不可以以数字开头
3. 不可以使用关键字和保留字，但可以包含关键字和保留字
4. 严格区分大小写，长度无限制
5. 不能包含空格



## 关键字

关键字所有字母都为小写

| **用于定义数据类型的关键字**                       |            |           |              |          |
| -------------------------------------------------- | ---------- | --------- | ------------ | -------- |
| class                                              | interface  | enum      | byte         | short    |
| int                                                | long       | float     | double       | char     |
| boolean                                            | void       |           |              |          |
| **用于定义数据类型值的关键字**                     |            |           |              |          |
| true                                               | false      | null      |              |          |
| **用于定义流程的关键字**                           |            |           |              |          |
| if                                                 | else       | switch    | case         | default  |
| while                                              | do         | for       | break        | continue |
| return                                             |            |           |              |          |
| **用于定义访问权限修饰符的关键字**                 |            |           |              |          |
| private                                            | protected  | public    |              |          |
| **用于定义类、函数、变量修饰符的关键字**           |            |           |              |          |
| abstract                                           | final      | static    | synchronized |          |
| **用于定义类与类之间关系的关键字**                 |            |           |              |          |
| extends                                            | implements |           |              |          |
| **用于定义建立实例、引用实例以及判断实例的关键字** |            |           |              |          |
| new                                                | this       | super     | instanceof   |          |
| **用于处理异常的关键字**                           |            |           |              |          |
| tyr                                                | catch      | finally   | throw        | throws   |
| **用于包的关键字**                                 |            |           |              |          |
| package                                            | import     |           |              |          |
| **其他修饰符关键字**                               |            |           |              |          |
| native                                             | strictfp   | transient | volatile     | assert   |



## 保留字

现Java版本尚未使用，但以后版本可能会作为关键字使用。

| byValue  | cast  | future | generic | inner |
| -------- | ----- | ------ | ------- | ----- |
| operator | outer | rest   | var     | goto  |
| const    |       |        |         |       |



## Java基本命名规范

### 包名

多单词组成时所有字母都小写，例如：`aaa.bbb.ccc`



### 类名、接口名

多单词组成时，所有单词首字母大写，例如：`XxxYyyZzz`（大驼峰）



### 变量名、方法名

多单词组成时，第一个单词首字母小写，第二个单词开始每个单词首字母大写，例如：`xxxYyyZzz`（小驼峰）



### 常量名

所有字母大写，多单词组成时，每个单词用下划线连接，例如：`XXX_YYY_ZZZ`

