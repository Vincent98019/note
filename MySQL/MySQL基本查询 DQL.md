
select语句屏蔽了物理层的操作，用户不必关心数据的真实存储，交由数据库高效的查找数据。

## 普通查询

> **执行顺序**：词法分析与优化，读取SQL语句 → FROM，选择数据来源 → SELECT，选择输出内容

* `select 列名 from 表名 [where 条件表达式]`：查询表中的数据
* `select * from 表名`：查询表所有行和列的数据
  * `*`代表所有字段，一般不建议使用
* `select 字段名 1, 字段名 2, 字段名 3, ... from 表名`：查询指定列

```sql
-- 查询所有的学生：
select * from student;

-- 查询 student 表中的 name 和 age 列
select name, age from student;
```


## 使用别名查询

* `select 字段名1 as 别名, 字段名2 as 别名... from 表名`：对列指定别名
* `select 表别名.字段名1 as 别名, 表别名.字段名2 as 别名... from 表名 as 表别名`：对列和表同时指定别名


**使用别名的好处：**

* 显示的时候使用新的名字，并不修改表的结构。**`as`可以省略。**
* 用于多表查询操作

```bash
-- 使用别名
select name as 姓名,age as 年龄 from student;

-- 表使用别名
select st.name as 姓名,age as 年龄 from student as st;
```


## 分页查询

**LIMIT**是限制的意思，所以`LIMIT`的作用就是限制查询记录的条数。

> **执行顺序**：FROM → SELECT → LIMIT
> 
* `select 字段名 from 表名 limit 起始位置, 偏移量`：分页查询
   * 起始位置：起始行数，从 0 开始计数，如果省略，默认就是0
   * 偏移量：返回的行数
   * 公式：**开始的索引 = （当前的页码 - 1） \* 每页显示的条数**

```sql
-- 查询学生表中数据，从第 3 条开始显示，显示 6 条。
select * from student3 limit 2, 6;
```


## 排序

通过`ORDER BY`子句，可以将查询出的结果进行排序（排序只是显示方式，不会影响数据库中数据的顺序）。

> **执行顺序**：FROM → SELECT → ORDER BY → LIMIT

* `select 字段名 from 表名 order by 字段名 [asc|desc]`：单列排序，只按某一个字段进行排序，单列排序。
* `select 字段名 from 表名 order by 字段名1 [asc|desc], 字段名2 [asc|desc]`：组合排序，同时对多个字段进行排序，如果第 1 个字段相等，则按第 2 个字段排序，依次类推。
  * **ASC**：升序，默认值
  * **DESC**：降序

> 如果排序列是数字类型，数据库就按照数字大小排序，如果是日期类型，就按照日期大小，如果是字符串就按照字符集序号排序。



```sql
-- 查询所有数据,使用年龄降序排序
select * from student order by age desc;

-- 查询所有数据,在年龄降序排序的基础上，如果年龄相同再以数学成绩升序排序
select * from student order by age desc, math asc;
```


## 清除重复值

* `select distinct 字段名 from 表名`：查询指定列并且结果不出现重复数据

> 使用 `DISTINCT` 的SELECT子句中只能查询一列数据，如果查询多列，去除重复记录就会失效，一条SELECT语句中，只能使用一次。

```sql
-- 查询学生来自于哪些地方
select address from student;
-- 去掉重复的记录
select distinct address from student;
```


## 查询结果参与运算

* `select 列名1+固定值 from 表名`：某列数据和固定值运算
* `select 列名1+列名2 from 表名`：某列数据和其他列数据参与运算

> **注意**：参与运算的必须是数值类型

```sql
-- 某列数据和固定值运算
select age + 1 from student;

-- 某列数据和其他列数据参与运算，英语和数学分数之和
select english + math from student;
```