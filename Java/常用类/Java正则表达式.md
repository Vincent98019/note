
正则表达式的元字符可以大致分为六类：

1. 限定符
2. 选择匹配符
3. 分组组合和反向引用符
4. 特殊字符
5. 字符匹配符
6. 定位符


## 转义符

`\\` 符号在检索某些特殊字符的时候可以使用，否则检索不到结果。

在Java中是`\\` ，而其他语言则是`\` 。

一般 ` . * + ( ) $ / \ ? [ ] { } ` 都可能需要转义符

```java
public static void main(String[] args) {
    String content = "abc$((abfs)($(123";
    // 正则表达式：匹配左括号，需要用到转义符
    String regex = "\\(";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


## 字符匹配符

| **符号** | **描述**                                                     | **示例**       | **解释**                                                     |
| -------- | ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| `[]`     | 可接收的字符列表                                             | `[efgh]`       | 匹配e、f、g、h中的任意1个字符                                |
| `[^]`    | 不接收的字符列表                                             | `[^abc]`       | 除了a、b、c以外的任意1个字符，包括数字和特殊符号             |
| `-`      | 连续字符                                                     | `A-Z`          | 任意一个大写字母                                             |
| `.`      | 匹配除`\n` 以外的任何字符                                    | `a..b`         | 以a开头，b结尾，中间有两个任意字符，长度为4的字符串(aaab，asdb，a*+b) |
| `\\d`    | 匹配单个数字字符，相当于`[0-9]`                              | `\\d{3(\\d)?`  | 包含3个或4个数字的字符串(123，4528)                          |
| `\\D`    | 匹配单个非数字字符，相当于`[^0-9]`                           | `\\D(\\d*)`    | 非数字开头，之后任意个数字字符串(a，A528)                    |
| `\\w`    | 匹配单个数字、大小写字母、下划线的字符，相当于`[0-9a-zA-Z]`  | `\\d{3}\\w{4}` | 3个数字开头长度为7的数字字母字符串(235ads5，123412a)         |
| `\\W`    | 匹配单个非数字、大小写字母、下划线的字符，相当于`[^0-9a-zA-Z]` | `\\W+\\d{2}`   | 一个非数字字母开头，2个数字结尾的字符串(#35，\$%dassd25)     |
| `\\s`    | 匹配任何空白字符(空格、制表符等)                             |                |                                                              |
| `\\S`    | 匹配任何非空白字符                                           |                |                                                              |



### 大小写

Java正则表达式默认是区分字母大小写的，`(?i)` 表示编译标记。

下面几个示例实现不区分大小写：

* `(?i)abc`：表示abc都不区分大小写
* `a(?i)bc`：表示bc不区分大小写
* `a((?i)b)c`：表示b不区分大小写
* `Pattern.CASE_INSENSITIVE` ：不区分大小写



```java
public static void main(String[] args) {
    String content = "a11c8abcABC _";
    // 匹配任意一个小写字母
    String regex1 = "[a-z]";
    // 匹配任意一个大写字母
    String regex2 = "[A-Z]";
    // 匹配小写abc，必须是abc连起来才能匹配上
    String regex3 = "abc";
    // 匹配不区分大小写abc
    Pattern pattern1 = Pattern.compile(regex3, Pattern.CASE_INSENSITIVE);
    // 匹配不区分大小写abc
    String regex4 = "(?i)abc";
    // 匹配数字
    String regex5 = "[0-9]";
    String regex6 = "\\d";
    // 匹配除小写字母其他所有字符
    String regex7 = "[^a-z]";
    // 匹配abcd中任意一个字符
    String regex8 = "[abcd]";
    // 匹配数字、字母、下划线中任意一个字符
    String regex9 = "\\w";
    // 匹配空格、制表符
    String regex10 = "\\s";
    // 匹配除\n以外的所有字符，匹配.本身的话使用\\.
    String regex = ".";

    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


## 选择匹配符

| **符号** | **描述**                |
| -------- | ----------------------- |
| `\|`      | 匹配 `\|` 前或后的表达式 |



```java
public static void main(String[] args) {
    String content = "a11c8abcABC _";
    // 匹配abc或者ABC再或者aaa
    String regex = "abc|ABC|aaa";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


## 限定符

用于指定其前面的字符和组合项连续出现多少次。

| **符号** | **描述**                       | **示例**      | **解释**                                                     |
| -------- | ------------------------------ | ------------- | ------------------------------------------------------------ |
| `*`      | 指定字符重复0次或n次           | `(abc)*`      | 仅包含任意个abc的字符串，等效于`\w*` (abc，abcabcabc)        |
| `+`      | 指定字符重复1次或n次(最少一次) | `m+(abc)*`    | m开头，后面接任意个abc的字符串(m，mabc，mabcabcabc)          |
| `?`      | 指定字符重复0次或1次(最多一次) | `m+abc?`      | m开头，后面接ab或abc的字符串(mab，mabc，mmmabc)              |
| `{n}`    | 只能输入n个字符                | `[abcd]{3}`   | 由abcd中字母组成长度为3的字符串(abc，bcd，adc)               |
| `{n,}`   | 指定至少n个匹配                | `[abcd]{3,}`  | 由abcd中字母组成的任意长度不小于3的字符串(aab，bdc，aaadbc)  |
| `{n,m}`  | 指定至少n个但不多余m个匹配     | `[abcd]{3,5}` | 由abcd中字母组成的任意长度不小于3的字符串，并且长度不大于5的字符串(abc，abcd，aaaaa，bcdab) |



```java
public static void main(String[] args) {
    String content = "1111111aaaaaahello";
    // 匹配aaa
    String regex1 = "a{3}";
    // 匹配1111，只会匹配(0,3)，后面3个1不会被匹配到
    String regex2 = "1{4}";
    // 匹配两位连续的数字，这里会匹配到3个11
    String regex3 = "\\d{2}";
    // 匹配aaa或者aaaa，Java中默认会先匹配多的，因为只有6个a，所以是aaaa
    String regex4 = "a{3,4}";
    // 匹配连续2-5个数字的组合
    String regex5 = "\\d{2,5}";
    // 匹配一个1或者多个1，贪婪匹配，所以直接7个1
    String regex6 = "1+";
    // 匹配0个1或者多个1，贪婪匹配，所以直接7个1和一些没有1
    String regex7 = "1*";
    // 匹配1后面有个a，也可以没有
    String regex = "1a?";
    
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


## 定位符

定位符规定要匹配的字符串出现的位置，比如在字符串开始的位置或者结束的位置。

| **符号** | **描述**               | **示例**           | **解释**                                                 |
| -------- | ---------------------- | ------------------ | -------------------------------------------------------- |
| `^`      | 指定起始字符           | `^[0-9]+[a-z]\*`   | 至少一个数字开头，后面接任意个小写字母(123，6aa，555edf) |
| `$`      | 指定结束字符           | `^[0-9]\\-[a-z]+$` | 1个数字开头，接"-"，接至少一个小写字母(1-a)              |
| `\\b`    | 匹配目标字符串的边界   | `hu\\b`            | 边界指空格或者结束位置(huwenlong wl**hu** haha**hu**)    |
| `\\B`    | 匹配目标字符串的非边界 | `hu\\B`            | 和\\\\b相反(**hu**wenlong wlhu hahahu)                   |

```java
public static void main(String[] args) {
    String content = "123abc";
    // 匹配至少一个数字开头，后面接任意小写字母，匹配的是整串
    String regex1 = "^[0-9]+[a-z]*";
    // 匹配至少一个数字开头，必须一个小写字母结尾，匹配的是整串
    String regex = "^[0-9]+[a-z]+$";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


```java
public static void main(String[] args) {
    String content = "huwenlong wlhu huhahahu";
    // 匹配在边界的hu，空格前的+字符串末尾的
    String regex1 = "hu\\b";
    // 匹配不在边界的hu，空格后的+字符串开头的
    String regex = "hu\\B";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


## 分组

### 捕获分组

| **常用分组构造形式** | **描述**                                                     |
| -------------------- | ------------------------------------------------------------ |
| `(pattern)`          | 非命名捕获，捕获匹配的子字符串，编号为0的第一个捕获是由整个正则表达式模式匹配的文本，其它捕获的结果则根据左括号的顺序从1开始自动编号。 |
| `(?<name>pattern)`   | 命名捕获，将匹配的子字符串捕获到一个组名称或2编号名称中，用于name的字符串不能包含任何符号，并且不能以数字开头。可以使用`''`代替`<>`，例如`(?'name')` |



```java
public static void main(String[] args) {
    String content = "huwenlong wl7787 huhaha1541hu";
    // 匹配四个数字
    String regex1 = "\\d\\d\\d\\d";
    // 非命名分组：
    // 匹配四个数字，其中每两个数字为一组
    // matcher.group(0)：匹配到的所有字符串
    // matcher.group(1)：匹配到的第一组的内容，也就是前两个数字
    // matcher.group(2)：匹配到的第二组的内容，也就是后两个数字
    String regex2 = "(\\d\\d)(\\d\\d)";
    String regex = "(?<g1>\\d\\d)(?<g2>\\d\\d)";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
        System.out.println("第一组：" + matcher.group(1));
        System.out.println("第二组：" + matcher.group(2));
        System.out.println("命名分组-第一组：" + matcher.group("g1"));
        System.out.println("命名分组-第二组：" + matcher.group("g2"));
        
    }
}
```


### 非捕获分组

| **常用分组构造形式** | **描述**                                                     |
| -------------------- | ------------------------------------------------------------ |
| `(?:pattern)`        | 非捕获匹配，匹配pattern但不捕获该匹配的子表达式，不存储供以后使用的匹配。这对于用"or"字符(`|`)组合模式部件的情况很有用。例如：`'industr(?:y|ies)'` 是比`'industry|industries'`更经济的表达式 |
| `(?=pattern)`        | 非捕获匹配，例如：`'Windows (?=95|98|NT|2000)'`匹配`"Windows 2000"`中的`"Windows"`，但不匹配`"Windows 3.1"`中的`"Windows"`。 |
| `(?!pattern)`        | 非捕获匹配，该表达式匹配不处于匹配pattern的字符串的起始点的搜索字符串。例如：`'Windows (?!95|98|NT|2000)'`匹配`"Windows 3.1"`中的`"Windows"`，但不匹配`"Windows 2000"`中的`"Windows"`。 |

```java
public static void main(String[] args) {
    String content = "hellohuwen arborhuwl huwonghello";
    // 正常匹配
    String regex1 = "huwen|huwl|huwong";
    // 非捕获分组，上面的等价于：
    String regex2 = "huw(?:en|l|ong)";
    // 非捕获分组，huw后面包含了以下两种情况才会被匹配，
    // 但是不包含下面两种字符，所以得到的结果是两个huw
    String regex3 = "huw(?=en|l)";
    // 非捕获分组，huw后面不包含以下两种情况才会被匹配，
    // 但是不包含下面两种字符，所以得到的结果是一个huw
    String regex = "huw(?!en|l)";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


## 非贪婪匹配

一般情况下，正则表达式会贪婪匹配，可以看限定符章节，而在后面加个`?` 就是非贪婪匹配。

```java
public static void main(String[] args) {
    String content = "1111111aaaaaahello";
    // 匹配数字，贪婪匹配会匹配到1111111
    String regex1 = "\\d+";
    // 匹配数字，非贪婪匹配会匹配到七个1
    String regex = "\\d+?";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    while (matcher.find()) {
        System.out.println("截取到的字符串：" + matcher.group(0));
    }
}
```


## 反向引用

* **分组：** 用圆括号`()`组成一个比较复杂的匹配模式，一个圆括号就是一个子表达式(一个分组)。
* **捕获：** 把正则表达式中子表达式(分组)匹配的内容，保存到内存中，以数字编号或显式命名的组里，方便后面引用。从左向右，以分组的左括号为标志，第一个出现的分组的组号为1，第二个为2，以此类推，组0表示整个正则式。
* **反向引用：** 小括号的内容被捕获后，可以在这个括号后被使用，从而写出一个比较实用的匹配模式。这种引用既可以在正则表达式的内部，也可以在外部，内部反向引用**\\\\分组号**，外部反向引用**\$分组号**。

```java
匹配两个连续相同的数字：(\\d)\\1    // 先匹配一个数字，再反向引用一次，1表示分组1号也就是第一个括号
匹配五个连续相同的数字：(\\d)\\1{4}    // 先匹配一个数字，再反向引用四次
匹配四个数字，中间两个数相同，两边两个数相同，例如：1221,3553,2332
(\\d)(\\d)\\2\\1    // \\2表示第二组反向引用一次，\\1表示第一组反向引用一次
```


## 示例

```java
public static void main(String[] args) {
    String content = "https://space.bilibili.com/124536558?spm_id_from=333.1007.0.0";
    // 匹配多个汉字，整体匹配
    String regex1 = "^[\u0391-\uffe5]+$";
    // 匹配邮政编码，1-9开头的六位数字
    String regex2 = "^[1-9]\\d{5}$";
    // 匹配QQ，1-9开头的5-10位数字
    String regex3 = "^[1-9]\\d{4,9}$";
    // 匹配手机号码，必须是13 14 15 18开头的11位数字
    String regex4 = "^1[3|4|5|8]\\d{9}$";
    // 匹配url []中的 ? * . 表示的是本身，不是转义符，所以不用使用\\转义
    String regex = "^((http|https)://)([\\w-]+\\.)+[\\w-]+(/[\\w-?=&/%.#]*)?$";
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(content);
    if (matcher.find()) {
        System.out.println("满足格式");
    } else {
        System.out.println("不满足格式");
    }
}
```

