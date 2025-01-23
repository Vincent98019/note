函数是指一段可以直接被另一段程序调用的程序或代码。MySQL已经提供了一些函数。

MySQL中的函数主要分为以下四类： 字符串函数、数值函数、日期函数、流程函数。



## 字符串函数

**常用函数：**
* `concat(S1,S2,...Sn)`：字符串拼接，将S1，S2，... Sn拼接成一个字符串。
* `lower(str)`：将字符串str全部转为小写。
* `upper(str)`：将字符串str全部转为大写。
* `lpad(str, n, pad)`：左填充，用字符串pad对str的左边进行填充，达到n个字符串长度。
* `rpad(str, n, pad)`：右填充，用字符串pad对str的右边进行填充，达到n个字符串长度。
* `trim(str)`：去掉字符串头部和尾部的空格。
* `substring(str, start, len)`：返回从字符串str从start位置起的len个长度的字符串，**索引值从1开始。**



```sql
-- concat(S1,S2,...Sn)：字符串拼接，将S1，S2，... Sn拼接成一个字符串
select concat('hello', 'mysql');

-- lower(str)：将字符串str全部转为小写
select lower('HELLO');

-- upper(str)：将字符串str全部转为大写
select upper('hello');

-- lpad(str, n, pad)：左填充，用字符串pad对str的左边进行填充，达到n个字符串长度
select lpad('hello', 20, ',');

-- rpad(str, n, pad)：右填充，用字符串pad对str的右边进行填充，达到n个字符串长度
select rpad('mysql', 10, '-');

-- trim(str)：去掉字符串头部和尾部的空格
select trim('  hello mysql  ');

-- substring(str, start, len)：返回从字符串str从start位置起的len个长度的字符串
select substring('hello mysql', 1, 5);
```


## 数值函数

**常用函数：**

* `ceil(x)`：向上取整。
* `floor(x)`：向下取整。
* `mod(x,y)`：返回x/y的模。
* `rand()`：返回0\~1内的随机数。
* `round(x,y)`：求参数x的四舍五入的值，保留y位小数。

```sql
-- ceil：向上取整
select ceil(1.1);

-- floor：向下取整
select floor(1.9);

-- mod：取模
select mod(7,4);

-- rand：获取随机数
select rand();

-- round：四舍五入
select round(2.344,2);
```


## 日期函数

**常用函数：**

* `curdate()`：返回当前日期。
* `curtime()`：返回当前时间。
* `now()`：返回当前日期和时间。
* `year(date)`：获取指定date的年份。
* `month(date)`：获取指定date的月份。
* `day(date)`：获取指定date的日期。
* `date_add(date, INTERVAL expr type)`：返回一个日期/时间值加上一个时间间隔expr后的时间值。type可选年月日。
* `datediff(date1,date2)`：返回起始时间date1和结束时间date2之间的天数。

```sql
-- curdate：当前日期
select curdate();

-- curtime：当前时间
select curtime();

-- now：当前日期和时间
select now();

-- YEAR , MONTH , DAY：当前年、月、日
select YEAR(now());
select MONTH(now());
select DAY(now());

-- date_add：增加指定的时间间隔
select date_add(now(), INTERVAL 70 YEAR );

--  datediff：获取两个日期相差的天数
select datediff('2021-10-01', '2021-12-01');
```


## 流程函数

流程函数也是很常用的一类函数，可以在SQL语句中实现条件筛选，从而提高语句的效率。

**常用函数：**

* `if(value , t , f)`：如果value为true，则返回t，否则返回f。
* `ifnull(value1 , value2)`：如果value1不为空，返回value1，否则返回value2。
* `case when [ val1 ] then [res1] ... else [ default ] end`：如果val1为true，返回res1，... 否则返回default默认值。
* `case [ expr ] when [ val1 ] then [res1] ... else [ default ] end`：如果expr的值等于val1，返回res1，... 否则返回default默认值。

```sql
-- if
select if(false, 'Ok', 'Error');

-- ifnull
select ifnull('Ok','Default');
select ifnull('','Default');
select ifnull(null,'Default');

--  case when then else end
-- 查询emp表的员工姓名和工作地址 (北京/上海 ----> 一线城市 , 其他 ----> 二线城市)
select 
    name, 
    (
        case workaddress 
        when '北京' then '一线城市' 
        when '上海' then '一线城市' 
        else '二线城市' end
    ) as '工作地址' 
from emp;
```