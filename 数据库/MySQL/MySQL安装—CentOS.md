

## 安装

第一步： 在linux上创建MySQL文件夹

![](assets/MySQL安装—CentOS/49459fa3d11d439a9186a51c53dced59_MD5.png)

第二步：下载并上传MySQL安装包，上传到刚刚创建的目录中

![](assets/MySQL安装—CentOS/b3959d070fb8000491240e2c4b4acbb9_MD5.png)

第三步：看下系统里面有没有自带的mysql，有的话卸载

```bash
rpm -qa | grep mysql
```

有的话卸载 `rpm -e 版本号 --nodeps`

![](assets/MySQL安装—CentOS/7d35d9072d1021385d94c87728581fa6_MD5.png)

第四步：解压安装包

```bash
tar -xvf mysql-8.0.25-1.el8.x86_64.rpm-bundle.tar 
```

![](assets/MySQL安装—CentOS/04fe159363229f40efcf3742f0de8d5d_MD5.png)

第五步：安装common

```bash
rpm -ivh mysql-community-common-8.0.25-1.el8.x86_64.rpm --nodeps --force 
```

![](assets/MySQL安装—CentOS/5dd469a1bcc33037001295ceee57c1a9_MD5.png)

第六步：安装libs

```bash
rpm -ivh mysql-community-libs-8.0.25-1.el8.x86_64.rpm --nodeps --force
```

![](assets/MySQL安装—CentOS/921a6f4579ddd244faf422ed4b3fe2a8_MD5.png)

第七步：安装client

```bash
rpm -ivh mysql-community-client-8.0.25-1.el8.x86_64.rpm --nodeps --force
```

![](assets/MySQL安装—CentOS/a300e005b6579a2b420db78b44e441d4_MD5.png)


第八步：安装server

```bash
rpm -ivh mysql-community-server-8.0.25-1.el8.x86_64.rpm --nodeps --force
```
![](assets/MySQL安装—CentOS/a2f13fa6c6a06e8cdb29c5b6dbb34425_MD5.png)


第九步：查看安装情况

```bash
rpm -qa | grep mysql
```

![](assets/MySQL安装—CentOS/9aba9d7c63259389813c445a269146a1_MD5.png)

安装到这里结束，下面是配置


## 配置

第十步：对 mysql 数据库的初始化和相关配置

一行一行执行

```bash
mysqld --initialize;
chown mysql:mysql /var/lib/mysql -R;
systemctl start mysqld.service;
systemctl enable mysqld;
```

![](assets/MySQL安装—CentOS/ef786ce75aa6857f2de22d7a36c417e2_MD5.png)


第十一步：查看数据库的密码

```bash
cat /var/log/mysqld.log | grep password
```

![](assets/MySQL安装—CentOS/5303f4b4d2f516cecc0758f76c55be0c_MD5.png)


第十二步：登录

```bash
mysql -uroot -p
```

然后回车，后输入密码，然后再回车

![](assets/MySQL安装—CentOS/4a1481aa1501b2c826713d7ac2386a2e_MD5.png)


第十三步：改密码

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_passwordBY '新密码';
```

![](assets/MySQL安装—CentOS/e806adf374bec84aa6ebd472c9026ecb_MD5.png)


第十四步：退出并重新登录

```bash
exit;
mysql -uroot -p
```

然后回车，后输入密码，然后再回车

![](assets/MySQL安装—CentOS/92788982ac2f9af498f90c2dcb88f698_MD5.png)


第十五步：远程访问授权

一行命令一个图，成功的话报OK

```bash
create user 'root'@'%' identified with mysql_native_password by 'root';
```

![](assets/MySQL安装—CentOS/f10b24b8c9ee390ad5e417241c37238c_MD5.png)


```bash
grant all privileges on *.* to 'root'@'%' with grant option;
```

![](assets/MySQL安装—CentOS/e49116d7c59830845e578c815167b8d4_MD5.png)


```bash
flush privileges;
```

![](assets/MySQL安装—CentOS/593883df69959bf3ee0e57af534517c9_MD5.png)


第十六步：修改root用户远程访问权限

一行一行执行

```bash
use mysql;
select host, user from user;
GRANT ALL ON *.* TO 'root'@'%';
flush privileges;
```

![](assets/MySQL安装—CentOS/14938e58a703da4928ec69b8dd268dd3_MD5.png)


```bash
-- 修改密码为用不过期
mysql> ALTER USER 'root'@'%' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; 
Query OK, 0 rows affected (0.02 sec)
 
-- 修改密码并指定加密规则为mysql_native_password
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)
 
-- 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```
