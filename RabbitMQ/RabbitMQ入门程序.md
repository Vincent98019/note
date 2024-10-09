
用 Java 编写两个程序。发送单个消息的生产者和接收消息并打印出来的消费者。

在下图中，“ P”是生产者，“ C”是消费者。中间的框是一个队列(RabbitMQ)代表使用者保留的消息缓冲区。

![](assets/RabbitMQ入门程序/d3077e785bce93eeb7ae4f31104d1bce_MD5.png)




## 搭建环境

创建一个空的Maven工程。添加下面的依赖：

```xml
<dependencies>

    <!--rabbitmq 依赖客户端-->
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.8.0</version>
    </dependency>

</dependencies>
```

## 生产者：发消息

```java
public class Producer {

    // 队列的名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        // 1. 创建一个连接工厂，并设置MQ的相关信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.182.130");
        factory.setUsername("admin");
        factory.setPassword("123123");

        // 2. 创建连接
        Connection connection = factory.newConnection();
        // 3. 获取信道
        Channel channel = connection.createChannel();
        /* 4. 创建队列(入门程序，先不做交换机，有默认的交换机)
         *     4.1 队列名称
         *     4.2 队列里面的消息是否持久化 默认消息存储在内存中
         *     4.3 该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
         *     4.4 是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
         *     4.5 其他参数
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        String message = "hello world";
        /* 5. 发送一个消息
         *     5.1 发送到那个交换机
         *     5.2 路由的 key 是哪个
         *     5.3 其他的参数信息
         *     5.4 发送消息的消息体
         */
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println("消息发送完毕");
    }
}
```

## 消费者：接收消息

```java
public class Consumer {

    // 队列的名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        // 1. 创建一个连接工厂，并设置MQ的相关信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.182.130");
        factory.setUsername("admin");
        factory.setPassword("123123");

        // 2. 创建连接
        Connection connection = factory.newConnection();
        // 3. 获取信道
        Channel channel = connection.createChannel();
        System.out.println("等待接收消息.........");

        // 4. 接收消息，并交付消息
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            System.out.println(message);
        };
        // 5. 取消消息时的回调，如在消费的时候队列被删除掉了
        CancelCallback cancelCallback = (consumerTag) 
                -> System.out.println("消息消费被中断");

        /*
         * 6. 消费者消费消息
         *     6.1 消费哪个队列
         *     6.2 消费成功之后是否要自动应答(true:自动应答；false:手动应答)
         *     6.3 消费者成功消费的回调
         *     6.4 消费者取消消费的回调
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```