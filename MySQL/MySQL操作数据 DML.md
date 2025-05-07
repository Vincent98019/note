## 添加数据

* `insert [ignore] [into] 表名 [字段名] values (字段值) [,(字段值2)...]`：添加数据


* `insert into 表名 (字段名 1, 字段名 2, …) values (值 1, 值 2, ...)`：给指定字段添加数据
* `insert into 表名 values (值 1, 值 2, …)`：给全部字段添加数据，则默认给所有列添加值
* `insert into 表名 (字段名 1, 字段名 2, ...) values (值 1, 值 2, ...), (值 1, 值 2, ...)`：批量添加数据
* `insert into 表名 set 字段1=值1, 字段2=值2, ... `：给指定字段添加数据，mysql的方言语法
* `insert ignore into 表名....`：如果插入某一行有冲突的话，则插入没有冲突的行
  * `ignore`：会让insert只插入数据库不存在的记录



```sql
-- 如果只插入部分列，必须写列名
insert into student (id, name, age, sex) values (1, '孙悟空', 20, '男');
insert into student (id, name, age, sex) values (2, '孙悟天', 16, '男');

-- 插入所有列
insert into student values (3, '孙悟饭', 18, '男', '龟仙人洞中');
```


> 注意事项：
> 
> * 插入数据时，指定的字段顺序需要与值的顺序一致；
> * 字符串和日期型数据应该包含在引号中；
> * 插入的数据大小应该在字段规定的范围内；
> * 没有添加数据的字段会使用 NULL。

## 更新数据

* `update [ignore] 表名 set 字段1=值1, 字段2=值2, ... [where 条件1 ...] [order by ...] [limit ...]`：更新一个或多个字段
* `update 表名 set 字段名=值`：不带条件修改数据，修改所有的行
* `update 表名 set 字段名=值 where 字段名=值`：	带条件修改数据
* `update 表1 join 表2 on 条件 set 字段1=值1, 字段2=值2, ... `：	修改多张表的记录
* `update 表1, 表2 set 字段1=值1, 字段2=值2, ... where 条件...`：	表内连接语法
* `update 表1 [ lift | right ] join 表2 on 条件 set 字段1=值1, 字段2=值2 , ...`：表外连接语法



```sql
-- 不带条件修改数据，将所有的性别改成女
update student set sex = '女';
-- 带条件修改数据，将 id 号为 2 的学生性别改成男
update student set sex='男' where id = 2;
-- 一次修改多个列，把 id 为 3 的学生，年龄改成 26 岁，address 改成北京
update student set age=26, address='北京' where id = 3;
```


## 删除数据

* `delete [ignore] from 表名 [where 条件1, 条件2, ...] [order by ...] [limit ...]`：删除记录，不写条件的话删全部
* `delete from 表名`：不带条件删除数据
* `delete from 表名 where 字段名=值`：带条件删除数据
* `delete 表1, ... from 表1 join 表2 on 条件 [where 条件1, 条件2, ...] [order by ...] [limit ...]`：表内连接删除
* `delete 表1, 表2 from 表1 [ lift | right ] join 表2 on 条件 set 字段1=值1, 字段2=值2 , ...`：表外连接删除
* `truncate table 表名`：使用 truncate 删除表中所有记录



```sql
-- 带条件删除数据，删除 id 为 1 的记录
delete from student where id=1;
-- 不带条件删除数据,删除表中的所有数据
delete from student;
```


**truncate 和 delete 的区别：**

* truncate相当于删除表的结构，再创建一张表。
* delete是在事务机制下删除记录，删除记录之前，先把将要删除的记录保持到日志文件里，然后再删除记录。
* truncate语句在事务机制之外删除记录，速度远超过delete语句

