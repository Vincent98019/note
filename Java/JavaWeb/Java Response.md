
业务处理完后，后台就需要给前端返回业务处理的结果即响应数据，把响应数据封装到response对象中，后台服务器\[Tomcat\]会解析response对象，按照 **\[响应行+响应头+响应体\]** 格式拼接结果，浏览器最终解析结果，把内容展示在浏览器给用户浏览。



## 继承体系

![](assets/Java%20Response/5e41c9976871aec49934f4c62c0592e7_MD5.png)


Reponse的继承体系和Request的继承体系非常相似。



## 常用方法

### 设置响应行

![](assets/Java%20Response/021e8466225136a778bb77b91c0e43f0_MD5.png)


* `void setStatus(int sc)`：设置响应状态码。

### 设置响应头

![](assets/Java%20Response/f7bfa65dcd193b212f594a274682beac_MD5.png)


* `void setHeader(String name, String value)`：设置响应头键值对。



### 设置响应体

![](assets/Java%20Response/234f9f7f02a4ab1d9f43a138d8f11f4e_MD5.png)


对于响应体，是通过字符、字节输出流的方式往浏览器写。

* `PrintWriter getWriter()`：获取字符输出流。
* `ServletOutputStream getOutputStream()`：获取字节输出流。



## 请求重定向

Response重定向(redirect)，一种资源跳转方式。

**重定向的特点：**

* 浏览器地址栏路径发送变化
* 可以重定向到任何位置的资源(服务内容、外部均可)
* 两次请求，不能在多个资源使用request共享数据

**重定向的流程：**

1. 浏览器发送请求给服务器，服务器中对应的资源A接收到请求
2. 资源A现在无法处理该请求，就会给浏览器响应一个302的状态码+location的一个访问资源B的路径
3. 浏览器接收到响应状态码为302就会重新发送请求到location对应的访问地址去访问资源B
4. 资源B接收到请求后进行处理并最终给浏览器响应结果，这整个过程就叫重定向

![](assets/Java%20Response/d5a61068cb24a98a3d19699995ff53c2_MD5.png)


重定向的实现方式：

```java
resp.setStatus(302);
resp.setHeader("location", "资源B的访问路径");
```

![](assets/Java%20Response/72edb5e03d3242e7da39d32fa100d8f3_MD5.png)




**常用方法**

* `setStatus(int status)`：设置状态码。
* `setHeader(String key, String value)`：设置响应头。
* `sendRedirect(String uri)`：重定向的快捷方法，直接传入uri即可。



```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        System.out.println("resp1....");
        //重定向
        //1.设置响应状态码 302
        response.setStatus(302);
        //2. 设置响应头 Location
        response.setHeader("Location","/request-demo/resp2");

        // 重定向，简化写法
        // resposne.sendRedirect("/request-demo/resp2")；
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```


```java
@WebServlet("/resp2")
public class ResponseDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        System.out.println("resp2....");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```


## 资源路径

* 浏览器使用：需要加虚拟目录(项目访问路径)
* 服务端使用：不需要加虚拟目录



**相对路径：通过相对路径不可以确定唯一资源**，如：`./index.html`	不以`/`开头，以 `.` 开头路径

**规则：** 找到当前资源和目标资源之间的相对位置关系

* `./`：当前目录
* `../`：后退一级目录

---

**绝对路径：通过绝对路径可以确定唯一资源**，如：`http://localhost/arbor/responseDemo2`，以`/`开头的路径。

**规则：**

* **给客户端浏览器使用：** 需要加虚拟目录(项目的访问路径)
* **给服务器使用：** 不需要加虚拟目录



## 响应字符数据

要想将字符数据写回到浏览器，需要两步：

1. 通过Response对象获取字符输出流：`PrintWriter writer = resp.getWriter();`
2. 通过字符输出流写数据：`writer.write("aaa");`

> `getWriter()`获取的流的默认编码是ISO-8859-1，所以要设置流的默认编码格式(获取流之前设置)。



```java
//设置响应的数据格式及数据的编码
// text/html：告诉浏览器返回的数据类型是HTML类型数据
response.setContentType("text/html;charset=utf-8");
// response.setHeader("content-type","text/html;charset=utf-8"); 和上面一行代码实现的效果一致
PrintWriter writer = resp.getWriter();
writer.write("你好");
```


## 响应字节数据

将字节数据写回到浏览器，需要两步：

1. 通过Response对象获取字节输出流：`ServletOutputStream outputStream = resp.getOutputStream();`
2. 通过字节输出流写数据：`outputStream.write(字节数据);`


**返回一个图片文件到浏览器：**

```java
@WebServlet("/resp4")
public class ResponseDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        //1. 读取文件
        FileInputStream fis = new FileInputStream("d://a.jpg");
        //2. 获取response字节输出流
        ServletOutputStream os = response.getOutputStream();
        //3. 完成流的copy
        byte[] buff = new byte[1024];
        int len = 0;
        while ((len = fis.read(buff))!= -1){
            os.write(buff,0,len);
        }
        fis.close();
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) 
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```