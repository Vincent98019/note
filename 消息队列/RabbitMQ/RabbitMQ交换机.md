假设工作队列背后，每个任务都恰好交付给一个消费者(工作进程)。而将消息传达给多个消费者，这种模式称为“发布/订阅”。

> 例：构建一个简单的日志系统。它将由两个程序组成：第一个程序将发出日志消息，第二个程序是消费者。会启动两个消费者，一个消费者接收到消息后把日志存储在磁盘，另外一个消费者接收到消息后把消息打印在屏幕上，事实上第一个程序发出的日志消息将广播给所有消费者。



## Exchanges

RabbitMQ 消息传递模型的核心思想是：生产者生产的消息从不会直接发送到队列。

通常生产者不知道消息传递到了哪些队列中，生产者只能将消息发送到交换机(exchange)，**交换机工作的内容非常简单，接收来自生产者的消息，并将它们推入队列。** 交换机必须确切知道如何处理收到的消息。

交换机的类型决定把消息放到特定队列，还是放到许多队列中，或者丢弃它们。

![](assets/RabbitMQ交换机/d5c6c3f84866006871d83c60114fd536_MD5.png)

**Exchanges 的类型：**

* 直接(direct)
* 主题(topic)
* 标题(headers)
* 扇出(fanout)



## 默认exchange

```java
channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
```

第一个参数是交换机的名称。空字符串表示默认或无名称交换机，消息能路由发送到队列中其实是由routingKey(bindingkey)绑定 key 指定的，如果它存在的话。



## 临时队列

当连接 RabbitMQ 时，需要一个全新的空队列，为此可以创建一个具有随机名称的队列，或者能让服务器随机生成队列名称。一旦断开了消费者连接，队列将被自动删除。

创建临时队列：

```java
String queueName = channel.queueDeclare().getQueue();
```

![](assets/RabbitMQ交换机/3983e1e88d8bebdd3cfbf8f745ef664f_MD5.png)



## 绑定(bindings)

实是 exchange 和 queue 之间的桥梁，比如说下面这张图就是 X 与 Q1 和 Q2 进行了绑定：

![](assets/RabbitMQ交换机/2435e2693cd1adc8597893737ecd976e_MD5.png)



## Fanout

Fanout 类型非常简单。将接收到的所有消息广播到它知道的所有队列中。

系统中默认有些Fanout类型的exchange：

![](assets/RabbitMQ交换机/a1c60742f926bc1b483a6935c6b39ee0_MD5.png)

案例：

消费者：将接收到的消息打印在控制台

```java
public class Worker0601 {

    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机，交换机的名字为logs
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        // 生成一个临时队列，名称随机，当消费者与队列断开连接时，该队列对自动删除
        String queueName = channel.queueDeclare().getQueue();
        // 将交换机和随机队列绑定，routingKey也就是bindingKey为空字符串
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println("等待接收消息，该队列会将收到的消息打印出来....");

        // 接收到消息后的处理逻辑：
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println("收到的消息：" + message);
        };
        // 取消消息时的回调：
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 消费消息
        channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
    }
}
```


消费者：将接收到的消息存储在磁盘

```java
public class Worker0602 {

    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机，交换机的名字为logs
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        // 生成一个临时队列，名称随机，当消费者与队列断开连接时，该队列对自动删除
        String queueName = channel.queueDeclare().getQueue();
        // 将交换机和随机队列绑定，routingKey也就是bindingKey为空字符串
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println("等待接收消息，该队列会将收到的消息写入文件....");

        // 接收到消息后的处理逻辑：
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            File file = new File("D:\\rabbitmq_info.txt");
            BufferedWriter bw = new BufferedWriter(new FileWriter(file, true));
            bw.write(message);
            bw.newLine();
            bw.close();
            System.out.println("数据写入文件成功");
        };
        // 取消消息时的回调：
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 消费消息
        channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
    }
}
```


生产者：发送消息给两个消费者接收

```java
public class Task0601 {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机，交换机的名字为logs
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        Scanner sc = new Scanner(System.in);
        System.out.println("请输入信息：");
        while (sc.hasNext()) {
            String message = sc.nextLine();
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println("生产者发出消息" + message);
        }
    }
}
```


## Direct

