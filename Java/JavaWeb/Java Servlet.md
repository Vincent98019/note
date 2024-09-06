
Servlet（Server Applet）服务器小程序，主要功能用于生成动态Web内容，Servlet就是一个接口，定义了Java类被浏览器访问到(Tomcat识别)的规则。

![](assets/Java%20Servlet/de21a4648e785215de7c7243827102d7_MD5.png)




## 快速入门

1. 创建Web项目，导入Servlet依赖坐标。

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <!-- provided指的是在编译和测试过程中有效，最后生成的war包时不会加入
        因为Tomcat的lib目录中已经有servlet-api这个jar包，
        如果在生成war包的时候生效就会和Tomcat中的jar包冲突，导致报错-->
    <scope>provided</scope>
</dependency>
```


2. 定义一个类，实现Servlet接口，并重写接口中所有方法，并在service方法中输入一句话

```java
import javax.servlet.*;
import java.io.IOException;

public class ServletDemo1 implements Servlet {
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) 
            throws ServletException, IOException {
        System.out.println("你好，Servlet");
    }

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {}

    @Override
    public ServletConfig getServletConfig() { return null; }

    @Override
    public String getServletInfo() { return null; }

    @Override
    public void destroy() {}

}
```


3. 在类上使用`@WebServlet`注解，配置该Servlet的访问路径

```java
@WebServlet("/hello001")
public class ServletDemo1 implements Servlet {...}
```


4. 启动Tomcat，浏览器中输入URL地址访问该Servlet，然后控制台就会打印刚刚写的`"你好，servlet"` 。

![](assets/Java%20Servlet/7fcc9241dc19a727059aec7b60cba49b_MD5.png)


#### 控制台中文乱码

Servlet在打印中文时，出现乱码，可以进行如下配置：

![](assets/Java%20Servlet/c3e5429036798dc296ec8f60239a9c5a_MD5.png)


```Plain Text
-Dfile.encoding=UTF-8
```

![](assets/Java%20Servlet/23460441939d2e7b6df30c0afdb6813c_MD5.png)




## 执行流程

浏览器发出`http://localhost:8080/hello/hello001`请求，从请求中可以解析出三部分内容，分别是`localhost:8080`、`hello`、`hello001` 。

1. 根据`localhost:8080`可以找到要访问的Tomcat Web服务器
2. 根据`hello`可以找到部署在Tomcat服务器上的web项目
3. 根据`hello001`可以找到要访问的是项目中的哪个Servlet类，根据@WebServlet后面的值进行匹配
4. 找到ServletDemo1这个类后，Tomcat Web服务器就会为ServletDemo1这个类创建一个对象，然后调用对象中的service方法

> ServletDemo1实现了Servlet接口，所以类中必然会重写service方法供Tomcat Web服务器进行调用。
> service方法中有ServletRequest和ServletResponse两个参数，ServletRequest封装的是请求数据，ServletResponse封装的是响应数据



## 生命周期

Servlet运行在Servlet容器(web服务器)中，其生命周期由容器来管理，分为4个阶段：

1. **加载和实例化：** 默认情况下，当Servlet第一次被访问时，由容器创建Servlet对象。

> 默认情况，Servlet会在第一次访问被容器创建，但是如果创建Servlet比较耗时的话，那么第一个访问的人等待的时间就比较长，用户的体验就比较差，这时，可以设置将Servlet放到服务器启动的时候来创建：

```java
@WebServlet(urlPatterns = "/demo1", loadOnStartup = 1)
// loadOnstartup的取值有两类情况
// （1）负整数：第一次访问时创建Servlet对象，默认为-1
// （2）0或正整数：服务器启动时创建Servlet对象，数字越小优先级越高
```

2. **初始化：** 在Servlet实例化之后，容器将调用Servlet的`init()`方法初始化这个对象，完成一些如加载配置文件、创建连接等初始化的工作。**该方法只调用一次。**
3. **请求处理：** 每次请求Servlet时，Servlet容器都会调用Servlet的`service()`方法对请求进行处理。
4. **服务终止：** 当需要释放内存或者容器关闭时，容器就会调用Servlet实例的`destroy()`方法完成资源的释放。在`destroy()`方法调用之后，容器会释放这个Servlet实例，该实例随后会被Java的垃圾收集器所回收。



## 常用方法

* `void init(ServletConfig config)`：初始化方法，在Servlet被创建时执行，只执行一次。
* `void service(ServletRequest req, ServletResponse res)`：提供服务方法， 每次Servlet被访问，都会调用该方法。
* `void destroy()`：销毁方法，当Servlet被销毁时，调用该方法。在内存释放或服务器关闭时销毁Servlet。
* `String getServletInfo()`：获取Servlet信息，该方法用来返回Servlet的相关信息，没有什么太大的用处，一般返回一个空字符串即可。
* `ServletConfig getServletConfig()`：获取ServletConfig对象。创建Servlet对象的时候会调用init方法，必定会传入一个ServletConfig对象，只需要将服务器传过来的ServletConfig进行返回即可。

```java
@WebServlet(urlPatterns = "/hello003", loadOnStartup = 1)
public class ServletDemo3 implements Servlet {

    private ServletConfig servletConfig;

    public void init(ServletConfig config) throws ServletException {
        this.servletConfig = config;
        System.out.println("init...");
    }
    public ServletConfig getServletConfig() {
        return servletConfig;
    }

    // ...
}
```


## 体系结构

![](assets/Java%20Servlet/d08607e45f5535e0675ac9902ebc881e_MD5.png)


**GenericServlet：** 将Servlet接口中其他的方法做了默认空实现，只将`service()`方法作为抽象，将来定义Servlet类时，可以继承GenericServlet，实现`service()`方法即可

