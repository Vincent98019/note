
约束是作用于表中字段上的规则，用于限制存储在表中的数据。保证数据库中数据的正确、有效性和完整性。

| **关键字**      | **约束**                     | **描述**                                                 |
| --------------- | ---------------------------- | -------------------------------------------------------- |
| not null        | 非空约束                     | 限制该字段的数据不能为null                               |
| unique          | 唯一约束                     | 保证该字段的所有数据都是唯一、不重复的                   |
| primary key     | 主键约束                     | 主键是一行数据的唯一标识，要求非空且唯一                 |
| default         | 默认约束                     | 保存数据时，如果未指定该字段的值，则采用默认值           |
| check           | 检查约束<br>(8.0.16版本之后) | 保证字段值满足某一个条件                                 |
| foreign key     | 外键约束                     | 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 |
| unsigned        | 无符号                       | 只能是0或正整数                                          |
| auto\_increment | 自动增长                     | 字段类型必须是整数类型                                   |

> 约束是作用于表中字段上的，可以在创建表/修改表的时候添加约束。



## 主键约束

用来唯一标识数据库中的每一条记录，通常不用业务字段作为主键，单独给每张表设计一个 id 的字段，把 id 作为主键。主键是给数据库和程序使用的，不是给最终的客户使用的。所以主键有没有含义没有关系，只要不重复，非空就行。

* 主键约束要求字段的值在全表必须唯一，而且不能为null值。
* 建议主键使用数字类型，因为数字的检索速度非常快，而且还可以设置自增长。

| **关键字**      | **描述**                             |
| --------------- | ------------------------------------ |
| primary key     | 字段值唯一，且不能为null             |
| auto\_increment | 表示自动增长(字段类型必须是整数类型) |
| unsigned        | 无符号                               |



* `字段名 字段类型 primary key`：在创建表的时候给字段添加主键
* `alter table 表名 add primary key(字段名);`：在已有表中添加主键
* `caeate table 表名(列名 int primary key auto_increment) auto_increment=起始值;`：修改自增长的默认值起始值
* `alter table 表名 auto_increment=起始值;`：创建好以后修改起始值



> **delete 和 truncate 对自增长的影响：**
> * **delete**：删除所有的记录之后，自增长没有影响。  
> * **truncate**：删除以后，自增长又重新开始。

```sql
-- 创建表学生表 st5, 包含字段(id, name, age)将 id 做为主键
create table st5 (
id int primary key, -- id 为主键
name varchar(20),
age int
)
desc st5;

-- 删除 st5 表的主键
alter table st5 drop primary key;
-- 添加主键
alter table st5 add primary key(id);

-- AUTO_INCREMENT 的默认开始值是 1，修改起始值
-- 指定起始值为 1000
create table st4 (
id int primary key auto_increment,
name varchar(20)
) auto_increment = 1000;

-- 创建好以后修改起始值
alter table st4 auto_increment = 2000;

-- 创建t_teacher表，主键是整数无符号类型，并且自增长
create table t_teacher(
id int unsigned primary key auto_increment,
name varchar(20) not null,
tel char(11) not null unique
);
```


## 唯一约束

* 唯一约束要求字段值如果不为null，那么在全表必须唯一。

| **关键字** | **描述**                 |
| ---------- | ------------------------ |
| unique     | 字段值唯一，且可以为null |



* `字段名 字段类型 unique`：表中某一列不能出现重复的值
* `alter table 表名 drop index 字段名;`：删除唯一约束

```sql
-- 创建学生表 st7, 包含字段(id, name),name 这一列设置唯一约束
create table st7 (
id int,
name varchar(20) unique
)
```


## 非空约束

* 非空约束要求字段的值不能为null值
* null值是没有值，而不是""空字符串

| **关键字** | **描述**         |
| ---------- | ---------------- |
| not null   | 字段值不能为null |
| default    | 默认值           |



* `字段名 字段类型 not null`：某一列不能为 null
* `字段名 字段类型 default 默认值`：不传参数的话，指定一个默认值



> **如果字段设置了非空与唯一约束，该字段与主键的区别：**
> * 主键在一个表中，只能有一个。不能出现多个主键。主键可以单列，也可以是多列。
> * 自增长只能用在主键上。

