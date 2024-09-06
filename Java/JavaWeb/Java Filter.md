
Filter 表示过滤器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一。

过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能。

![](assets/Java%20Filter/82fee6b919175895ba01bb88a79b2360_MD5.png)


过滤器一般完成一些通用的操作。



### 快速入门

1. 定义类，实现 Filter接口，并重写其所有方法。

![](assets/Java%20Filter/d23dfa3ca32a3f9c0a949aceee07f745_MD5.png)


2. 配置Filter拦截资源的路径：在类上定义`@WebFilter`注解。而注解的`value`属性值`/*`表示拦截所有的资源。

![](assets/Java%20Filter/d1daca665fdd23f87b9592d47201b390_MD5.png)


3. 在doFilter方法中输出一句话，并放行。

![](assets/Java%20Filter/4882afa261085cfd20b41d66e18e61c6_MD5.png)


> `chain.doFilter(request,response);`就是放行，让其访问本该访问的资源。



### 执行流程

![](assets/Java%20Filter/05da9e003085dccbf197786ff0888789_MD5.png)
![](assets/Java%20Filter/f2a59b822da7817b96f58b0395e3fc36_MD5.png)
![](assets/Java%20Filter/c3422939f38d208b2661501bb05f63be_MD5.png)




### 拦截路径配置

拦截路径表示 Filter 会对请求的哪些资源进行拦截，使用`@WebFilter`注解进行配置。如：`@WebFilter("拦截路径")`



**拦截路径有如下四种配置方式：**



* **拦截具体的资源：**`/index.jsp`，只有访问index.jsp时才会被拦截
* **目录拦截：**`/user/*`，访问/user下的所有资源，都会被拦截
* **后缀名拦截：**`*.jsp`，访问后缀名为jsp的资源，都会被拦截
* **拦截所有：**`/*`，访问所有资源，都会被拦截



### 过滤器链

过滤器链是指在一个Web应用，可以配置多个过滤器，这多个过滤器称为过滤器链。

![](assets/Java%20Filter/eb7ffb8b43d028317e4ce21294cfd147_MD5.png)


上图中的过滤器链执行是按照以下流程执行：



1. 执行`Filter1`的放行前逻辑代码
2. 执行`Filter1`的放行代码
3. 执行`Filter2`的放行前逻辑代码
4. 执行`Filter2`的放行代码
5. 访问到资源
6. 执行`Filter2`的放行后逻辑代码
7. 执行`Filter1`的放行后逻辑代码



使用注解配置Filter的优先级是按照过滤器类名(字符串)的自然排序。比如有如下两个名称的过滤器：`BFilterDemo`和 `AFilterDemo`。那一定是 `AFilterDemo`过滤器先执行。
