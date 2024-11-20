## 数据类型

### 数值类型

| **类型**        | **大小** | **有符号(SIGNED)范围**                                | **无符号(UNSIGNED)范围**                                  | **描述**           |
| --------------- | -------- | ----------------------------------------------------- | --------------------------------------------------------- | ------------------ |
| tinyint         | 1 byte   | (-128，127)                                           | (0，255)                                                  | 小整数值           |
| smallint        | 2 bytes  | (-32768，32767)                                       | (0，65535)                                                | 大整数值           |
| mediumint       | 3 bytes  | (-8388608，8388607)                                   | (0，16777215)                                             | 大整数值           |
| int(integer)    | 4 bytes  | (-2147483648，2147483647)                             | (0，4294967295)                                           | 大整数值           |
| bigint          | 8 bytes  | (-2^63，2^63-1)                                       | (0，2^64-1)                                               | 极大整数值         |
| float\[(n,m)\]  | 4 bytes  | (-3.402823466 E+38，<br>3.402823466351 E+38)          | 0 和 (1.175494351 E-38，3.402823466 E+38)                 | 单精度浮点数值     |
| double\[(n,m)\] | 8 bytes  | (-1.7976931348623157 E+308，1.7976931348623157 E+308) | 0 和 (2.2250738585072014 E-308，1.7976931348623157 E+308) | 双精度浮点数值     |
| decimal(n,m)    |          | 依赖于M(精度)和D(标度)的值                            | 依赖于M(精度)和D(标度)的值                                | 小数值(精确定点数) |

* `decimal(10,2)`：10代表整数位加小数位一共10位，2代表精确到小数点后2位

### 字符串类型

| **类型**   | **大小**              | **描述**                               |
| ---------- | --------------------- | -------------------------------------- |
| char(m)    | 0-255 bytes           | 定长字符串，无论使用几个字符都占满全部 |
| varchar(m) | 0-65535 bytes         | 变长字符串，使用几个字符就占用几个     |
| tinyblob   | 0-255 bytes           | 不超过255个字符的二进制数据            |
| tinytext   | 0-255 bytes           | 短文本字符串                           |
| blob       | 0-65 535 bytes        | 二进制形式的长文本数据                 |
| text       | 0-65 535 bytes        | 长文本数据                             |
| mediumblob | 0-16 777 215 bytes    | 二进制形式的中等长度文本数据           |
| mediumtext | 0-16 777 215 bytes    | 中等长度文本数据                       |
| longblob   | 0-4 294 967 295 bytes | 二进制形式的极大文本数据               |
| longtext   | 0-4 294 967 295 bytes | 极大文本数据                           |



### 日期时间类型

| **类型**  | **大小** | **范围**                               | **格式**            | **描述**                 |
| --------- | -------- | ------------------------------------------ | ------------------- | ------------------------ |
| date      | 3        | 1000-01-01 至 9999-12-31                   | YYYY-MM-DD          | 日期值                   |
| time      | 3        | \-838:59:59 至 838:59:59                   | HH:MM:SS            | 时间值或持续时间         |
| year      | 1        | 1901 至 2155                               | YYYY                | 年份值                   |
| datetime  | 8        | 1000-01-01 00:00:00至 9999-12-31 23:59:59  | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| timestamp | 4        | 1970-01-01 00:00:01 至 2038-01-19 03:14:07 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值，时间戳 |


## 创建表

* 创建一个新的表：

```sql
create table 表名 (
    列名1 数据类型 [约束] [comment 列1注释],
    列名2 数据类型 [约束] [comment 列2注释],
    ......
    列名n 数据类型 [约束] [comment 列n注释]
)[comment 表注释];
```

* `create table 新表名 like 旧表名`：快速创建一个表结构相同的表。


```sql
-- 创建 student 表包含 id,name,birthday 字段
create table student (
    id int unsigned comment '编号',     -- 整数，unsigned是一个约束，代表无符号整数
    name varchar(20),    	         -- 字符串
    birthday date	                 -- 日期，最后没有逗号
) comment '学生表';

-- 创建 s1 表，s1 表结构和 student 表结构相同
create table s1 like student;
```

## 查看表

* `show tables`：查看某个数据库中的所有表
* `desc 表名`：查看表结构
* `show create table 表名`：查看创建表的 SQL 语句

```sql
-- 查看db1数据库中的所有表
use db1;
show tables;

-- 查看 student 表的结构
desc student;

-- 查看 student 的创建表 SQL 语句
show create table student;
```


## 修改表

* 添加列：

```sql
alter table 表名
    add  列名1  数据类型 [约束] [comment 注释],
    add  列名2  数据类型 [约束] [comment 注释],
    ...... ;
```


* 修改列类型：

```sql
alter table 表名 
    modify 列名1 数据类型 [约束] [comment 注释],
    modify 列名2 数据类型 [约束] [comment 注释],
...... ;
```


* 修改列名：

```sql
alter table 表名
    change 旧列名1 新列名1 数据类型 [约束] [comment 注释],
    change 旧列名2 新列名2 数据类型 [约束] [comment 注释],
...... ;
```


* 删除列：

```sql
alter table 表名
    drop 列名1,
    drop 列名2,
...... ;
```


* 修改表名：

```sql
rename table 表名 to 新表名;
```


* 修改字符集：

```sql
alter table 表名 character set 字符集;
```




```sql
-- 为学生表添加一个新的字段 remark,类型为 varchar(20)
alter table student add remark varchar(20);

-- 将 student 表中的 remark 字段的改成 varchar(100)
alter table student modify remark varchar(100);

-- 将 student 表中的 remark 字段名改成 intro，类型 varchar(30)
alter table student change remark intro varchar(30);

-- 删除 student 表中的字段 home_tel 和 home_address
alter table student
drop home_tel,
drop home_address;

-- 将学生表 student 改名成 student2
rename table student to student2;

-- 将 student2 表的编码修改成 gbk
alter table student2 character set gbk;
```

## 删除表

* `drop table 表名`：直接删除表
* `drop table if exists 表名`：判断表是否存在，如果存在则删除表
* `truncate table 表名`：删除表，并重新创建该表

```sql
-- 直接删除表 s1 表
drop table s1;

-- 判断表是否存在并删除 s1 表
drop table if exists `create`;
```