Listener 表示监听器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一。

监听器可以监听就是在`application`，`session`，`request`三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件。

`application`是`ServletContext`类型的对象。


`ServletContext`代表整个web应用，在服务器启动的时候，tomcat会自动创建该对象。在服务器关闭时会自动销毁该对象。



JavaWeb 提供了8个监听器：

![](assets/Java%20Listener/393b3a6e3487183d9f81a2561d18ab41_MD5.png)


`ServletContextListener`是用来监听`ServletContext`对象的创建和销毁。

`ServletContextListener`接口中有以下两个方法：

* `void contextInitialized(ServletContextEvent sce)`：`ServletContext`对象被创建了会自动执行的方法。
* `void contextDestroyed(ServletContextEvent sce)`：`ServletContext`对象被销毁时会自动执行的方法。

```java
@WebListener
public class ContextLoaderListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        //加载资源
        System.out.println("ContextLoaderListener...");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        //释放资源
    }
}
```


> 启动服务器，就可以在启动的日志信息中看到`contextInitialized()`方法输出的内容，同时也说明了`ServletContext`对象在服务器启动的时候被创建了。
