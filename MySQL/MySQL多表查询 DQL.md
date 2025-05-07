## 多表关系
项目开发中，在进行数据库表结构设计时，会根据业务需求及业务模块之间的关系，分析并设计表结构，由于业务之间相互关联，所以各个表结构之间也存在着各种联系，基本上分为三种：

* 一对多(多对一)
* 多对多
* 一对一



### 一对多
部门与员工的关系：一个部门对应多个员工，一个员工对应一个部门。

实现：在多的一方建立外键，指向一的一方的主键。
![](assets/MySQL多表查询%20DQL/083651587ee45f84dd35cafff18aa34d_MD5.png)



### 多对多
学生与课程的关系：一个学生可以选修多门课程，一门课程也可以供多个学生选择。

实现：建立第三张中间表，中间表至少包含两个外键，分别关联两方主键。
![](https://i-blog.csdnimg.cn/blog_migrate/8f1e22c787b442f97145c8f0c4e1a710.png)


### 一对一
用户与用户详情的关系：一对一关系，多用于单表拆分，将一张表的基础字段放在一张表中，其他详情字段放在另一张表中，以提升操作效率。

实现：在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的(UNIQUE)。
![](assets/MySQL多表查询%20DQL/e6b2b4ee56a2214cbc3760847c2cc43c_MD5.png)

## 笛卡尔积
笛卡尔积：笛卡尔乘积是指在数学中，两个集合A集合 和 B集合的所有组合情况。
![](assets/MySQL多表查询%20DQL/2b8a5412f8c6e2563cfd25ad239fb4cd_MD5.png)


而在多表查询中，需要消除无效的笛卡尔积，只保留两张表关联部分的数据。
![](assets/MySQL多表查询%20DQL/1bee3313d3fb57c9575e90f3ee09106e_MD5.png)


```sql
-- 设置过滤条件 Column 'id' in where clause is ambiguous
select * from emp,dept where id=5;
select * from emp,dept where emp.`dept_id` = dept.`id`;
-- 查询员工和部门的名字
select emp.`name`, dept.`name` from emp,dept where emp.`dept_id` = dept.`id`;
```


## 多表查询分类
* 内连接：相当于查询A、B交集部分数据
* 外连接：
   * 左外连接：查询左表所有数据，以及两张表交集部分数据
   * 右外连接：查询右表所有数据，以及两张表交集部分数据
* 自连接：当前表与自身的连接查询，自连接必须使用表别名
![](assets/MySQL多表查询%20DQL/5894f69fa58c8c9b6ac7637128489e36_MD5.png)


* 子查询


## 表连接
### 内连接
内连接查询的是两张表交集部分的数据。(也就是绿色部分的数据)。

![](assets/MySQL多表查询%20DQL/5894f69fa58c8c9b6ac7637128489e36_MD5.png)

内连接的语法分为两种：隐式内连接、显式内连接。

* `select 字段名 from 左表, 右表 where 条件`：**隐式内连接**，看不到 join 关键字，条件使用 where 指定。
* `select 字段名 from 左表 [inner] join 右表 where 条件`：**显式内连接**，可以省略 inner。



> **总结：**
> * 内连接的数据表不一定必须有同名字段，只要字段之间符合逻辑关系就可以。
> * 相同的数据表也可以做表连接。
> * 结果集也可以作为一张“表”来跟其他表连接。



```sql
-- 隐式内连接
select * from emp,dept where emp.`dept_id` = dept.`id`;

-- 确定表连接条件，员工表.dept_id = 部门表.id 的数据才是有效的
select * from emp e inner join dept d on e.`dept_id` = d.`id`;

-- 查询与SCOTT相同部门的员工都有谁
select e2.ename
from t_emp e1 join t_emp e2 on e1.deptno = e2.deptno 
where e1.ename = "SCOTT" and e2.ename != "SCOTT"

-- 查询月薪超过公司平均月薪的员工信息
select e.*
from t_emp e 
join (select avg(sal) avg from t_emp) t 
on e.sal >= t.avg
```


### 外连接
外连接与内连接的区别在于，除了符合条件的记录之外，结果集中还会保留不符合条件的记录。左外连接就是保留左表所有的记录，与右表做连接。如果右表有符合条件的记录就与左表连接。如果右表没有符合条件的记录，就用NULL与左表连接。右外连接也是如此。

内连接只保留符合条件的记录，所以查询条件写在ON子句和WHERE子句中的效果是相同的。但是外连接里，条件写在WHERE子句里，不合符条件的记录是会被过滤掉的，而不是保留下来。

* `select 字段名 from 左表 left [outer] join 右表 on 条件`：**左外连接**，outer 可以省略，用左边表的记录去匹配右边表的记录，如果符合条件的则显示；否则，显示 NULL。
> 左外连接相当于查询表1(左表)的所有数据，也包含表1和表2交集部分的数据。



* `select 字段名 from 左表 right [outer] join 右表 on 条件`：**右外连接**，outer 可以省略，用右边表的记录去匹配左边表的记录，如果符合条件的则显示；否则，显示 NULL
> 右外连接相当于查询表2(右表)的所有数据，也包含表1和表2交集部分的数据。



```sql
-- 在员工表中增加一个员工
insert into emp values (null, '沙僧','男',6666,'2013-12-05',null);
select * from emp;
-- 使用内连接查询
select * from dept inner join emp on dept.`id` = emp.`dept_id`;
-- 使用右外连接查询
select * from dept right join emp on dept.`id` = emp.`dept_id`;
```


### 自连接
自连接查询就是把一张表连接查询多次。可以是内连接查询，也可以是外连接查询。

* `select 字段列表 from 表A 别名A join 表A 别名B on 条件 ... ;`：自连接查询

> 在自连接查询中，必须要为表起别名，要不然不清楚所指定的条件、返回的字段，到底是哪一张表的字段。

```sql
-- 查询员工 及其 所属领导的名字
select a.name , b.name from emp a , emp b where a.managerid = b.id;

--  查询所有员工 emp 及其领导的名字 emp , 如果员工没有领导, 也需要查询出来表结构: emp a , emp b
select a.name '员工', b.name '领导' from emp a left join emp b on a.managerid = b.id;
```


## 联合查询
union查询，就是把多次查询的结果合并起来，形成一个新的查询结果集。

* `(查询语句) UNION (查询语句) UNION (查询语句) ...`：UNION关键字可以将多个查询语句的结果集进行合并

> - 对于联合查询的多张表的列数必须保持一致，字段类型也需要保持一致。
> - union all 会将全部的数据直接合并在一起，union 会对合并之后的数据去重。
> - 如果多条查询语句查询出来的结果，字段数量不一致，会报错。



```sql
-- 将薪资低于 5000 的员工 , 和 年龄大于 50 岁的员工全部查询出来
-- 不去重
select * from emp where salary < 5000 
union all 
select * from emp where age > 50;
```
![](assets/MySQL多表查询%20DQL/4943ab352a95cde27b76a251695f43d4_MD5.png)


```sql
-- 去重
select * from emp where salary < 5000 
union 
select * from emp where age > 50;
```
![](assets/MySQL多表查询%20DQL/3a352555eaa8bb5e8e4b8f42abebe97e_MD5.png)




## 子查询
子查询可以写在三个地方：WHERE子句、FROM子句、SELECT子句，但是只有FROM子句子查询是最可取的。

根据子查询结果不同，分为：

* 标量子查询(子查询结果为单个值）
* 列子查询(子查询结果为一列)
* 行子查询(子查询结果为一行)
* 表子查询(子查询结果为多行多列)

> 查询语句执行的时候要多次的依赖于子查询的结果，这类子查询被称作相关子查询，WHERE子查询和SELECT子查询都属于相关子查询。因为相关子查询要反复多次执行，所以应该避免使用。



### EXISTS关键字
EXISTS关键字是把原来在子查询之外的条件判断，写到了子查询的里面。

* `select …… from 表名 where [not] exists ( 子查询 );`



### 标量子查询(结果是一个值)
子查询结果只要是单行单列，肯定在 WHERE 后面作为条件，父查询使用：比较运算符，如：> 、<、<>、= 等。

* `select 查询字段 from 表 where 字段 = (子查询);`



```sql
-- 1) 查询最高工资是多少
select max(salary) from emp;
-- 2) 根据最高工资到员工表查询到对应的员工信息
select * from emp where salary = (select max(salary) from emp);

-- 1) 查询平均工资是多少
select avg(salary) from emp;
-- 2) 到员工表查询小于平均的员工信息
select * from emp where salary < (select avg(salary) from emp);
```


### 列子查询(结果是多行单列)
子查询返回的结果是一列（可以是多行），这种子查询称为列子查询。

常用的操作符：in、not in、any、some、all

|**操作符**|**描述**|
| ----- | ----- |
|in|在指定的集合范围之内，多选一|
|not in|不在指定的集合范围之内|
|any|子查询返回列表中，有任意一个满足即可|
|some|与ANY等同，使用SOME的地方都可以使用ANY|
|all|子查询返回列表的所有值都必须满足|



* `select 查询字段 from 表 where 字段 in (子查询);`



```sql
-- 先查询大于 5000 的员工所在的部门 id
select dept_id from emp where salary > 5000;
-- 再查询在这些部门 id 中部门的名字 Subquery returns more than 1 row
select name from dept where id = (select dept_id from emp where salary > 5000);
select name from dept where id in (select dept_id from emp where salary > 5000);

-- 先查询开发部与财务部的 id
select id from dept where name in('开发部','财务部');
-- 再查询在这些部门 id 中有哪些员工
select * from emp where dept_id in (select id from dept where name in('开发部','财务部'));

-- 查询比 财务部 所有人工资都高的员工信息
-- 1.查询所有 财务部 人员工资
select salary from emp where dept_id = (select id from dept where name = '财务部');
-- 2.比 财务部 所有人工资都高的员工信息
select * from emp where salary > all ( select salary from emp where dept_id = (select id from dept where name = '财务部') );

-- 查询比研发部其中任意一人工资高的员工信息
-- 1.查询研发部所有人工资
select salary from emp where dept_id = (select id from dept where name = '研发部');
-- 2.比研发部其中任意一人工资高的员工信息
select * from emp where salary > any (select salary from emp where dept_id = (select id from dept where name = '研发部'));
```


### 行子查询(结果是一行多列)
子查询返回的结果是一行（可以是多列），这种子查询称为行子查询。

常用的操作符：= 、<> 、IN 、NOT IN

```sql
-- 查询与 "张无忌" 的薪资及直属领导相同的员工信息
-- 1. 查询 "张无忌" 的薪资及直属领导
select salary, managerid from emp where name = '张无忌';
-- 2. 查询与 "张无忌" 的薪资及直属领导相同的员工信息 ;
select * from emp where (salary,managerid) = (select salary, managerid from emp where name = '张无忌');
```


### 表子查询(结果是多行多列)
子查询返回的结果是多行多列，这种子查询称为表子查询。子查询结果只要是多列，一般在 from 后面作为表。

* `select 查询字段 from (子查询) 表别名 where 条件;`

子查询作为表需要取别名，否则这张表没有名称则无法访问表中的字段。



```sql
-- 查询与 "鹿杖客" , "宋远桥" 的职位和薪资相同的员工信息
-- 1.  查询 "鹿杖客" , "宋远桥" 的职位和薪资
select job, salary from emp where name = '鹿杖客' or name = '宋远桥';
-- 2. 查询与 "鹿杖客" , "宋远桥" 的职位和薪资相同的员工信息
select * from emp where (job,salary) in ( select job, salary from emp where name = '鹿杖客' or name = '宋远桥' );

-- 查询入职日期是 "2006-01-01" 之后的员工信息 , 及其部门信息
-- 1. 入职日期是 "2006-01-01" 之后的员工信息
select * from emp where entrydate > '2006-01-01';
-- 2. 查询这部分员工, 对应的部门信息;
select e.*, d.* from (select * from emp where entrydate > '2006-01-01') e left join dept d on e.dept_id = d.id ;
```


## 子查询效率问题
### WHERE子查询
这种子查询最简单，最容易理解，但是却是效率很低的子查询。
![](assets/MySQL多表查询%20DQL/ff84a7b21e8c82be6970147adc91d650_MD5.png)


**用表连接替代WHERE子查询：**

表连接的优点是子查询只执行一次，查询效率特别高
![](assets/MySQL多表查询%20DQL/d0977f3f4649bdfa30240005a9edaff9_MD5.png)


### FROM子查询
这种子查询只会执行一次，所以查询效率很高。
![在这里插入图片描述](assets/MySQL多表查询%20DQL/703a7a369f24f8c28ebf37897b40475f_MD5.png)


### SELECT子查询
这种子查询每输出一条记录的时候都要执行一次，查询效率很低。
![](assets/MySQL多表查询%20DQL/9cf8fcefcfa0cad00295a2c354fd8b39_MD5.png)

