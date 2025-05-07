
在生产环境中由于一些不明原因，导致 RabbitMQ 重启，在 RabbitMQ 重启期间生产者消息投递失败，导致消息丢失，需要手动处理和恢复。 在这样比较极端的情况，RabbitMQ 集群不可用的时候，无法投递的消息该如何处理呢？


## 发布确认

确认机制方案：

![](assets/RabbitMQ发布确认高级/efa13a2f9533f03db806b570f5e3eee6_MD5.png)



代码架构图：

![](assets/RabbitMQ发布确认高级/8bce1568374137265ec8248319e88031_MD5.png)



### 配置文件

在application.properties文件中加入下面的配置：

```yaml
spring.rabbitmq.publisher-confirm-type=correlated
```

* `NONE`：禁用发布确认模式，是默认值
* `CORRELATED`：发布消息成功到交换器后会触发回调方法
* `SIMPLE`：有两种效果，其一效果和 CORRELATED 值一样会触发回调方法，其二在发布消息成功后使用 rabbitTemplate 调用 waitForConfirms 或 waitForConfirmsOrDie 方法等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑。

> 要注意的是waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker。



### 配置类

```java
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    /**
     * 声明交换机
     */
    @Bean("confirmExchange")
    public DirectExchange confirmExchange() {
        return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }

    /**
     * 声明队列
     * 该队列是确认队列
     */
    @Bean("confirmQueue")
    public Queue confirmQueue() {
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    /**
     * 声明交换机与队列绑定
     */
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue queue,
                                @Qualifier("confirmExchange") DirectExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("key1");
    }
}
```


### 回调接口

```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback {
    /**
     * 交换机不管是否收到消息的一个回调方法
     *
     * @param correlationData 消息相关数据
     * @param ack             交换机是否收到消息
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (ack) {
            log.info("交换机已经收到 id 为：{}的消息", id);
        } else {
            log.info("交换机还未收到 id 为：{}消息，由于原因：{}", id, cause);
        }
    }
}
```


### 消息生产者

```java
@Slf4j
@RestController
@RequestMapping("/confirm")
public class Producer {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private MyCallBack myCallBack;

    /**
     * 依赖注入 rabbitTemplate 之后再设置它的回调对象
     */
    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(myCallBack);
    }

    @GetMapping("sendMessage/{message}")
    public void sendMessage(@PathVariable String message) {
        // 指定消息 id 为 1
        CorrelationData correlationData1 = new CorrelationData("1");
        String routingKey = "key1";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, message, correlationData1);
        log.info("发送消息routingKey={}，内容={}", routingKey, message);

        CorrelationData correlationData2 = new CorrelationData("2");
        routingKey = "key2";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, message, correlationData2);
        log.info("发送消息routingKey={}，内容={}", routingKey, message);
    }
}
```


### 消息消费者

```java
@Slf4j
@Component
public class ConfirmConsumer {

    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    @RabbitListener(queues = CONFIRM_QUEUE_NAME)
    public void receiveMsg(Message message) {
        String msg = new String(message.getBody());
        log.info("接收到队列 confirm.queue 消息:{}", msg);
    }
}
```


### 测试

启动项目，发起一个请求：`http://localhost:8080/confirm/sendMessage/xx1`

发送了两条消息，第一条消息的 RoutingKey 为 "key1"，第二条消息的 RoutingKey 为 "key2"，两条消息都成功被交换机接收，也收到了交换机的确认回调，但消费者只收到了一条消息，因为第二条消息的 RoutingKey 与队列的 BindingKey 不一致，也没有其它队列能接收这个消息，所有第二条消息被直接丢弃了。

![](assets/RabbitMQ发布确认高级/140302a7341232a3c12bac4984216cec_MD5.png)



## 回退消息

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息不可路由，那么消息会被直接丢弃，生产者是不知道消息被丢弃的。通过**设置mandatory参数**可以在当消息传递过程中不可达目的地时将消息返回给生产者。

