延时队列内部是有序的，最重要的特性就是延时，延时队列中的元素是希望在指定时间到了以后或之前取出和处理，简单来说，延时队列就是用来存放需要在指定时间被处理的元素的队列。

## 使用场景

1. 订单在十分钟之内未支付则自动取消
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
3. 用户注册成功后，如果三天内没有登陆则进行短信提醒。
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议。

这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务，如：发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；使用定时任务，一直轮询数据，每秒查一次，取出需要被处理的数据，然后处理，如果数据量比较少，可以这样做，但对于数据量比较大，并且时效性较强的场景，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万级别，对这么庞大的数据量使用轮询的方式是不可取的，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下。

![](assets/RabbitMQ延时队列/b1498efa0b969ab58cbc4fdfcc3f8136_MD5.png)



## TTL

TTL 是 RabbitMQ 中一个消息或者队列的属性，表明一条消息或者该队列中的所有消息的最大存活时间，单位是毫秒。

如果一条消息设置了 TTL 属性或者进入了设置TTL 属性的队列，这条消息如果在TTL设置的时间内没有被消费，则会成为"死信"。如果同时配置了队列的TTL和消息的TTL，会使用较小的值，有两种方式设置 TTL。



### 队列设置TTL

在创建队列的时候设置队列的`x-message-ttl`属性，使用SpringBoot整合设置：

```java
/**
 * 声明 A 队列，并设置TTL时间为 10 秒
 */
@Bean("aQueue")
public Queue aQueue() {
    Map<String, Object> args = new HashMap<>(3);
    // 声明队列的 TTL
    args.put("x-message-ttl", 10000);
    return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
}
```


创建两个队列 QA 和 QB，两者队列 TTL 分别设置为 10S 和 40S，然后在创建一个交换机 X 和死信交换机 Y，它们的类型都是direct，创建一个死信队列 QD，它们的绑定关系如下：

![](assets/RabbitMQ延时队列/fc263d4e95cd3508d320216c0eb5622f_MD5.png)



**创建配置文件类：**

```java
import org.springframework.amqp.core.*;

@Configuration
public class TtlQueueConfig {

    public static final String X_EXCHANGE = "X";
    public static final String QUEUE_A = "A";
    public static final String QUEUE_B = "B";
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    public static final String DEAD_LETTER_QUEUE = "D";

    /**
     * 声明 X 交换机
     */
    @Bean("xExchange")
    public DirectExchange xExchange() {
        return new DirectExchange(X_EXCHANGE);
    }

    /**
     * 声明 Y 死信交换机
     */
    @Bean("yExchange")
    public DirectExchange yExchange() {
        return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
    }

    /**
     * 声明 A 队列，绑定死信交换机，并设置TTL时间为 10 秒
     */
    @Bean("aQueue")
    public Queue aQueue() {
        Map<String, Object> args = new HashMap<>(3);
        // 声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        // 声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        // 声明队列的 TTL
        args.put("x-message-ttl", 10000);
        return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
    }

    /**
     * 声明 A 队列与 X 交换机绑定
     */
    @Bean
    public Binding aQueueBindingX(@Qualifier("aQueue") Queue aQueue,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(aQueue).to(xExchange).with("XA");
    }

    /**
     * 声明 B 队列，绑定死信交换机，并设置TTL时间为 40 秒
     */
    @Bean("bQueue")
    public Queue bQueue() {
        Map<String, Object> args = new HashMap<>(3);
        //声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        //声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        //声明队列的 TTL
        args.put("x-message-ttl", 40000);
        return QueueBuilder.durable(QUEUE_B).withArguments(args).build();
    }

    /**
     * 声明队列A与交换机X绑定
     */
    @Bean
    public Binding bQueueBindingX(@Qualifier("bQueue") Queue bQueue,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(bQueue).to(xExchange).with("XB");
    }

    /**
     * 声明死信队列 D
     */
    @Bean("dQueue")
    public Queue dQueue() {
        return new Queue(DEAD_LETTER_QUEUE);
    }

    /**
     * 声明死信队列 D 与死信交换机 Y 绑定
     */
    @Bean
    public Binding dDeadLetterQueueBindingY(@Qualifier("dQueue") Queue queueD,
                                            @Qualifier("yExchange") DirectExchange yExchange) {
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }
}
```


**消息生产者：**

