
SqlMapConfig.xml配置文件

## 配置的内容和顺序

1. properties（属性）
	- property
2. settings（全局配置参数）
	- setting
3. typeAliases（类型别名）
	- typeAliase
	- package
4. typeHandlers（类型处理器）
5. objectFactory（对象工厂）
6. plugins（插件）
7. environments（环境集合属性对象）
	- environment（环境子属性对象）
		- transactionManager（事务管理）
		- dataSource（数据源）
8. mappers（映射器）
	- mapper
	- package

## properties（属性）

方式一：

```xml
<properties>
    <property name="jdbc.driver" value="com.mysql.jdbc.Driver"/>
	<property name="jdbc.url" value="jdbc:mysql://localhost:3306/mybatisdb"/>
    <property name="jdbc.username" value="root"/>
	<property name="jdbc.password" value="1019"/>
</properties>
```

方式二：

- **在 classpath 下定义 db.properties 文件**
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatisdb
jdbc.username=root
jdbc.password=1019
```

- **properties 标签配置**
```xml
<!-- 配置连接数据库的信息
resource 属性：用于指定 properties 配置文件的位置，要求配置文件必须在类路径下
resource="jdbcConfig.properties"
-->
<properties url=
"file:///E:/IdeaProjects/mybatis02/src/main/resources/jdbcConfig.properties">
</properties>
```

- **此时 dataSource 标签就变成了引用上面的配置**
```xml
<dataSource type="POOLED">
    <property name="driver" value="${jdbc.driver}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</dataSource>
```


## typeAliases（类型别名）

```xml
<typeAliases>
	<!-- 单个别名定义 -->
    <typeAlias alias="user" type="com.arbor.domain.User"/>
	<!-- 批量别名定义，扫描整个包下的类，别名为类名（首字母大写或小写都可以） -->
    <package name="com.arbor.domain"/>
	<package name="其它包"/>
</typeAliases>
```


## mappers（映射器）

常用标签&属性

| **标签&属性**                 | **描述**                                                                  |
| ------------------------- | ----------------------------------------------------------------------- |
| `<mapper resource=" " />` | 使用相对于类路径的资源                                                             |
| 例如：                       | `<mapper resource="com/arbor/dao/UserDao.xml" />`                       |
| `<mapper class=" " />`    | 使用 mapper 接口类路径  <br>**要求 mapper 接口名称和 mapper 映射文件名称相同，且放在同一个目录中。**     |
| 例如：                       | `<mapper class="com.arbor.dao.UserDao"/>`                               |
| `<package name=""/>`      | 注册指定包下的所有 mapper 接口  <br>**要求 mapper 接口名称和 mapper 映射文件名称相同，且放在同一个目录中。** |
| 例如：                       | `<package name="cn.arbor.mybatis.mapper"/>`                             |
