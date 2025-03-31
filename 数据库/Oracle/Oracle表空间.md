## 创建表空间

ORACLE 数据库的逻辑单元。 **数据库---表空间**

一个表空间可以与多个数据文件（物理结构）关联一个数据库下可以建立多个表空间，一个表空间可以建立多个用户、一个用户下可以建立多个表。

```sql
-- 创建表空间
create tablespace study
datafile 'e:\Oracle\study.dbf'  -- 表空间逻辑的位置
size 100m                       -- 首次占用空间100m
autoextend on                   -- 如果不够了，后续继续增加
next 10m;                       -- 每次增加10m
```

- **study**：表空间名称
- **datafile**：指定表空间对应的数据文件 
- **size**：后定义的是表空间的初始大小
- **autoextend on**：自动增长，当表空间存储都占满时，自动增长
- **next**：后指定的是一次自动增长的大小

## 删除表空间

```sql
-- 删除表空间
drop tablespace study;
```