
## 查询

* `show databases` ：查看所有的数据库
* `show create database 数据库名` ：查看数据库的定义信息

```sql
-- 查看所有的数据库
show databases;

-- 查看某个数据库的定义信息
show create database db3;
show create database db1;
```


## 创建

* `create database [if not exists] 数据库名 [default character set 字符集] [collate 排序规则]`：创建数据库
  * `if not exists`：判断数据库是否已经存在，不存在则创建数据库
  * `default character set 字符集`：指定字符集，一般字符集会使用utf8字符集，但是在MySQL中可能有些字符会有4个字节，所以一般字符集指定为**utf8mb4**
  * `collate 排序规则`：执行排序规则

```sql
-- 直接创建数据库 db1
create database db1;

-- 判断是否存在，如果不存在则创建数据库 db2
create database if not exists db2;

-- 创建数据库并指定字符集为 gbk
create database db3 default character set gbk;

-- 创建db4数据库，判断是否存在，并制定字符集为gbk
create database if not exists db4 character set gbk;
```


## 修改

* `alter database 数据库名 default character set 字符集`：修改数据库默认的字符集

```sql
-- 将 db3 数据库的字符集改成 utf8
alter database db3 character set utf8;
```


## 删除

* `drop database [if exists] 数据库名`：判断数据库存在，存在再删除
  * `if exists`：判断数据库存在，存在再删除

```sql
-- 删除 db2 数据库
drop database db2;

-- 判断数据库存在，存在再删除
drop database if exists db3;
```


## 使用

* `select database()`：	查看正在使用的数据库，使用的一个 mysql 中的全局函数
* `use 数据库名`：使用/切换数据库

```sql
-- 查看正在使用的数据库
select database();

-- 改变要使用的数据库
use db4;
```