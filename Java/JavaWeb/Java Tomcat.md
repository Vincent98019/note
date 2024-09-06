## web服务器软件

服务器是安装了服务器软件的计算机，在web服务器软件中，可以部署web项目，让用户通过浏览器来访问这些项目。

Web服务器是一个应用程序（软件），对HTTP协议的操作进行封装，使得程序员不必直接对协议进行操作，让Web开发更加便捷。主要功能是"提供网上信息浏览服务"。



web服务器软件：接收用户的请求，处理请求，做出响应。



常见的java相关的web服务器软件：

* **webLogic：** oracle公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
* **webSphere：** IBM公司，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
* **JBOSS：** JBOSS公司的，大型的JavaEE服务器，支持所有的JavaEE规范，收费的。
* **Tomcat：** Apache基金组织，中小型的JavaEE服务器，仅仅支持少量的JavaEE规范servlet/jsp。开源的，免费的。



## Tomcat

Tomcat是Apache软件基金会一个核心项目，是一个开源免费的轻量级Web服务器，支持Servlet/JSP少量JavaEE规范。

因为Tomcat支持Servlet/JSP规范，所以Tomcat也被称为Web容器、Servlet容器。Servlet需要依赖Tomcat才能运行。

Tomcat的官网：[https://tomcat.apache.org/ ](https://tomcat.apache.org/)

Tomcat在解压缩的时候，解压所在的目录可以任意，但最好解压到一个不包含中文和空格的目录，因为后期在部署项目的时候，如果路径有中文或者空格可能会导致程序部署失败。

![](assets/Java%20Tomcat/962d5df94f464b416f8947b899782c37_MD5.png)


* bin：目录下有两类文件，一种是以`.bat`结尾的，是Windows系统的可执行文件，一种是以`.sh`结尾的，是Linux系统的可执行文件。
* webapps：就是以后项目部署的目录。



## 启动/关闭

启动：双击`bin\startup.bat` 即可

关闭：

* 强制关闭：直接x掉运行窗口，不建议
* 正常关闭：`bin\shutdown.bat`
* 正常关闭：`ctrl+c`



### 可能遇到的问题
#### 启动窗口一闪而过

没有正确配置JAVA_HOME环境变量，或已经开启了Tomcat

#### 控制台中文乱码

tomcat的日志配置文件的编码需要修改，找到tomcat安装目录，找到`conf/logging.properties`文件，将其中的`java.util.logging.ConsoleHandler.encoding = UTF-8`的修改为`GBK` 即可。



#### 端口号冲突

![](assets/Java%20Tomcat/dc5fcd8a72f20b8e4ddd6e146c3c4965_MD5.png)


* 暴力解决：找到占用的端口号，并且找到对应的进程，杀死该进程 `netstat -ano`
* 温柔解决：修改自身的端口号，在`conf/server.xml`文件中修改，一般会将tomcat的默认端口号修改为80。80端口号是http协议的默认端口号，在访问时，就不用输入端口号。

```xml
<Connector port="8888" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8445" />
```


## 部署

1. 直接将项目放到webapps目录下即可。

![](assets/Java%20Tomcat/807190e6e1a27a8f9ac1f80540bc64b9_MD5.png)
![](assets/Java%20Tomcat/10823b0fc8c3e12f3bfe5874f6047861_MD5.png)
![](assets/Java%20Tomcat/aa7e3eb69d9b38590704f6bc98d0e1e9_MD5.png)


2. 将项目打成一个war包，再将war包放置到webapps目录下。

![](assets/Java%20Tomcat/a3b71529746e1dbc9a298a134d5a94f4_MD5.png)


## 配置

可以在conf/server.xml文件中进行映射配置。

比如：项目存放的路径为`D:\hello` ，不放在webapps目录中，可以在`<Host>`标签体中配置：

```xml
<Context docBase="D:\hello" path="/hehe" />
```

* `docBase`：项目存放的路径
* `path`：虚拟目录   

![](assets/Java%20Tomcat/53ed1c8b44c674e31773ca65ea62a995_MD5.png)


也可以在`conf\Catalina\localhost`创建任意名称的xml文件。url映射的路径就是xml文件的名称。

在文件中编写：

```xml
<Context docBase="D:\hello" />
```

![](assets/Java%20Tomcat/99ab5f6ed7974b0c2ab419afc3b0ef03_MD5.png)
![](assets/Java%20Tomcat/64487d4b55109dd25e231403138a61d6_MD5.png)


## Maven创建Web项目

Web项目的结构分为：**开发中的项目**和**开发完可以部署的Web项目**，这两种项目的结构是不一样的。

**开发中的项目：**

![](assets/Java%20Tomcat/ca02c1afb6c0a093b3adcd2b8c333255_MD5.png)


需要在pom.xml文件中配置打包方式为war(默认为jar)，这样使用Maven的`package` 命令即可生成war包。

![](assets/Java%20Tomcat/4e3eb778c4f2b80b27fde485cd996172_MD5.png)


使用骨架创建web项目：

![](assets/Java%20Tomcat/4cce2818b283f4b437d5d2b371575dc6_MD5.png)


不使用骨架创建web项目：

![](assets/Java%20Tomcat/32a694898ab6e3194db70d952578049f_MD5.png)

**开发完成部署的Web项目：**

![](assets/Java%20Tomcat/22ce4069de96baf24da250f5e8bf4d2d_MD5.png)


* 开发项目通过执行Maven打包命令`package`，可以获取到部署的Web项目目录
* 编译后的Java字节码文件和resources的资源文件，会被放到WEB-INF下的classes目录下
* pom.xml中依赖坐标对应的jar包，会被放入WEB-INF下的lib目录下



## IDEA集成Tomcat

### 第一种：使用本地的Tomcat

![](assets/Java%20Tomcat/0e9259c4bf65d79484b20ed44995ecdc_MD5.png)




### 第二种：使用Tomcat Maven插件

Maven Tomcat插件目前只有Tomcat7版本，没有更高的版本可以使用，要想修改Tomcat的端口和访问路径，可以直接修改pom.xml。

1. 在pom.xml中添加Tomcat插件

```xml
<build>
    <plugins>
    	<!--Tomcat插件 -->
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
            	<port>80</port><!--访问端口号 -->
                <!--项目访问路径
                    未配置访问路径: http://localhost:80/tomcat-demo2/a.html
                    配置/后访问路径: http://localhost:80/a.html
                    如果配置成 /hello，访问路径会变成 http://localhost:80/hello/a.html
		-->
                <path>/</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```


启动：

![](assets/Java%20Tomcat/19a44aaebaaab0dd963e4f94bbdcf380_MD5.png)


### 可能遇到的问题

#### IDEA输出乱码问题

因为IDEA是使用UTF-8格式的，而Windows使用的是GBK，所以要把刚刚修改的编码重新修改为UTF-8格式的：

找到tomcat安装目录，找到`conf/logging.properties`文件，确保其中的`java.util.logging.ConsoleHandler.encoding = UTF-8`的编码格式为`UTF-8` 。

![](assets/Java%20Tomcat/d34c72116172edc6b8a14e2613d99148_MD5.png)


#### 看不到Run Maven和Debug Maven

使用插件启动时，如果选中项目并右键点击后，看不到Run Maven和Debug Maven，这个时候就需要在IDEA中下载Maven Helper插件，具体的操作方式为: File --> Settings --> Plugins --> Maven Helper --> Install，安装完后按照提示重启IDEA，就可以看到了。

![](assets/Java%20Tomcat/691f64e00d8f4cee53a87a64efdf2cf5_MD5.png)
