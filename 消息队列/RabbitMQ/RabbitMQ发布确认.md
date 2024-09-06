生产者可以将信道设置成 confirm 模式，当信道进入 confirm 模式后，所有在该信道上面发布的消息都将会被指派一个唯一的 ID(从 1 开始)，消息被投递到所有匹配的队列之后，broker 就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了。


如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。

confirm 模式的好处是异步的，一旦发布一条消息，生产者就可以在信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者便可以通过回调方法来处理该确认消息，如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息，生产者应用程序同样可以在回调方法中处理该 nack 消息。



## 开启发布确认

发布确认默认没有开启，如果要开启需要调用方法 confirmSelect，每当要使用发布确认，都要在 channel 上调用该方法。

```java
// 创建一个连接工厂，并设置MQ的相关信息
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("192.168.182.130");
factory.setUsername("admin");
factory.setPassword("123123");
// 创建连接
Connection connection = factory.newConnection();
// 获取信道
Channel channel = connection.createChannel();
// 开启发布确认
channel.confirmSelect();
```


## 单个确认发布

是一种同步确认发布的方式，发布一个消息被确认发布后，后续的消息才能继续发布。

`waitForConfirmsOrDie(long)`方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。

这种确认方式有一个最大的缺点，发布速度特别的慢。

因为如果没有确认发布的消息就会阻塞所有后续消息的发布，这种方式最多提供每秒不超过数百条发布消息的吞吐量。对于某些应用程序来说这可能已经足够了。

```java
public class Task05 {
    // 队列的名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        // 创建队列
        Channel channel = RabbitMqUtil.getChannel();
        // 开启发布确认
        channel.confirmSelect();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 计时
        long startTime = System.currentTimeMillis();
        // 发送消息
        for (int i = 1; i <= 10000; i++) {
            String message = "第" + i + "条消息";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            // 服务端返回 false 或超时时间内未返回，生产者可以消息重发
            boolean flag = channel.waitForConfirms();
            if (flag) {
                System.out.println("消息发送完毕：" + message);
            }
        }
        long endTime = System.currentTimeMillis();
        System.out.println("耗时：" + (endTime - startTime) + "毫秒");
    }
}
```

![](assets/RabbitMQ发布确认/51651563e5c11a3653764eb547dad249_MD5.png)



## 批量确认发布

上面的方式非常慢，与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量。

这种方式的缺点就是：当发生故障导致发布出现问题时，不知道是哪个消息出了问题，必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。

这种方案仍然是同步的，一样阻塞消息的发布。

```java
public class Task05 {
    // 队列的名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        // 创建队列
        Channel channel = RabbitMqUtil.getChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 开启发布确认
        channel.confirmSelect();
        //批量确认消息大小
        int batchSize = 100;
        //未确认消息个数
        int outstandingMessageCount = 0;

        // 计时
        long startTime = System.currentTimeMillis();
        // 发送消息
        for (int i = 1; i <= 10000; i++) {
            String message = "第" + i + "条消息";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            outstandingMessageCount++;
            if (outstandingMessageCount == batchSize) {
                channel.waitForConfirms();
                outstandingMessageCount = 0;
            }
            System.out.println("消息发送完毕：" + message);
        }
        //为了确保还有剩余没有确认消息 再次确认
        if (outstandingMessageCount > 0) {
            channel.waitForConfirms();
        }
        long endTime = System.currentTimeMillis();
        System.out.println("耗时：" + (endTime - startTime) + "毫秒");
    }
}
```

![](assets/RabbitMQ发布确认/ba00a6f399e7a19123685823e7a77a20_MD5.png)



## 异步确认发布

异步确认利用回调函数来达到消息可靠性传递，也通过函数回调来保证投递成功。

![](assets/RabbitMQ发布确认/1d9f9aac90078d73ed35dd51b913bf44_MD5.png)

> **处理异步未确认消息：** 最好的解决方案就是把未确认的消息放到一个基于内存的能被发布线程访问的队列，比如用 ConcurrentLinkedQueue 这个队列在 confirm callbacks 与发布线程之间进行消息的传递。



```java
public class Task0501 {
    // 队列的名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 开启发布确认
        channel.confirmSelect();
        /* 线程安全有序的一个哈希表，适用于高并发的情况
         * 1.轻松的将序号与消息进行关联
         * 2.轻松批量删除条目 只要给到序列号
         * 3.支持并发访问
         */
        ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();

        /* 确认收到消息的一个回调
         * 1.消息序列号
         * 2.true 可以确认小于等于当前序列号的消息  false 确认当前序列号消息
         */
        ConfirmCallback ackCallback = (sequenceNumber, multiple) -> {
            if (multiple) {
                // 返回的是小于等于当前序列号的未确认消息 是一个 map
                ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(sequenceNumber, true);
                // 清除该部分未确认消息
                confirmed.clear();
            } else {
                // 只清除当前序列号的消息
                outstandingConfirms.remove(sequenceNumber);
            }
        };

        ConfirmCallback nackCallback = (sequenceNumber, multiple) -> {
            String message = outstandingConfirms.get(sequenceNumber);
            System.out.println("发布的消息" + message + "未被确认，序列号" + sequenceNumber);
        };

        /* 添加一个异步确认的监听器
         * 1.确认收到消息的回调
         * 2.未收到消息的回调
         */
        channel.addConfirmListener(ackCallback, nackCallback);

        // 计时
        long startTime = System.currentTimeMillis();
        // 发送消息
        for (int i = 1; i <= 10000; i++) {
            String message = "第" + i + "条消息";
            /* channel.getNextPublishSeqNo()获取下一个消息的序列号
             * 通过序列号与消息体进行一个关联
             * 全部都是未确认的消息体
             */
            outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完毕：" + message);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("耗时：" + (endTime - startTime) + "毫秒");
    }
}
```

![](assets/RabbitMQ发布确认/9b3b71d2a5e7b26850fb4d1d97b48ca1_MD5.png)



## 发布确认速度对比

* 单独发布消息：同步等待确认，简单，但吞吐量非常有限。
* 批量发布消息：批量同步等待确认，简单，合理的吞吐量，一旦出现问题但很难推断出是那条消息出现了问题。
* 异步处理：最佳性能和资源使用，在出现错误的情况下可以很好地控制，但是实现起来稍微难些。

