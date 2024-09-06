

TCP通信能实现两台计算机之间的数据交互，通信的两端，要严格区分为**客户端（Client）**与**服务端（Server）**。



**两端通信时步骤：**

1. 服务端程序需要先启动，等待客户端的连接。
2. 客户端主动连接服务器端，连接成功才能通信。服务端不可以主动连接客户端。



**在Java中，提供了两个类用于实现TCP通信程序：**

* 客户端：`java.net.Socket`类表示。创建`Socket`对象，向服务端发出连接请求，服务端响应请求，两者建立连接开始通信。
* 服务端：`java.net.ServerSocket`类表示。创建`ServerSocket`对象，相当于开启一个服务，并等待客户端的连接。



## InetAddress类

这个类代表一个互联网协议（IP）地址。 

**常用方法**

* `static InetAddress getLocalHost()`：返回本地主机的地址。
* `static InetAddress getByName(String host)`：根据主机名获取主机信息。
* `String getHostAddress()`：根据InetAddress对象获取ip地址。
* `String getHostName()`：根据InetAddress对象获取主机名。

```java
public static void main(String[] args) throws UnknownHostException {
    // 获取本机的InetAddress对象
    InetAddress localHost = InetAddress.getLocalHost();
    System.out.println(localHost);
    // 根据本机的主机名获取InetAddress对象
    InetAddress host = InetAddress.getByName("DESKTOP-3G22U7M");
    System.out.println(host);
    // 根据域名获取InetAddress对象
    InetAddress host2 = InetAddress.getByName("www.baidu.com");
    System.out.println(host2);
    // 根据InetAddress对象获取ip地址
    String hostAddress = host2.getHostAddress();
    System.out.println(hostAddress);
    // 根据InetAddress对象获取主机名
    String hostName = host2.getHostName();
    System.out.println(hostName);
}
```




## Socket类

`java.net.Socket`类实现客户端套接字（也可以就叫“套接字”）。该类实现客户端**套接字**，套接字指的是两台设备之间通讯的端点。

> Socket允许程序把网络连接当成一个流，数据在两个Socket间通过IO传输。


**构造方法**

* `public Socket(String host, int port)`：创建套接字对象并将其连接到指定主机上的指定端口号。如果指定的host是null ，则相当于指定地址为回送地址。
  * `String host`：服务器主机的名称/服务器的IP地址。
  * `int port`：服务器的端口号。



```java
Socket client = new Socket("127.0.0.1", 6666);
```


> 回送地址(**127.x.x.x**) 是本机回送地址(Loopback Address)，主要用于网络软件测试以及本地机进程间通信，无论什么程序，一旦使用回送地址发送数据，立即返回，不进行任何网络传输。



**常用方法**

* `public InputStream getInputStream()`：返回此套接字的输入流。

> 如果此Scoket具有相关联的通道，则生成的InputStream 的所有操作也关联该通道。
> **关闭生成的InputStream也将关闭相关的Socket。**


* `public OutputStream getOutputStream()`： 返回此套接字的输出流。

> 如果此Scoket具有相关联的通道，则生成的OutputStream 的所有操作也关联该通道。
> **关闭生成的OutputStream也将关闭相关的Socket。**


* `public void close()`：关闭此套接字。
  一旦一个socket被关闭，它不可再使用。

> **关闭此socket也将关闭相关的InputStream和OutputStream 。**

* `public void shutdownOutput()`： 禁用此套接字的输出流。 
  任何先前写出的数据将被发送，随后终止输出流。


## ServerSocket类

`java.net.ServerSocket`类：这个类实现了服务器套接字，该对象等待通过网络的请求。

**构造方法**

* `public ServerSocket(int port)`：使用该构造方法在创建ServerSocket对象时，就可以将其绑定到一个指定的端口号上，参数port就是端口号。

```java
ServerSocket server = new ServerSocket(6666);
```

**常用方法**

* `public Socket accept()`：侦听并接受连接，返回一个新的Socket对象，用于和客户端实现通信。该方法会一直阻塞直到建立连接。



## TCP网络程序

### TCP通信分析

1. 【服务端】启动，创建ServerSocket对象，等待连接。
2. 【客户端】启动，创建Socket对象，请求连接。
3. 【服务端】接收连接，调用accept方法，并返回一个Socket对象。
4. 【客户端】Socket对象，获取OutputStream，向服务端写出数据。
5. 【服务端】Scoket对象，获取InputStream，读取客户端发送的数据。

到此，客户端向服务端发送数据成功。

6. 【服务端】Socket对象，获取OutputStream，向客户端回写数据。
7. 【客户端】Scoket对象，获取InputStream，解析回写数据。
8. 【客户端】释放资源，断开连接。

自此，服务端向客户端回写数据。



### 服务端实现

```java
public class ServerTCP {
    public static void main(String[] args) throws IOException {
	System.out.println("服务端启动 , 等待连接 .... ");
        // 1.创建 ServerSocket对象，绑定端口，开始等待连接
        ServerSocket ss = new ServerSocket(6666);
        // 2.接收连接 accept 方法, 返回 socket 对象.
        Socket server = ss.accept();
        // 3.通过socket 获取输入流
        InputStream is = server.getInputStream();
        // 4.一次性读取数据
        // 4.1 创建字节数组
        byte[] b = new byte[1024];
        // 4.2 据读取到字节数组中.
        int len = is.read(b)；
        // 4.3 解析数组,打印字符串信息
        String msg = new String(b, 0, len);
        System.out.println(msg);
        // =================回写数据=======================
        // 5. 通过 socket 获取输出流
        OutputStream out = server.getOutputStream();
        // 6. 回写数据
        out.write("我很好,谢谢你".getBytes());
        // 7.关闭资源.
        out.close();
        is.close();
        server.close();
    }
}
```


### 客户端实现

```java
public class ClientTCP {
    public static void main(String[] args) throws Exception {
        System.out.println("客户端 发送数据");
        // 1.创建 Socket ( ip , port ) , 确定连接到哪里.
        Socket client = new Socket("localhost", 6666);
        // 2.通过Scoket,获取输出流对象 
        OutputStream os = client.getOutputStream();
        // 3.写出数据.
        os.write("你好么? tcp ,我来了".getBytes());
        // ==============解析回写=========================
        // 4. 通过Scoket,获取 输入流对象
        InputStream in = client.getInputStream();
        // 5. 读取数据数据
        byte[] b = new byte[100];
        int len = in.read(b);
        System.out.println(new String(b, 0, len));
	// 6. 关闭资源 .
   	in.close();
        os.close();
        client.close();
    }
}
```