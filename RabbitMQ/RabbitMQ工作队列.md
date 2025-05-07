工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。

相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。


## 轮询分发消息

启动一个消息发送线程，两个工作线程，**需要保证一个消息只能被处理一次，不能处理多次**，多个工作线程之前是竞争关系。

### 抽取工具类

```java
public class RabbitMqUtil {
    public static Channel getChannel() throws Exception{
        // 创建一个连接工厂，并设置MQ的相关信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.182.130");
        factory.setUsername("admin");
        factory.setPassword("123123");
        // 创建连接
        Connection connection = factory.newConnection();
        // 获取信道
        return connection.createChannel();
    }
}
```


### 启动两个工作线程

```java
public class Worker01 {

    // 队列的名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        System.out.println("线程02：等待接收消息.........");

        // 1. 接收消息，并交付消息
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            System.out.println(message);
        };
        // 2. 取消消息时的回调
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");
        // 3. 消费者消费消息
        Channel channel = RabbitMqUtil.getChannel();
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}

```


### 启动一个发送线程

```java
public class Task01 {
    // 队列的名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        // 创建队列
        Channel channel = RabbitMqUtil.getChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 发送一个消息
        for (int i = 0; i < 10; i++) {
            String message = "第" + i + "条消息";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完毕：" + message);
        }
    }
}
```


## 消息应答

消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，将丢失正在处理的消息。

为了保证消息在发送过程中不丢失，rabbitmq 引入消息应答机制，消息应答就是：**消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。**



### 自动应答

消息发送后立即被认为已经传送成功，这种模式需要**在高吞吐量和数据传输安全性方面做权衡**，如果消息在接收到之前，消费者出现连接或者 channel 关闭，消息就会丢失了。

另一方面消费者**没有对传递的消息数量进行限制**，有可能接收太多来不及处理的消息，导致消息积压，使内存耗尽，最终被操作系统杀死，这种模式**仅适用在消费者可以高效处理这些消息的情况下使用**。



### 手动应答

手动应答的好处是可以批量应答并且减少网络拥堵。



#### 应答方法

**Channel类：**

`basicAck(long deliveryTag, boolean multiple)`：用于肯定确认，告知RabbitMQ已经处理消息，可以将其丢弃。

* `multiple` ：是否批量应答，如果设置为true，则会将信道中还未应答的消息应答。false只会应答指定的消息。

`basicNack(long deliveryTag, boolean multiple, boolean requeue)`：用于否定确认

`basicReject(long deliveryTag, boolean requeue)`：用于否定确认，不处理该消息了直接拒绝，可以将其丢弃了



#### 消息自动重新入队

如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息
未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

![](assets/RabbitMQ工作队列/10f7e32dd52d55cae66dd145d42253bc_MD5.png)



#### Demo

生产者：

```java
public class Task02 {
    // 队列的名称
    private final static String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        // 创建队列
        Channel channel = RabbitMqUtil.getChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 发送一个消息
        for (int i = 0; i < 10; i++) {
            String message = "第" + i + "条消息";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完毕：" + message);
        }
    }
}
```


两个消费者：

```java
public class Worker02 {

    // 队列的名称
    private final static String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        System.out.println("线程02：等待接收消息.........");

        boolean autoAck = false;
        // 1. 接收消息，并交付消息，手动应答
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            SleepUtils.sleep(1);
            String message = new String(delivery.getBody());
            System.out.println("收到的消息：" + message);
            // 手动应答：
            // 参数1：消息标记的tag
            // 参数2：是否批量应答
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        // 2. 取消消息时的回调
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");
        // 3. 消费者消费消息
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
    }
}
```

```java
public class Worker03 {

    // 队列的名称
    private final static String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        System.out.println("线程03：等待接收消息.........");

        boolean autoAck = false;
        // 1. 接收消息，并交付消息，手动应答
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            SleepUtils.sleep(30);
            String message = new String(delivery.getBody());
            System.out.println("收到的消息：" + message);
            // 手动应答：
            // 参数1：消息标记的tag
            // 参数2：是否批量应答
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        // 2. 取消消息时的回调
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");
        // 3. 消费者消费消息
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
    }
}
```


> 正常情况下消息发送方发送两个消息两个线程分别接收到消息并进行处理，但这里设置的是，一个线程处理的时间为1秒，另一个线程处理的时间为30秒。

> 发送10条，线程1很快处理完消息，而线程2还没有处理，此时把线程2关掉(模拟宕机或异常)，消息会重新发给线程1，被线程1处理，就说明数据不会丢失。

![](assets/RabbitMQ工作队列/8035b9eb6b549ed4c1136e6fb11f8f8b_MD5.png)



## 持久化

上面是如何处理任务不丢失的情况，持久化可以保障当 RabbitMQ 服务停掉以后消息生产者发送过来的消息不丢失。

> 默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。

确保消息不会丢失需要做两件事：**将队列和消息都标记为持久化**



### 队列持久化

之前创建的队列都是非持久化的，rabbitmq 如果重启的话，队列就会被删除掉，如果要队列实现持久化**需要在声明队列的时候把 durable 参数设置为持久化**。

如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新创建一个持久化的队列，不然就会出现错误。

![](assets/RabbitMQ工作队列/561b6a97e1634a0459e5e3a6af818540_MD5.png)



非持久化队列和持久化队列在UI界面的区别：

![](assets/RabbitMQ工作队列/545a53b3878f707cd50a9b1e2264c8cd_MD5.png)

持久化队列后，即使重启 rabbitmq 队列也依然存在。



### 消息持久化