```java
@Slf4j
@RestController
public class MessageProducer implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * rabbitTemplate 注入之后就设置该值
     */
    @PostConstruct
    private void init() {
        // 设置收到消息的回调交给谁处理
        rabbitTemplate.setConfirmCallback(this);
        // true：交换机无法将消息进行路由时，会将该消息返回给生产者
        // false：如果发现消息无法进行路由，则直接丢弃
        rabbitTemplate.setMandatory(true);
        // 设置回退消息交给谁处理
        rabbitTemplate.setReturnCallback(this);
    }

    @GetMapping("/sendMessage/{message}")
    public void sendMessage(@PathVariable String message) {
        // 让消息绑定一个 id 值
        CorrelationData correlationData1 = new CorrelationData(UUID.randomUUID().toString());
        rabbitTemplate.convertAndSend("confirm.exchange", "key1", message + "key1", correlationData1);
        log.info("发送消息id为:{}内容为{}", correlationData1.getId(), message + "key1");

        CorrelationData correlationData2 = new CorrelationData(UUID.randomUUID().toString());
        rabbitTemplate.convertAndSend("confirm.exchange", "key2", message + "key2", correlationData2);
        log.info("发送消息id为:{}内容为{}", correlationData2.getId(), message + "key2");
    }

    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (ack) {
            log.info("交换机收到消息确认成功，id:{}", id);
        } else {
            log.error("消息id:{}未成功投递到交换机，原因是:{}", id, cause);
        }
    }

    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, 
            String exchange, String routingKey) {
        log.info("消息{}被服务器退回，退回原因:{}，交换机是:{}，路由 key:{}", new String(message.getBody()), replyText,
                exchange, routingKey);
    }
}
```

![](assets/RabbitMQ发布确认高级/424fe82635a2f25cb48f2955c6ece65f_MD5.png)



## 备份交换机

使用 mandatory 参数和回退消息，可以对无法投递消息的感知能力，在生产者的消息无法被投递时发现并处理。但有时候，并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。

而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。而且设置 mandatory 参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑。

如果既不想丢失消息，又不想增加生产者的复杂性，可以为队列设置死信交换机来存储处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。

在 RabbitMQ 中，有一种备份交换机的机制存在，可以很好的应对这个问题。备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout，这样就能把所有消息都投递到与其绑定的队列中，然后在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列。当然，还可以建立一个报警队列，用独立的消费者来进行监测和报警。



![](assets/RabbitMQ发布确认高级/4349bfbcc6456a5222676b6f9d8bdc01_MD5.png)



### 配置类

```java
@Configuration
public class ConfirmConfig {

    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    public static final String BACKUP_EXCHANGE_NAME = "backup.exchange";
    public static final String BACKUP_QUEUE_NAME = "backup.queue";
    public static final String WARNING_QUEUE_NAME = "warning.queue";

    /**
     * 声明备份交换机
     */
    @Bean("backupExchange")
    public FanoutExchange backupExchange() {
        return new FanoutExchange(BACKUP_EXCHANGE_NAME);
    }

    /**
     * 声明警告队列
     */
    @Bean("warningQueue")
    public Queue warningQueue() {
        return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
    }

    /**
     * 声明警告队列和备份交换机绑定
     */
    @Bean
    public Binding warningBinding(@Qualifier("warningQueue") Queue queue,
                                  @Qualifier("backupExchange") FanoutExchange backupExchange) {
        return BindingBuilder.bind(queue).to(backupExchange);
    }

    /**
     * 声明备份队列
     */
    @Bean("backQueue")
    public Queue backQueue() {
        return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }

    /**
     * 声明备份队列和备份交换机绑定
     */
    @Bean
    public Binding backupBinding(@Qualifier("backQueue") Queue queue,
                                 @Qualifier("backupExchange") FanoutExchange backupExchange) {
        return BindingBuilder.bind(queue).to(backupExchange);
    }


    /**
     * 声明交换机
     */
    @Bean("confirmExchange")
    public DirectExchange confirmExchange() {
        ExchangeBuilder exchangeBuilder = ExchangeBuilder
                .directExchange(CONFIRM_EXCHANGE_NAME)
                .durable(true)
                // 设置该交换机的备份交换机
                .withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME);
        return exchangeBuilder.build();
    }

    /**
     * 声明队列
     * 该队列是确认队列
     */
    @Bean("confirmQueue")
    public Queue confirmQueue() {
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    /**
     * 声明交换机与队列绑定
     */
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue queue,
                                @Qualifier("confirmExchange") DirectExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("key1");
    }
}
```


### 消息生产者

和上面的保持不变



### 消息消费者

```java
@Slf4j
@Component
public class ConfirmConsumer {

    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    @RabbitListener(queues = CONFIRM_QUEUE_NAME)
    public void receiveMsg(Message message) {
        String msg = new String(message.getBody());
        log.info("接收到队列 confirm.queue 消息:{}", msg);
    }

    public static final String WARNING_QUEUE_NAME = "warning.queue";

    @RabbitListener(queues = WARNING_QUEUE_NAME)
    public void receiveWarningMsg(Message message) {
        String msg = new String(message.getBody());
        log.error("报警发现不可路由消息：{}", msg);
    }
}
```


### 测试

启动项目，发起一个请求：`http://localhost:8080/sendMessage/aaaa`

![](assets/RabbitMQ发布确认高级/071b46528610958a69d0e4b4629735ae_MD5.png)

mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，经过上面结果显示答案是**备份交换机优先级高**。

