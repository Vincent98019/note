
## 新建表

```sql
Create table 表名(
	字段1 数据类型 [default 默认值],
	字段2 数据类型 [default 默认值],
	...
	字段n 数据类型 [default 默认值]
);
```

例如创建 person 表

```sql

create table person(
	pid      number(10),
	name     varchar2(10),
	gender   number(1) default 1,
	birthday date
);

insert into person(pid, name, gender, birthday)
values(1, '张三', 1, to_date('1999-12-22', 'yyyy-MM-dd'));
``` 

## 删除表

```sql
DROP TABLE 表名
```
  

## 修改表

在 sql 中使用 alter 可以修改表

```sql
-- 添加一列
-- 在person表中添加gender列，类型是长度为1的数值型
alter table person add (gender number(1));

-- 修改列类型
alter table person modify gender char(1);

-- 修改列名称
-- 将person表的gender字段名改为sex
alter table person rename column gender to sex;

-- 删除一列
alter table person drop column sex;
```

添加列：

```sql
ALTER TABLE 表名称 ADD(列名1 类型 [DEFAULT 默认值]，列名2 类型 [DEFAULT 默认值]...)
```
  

修改列：

```sql
ALTER TABLE 表名称 MODIFY(列名1 类型 [DEFAULT 默认值]，列名2 类型[DEFAULT 默认值]...)
```
  

修改列名:

```sql
ALTER TABLE 表名称 RENAME 列名1 TO 列名2
```