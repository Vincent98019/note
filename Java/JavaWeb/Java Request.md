
Request是请求对象，Response是响应对象。

![](assets/Java%20Request/764a49941187229eec68af8866c5d6d3_MD5.png)


浏览器会发送HTTP请求到后台服务器\[Tomcat\]，请求中会包含很多请求数据 **\[请求行+请求头+请求体\]** ，后台服务器\[Tomcat\]会对HTTP请求中的数据进行解析并把解析结果存入到Request对象，可以从Request对象中获取请求的相关参数。

## 继承体系

![](assets/Java%20Request/05fc74698cbb4f342093e6d94f27bd97_MD5.png)



因为ServletRequest和HttpServletRequest都是接口，无法直接创建对象，所以Tomcat的实现类通过解析并创建对象后，向上转型为接口类型以供使用。

通过debug可以看到ServletRequest的运行类型其实就是RequestFacade，而Response同理。

![](assets/Java%20Request/24310f453a86dded9fed1bcfda085d9a_MD5.png)


## 常用方法

### 获取请求行数据

![](assets/Java%20Request/06ede94d2033f9971783cab396ae3dd0_MD5.png)


* `String getMethod()`：获取请求方式，例：`GET`。
* `String getContextPath()`：获取虚拟目录(项目访问路径)，例：`/request-demo`。
* `StringBuffer getRequestURL()`：获取URL(统一资源定位符)，例： `http://localhost:8080/request-demo/req1`。
* `String getRequestURI()`：获取URI(统一资源标识符)，例： `/request-demo/req1`。
* `String getQueryString()`：获取请求参数(GET方式)，例： `username=zhangsan&password=123`。

> URL：统一资源定位符，例：http://localhost/arbor/demo1  中华人民共和国
> URI：统一资源标识符，例：/arbor/demo1 共和国



```java
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // String getMethod()：获取请求方式： GET
        String method = req.getMethod();

        // String getContextPath()：获取虚拟目录(项目访问路径)：/request-demo
        String contextPath = req.getContextPath();

        // StringBuffer getRequestURL(): 获取URL(统一资源定位符)：http://localhost:8080/request-demo/req1
        StringBuffer url = req.getRequestURL();

        // String getRequestURI()：获取URI(统一资源标识符)： /request-demo/req1
        String uri = req.getRequestURI();

        // String getQueryString()：获取请求参数（GET方式）： username=zhangsan
        String queryString = req.getQueryString();
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```


### 获取请求头数据

请求头的数据格式为`key: value`。

![](assets/Java%20Request/f6015a97b9db256ef63fd4f3b8834e99_MD5.png)


* `String getHeader(String name)`：所以根据请求头名称获取对应值

```java
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取请求头: user-agent: 浏览器的版本信息
        String agent = req.getHeader("user-agent");
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}

```


### 获取请求体数据

浏览器在发送GET请求的时候是没有请求体的，所以请求方式为POST时获取请求体。

![](assets/Java%20Request/2a1f1e6915159a8a10e47c8e57673081_MD5.png)


* `InputStream getInputStream()`：获取字节输入流，如果前端发送的是字节数据，比如传递的是文件数据，则使用该方法。
* `BufferedReader getReader()`：获取字符输入流，如果前端发送的是纯文本数据，则使用该方法。

> BufferedReader流是通过request对象来获取的，当请求完成后request对象就会被销毁，request对象被销毁后，BufferedReader流就会自动关闭，所以不需要手动关闭流。

```java
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
         //获取post 请求体：请求参数
        //1. 获取字符输入流
        BufferedReader br = req.getReader();
        //2. 读取数据
        String line = br.readLine();
    }
}
```


### 【通用】获取请求参数

* `Map<String,String[]> getParameterMap()`：获取所有参数Map集合。
* `String[] getParameterValues(String name)`：根据名称获取参数值（数组）。
* `String getParameter(String name)`：根据名称获取参数值(单个值)。

```java
@WebServlet("/req2")
public class RequestDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //GET请求逻辑
        System.out.println("get....");
        //1. 获取所有参数的Map集合
        Map<String, String[]> map = req.getParameterMap();
        for (String key : map.keySet()) {
            // username:zhangsan lisi
            System.out.print(key+":");

            //获取值
            String[] values = map.get(key);
            for (String value : values) {
                System.out.print(value + " ");
            }

            System.out.println();
        }

        System.out.println("------------");
        // 2. 获取单个参数的数组值
        String[] hobbies = req.getParameterValues("hobby");
        for (String hobby : hobbies) {
            System.out.println(hobby);
        }

        // 3. 获取单个参数的单个值
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        System.out.println(username);
        System.out.println(password);
        
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```




