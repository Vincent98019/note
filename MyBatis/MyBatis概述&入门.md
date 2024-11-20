

## 概述

MyBatis最初是Apache的一个开源项目iBatis，2010年6月这个项目由Apache Software Foundation迁移到了Google Code。随着开发团队转投Google Code旗下， iBatis3.x正式更名为MyBatis。代码于2013年11月迁移到Github。

iBatis一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。 iBatis提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。

MyBatis官网：[https://mybatis.org/mybatis-3/zh/index.html](https://mybatis.org/mybatis-3/zh/index.html)

GitHub：[https://github.com/mybatis/mybatis-3](https://github.com/mybatis/mybatis-3)

### MyBatis特性

- MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架
    
- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
    
- MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old JavaObjects，普通的Java对象）映射成数据库中的记录
    
- MyBatis 是一个 半自动的ORM（Object Relation Mapping）框架
    

### 和其它持久化层技术对比

#### JDBC

- SQL 夹杂在Java代码中耦合度高，导致硬编码内伤
    
- 维护不易且实际开发需求中 SQL 有变化，频繁修改的情况多见
    
- 代码冗长，开发效率低
    

#### Hibernate 和 JPA

- 操作简便，开发效率高
    
- 程序中的长难复杂 SQL 需要绕过框架
    
- 内部自动生产的 SQL，不容易做特殊优化
    
- 基于全映射的全自动框架，大量字段的 POJO 进行部分映射时比较困难
    
- 反射操作太多，导致数据库性能下降
    

#### MyBatis

- 轻量级，性能出色
    
- SQL 和 Java 编码分开，功能边界清晰（Java代码专注业务、SQL语句专注数据）
    
- 开发效率稍逊于HIbernate，但是完全能够接受
    

## 入门程序

创建一个Maven项目，这里先不使用Spring。

引入依赖：

```xml
<dependencies>  
​  
    <!-- Mybatis核心 -->  
    <dependency>  
        <groupId>org.mybatis</groupId>  
        <artifactId>mybatis</artifactId>  
        <version>3.5.7</version>  
    </dependency>  
​  
    <!-- junit测试 -->  
    <dependency>  
        <groupId>junit</groupId>  
        <artifactId>junit</artifactId>  
        <version>4.13.2</version>  
        <scope>test</scope>  
    </dependency>  
​  
    <!-- MySQL驱动 -->  
    <dependency>  
        <groupId>mysql</groupId>  
        <artifactId>mysql-connector-java</artifactId>  
        <version>8.0.31</version>  
    </dependency>  
​  
</dependencies>
```

创建MyBatis的核心配置文件：

> 习惯上命名为 `mybatis-config.xml` ，并非强制要求改文件名。整合Spring后，这个配置文件可以省略。

核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息。

核心配置文件存放的位置是 `src/main/resources` 目录下。

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-config.dtd">  
​  
<configuration>  
    <!--设置连接数据库的环境-->  
    <environments default="development">  
        <environment id="development">  
            <transactionManager type="JDBC"/>  
            <dataSource type="POOLED">  
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
                <property name="url" value="jdbc:mysql://localhost:3306/MyBatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>  
                <property name="username" value="root"/>  
                <property name="password" value="123123"/>  
            </dataSource>  
        </environment>  
    </environments>  
    <!--引入映射文件-->  
    <mappers>  
        <mapper resource="mappers/UserMapper.xml"/>  
    </mappers>  
</configuration>
```

创建实体类：
```java
@Getter
@Setter
public class User {
	private Integer id;
	private String name;
}
```


编写持久层接口 UserDao：
也可以起名为UserMapper
```java
public interface UserDao {
	List<User> findAll();
}
```


编写持久层接口的映射文件 UserDao.xml ：
- 创建位置：必须和持久层接口在相同的包中。
- 名称：必须以持久层接口名称命名文件名，扩展名是.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper 
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    
<mapper namespace="com.arbor.dao.UserDao">
	<!-- 配置查询所有操作 -->
    <select id="findAll" resultType="com.arbor.domain.User">
		select * from user
	</select>
</mapper>
```


测试类：
```java
public class MybatisTest {
    
    @Test
	public void testFindAll() throws Exception {
		//1.读取配置文件
		InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
		//2.创建 SqlSessionFactory 的构建者对象
		SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
		//3.使用构建者创建工厂对象 SqlSessionFactory
		SqlSessionFactory factory = builder.build(in);
		//4.使用 SqlSessionFactory 生产 SqlSession 对象
		SqlSession session = factory.openSession();
		//5.使用 SqlSession 创建 dao 接口的代理对象
		UserDao userDao = session.getMapper(UserDao.class);
		//6.使用代理对象执行查询所有方法
		List<User> users = userDao.findAll();
		for(User user : users) {
			System.out.println(user);
		}
		//7.释放资源
		session.close();
		in.close();
	}
}
```

## 基于注解的使用

在使用基于注解的 Mybatis 配置时，移除 xml 的映射配置（UserDao.xml）。

在持久层接口中添加注解：
```java
public interface UserDao {
	@Select("select * from user")
	List<User> findAll();
}
```

修改mybatis-config.xml文件：
```xml
<!-- 告知 mybatis 映射配置的位置 -->
<mappers>
    <mapper class="com.arbor.dao.UserDao"/>
</mappers>
```