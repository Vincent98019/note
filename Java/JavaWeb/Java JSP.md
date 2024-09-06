
JSP（Java Server Page）是J2EE的功能模块，是Java服务器页面，由Web服务器执行，作用就是降低动态网页开发难度，将Java代码与HTML分离，降低开发难度，本质就是Servlet。

**Servlet的缺点：**
- 静态HTML与动态Java代码混合在一起，难以维护
- Servlet利用`out.println()`语句输出，开发效率低下

**JSP的运行要求：**
- 可正常运行的Tomcat

- 所有JSP页面扩展名必须是 `.jsp`
- JSP页面应放在Web应用程序目录下

**JSP的执行过程：**

![](assets/Java%20JSP/f2cf96ecb15e15de92c1c88f160fd089_MD5.jpg)

![](assets/Java%20JSP/628f846dc62f34c3c9f85db6ecb229d3_MD5.jpg)


## 基本语法

### JSP代码块

JSP代码块用于在JSP中嵌入Java代码

语法：`<% java代码 %>`

例如：
```java
<%System.out.println("Hello World!");%>
```

### JSP声明构造块

JSP声明构造块用于声明变量或方法

语法：`<%! 声明语句 %>`
例如：
```java
<%! public int add(int a,int b){return a+b;} %>
```

### JSP输出指令

JSP输出指令用于在JSP页面中显示java代码执行结果

语法：`<%= java代码 %>`

例如：
```java
<%= "<b>" + name + "</b>" %>
```

### JSP处理指令

JSP处理指令用于提供JSP执行过程中的辅助信息

语法：`<%@ jsp指令 %>`
例如：
```java
<%@ page import="java.util.*" %>
```

**常用处理指令：**
- `<%@ page %>`：定义当前JSP页面全局设置
- `<%@ include %>`：将其他JSP页面与当前JSP页面合并
- `<%@ taglib %>`：引入JSP标签库


用于配置JSP页面，导入资源文件

语法：
```java
<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>
```
- `page`：配置JSP页面的
	- `contentType`：等同于`response.setContentType()`，设置响应体的mime类型以及字符集，设置当前jsp页面的编码（只能是高级的IDE才能生效，如果使用低级工具，则需要设置pageEncoding属性设置当前页面的字符集）
	- `import`：导包
	- `errorPage`：当前页面发生异常后，会自动跳转到指定的错误页面
	- `isErrorPage`：标识当前也是是否是错误页面。true：是，可以使用内置对象exception；false：否，默认值，不可以使用内置对象exception
- `include`：页面包含的，导入页面的资源文件 `<%@include file="top.jsp"%>`
- `taglib`：导入资源 `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`
	- `prefix`：前缀，自定义的

### 注释

- `<%-- 注释 --%>`：JSP注释，被注释语句不做任何处理
- `//` 、`/*..*/`：用于注释`<%%>`中的java代码，被注释代码不执行
- `<!-- html -->`：HTML注释，被注释的语句不会被浏览器解释

## 内置对象

在jsp页面中不需要创建，直接使用的对象

| 变量名 | 真实类型 | 作用 |
|----|----|----|
| pageContext | PageContext | 当前页面共享数据，还可以获取其他八个内置对象 |
| request | HttpServletRequest | 一次请求访问的多个资源(转发) |
| session | HttpSession | 一次会话的多个请求间 |
| application | ServletContext | 所有用户间共享数据 |
| response | HttpServletResponse | 响应对象 |
| page | Object | 当前页面(Servlet)的对象  this |
| out | JspWriter | 输出对象，数据输出到页面上 |
| config | ServletConfig | Servlet的配置对象 |
| exception | Throwable | 异常对象 |