## 中文乱码问题

POST的请求参数是通过request的`getReader()`来获取流中的数据，而Tomcat在获取流的时候采用的编码是ISO-8859-1，ISO-8859-1编码是不支持中文的，所以会出现乱码。

可以通过`request.setCharacterEncoding("UTF-8")`设置编码，UTF-8也可以写成小写。

* `request.setCharacterEncoding(String encode)`：设置request处理流的编码。



```java
@WebServlet("/req4")
public class RequestDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        //1. 解决乱码: POST getReader()
        //设置字符输入流的编码，设置的字符集要和页面保持一致
        request.setCharacterEncoding("UTF-8");
       //2. 获取username
       String username = request.getParameter("username");
       System.out.println(username);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```


Tomcat8已经将Get方式乱码问题解决了，而8以下的版本需要解决：

GET请求获取请求参数的方式是`request.getQueryString()`，`request.setCharacterEncoding("utf-8")`是设置request处理流的编码。而`getQueryString`方法并没有通过流的方式获取数据，所以GET请求不能用设置编码的方式来解决中文乱码问题。

![](assets/Java%20Request/17b2ad43082eb388e798b8d44360cf19_MD5.png)

**乱码原因：**

1. 浏览器通过HTTP协议发送请求和数据给后台服务器（Tomcat)
2. 浏览器在发送HTTP的过程中会对中文数据进行URL编码
3. 在进行URL编码的时候会采用页面`<meta>`标签指定的UTF-8的方式进行编码，`张三`编码后的结果为`%E5%BC%A0%E4%B8%89`
4. 后台服务器(Tomcat)接收到`%E5%BC%A0%E4%B8%89`后会默认按照`ISO-8859-1`进行URL解码
5. 由于前后编码与解码采用的格式不一样，就会导致后台获取到的数据为乱码。



**解决方式：**

```java
@WebServlet("/req4")
public class RequestDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        //1. 解决乱码：POST，getReader()
        //request.setCharacterEncoding("UTF-8");//设置字符输入流的编码

        //2. 获取username
        String username = request.getParameter("username");
        System.out.println("解决乱码前："+username);

        //3. GET,获取参数的方式：getQueryString
        // 乱码原因：tomcat进行URL解码，默认的字符集ISO-8859-1
       /* //3.1 先对乱码数据进行编码：转为字节数组
        byte[] bytes = username.getBytes(StandardCharsets.ISO_8859_1);
        //3.2 字节数组解码
        username = new String(bytes, StandardCharsets.UTF_8);*/
        username  = new String(username.getBytes(StandardCharsets.ISO_8859_1), StandardCharsets.UTF_8);
        System.out.println("解决乱码后："+username);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```


## 请求转发

请求转发(forward)，一种在服务器内部的资源跳转方式。

**转发的特点：**

* 浏览器地址栏路径不发生变化
* 只能转发到当前服务器的内部资源
* 一次请求，可以在转发资源间使用request共享数据



**转发的流程：**

1. 浏览器发送请求给服务器，服务器中对应的资源A接收到请求
2. 资源A处理完请求后将请求发给资源B
3. 资源B处理完后将结果响应给浏览器
4. 请求从资源A到资源B的过程就叫请求转发

![](assets/Java%20Request/b3ba1aae24ea694cd111aaa097e55cc3_MD5.png)


请求转发的实现方式：

`req.getRequestDispatcher("资源B路径").forward(req, resp);`

而转发时，可以设置数据共享和传输，Request中有三个方法：

**常用方法**

* `RequestDispatcher getRequestDispatcher(String path)`：通过request对象获取请求转发器对象。

* `forward(ServletRequest request, ServletResponse response)`：使用RequestDispatcher对象来进行转发

* `void setAttribute(String name, Object o)`：存储数据到request域\[范围,数据是存储在request对象\]中。
* `Object getAttribute(String name)`：根据key获取值。
* `void removeAttribute(String name)`：根据key删除该键值对。

> 域对象：一个有作用范围的对象，可以在范围内共享数据

> request域：代表一次请求的范围，一般用于请求转发的多个资源中共享数据



* `ServletContext getServletContext()`：获取ServletContext。



```java
@WebServlet("/req5")
public class RequestDemo5 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        System.out.println("demo5...");
        //存储数据
        request.setAttribute("msg","hello");
        //请求转发
        request.getRequestDispatcher("/req6").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```


```java
@WebServlet("/req6")
public class RequestDemo6 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
        throws ServletException, IOException {
        System.out.println("demo6...");
        //获取数据
        Object msg = request.getAttribute("msg");
        System.out.println(msg);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```