对于上面的案例，希望将日志消息写入磁盘的程序仅写入严重错误(errros)，而不存储警告(warning)或信息(info)日志
消息避免浪费磁盘空间。Fanout 交换类型只能进行无意识的广播，所以这里使用 direct 类型来进行替换，它工作方式是，消息只去到它绑定的routingKey 队列中去。

![](assets/RabbitMQ交换机/d25388670e0aa40a46c72d49b2c51f49_MD5.png)

在上面这张图中，可以看到 X 绑定了两个队列，绑定类型是 direct。队列Q1 绑定键为 orange，队列 Q2 绑定键有两个，一个绑定键为 black，另一个绑定键为 green。

在这种绑定情况下，生产者发布消息到 exchange 上，绑定键为 orange 的消息会被发布到队列Q1。绑定键为black和green的消息会被发布到队列 Q2，其他消息类型的消息将被丢弃。



**多重绑定：**

![](assets/RabbitMQ交换机/a82bd14986f8dcf9199ce7a7445419ca_MD5.png)

当然如果 exchange 的绑定类型是direct，但是它绑定的多个队列的 key 如果都相同，在这种情况下虽然绑定类型是 direct 但是它表现的就和 fanout 有点类似了，就跟广播差不多，如上图所示。



案例：

![](assets/RabbitMQ交换机/3ff38919b233059a5dd3684362fb3df7_MD5.png)

消息发送者：

```java
public class Task0701 {
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        Map<String, String>  bindingKeyMap = new HashMap<>();
        bindingKeyMap.put("info","普通 info 信息");
        bindingKeyMap.put("warning","警告 warning 信息");
        bindingKeyMap.put("error","错误 error 信息");
        // debug 没有消费这接收这个消息 所有就丢失了
        bindingKeyMap.put("debug","调试 debug 信息");
        for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {
            String bindingKey = bindingKeyEntry.getKey();
            String message = bindingKeyEntry.getValue();
            channel.basicPublish(EXCHANGE_NAME, bindingKey, null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println("生产者发出消息:" + message);
        }
    }
}
```


消息接收者：

```java
public class Worker0701 {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机，交换机的名字为logs
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        // 声明一个队列
        String queueName = "console";
        channel.queueDeclare(queueName, false, false, false, null);
        // 将交换机和队列绑定
        channel.queueBind(queueName, EXCHANGE_NAME, "info");
        channel.queueBind(queueName, EXCHANGE_NAME, "warning");

        System.out.println("等待接收消息，该队列会将收到的消息打印出来....");

        // 接收到消息后的处理逻辑：
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            message = "接收绑定键：" + delivery.getEnvelope().getRoutingKey() + "，消息：" + message;
            System.out.println(message);
        };
        // 取消消息时的回调：
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 消费消息
        channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
    }
}
```


消息接收者：

```java
public class Worker0702 {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机，交换机的名字为logs
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        // 声明一个队列
        String queueName = "disk";
        channel.queueDeclare(queueName, false, false, false, null);
        // 将交换机和队列绑定
        channel.queueBind(queueName, EXCHANGE_NAME, "error");

        System.out.println("等待接收消息，该队列会将收到的消息写入文件....");

        // 接收到消息后的处理逻辑：
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            message = "接收绑定键：" + delivery.getEnvelope().getRoutingKey() + "，消息：" + message;
            File file = new File("D:\\rabbitmq_info.txt");
            BufferedWriter bw = new BufferedWriter(new FileWriter(file, true));
            bw.write(message);
            bw.newLine();
            bw.close();
            System.out.println("错误日志写入文件成功");
        };
        // 取消消息时的回调：
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 消费消息
        channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
    }
}
```


## Topics

尽管使用direct交换机改进了日志系统，仍然存在局限性，比如想接收的日志类型有info.base 和 info.advantage，某个队列只想 info.base 的消息，那这个时候direct 就办不到了。这个时候就只能使用 topic 类型。

发送到类型是 topic 交换机的消息的 routing\_key 不能随意写，必须满足一定的要求，它**必须是一个单词列表，以点号分隔开**。这些单词可以是任意单词，比如说："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit"这种类型的。当然这个单词列表最多不能超过 255 个字节。

在这个规则列表中，有两个替换符：

