DCL英文全称是Data Control Language(数据控制语言)，用来管理数据库用户、控制数据库的访问权限。

## 查询用户

* `select * from mysql.user`：查询所有用户

> 其中 Host代表当前用户访问的主机，如果为localhost，仅代表只能够在当前本机访问，是不可以
> 远程访问的。 

> User代表的是访问该数据库的用户名。**在MySQL中需要通过Host和User来唯一标识一个用户。**

![](assets/MySQL用户管理%20DCL/f0a3de749e26ff6b301f66b23f182d11_MD5.png)



## 创建用户

* `create user '用户名'@'主机名' identified by '密码'`：创建一个用户

> 主机名可以使用`%`通配。`localhost` 表示MySQL服务的主机。



```sql
create user 'arbor'@'localhost' identified by 'arbor1019';
```


## 修改用户密码

* `alter user '用户名'@'主机名' identified with mysql_native_password BY '新密码'`：修改用户密码
  * `mysql_native_password` ：表示加密方式



```sql
alter user 'arbor'@'localhost' identified with mysql_native_password by 'arbor1019';
```


## 删除用户

* `drop user '用户名'@'主机名'`：删除用户

```sql
drop user 'arbor'@'localhost';
```




## 权限控制

| **权限**           | **描述**             |
| ------------------ | -------------------- |
| all，all privleges | 所有权限             |
| select             | 查询数据             |
| insert             | 插入数据             |
| update             | 修改数据             |
| delete             | 删除数据             |
| alter              | 修改表               |
| drop               | 删除数据库、表、视图 |
| create             | 创建数据库、表       |

上述只是简单罗列了常见的几种权限描述，其他权限描述及含义，可以直接参考官方文档。



## 查询权限

* `show grants for '用户名'@'主机名'`：查询一个用户的权限

```sql
show grants for 'arbor'@'localhost';
```


## 授予权限

* `grant 权限列表 on 数据库.表名 to '用户名'@'主机名'`：给一个用户授予权限

> 多个权限之间，使用逗号分隔；如果是所有数据库或者所有表时，可以使用`*`代替

```sql
grant all on test.* to 'arbor'@'localhost';
```


## 撤销权限

* `revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名'`：给一个用户撤销权限

> 多个权限之间，使用逗号分隔

```sql
revoke all on test.* from 'arbor'@'localhost';
```