要想让消息实现持久化需要在消息生产者修改代码，`MessageProperties.PERSISTENT_TEXT_PLAIN`添加这个属性。

将消息标记为持久化并不能完全保证不会丢失消息。尽管告诉 RabbitMQ 将消息保存到磁盘，依然存在**当消息刚准备存储在磁盘的时候但是还没有存储完，消息还在缓存的一个间隔点。** 此时并没有真正写入磁盘。持久性保证并不强，对于简单任务队列而言绰绰有余。如果需要更强有力的持久化策略，可以进行发布确认。

```java
public class Task02 {
    // 队列的名称
    private final static String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        // 创建队列
        Channel channel = RabbitMqUtil.getChannel();
        // 队列持久化
        boolean durable = true;
        channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
        // 发送一个消息
        for (int i = 0; i < 10; i++) {
            String message = "第" + i + "条消息";
            
            // 消息持久化
            channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
            // 消息非持久化
            // channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            
            System.out.println("消息发送完毕：" + message);
        }
    }
}
```


## 不公平分发

RabbitMQ分发消息采用的轮询分发，在某种场景下这种策略并不是很好，比方说有两个消费者在处理任务，其中有个消费者1处理任务的速度非常快，而另外一个消费者2处理速度却很慢，这个时候采用轮询分发的话就会导致处理速度快的消费者很大一部分时间处于空闲状态，而处理慢的消费者一直在干活，这种分配方式在这种情况下其实就不太好，但是RabbitMQ并不知道这种情况它依然很公平的进行分发。

为了避免这种情况，可以设置参数`channel.basicQos(1);`

> 只有手动应答的情况下，不公平分发才有效，传参都设置为1才是不公平分发，否则是预取值的模式分发。

```java
public class Worker02 {

    // 队列的名称
    private final static String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        System.out.println("线程02：等待接收消息.........");

        boolean autoAck = false;
        // 1. 接收消息，并交付消息，手动应答
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            SleepUtils.sleep(1);
            String message = new String(delivery.getBody());
            System.out.println("收到的消息：" + message);
            // 手动应答
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        // 2. 取消消息时的回调
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 设置不公平分发
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);

        // 3. 消费者消费消息
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
    }
}
```


如果这个任务还没有处理完或者还没有应答，先不分配任务，然后rabbitmq会把该任务分配给空闲消费者，如果所有的消费者都没有完成任务，队列还在添加新任务，有可能会遇到队列被撑满的情况，就只能添加新的 worker 或者改变其他存储任务的策略。



### 预取值

**预取值定义通道上允许的未确认消息的最大数量。一旦数量达到配置的数量，RabbitMQ将停止在通道上传递更多消息。**

> 例如，假设在通道上有未确认的消息`[5, 6, 7, 8]`，并且通道的预取值为 4 (`channel.basicQos(4);`)，此时RabbitMQ将不会在该通道上再传递任何消息，除非至少有一个未应答的消息被确认。比方说`tag=6`这个消息刚刚被确认，RabbitMQ将会感知到这个情况并再发送一条消息。消息应答和Qos预取值对用户吞吐量有重大影响。

增加预取值可以提高向消费者传递消息的速度。虽然自动应答传输消息速率是最佳的，但已传递尚未处理的消息的数量也会增加，从而增加了消费者的RAM的消耗(随机存取存储器)，**应该小心使用具有无限预处理的自动确认模式或手动确认模式**，消费者消费了大量的消息如果没有确认的话，会导致消费者连接节点的内存消耗变大。

找到合适的预取值是一个反复试验的过程，不同的负载该值取值也不同，100 到 300 范围内的值通常可提供最佳的吞吐量，并且不会给消费者带来太大的风险。

预取值为 1 是最保守的，但吞吐量变得很低，特别是消费者连接延迟很严重或消费者连接等待时间较长的环境中。对于大多数应用来说，稍微高一点的值将是最佳的。



**发送线程：** 发送15条消息

**消费线程：** 处理消息时间1秒，预取值为2

```java
public class Worker02 {

    // 队列的名称
    private final static String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        System.out.println("线程02：等待接收消息.........");

        // 接收并交付消息，手动应答
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            SleepUtils.sleep(1);
            String message = new String(delivery.getBody());
            System.out.println("收到的消息：" + message);
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        // 取消消息时的回调
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 预取值
        int prefetchCount = 2;
        channel.basicQos(prefetchCount);

        // 消费者消费消息
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
    }
}
```


**消费线程：** 处理消息时间5秒，预取值为5

```java
public class Worker03 {

    // 队列的名称
    private final static String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        System.out.println("线程03：等待接收消息.........");

        // 接收并交付消息，手动应答
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            SleepUtils.sleep(5);
            String message = new String(delivery.getBody());
            System.out.println("收到的消息：" + message);
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        // 取消消息时的回调
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 预取值
        int prefetchCount = 5;
        channel.basicQos(prefetchCount);

        // 消费者消费消息
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
    }
}
```


**发送线程：** 发送15条消息

**消费线程：** 处理消息时间1秒，预取值为2

**消费线程：** 处理消息时间5秒，预取值为5

刚发送消息的时候，线程02和线程03进行轮询，直到将线程02的预取值填满，满了之后会将线程03的预取值填满。都满了之后，线程02是空闲的会一直处理消息，当线程03处理完第一条消息后，会填充进来一条消息，然后又满了，又发现线程02是空闲的，又给线程02处理。

![](assets/RabbitMQ工作队列/46d37144e8cc71d94ac5726994314988_MD5.png)