* \*(星号)可以代替一个单词
* #(井号)可以替代零个或多个单词



### 匹配案例

下图绑定关系如下：

* Q1-->绑定的是中间带 orange 带 3 个单词的字符串(\*.orange.\*)
* Q2-->绑定的是最后一个单词是 rabbit 的 3 个单词(\*.\*.rabbit)，第一个单词是 lazy 的多个单词(lazy.#)

![](assets/RabbitMQ交换机/00c077f816b7a85c51e3c36b4308ebe9_MD5.png)

上图是一个队列绑定关系图，他们之间数据接收情况：

* quick.orange.rabbit：被队列 Q1Q2 接收到
* lazy.orange.elephant：被队列 Q1Q2 接收到
* quick.orange.fox：被队列 Q1 接收到
* lazy.brown.fox：被队列 Q2 接收到
* lazy.pink.rabbit：虽然满足两个绑定但只被队列 Q2 接收一次
* quick.brown.fox：不匹配任何绑定不会被任何队列接收到会被丢弃
* quick.orange.male.rabbit：是四个单词不匹配任何绑定会被丢弃
* lazy.orange.male.rabbit：是四个单词但匹配 Q2



当队列绑定关系是下列这种情况时需要引起注意

* **当一个队列绑定键是#，这个队列将接收所有数据，像 fanout**
* **如果队列绑定键当中没有#和\*出现，该队列绑定类型就是 direct** 



案例：

消息发送者：

```java
public class Task0801 {
    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

        Map<String, String> bindingKeyMap = new HashMap<>();
        bindingKeyMap.put("quick.orange.rabbit", "被队列 Q1Q2 接收到");
        bindingKeyMap.put("lazy.orange.elephant", "被队列 Q1Q2 接收到");
        bindingKeyMap.put("quick.orange.fox", "被队列 Q1 接收到");
        bindingKeyMap.put("lazy.brown.fox", "被队列 Q2 接收到");
        bindingKeyMap.put("lazy.pink.rabbit", "虽然满足两个绑定但只被队列 Q2 接收一次");
        bindingKeyMap.put("quick.brown.fox", "不匹配任何绑定不会被任何队列接收到会被丢弃");
        bindingKeyMap.put("quick.orange.male.rabbit", "是四个单词不匹配任何绑定会被丢弃");
        bindingKeyMap.put("lazy.orange.male.rabbit", "是四个单词但匹配 Q2");

        for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {
            String bindingKey = bindingKeyEntry.getKey();
            String message = bindingKeyEntry.getValue();
            channel.basicPublish(EXCHANGE_NAME, bindingKey, null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println("生产者发出消息:" + message);
        }
    }
}
```


消息接收者：

```java
public class Worker0801 {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机，交换机的名字为logs
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        // 声明一个队列
        String queueName = "Q1";
        channel.queueDeclare(queueName, false, false, false, null);
        // 将交换机和队列绑定
        channel.queueBind(queueName, EXCHANGE_NAME, "*.orange.*");

        System.out.println("Q1等待接收消息....");

        // 接收到消息后的处理逻辑：
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            message = "接收绑定键：" + delivery.getEnvelope().getRoutingKey() + "，消息：" + message;
            System.out.println(message);
        };
        // 取消消息时的回调：
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 消费消息
        channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
    }
}
```


消息接收者：

```java
public class Worker0802 {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtil.getChannel();
        // 创建一个交换机，交换机的名字为logs
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        // 声明一个队列
        String queueName = "Q2";
        channel.queueDeclare(queueName, false, false, false, null);
        // 将交换机和队列绑定
        channel.queueBind(queueName, EXCHANGE_NAME, "*.*.rabbit");
        channel.queueBind(queueName, EXCHANGE_NAME, "lazy.#");

        System.out.println("Q2等待接收消息....");

        // 接收到消息后的处理逻辑：
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            message = "接收绑定键：" + delivery.getEnvelope().getRoutingKey() + "，消息：" + message;
            System.out.println(message);
        };
        // 取消消息时的回调：
        CancelCallback cancelCallback = (consumerTag) -> System.out.println("消息消费被中断");

        // 消费消息
        channel.basicConsume(queueName, true, deliverCallback, cancelCallback);
    }
}
```

