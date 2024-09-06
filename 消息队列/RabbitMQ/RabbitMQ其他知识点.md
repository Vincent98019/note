


## 幂等性

用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。

用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条。在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等。



**消息重复消费：**

消费者在消费MQ中的消息时，MQ已把消息发送给消费者，消费者在给MQ 返回 ack 时网络中断，故MQ未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。



**解决思路：**

MQ消费者的幂等性的解决一般使用全局ID、唯一标识（比如时间戳）、UUID，订单消费者每次消费消息时用该 id 先判断该消息是否已消费过。



**消费端的幂等性保障：**

在海量订单生成的业务高峰期，生产端有可能会重复发出消息，这时候消费端就要实现幂等性，这就意味着即使收到了一样的消息，消息永远不会被消费多次。

业界主流的幂等性有两种操作：

1. 唯一 ID+指纹码机制，利用数据库主键去重

> 指纹码：一些规则或者时间戳加别的服务给到的唯一信息码，它并不一定是系统生成的，基本都是由业务规则拼接而来，但是一定要保证唯一性，然后判断这个 id 是否存在数据库中，优势就是实现简单就一个拼接，查询判断是否重复；劣势就是在高并发时，如果是单个数据库就会有写入性能瓶颈，也可以采用分库分表提升性能，但不是最推荐的方式。

2. 利用 redis 的原子性去实现

> 利用 redis 执行 setnx 命令，天然具有幂等性。从而实现不重复消费。



## 优先级队列

比如在系统中有一个订单催付的场景，客户在天猫下订单，如果在设定的时间内未付款，那么就会给客户推送一条短信提醒。比如像苹果，小米这样大商家，他们的订单得到优先处理，redis的定时轮询只能用List做一个简简单单的消息队列，并不能实现一个优先级的场景。

订单量大了后采用 RabbitMQ 进行改造和优化，如果发现是大商家的订单给一个相对比较高的优先级，否则就是默认优先级。



### 添加优先级队列

要让队列实现优先级需要做的事情：

1. 队列需要设置为优先级队列
2. 需要设置消息的优先级
3. 消费者需要等待消息已经发送到队列中才去消费，这样才有机会对消息进行排序

![](assets/RabbitMQ其他知识点/a3455f929b0ecc2b1a0dacfc9f52f4d6_MD5.png)



#### 控制台页面添加

![](assets/RabbitMQ其他知识点/f62166c800c3937089e504ed58054438_MD5.png)



#### 代码中队列添加

```java
Map<String, Object> params = new HashMap();
params.put("x-max-priority", 10);
channel.queueDeclare("hello", true, false, false, params);
```


#### 代码中消息添加

```java
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
```


### 案例

消息发布者：

```java
public class Producer {
    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        // 给消息赋予一个 priority 属性
        AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
        for (int i = 1; i <= 10; i++) {
            String message = "info" + i;
            if (i == 5) {
                channel.basicPublish("", QUEUE_NAME, properties, message.getBytes());
            } else {
                channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            }
            System.out.println("发送消息完成:" + message);
        }
    }
}
```


消息接收者：

```java
public class Consumer {
    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        // 设置队列的最大优先级 最大可以设置到 255 官网推荐 1-10 如果设置太高比较吃内存和 CPU
        Map<String, Object> params = new HashMap();
        params.put("x-max-priority", 10);
        channel.queueDeclare(QUEUE_NAME, true, false, false, params);
        System.out.println("消费者启动等待消费..............");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String receivedMessage = new String(delivery.getBody());
            System.out.println("接收到消息:" + receivedMessage);
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, (consumerTag) -> {
            System.out.println("消费者无法消费消息时调用，如队列被删除");
        });
    }
}
```


## 惰性队列

RabbitMQ 从 3.6.0 版本开始引入了惰性队列的概念。

惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即**支持更多的消息存储**。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了。

默认情况下，生产者将消息发送到 RabbitMQ 的时候，队列中的消息会尽可能的存储在内存之中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。当 RabbitMQ 需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会耗费较长的时间，也会阻塞队列的操作，进而无法接收新的消息。

虽然 RabbitMQ 的开发者们一直在升级相关的算法，但是效果始终不太理想，尤其是在消息量特别大的时候。



### 两种模式

**队列具备两种模式：default 和 lazy。** 默认的为default 模式，在3.6.0 之前的版本无需做任何变更。lazy模式即为惰性队列的模式，可以通过调用`channel.queueDeclare`方法的时候在参数中设置，也可以通过Policy的方式设置，如果一个队列同时使用这两种方式设置的话，那么Policy的方式具备更高的优先级。

如果要通过声明的方式改变已有队列的模式的话，那么只能先删除队列，然后再重新声明一个新的。

在队列声明的时候可以通过`x-queue-mode`参数来设置队列的模式，取值为`default`和`lazy`。

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```


### 内存开销对比

![](assets/RabbitMQ其他知识点/81be31461d35d5892d2dc2b1f5591f18_MD5.png)

在发送一百万条消息，每条消息大概占1KB的情况下，普通队列占用内存是1.2GB，而惰性队列仅仅占用1.5MB。