```java
@Slf4j
@RequestMapping("/ttl")
@RestController
public class SendMsgController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("sendMsg/{message}")
    public void sendMsg(@PathVariable String message) {
        log.info("当前时间：{}，发送一条信息给两个 TTL 队列：{}", LocalDateTime.now(), message);
        rabbitTemplate.convertAndSend("X", "XA", "消息来自 ttl 为 10S 的队列: " + message);
        rabbitTemplate.convertAndSend("X", "XB", "消息来自 ttl 为 40S 的队列: " + message);
    }
}
```


**消息消费者：**

```java
@Slf4j
@Component
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = TtlQueueConfig.DEAD_LETTER_QUEUE)
    public void receiveD(Message message, Channel channel) {
        String msg = new String(message.getBody());
        log.info("当前时间：{}，收到死信队列信息{}", LocalDateTime.now(), msg);
    }
}
```


**测试：**

启动项目，发起一个请求 `http://localhost:8080/ttl/sendMsg/124123` 

![](assets/RabbitMQ延时队列/ca258e0bbd243a94becf2bcc0c8c7b78_MD5.png)

第一条消息在 10S 后变成了死信消息，然后被消费者消费掉，第二条消息在 40S 之后变成了死信消息，然后被消费掉，这样一个延时队列就打造完成了。



### 消息设置TTL

对每条消息设置TTL，使用SpringBoot整合设置：

```java
@GetMapping("sendMsg/{message}")
public void sendMsg(@PathVariable String message) {
    log.info("当前时间：{}，发送一条信息给两个 TTL 队列：{}", LocalDateTime.now(), message);
    rabbitTemplate.convertAndSend("X", "XA", "消息来自 ttl 为 10S 的队列: " + message);
    rabbitTemplate.convertAndSend("X", "XB", "消息来自 ttl 为 40S 的队列: " + message,
            // 消息来自设置了40S的队列，但消息的TTL是2S，如果都设置了，则以少的为准
            correlationData -> {
                correlationData.getMessageProperties().setExpiration("2000");
                return correlationData;
            });
}
```


在上面的案例中新增一个C队列，绑定关系如下，该队列不设置TTL 时间：

![](assets/RabbitMQ延时队列/0e4cddcdfb9dd918f7782b6555cb7d14_MD5.png)

**创建一个新的配置类：**

```java
@Component
public class MsgQueueConfig {

    public static final String QUEUE_C = "C";
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";

    /**
     * 声明 C 队列，绑定死信交换机
     */
    @Bean("cQueue")
    public Queue cQueue(){
        Map<String, Object> args = new HashMap<>(3);
        //声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        //声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        //没有声明 TTL 属性
        return QueueBuilder.durable(QUEUE_C).withArguments(args).build();
    }

    /**
     * 声明队列C与交换机X绑定
     */
    @Bean
    public Binding cQueueBindingX(@Qualifier("cQueue") Queue bQueue,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(bQueue).to(xExchange).with("XC");
    }
}
```


**消息生产者：**

```java
@GetMapping("sendMsg/{message}")
public void sendMsg(@PathVariable String message) {
    log.info("当前时间：{}，发送一条信息给两个 TTL 队列：{}", LocalDateTime.now(), message);
    rabbitTemplate.convertAndSend("X", "XA", "消息来自 ttl 为 10S 的队列: " + message);
    rabbitTemplate.convertAndSend("X", "XB", "消息来自 ttl 为 40S 的队列: " + message,
            // 消息来自设置了40S的队列，但消息的TTL是2S，如果都设置了，则以少的为准
            correlationData -> {
                correlationData.getMessageProperties().setExpiration("2000");
                return correlationData;
            });
}

@GetMapping("sendExpirationMsg/{message}/{ttlTime}")
public void sendMsg(@PathVariable String message, @PathVariable String ttlTime) {
    rabbitTemplate.convertAndSend("X", "XC", message, correlationData -> {
        correlationData.getMessageProperties().setExpiration(ttlTime);
        return correlationData;
    });
    log.info("当前时间：{}，发送一条时长{}毫秒 TTL 信息给队列 C: {}", LocalDateTime.now(), ttlTime, message);
}
```


**消息消费者：**

和上面的保持一致



**测试：**

启动项目，发起两个请求

*  `http://localhost:8080/ttl/sendExpirationMsg/你好2/20000` 
*  `http://localhost:8080/ttl/sendExpirationMsg/你好/2000` 

![](assets/RabbitMQ延时队列/60adbf5adb557b8273c73e6d64116132_MD5.png)

执行成功。



### 两者的区别