```sql
-- 创建表学生表 st8, 包含字段(id,name,gender)其中 name 不能为 NULL
create table st8 (
    id int,
    name varchar(20) not null,
    gender char(1)
)

-- 创建一个学生表 st9，包含字段(id,name,address)， 地址默认值是广州 
create table st9 (
    id int,
    name varchar(20),
    address varchar(20) default '广州'
)
```


## 外键约束

外键约束用来让两张表的数据之间建立连接，从而保证数据的一致性和完整性。

* 主表：一方，用来约束别人的表  
* 从表：多方，被别人约束的表

| **关键字**  | **描述**             |
| ----------- | -------------------- |
| foreign key | 保持关联数据的逻辑性 |



* 新建表时增加外键：

```sql
create table 表名(
    字段名 数据类型 [约束],
    ...... ,
    [constraint] [外键约束名称] foreign key(外键字段名) references 主表名(主键字段名)
);
```




* 已有表增加外键：

```sql
alter table 从表 add [constraint] [外键约束名称] foreign key (外键字段名) references 主表(主键字段名);
```




* 删除外键：

```sql
alter table 从表 drop foreign key 外键名称;
```




```sql
-- 创建从表 employee 并添加外键约束 emp_depid_fk
create table employee(
id int primary key auto_increment,
name varchar(20),
age int,
dep_id int, -- 外键对应主表的主键
-- 创建外键约束
constraint emp_depid_fk foreign key (dep_id) references department(id)
);

-- 删除 employee 表的 emp_depid_fk 外键
alter table employee drop foreign key emp_depid_fk;

-- 在 employee 表存在的情况下添加外键
alter table employee add constraint emp_depid_fk 
foreign key (dep_id) references department(id);
```


## 外键级联

添加了外键之后，再删除父表数据时产生的约束行为，就称为删除/更新行为。具体的删除/更新行为有以下几种：

| **行为**    | **说明**                                                     |
| ----------- | ------------------------------------------------------------ |
| no action   | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。**(与restrict一致) 默认行为。**  |
| restrict    | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。 **(与no action一致) 默认行为。** |
| cascade     | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有，则也删除/更新外键在子表中的记录。 |
| set null    | 当在父表中删除对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为null（这就要求该外键允许取null）。 |
| set default | 父表有变更时，子表将外键列设置成一个默认的值 (Innodb不支持)。 |



| **关键字**        | **描述**                                                     |
| ----------------- | ------------------------------------------------------------ |
| on update cascade | 级联更新，只能是创建表的时候创建级联关系。更新主表中的主键，从表中的外键列也自动同步更新 |
| on delete cascade | 级联删除                                                     |



* 在修改和删除主表的主键时，同时更新或删除副表的外键值，称为级联操作：

```sql
alter table 表名 add constraint 外键名称 foreign key (外键字段名称) references 主表名称(主表列名称) on  update cascade / on delete cascade;
```


```sql
-- 创建 employee 表，添加级联更新和级联删除
create table employee(
id int primary key auto_increment,
name varchar(20),
age int,
dep_id int, -- 外键对应主表的主键
-- 创建外键约束
constraint emp_depid_fk foreign key (dep_id) 
references department(id) on update cascade on delete cascade
)

-- 添加外键，并设置级联更新
ALTER TABLE employee ADD CONSTRAINT emp_dep_fk FOREIGN KEY ( dep_id ) REFERENCES department ( id ) ON UPDATE CASCADE;

-- 添加外键，并设置级联删除
ALTER TABLE employee ADD CONSTRAINT emp_dep_fk FOREIGN KEY ( dep_id ) REFERENCES department ( id ) ON DELETE CASCADE;

-- 添加外键，并设置级联更新并删除
ALTER TABLE employee ADD CONSTRAINT emp_dep_fk FOREIGN KEY ( dep_id ) REFERENCES department ( id ) ON UPDATE CASCADE ON DELETE CASCADE;

-- 删除或修改外键时，并把使用到该外键的修改为null
alter table emp add constraint fk_emp_dept_id foreign key (dept_id) references dept(id) on update set null on delete set null ;
```