**HttpServlet：** 对http协议的一种封装，简化操作，定义类继承HttpServlet，复写`doGet`/`doPost`方法



因为将来开发的都是B/S架构的web项目，都是HTTP协议，所以一般会通过继承HttpServlet来写代码：

```java
@WebServlet("/hello004")
public class ServletDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
            throws ServletException, IOException {
        //TODO GET 请求方式处理逻辑
        System.out.println("get...");
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) 
            throws ServletException, IOException {
        //TODO Post 请求方式处理逻辑
        System.out.println("post...");
    }
}
```

要想发送一个GET请求，请求该Servlet，只需要通过浏览器发送`http://localhost:8080/hello/hello004`，就能看到doGet方法被执行了。

要想发送一个POST请求，请求该Servlet，单单通过浏览器是无法实现的，这个时候就需要编写一个form表单来发送请求，在webapp下创建一个`a.html`页面，内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/hello/hello004" method="post">
        <input name="username"/><input type="submit"/>
    </form>
</body>
</html>
```


在HttpServlet的service方法中，先获取请求方式，再根据不同的请求方式调取对应的doXxx方法。

![](assets/Java%20Servlet/eb9caecde210916b222fbef242ec2889_MD5.png)


如果不继承使用HttpServlet，而是直接实现Servlet接口，这个操作则需要自己来完成，以用作不同的处理逻辑。



## @WebServlet

Servlet 3.x之后引入了“注解Annotation”特性。`@WebServlet()`是Servlet的核心注解。

**常用参数**

* `loadOnStartup`：创建时机，负数为第一次被访问时创建，0或正整数为在服务器启动的时候创建，默认为-1，数字越小优先级越高。

* `value`：同下
* `urlPatterns`：映射地址。



**配置规则**

* **精确匹配：** 一个Servlet可以定义多个访问路径： `@WebServlet({"/d4","/dd4","/ddd4"})`

```java
@WebServlet(urlPatterns = {"/demo7", "/demo8"})
public class ServletDemo7 extends HttpServlet {
    @Override
    protected void doGet(ServletRequest req, ServletResponse res) {
        System.out.println("demo7 get...");
    }
    @Override
    protected void doPost(ServletRequest req, ServletResponse res) {
    }
}
```

此时在浏览器上输入`http://localhost:8080/hello/demo7`,`http://localhost:8080/hello/demo8`这两个地址都能访问到ServletDemo7的doGet方法。



* **目录匹配：** 可以进入该目录下的所有路径都可以匹配，如果有精度更高的路径，则优先精度更高的。`@WebServlet({"/d4/*"})`

```java
@WebServlet(urlPatterns = "/user/*")
public class ServletDemo9 extends HttpServlet {

    @Override
    protected void doGet(ServletRequest req, ServletResponse res) {
        System.out.println("demo9 get...");
    }

    @Override
    protected void doPost(ServletRequest req, ServletResponse res) {
    }
}
```

此时如果访问`http://localhost:8080/hello/user` 、`http://localhost:8080/hello/user/a` 、`http://localhost:8080/hello/user/a/b` ，都可以访问到该`doGet` 方法。

但是如果同时有一个路径为`@WebServlet(urlPatterns = "/user/a")` 的路径，则会优先进入该路径下的方法，因为这个的精确度最高，所以优先级最高。



* **扩展名匹配：** 匹配指定的扩展名，使用扩展名时，不可以加`\`，不能写成`"/*.xx"`。`@WebServlet(urlPatterns = "*.do")` 。

```java
@WebServlet(urlPatterns = "*.do")
public class ServletDemo10 extends HttpServlet {

    @Override
    protected void doGet(ServletRequest req, ServletResponse res) {
        System.out.println("demo10 get...");
    }
    @Override
    protected void doPost(ServletRequest req, ServletResponse res) {
    }
}
```

此时，访问路径`http://localhost:8080/hello/[任意].do` ，都可以被匹配到。



* **任意匹配：** 匹配任何路径，可以写`\`或`\*` ，后者的优先级高于前者。

```java
@WebServlet(urlPatterns = "/*")
public class ServletDemo12 extends HttpServlet {
    @Override
    protected void doGet(ServletRequest req, ServletResponse res) {
        System.out.println("demo12 get...");
    }
    @Override
    protected void doPost(ServletRequest req, ServletResponse res) {
    }
}
```

`/`和`/*`的区别：

1. 当项目中的Servlet配置了`/`，会覆盖掉tomcat中的DefaultServlet，当其他的url-pattern都匹配不上时都会走这个Servlet。
2. 当项目中配置了`/*`，意味着匹配任意访问路径。
3. DefaultServlet是用来处理静态资源，如果配置了`/`会把默认的覆盖掉，就会引发请求静态资源的时候没有走默认的而是走了自定义的Servlet类，最终导致静态资源不能被访问。



## XML配置

`@WebServlet`是Servlet从3.0版本后开始支持注解配置，3.0版本前只支持XML配置文件的配置方法。

在web.xml中进行配置。

![](assets/Java%20Servlet/35a59cf6c90a01134c46e60c94177d29_MD5.png)




```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <!-- 声明servlet -->
    <servlet>
        <!-- servlet的别名 -->
        <servlet-name>demo13</servlet-name>
        <!--servlet对应的类-->
        <servlet-class>run.arbor.web.ServletDemo13</servlet-class>
        <!-- servlet的创建时机，可写可不写 -->
        <load-on-startup>0</load-on-startup>
    </servlet>

    <!-- 映射，将servlet与url绑定 -->
    <servlet-mapping>
        <!-- servlet的名称，要和上面的名称一致-->
        <servlet-name>demo13</servlet-name>
        <!-- servlet的访问路径-->
        <url-pattern>/demo13</url-pattern>
    </servlet-mapping>

</web-app>
```