设置队列的 TTL 属性，一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)。

设置消息的 TTL 属性，消息过期，不一定会被马上丢弃，消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间；如果不设置 TTL，表示消息永远不会过期，如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。



## 延时队列插件

上面消息TTL设置时，看起来没什么问题，但如果使用在消息属性上设置 TTL 的方式，消息可能并不会按时“死亡“，因为 RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列，如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行。

使用延时队列插件解决：



### 安装

官网下载地址：[https://www.rabbitmq.com/community-plugins.html](https://www.rabbitmq.com/community-plugins.html)

找到 `rabbitmq_delayed_message_exchange` 插件，点击`Releases`，进入github页面点击下载ez格式的文件，并将该文件上传到 RabbitMQ 的插件目录`/usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins`。

> 下载合适的版本，mq是3.8.8，插件就下载支持该版本的，否则执行下面命令会报错。

![](assets/RabbitMQ延时队列/6ee95d2952bb06f112d8e782743001c3_MD5.png)

执行命令：

```bash
 rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

> 如果执行了没反应，可以进入rabbitmq的安装目录执行，然后重启试试。

![](assets/RabbitMQ延时队列/72a2a6235c98fd14771cbdcd8ecce6f3_MD5.png)

![](assets/RabbitMQ延时队列/a80424f49552a98863212b7ee47065cd_MD5.png)



新增一个队列`delayed.queue`，一个交换机`delayed.exchange`，绑定关系如下：

![](assets/RabbitMQ延时队列/e167c67f5e6031a32fefc089aaf8f4fe_MD5.png)



### 配置文件类

```java
@Configuration
public class DelayedQueueConfig {
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

    /**
     * 声明 delayed.queue 队列
     */
    @Bean
    public Queue delayedQueue() {
        return new Queue(DELAYED_QUEUE_NAME);
    }

    /**
     * 声明 delayed.exchange 交换机
     * 交换机使用新类型的交换机：x-delayed-message
     */
    @Bean
    public CustomExchange delayedExchange() {
        Map<String, Object> args = new HashMap<>();
        // 自定义交换机的类型
        args.put("x-delayed-type", "direct");
        return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, args);
    }

    /**
     * 声明队列与交换机绑定
     */
    @Bean
    public Binding bindingDelayedQueue(@Qualifier("delayedQueue") Queue queue,
                                       @Qualifier("delayedExchange") CustomExchange delayedExchange) {
        return BindingBuilder.bind(queue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
    }
}
```


### 消息生产者

```java
public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

@GetMapping("sendDelayMsg/{message}/{delayTime}")
public void sendMsg(@PathVariable String message, @PathVariable Integer delayTime) {
    rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME, DELAYED_ROUTING_KEY, message, correlationData -> {
                correlationData.getMessageProperties().setDelay(delayTime);
                return correlationData;
            });
    log.info("当前时间：{}，发送一条延时 {} 毫秒的信息给队列 delayed.queue：{}", 
        LocalDateTime.now(), delayTime, message);
}
```


### 消息消费者

```java
@RabbitListener(queues = DelayedQueueConfig.DELAYED_QUEUE_NAME)
public void receiveDelayedQueue(Message message) {
    String msg = new String(message.getBody());
    log.info("当前时间：{}，收到延时队列的消息：{}", LocalDateTime.now(), msg);
}
```


### 测试

启动项目，发起两个请求

http://localhost:8080/ttl/sendDelayMsg/{message}/{delayTime}

*  `http://localhost:8080/ttl/sendDelayMsg/第1条消息/20000` 
*  `http://localhost:8080/ttl/sendDelayMsg/第2条消息/2000` 

![](assets/RabbitMQ延时队列/d1de2d6be451e8a1f3a14c0e0a01c5dc_MD5.png)

第二个消息被先消费掉，符合预期。



## 总结

延时队列在需要延时处理的场景下非常有用，使用 RabbitMQ 来实现延时队列可以很好的利用其特性，如：消息可靠发送、消息可靠投递、死信队列来保障消息至少被消费一次以及未被正确处理的消息不会被丢弃。

另外，通过 RabbitMQ 集群的特性，可以很好的解决单点故障问题，不会因为单个节点挂掉导致延时队列不可用或者消息丢失。

当然，延时队列还有很多其它选择，比如利用 Java 的 DelayQueue，利用 Redis 的 zset，利用 Quartz或者利用 kafka 的时间轮，这些方式各有特点，看需要适用的场景。