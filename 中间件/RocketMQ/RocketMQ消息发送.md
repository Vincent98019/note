#RocketMQ 

导入MQ客户端依赖：

```xml
<dependency>  
    <groupId>org.apache.rocketmq</groupId>  
    <artifactId>rocketmq-client</artifactId>  
    <version>4.9.4</version>  
</dependency>
```

消息发送者步骤分析：
1. 创建消息生产者producer，并制定生产者组名  
2. 指定Nameserver地址  
3. 启动producer  
4. 创建消息对象，指定主题Topic、Tag和消息体  
5. 发送消息  
6. 关闭生产者producer

消息消费者步骤分析：
1. 创建消费者Consumer，制定消费者组名  
2. 指定Nameserver地址  
3. 订阅主题Topic和Tag  
4. 设置回调函数，处理消息  
5. 启动消费者consumer

## 消息发送

### 发送同步消息

这种可靠性同步地发送方式使用的比较广泛，比如：重要的消息通知，短信通知。

```java
public static void main(String[] args) throws Exception {  
    //1.创建消息生产者producer，并制定生产者组名  
    DefaultMQProducer producer = new DefaultMQProducer("group1");  
    //2.指定Nameserver地址  
    producer.setNamesrvAddr("192.168.197.128:9876;192.168.197.128:9876");  
    //3.启动producer  
    producer.start();  
    for (int i = 0; i < 10; i++) {  
        //4.创建消息对象，指定主题Topic、Tag和消息体  
        /*  
         * 参数一：消息主题Topic  
         * 参数二：消息Tag  
         * 参数三：消息内容  
         */  
        Message msg = new Message("springboot-mq", "Tag1", ("Hello World" + i).getBytes());  
        //5.发送消息  
        SendResult result = producer.send(msg);  
        //发送状态  
        SendStatus status = result.getSendStatus();  
        System.out.println("发送结果:" + result);  
        //线程睡1秒  
        TimeUnit.SECONDS.sleep(1);  
    }  
    //6.关闭生产者producer  
    producer.shutdown();  
}
```


### 发送异步消息

异步消息通常用在对响应时间敏感的业务场景，即发送端不能容忍长时间地等待Broker的响应。

```java
public static void main(String[] args) throws Exception {  
    //1.创建消息生产者producer，并制定生产者组名  
    DefaultMQProducer producer = new DefaultMQProducer("group1");  
    //2.指定Nameserver地址  
    producer.setNamesrvAddr("192.168.197.128:9876;192.168.197.128:9876");  
    //3.启动producer  
    producer.start();  
    for (int i = 0; i < 10; i++) {  
        //4.创建消息对象，指定主题Topic、Tag和消息体  
        /*  
         * 参数一：消息主题Topic  
         * 参数二：消息Tag  
         * 参数三：消息内容  
         */  
        Message msg = new Message("base", "Tag2", ("Hello World" + i).getBytes());  
        //5.发送异步消息  
        producer.send(msg, new SendCallback() {  
            /**  
             * 发送成功回调函数  
             */  
            public void onSuccess(SendResult sendResult) {  
                System.out.println("发送结果：" + sendResult);  
            }  
            /**  
             * 发送失败回调函数  
             */  
            public void onException(Throwable e) {  
                System.out.println("发送异常：" + e);  
            }  
        });  
        //线程睡1秒  
        TimeUnit.SECONDS.sleep(1);  
    }  
    producer.shutdown();  
}
```


### 发送单向消息

这种方式主要用在不特别关心发送结果的场景，例如日志发送。

```java
public static void main(String[] args) throws Exception {  
    //1.创建消息生产者producer，并制定生产者组名  
    DefaultMQProducer producer = new DefaultMQProducer("group1");  
    //2.指定Nameserver地址  
    producer.setNamesrvAddr("192.168.197.128:9876;192.168.197.128:9876");  
    //3.启动producer  
    producer.start();  
    for (int i = 0; i < 3; i++) {  
        //4.创建消息对象，指定主题Topic、Tag和消息体  
        /*  
         * 参数一：消息主题Topic  
         * 参数二：消息Tag  
         * 参数三：消息内容  
         */  
        Message msg = new Message("base", "Tag3", ("Hello World，单向消息" + i).getBytes());  
        //5.发送单向消息  
        producer.sendOneway(msg);  
        //线程睡1秒  
        TimeUnit.SECONDS.sleep(5);  
    }  
    //6.关闭生产者producer  
    producer.shutdown();  
}
```


## 消费消息

### 负载均衡模式

消费者采用负载均衡方式消费消息，多个消费者共同消费队列消息，每个消费者处理的消息不同。

