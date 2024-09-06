## 文件上传分析

1. 【客户端】输入流，从硬盘读取文件数据到程序中。
2. 【客户端】输出流，写出文件数据到服务端。
3. 【服务端】输入流，读取文件数据到服务端程序。
4. 【服务端】输出流，写出文件数据到服务器硬盘中。



## 基本实现

### 服务端实现

```java
public class FileUpload_Server {
    public static void main(String[] args) throws IOException {
        System.out.println("服务器 启动..... ");
        // 1. 创建服务端ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        // 2. 建立连接 
        Socket accept = serverSocket.accept();
        // 3. 创建流对象
        // 3.1 获取输入流,读取文件数据
        BufferedInputStream bis = new BufferedInputStream(accept.getInputStream());
        // 3.2 创建输出流,保存到本地 .
        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.jpg"));
        // 4. 读写数据
        byte[] b = new byte[1024 * 8];
        int len;
        while ((len = bis.read(b)) != -1) {
        	bos.write(b, 0, len);
        }
        //5. 关闭 资源
        bos.close();
        bis.close();
        accept.close();
        System.out.println("文件上传已保存");
    }
}
```


### 客户端实现

```java
public class FileUPload_Client {
    public static void main(String[] args) throws IOException {
	// 1.创建流对象
        // 1.1 创建输入流,读取本地文件  
        BufferedInputStream bis  = new BufferedInputStream(new FileInputStream("test.jpg"));
        // 1.2 创建输出流,写到服务端 
        Socket socket = new Socket("localhost", 6666);
        BufferedOutputStream bos = new BufferedOutputStream(socket.getOutputStream());
        //2.写出数据. 
        byte[] b  = new byte[1024 * 8];
        int len ; 
        while ((len = bis.read(b)) != -1) {
	    bos.write(b, 0, len);
            bos.flush();
	}
        System.out.println("文件发送完毕");
        // 3.释放资源
        bos.close(); 
        socket.close();
        bis.close(); 
        System.out.println("文件上传完毕 ");
    }
}
```


## 文件上传优化

### 文件名称写死的问题

服务端保存文件的名称如果写死，最终导致服务器硬盘遇到相同文件名的文件时只会保留一个文件，建议使用系统时间优化，保证文件名称唯一，代码如下：

```java
FileOutputStream fis = new FileOutputStream(System.currentTimeMillis()+".jpg") // 文件名称
BufferedOutputStream bos = new BufferedOutputStream(fis);
// 自定义命名例如：域名+毫秒值+随机数+文件后缀
```


### 循环接收的问题

服务端只保存一个文件就关闭了，之后用户无法再上传，这是不符合实际的，使用循环改进，可以不断的接收不同用户的文件，代码如下：

```java
// 每次接收新的连接,创建一个Socket
while（true）{
    Socket accept = serverSocket.accept();
    // ......
}
```


### 效率问题

服务端在接收大文件时，可能耗费几秒钟的时间，此时不能接收其他用户上传，可以使用多线程技术优化，代码如下：

```java
while(true) {
    Socket accept = serverSocket.accept();
    // accept 交给子线程处理.
    new Thread(() -> {
        // ......
        InputStream bis = accept.getInputStream();
        // ......
    }).start();
}
```


### 最终实现

```java
public class FileUpload_Server {
    public static void main(String[] args) throws IOException {
	System.out.println("服务器 启动..... ");
        // 1. 创建服务端ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        // 2. 循环接收,建立连接
        while (true) {
	    Socket accept = serverSocket.accept();
            /* 
        	3. socket对象交给子线程处理,进行读写操作
        	Runnable接口中,只有一个run方法,使用lambda表达式简化格式
            */
	    new Thread(() -> {
	        try (
                    //3.1 获取输入流对象
                    BufferedInputStream bis = new BufferedInputStream(accept.getInputStream());
                    //3.2 创建输出流对象, 保存到本地 .
                    FileOutputStream fis = new FileOutputStream(System.currentTimeMillis() + ".jpg");
                    BufferedOutputStream bos = new BufferedOutputStream(fis);) {
                    // 3.3 读写数据
                    byte[] b = new byte[1024 * 8];
                    int len;
                    while ((len = bis.read(b)) != -1) {
                        bos.write(b, 0, len);
                    }
                    //4. 关闭 资源
                    bos.close();
                    bis.close();
                    accept.close();
                    System.out.println("文件上传已保存");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```


## 信息回写

前四步与基本文件上传一致
1. 【服务端】获取输出流，回写数据。
2. 【客户端】获取输入流，解析回写数据。

### 服务端实现

```java
public class FileUpload_Server {
    public static void main(String[] args) throws IOException {
        System.out.println("服务器 启动..... ");
        // 1. 创建服务端ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        // 2. 循环接收,建立连接
        while (true) {
	    Socket accept = serverSocket.accept();
            /*
              3. socket对象交给子线程处理,进行读写操作
              Runnable接口中,只有一个run方法,使用lambda表达式简化格式
            */
	    new Thread(() -> {
                try (
	            //3.1 获取输入流对象
                    BufferedInputStream bis = new BufferedInputStream(accept.getInputStream());
                    //3.2 创建输出流对象, 保存到本地 .
                    FileOutputStream fis = new FileOutputStream(System.currentTimeMillis() + ".jpg");
                    BufferedOutputStream bos = new BufferedOutputStream(fis);
                ) {
		    // 3.3 读写数据
                    byte[] b = new byte[1024 * 8];
                    int len;
                    while ((len = bis.read(b)) != -1) {
		         bos.write(b, 0, len);
                    }
                    
                    // 4.=======信息回写===========================
                    System.out.println("back ........");
                    OutputStream out = accept.getOutputStream();
                    out.write("上传成功".getBytes());
                    out.close();
                    //================================
                    //5. 关闭 资源
                    bos.close();
                    bis.close();
                    accept.close();
                    System.out.println("文件上传已保存");
                } catch (IOException e) {
                    e.printStackTrace();
                }
	    }).start();
        }
    }
}
```


### 客户端实现

```java
public class FileUpload_Client {
    public static void main(String[] args) throws IOException {
	// 1.创建流对象
        // 1.1 创建输入流,读取本地文件
        BufferedInputStream bis = new BufferedInputStream(new FileInputStream("test.jpg"));
        // 1.2 创建输出流,写到服务端
        Socket socket = new Socket("localhost", 6666);
        BufferedOutputStream bos = new BufferedOutputStream(socket.getOutputStream());
        //2.写出数据.
        byte[] b  = new byte[1024 * 8 ];
        int len ;
        while (( len  = bis.read(b))!=-1) {
	    bos.write(b, 0, len);
        }
	// 关闭输出流,通知服务端,写出数据完毕
        socket.shutdownOutput();
        System.out.println("文件发送完毕");
        // 3. =====解析回写============
        InputStream in = socket.getInputStream();
        byte[] back = new byte[20];
        in.read(back);
        System.out.println(new String(back));
        in.close();
        // ============================
        // 4.释放资源
        socket.close();
        bis.close();
    }
}
```