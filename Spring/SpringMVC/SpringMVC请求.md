
## 支持ant风格的路径

* ？：表示任意的单个字符
* \*：表示任意的0个或多个字符
* \*\*：表示任意的一层或多层目录

注意：在使用`**`时，只能使用`/**/xxx`的方式



## @RequestMapping

用于建立请求 URL 和处理请求方法之间的对应关系

**位置：**

* 类上，请求URL 的第一级访问目录。此处不写的话，就相当于应用的根目录
* 方法上，请求 URL 的第二级访问目录，与类上的使用@ReqquestMapping标注的一级目录一起组成访问虚拟路径

**属性：**

* **value：** 用于指定请求的URL。它和path属性的作用是一样的。可以是一个字符串类型的数组，表示该请求映射能够匹配多个请求地址所对应的请求。
* **method：** 用于指定请求的方式。也可以是一个数组，表示该请求映射能够匹配多种请求方式的请求。
* **params：** 用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的key和value必须和配置的一模一样
  * params = {"accountName"}，表示请求参数必须有accountName
  * params = {"moeny!100"}，表示请求参数中money不能是100
* **headers：** 限制请求头，使用方式类似于params。



### 派生注解

常用的请求方式有get，post，put，delete

但是目前浏览器只支持get和post，若在form表单提交时，为method设置了其他请求方式的字符串（put或delete），则按照默认的请求方式get处理，若要发送put和delete请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter。

对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解

* 处理get请求的映射–>@GetMapping
* 处理post请求的映射–>@PostMapping
* 处理put请求的映射–>@PutMapping
* 处理delete请求的映射–>@DeleteMapping



### 设置请求映射路径

当类上和方法上都添加了`@RequestMapping`注解，前端发送请求的时候，要和两个注解的value值相加匹配才能访问到。

@RequestMapping注解value属性前面加不加`/`都可以

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("user save ...");
        return "{'module':'user save'}";
    }
    
    @RequestMapping("/delete")
    @ResponseBody
    public String save(){
        System.out.println("user delete ...");
        return "{'module':'user delete'}";
    }
}
```


## 获取请求参数

### 通过ServletAPI获取

将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象

```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```


### 通过形参获取请求参数

在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参。

```java
@RequestMapping("/testParam")
public String testParam(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "success";
}
```

1. 若请求所传输的请求参数中有多个同名的请求参数，此时可以在控制器方法的形参中**设置字符串数组**或者**字符串类型**的形参接收此请求参数
2. 若使用字符串数组类型的形参，此参数的数组中包含了每一个数据
3. 若使用字符串类型的形参，此参数的值为每个数据中间使用逗号拼接的结果



### @RequestParam

@RequestParam是将请求参数和控制器方法的形参创建映射关系，一共有三个属性：

* **value：** 指定为形参赋值的请求参数的参数名
* **required：** 设置是否必须传输此请求参数，默认值为true

> 若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置defaultValue属性，则页面报错400：Required String parameter ‘xxx’ is not present；若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为null

* **defaultValue：** 不管required属性值为true或false，当value所指定的请求参数没有传输或传输的值为""时，则使用默认值为形参赋值



### @RequestHeader

@RequestHeader是将请求头信息和控制器方法的形参创建映射关系，一共有三个属性：value、required、defaultValue，用法同@RequestParam。



### @CookieValue

@CookieValue是将cookie数据和控制器方法的形参创建映射关系，一共有三个属性：value、required、defaultValue，用法同@RequestParam。



### 通过POJO获取请求参数

可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的请求参数的参数名和实体类中的属性名一致，那么请求参数就会为此属性赋值。

```java
public class User {
    private String name;
    private int age;
    //setter...getter...略
}
```

```java
@RequestMapping("/pojoParam")
@ResponseBody
public String pojoParam(User user){
    System.out.println("pojo参数传递 user ==> "+user);
    return "{'module':'pojo param'}";
}
```


### @PathVariable

SpringMVC路径中的占位符常用于RESTful风格中，当请求路径中将某些数据通过路径的方式传输到服务器中，就可以在相应的@RequestMapping注解的value属性中通过占位符{xxx}表示传输的数据，在通过@PathVariable注解，将占位符所表示的数据赋值给控制器方法的形参

```java
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username") String username){
    System.out.println("id:"+id+",username:"+username);
    return "success";
}
//最终输出的内容为-->id:1,username:admin
```


### 获取请求参数乱码

#### GET请求中文乱码

Tomcat8.5以后的版本已经处理了中文乱码的问题，但是IDEA中的Tomcat插件目前只到Tomcat7，所以需要修改pom.xml来解决GET请求中文乱码问题

```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
        <configuration>
          <port>80</port><!--tomcat端口号-->
          <path>/</path> <!--虚拟目录-->
          <uriEncoding>UTF-8</uriEncoding><!--访问路径编解码字符集-->
        </configuration>
      </plugin>
    </plugins>
  </build>
```


#### POST请求中文乱码

配置过滤器，SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效

```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    //乱码处理，CharacterEncodingFilter是在spring-web包中，所以用之前需要导入对应的jar包。
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        filter.setEncoding("UTF-8");
        return new Filter[]{filter};
    }
}
```

或者使用XML的方式(在web.xml中进行注册)：

```xml
<!--配置springMVC的编码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
