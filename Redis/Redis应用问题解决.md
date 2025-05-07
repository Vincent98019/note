
## 缓存穿透

key对应的数据在数据源不存在，每次针对此key的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

![](assets/Redis应用问题解决/23385d4a2185e5b718bea88493275977_MD5.png)


一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。



**解决方案：**

1. **对空值缓存：** 如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟。
2. **设置可访问的名单（白名单）：** 使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量，每次访问和bitmap里面的id进行比较，如果访问id不在bitmaps里面，进行拦截，不允许访问。
3. **采用布隆过滤器**： 将所有可能存在的数据哈希到一个足够大的bitmaps中，一个一定不存在的数据会被这个bitmaps拦截掉，从而避免了对底层存储系统的查询压力。

> 布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

4. **进行实时监控：**当发现Redis的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务。



## 缓存击穿

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

![](assets/Redis应用问题解决/1644565128eb22ade8548a1ed8681c48_MD5.png)


key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。



**解决方案：**

1. 预先设置热门数据：在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长。
2. 实时调整：现场监控哪些数据热门，实时调整key的过期时长。
3. 使用锁：
   1. 就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db
   2. 先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key
   3. 当操作返回成功时，再进行load db的操作，并回设缓存，最后删除mutex key
   4. 当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法


![](assets/Redis应用问题解决/75f3665f4b04c7026624d96335ea4f21_MD5.png)



## 缓存雪崩

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

> 缓存雪崩与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key。



正常访问：

![在这里插入图片描述](assets/Redis应用问题解决/967891592bad25c1e0ae744a8e239e5b_MD5.png)




缓存失效瞬间：

![在这里插入图片描述](assets/Redis应用问题解决/247abb6af7e9f9788b313b87939b948d_MD5.png)




**解决方案：**

缓存失效时的雪崩效应对底层系统的冲击非常可怕。

1. **构建多级缓存架构：** nginx缓存 + redis缓存 + 其他缓存（ehcache等）
2. **使用锁或队列：** 用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。**不适用高并发情况。**
3. **设置过期标志更新缓存：** 记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。
4. **将缓存失效时间分散开：** 比如可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

 

## 分布式锁

随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的Java API并不能提供分布式锁的能力。为了解决这个问题就需要一种跨JVM的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题。



**分布式锁主流的实现方案：**

1. 基于数据库实现分布式锁
2. 基于缓存（Redis等）
3. 基于Zookeeper



**每一种分布式锁解决方案都有各自的优缺点：**

1. 性能：redis最高
2. 可靠性：zookeeper最高



### 使用redis实现分布式锁

redis命令

```bash
set sku:1:info "OK" NX PX 10000
```

* EX second：设置键的过期时间为 second 秒。 `set key value ex second` 效果等同于 `setex key second value` 。
* PX millisecond ：设置键的过期时间为 millisecond 毫秒。 `set key value px millisecond` 效果等同于 `psetex key millisecond value` 。
* NX ：只在键不存在时，才对键进行设置操作。 set key value nx 效果等同于 `setnx key value` 。
* XX ：只在键已经存在时，才对键进行设置操作。

![](assets/Redis应用问题解决/54d5c6b7cb4830b2bf9b653860f0849d_MD5.png)


1. 多个客户端同时获取锁（setnx）
2. 获取成功，执行业务逻辑(从db获取数据，放入缓存)，执行完成释放锁（del）
3. 其他客户端等待重试



测试代码：

```java
@GetMapping("/testLock")
public void testLock(){
    //1获取锁，setne
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "111");
    //2获取锁成功、查询num的值
    if(lock){
        Object value = redisTemplate.opsForValue().get("num");
        //2.1判断num为空return
        if(StringUtils.isEmpty(value)){
            return;
        }
        //2.2有值就转成成int
        int num = Integer.parseInt(value+"");
        //2.3把redis的num加1
        redisTemplate.opsForValue().set("num", ++num);
        //2.4释放锁，del
        redisTemplate.delete("lock");

    }else{
        //3获取锁失败、每隔0.1秒再获取
        try {
            Thread.sleep(100);
            testLock();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```


如果是集群环境的话，需要在yml中配置集群信息：

```yaml
spring:
  redis:
    host: 192.168.182.133
    port: 6379
    database: 0
    timeout: 1800000
    lettuce:
      pool:
        max-active: 20
        max-wait: -1
        max-idle: 5
        min-idle: 0
    cluster:
      nodes: 192.168.182.133:6379,192.168.182.133:6380,192.168.182.133:6381,192.168.182.133:6382,192.168.182.133:6383,192.168.182.133:6384
```


重启，服务集群，通过网关压力测试：

`ab -n 1000 -c 100 http://192.168.0.123:8080/testLock`

在这之前，在Redis集群中设置num的值为0：

`set num 0`



测试结果：

刚好是1000

![](assets/Redis应用问题解决/8c6d11f786831e0f2c098acd1c57da8f_MD5.png)



### 优化 设置锁过期时间

setnx刚好获取到锁，业务逻辑出现异常，导致锁无法释放。

解决：设置过期时间，自动释放锁。

设置过期时间有两种方式：

1. 通过expire设置过期时间（缺乏原子性：如果在setnx和expire之间出现异常，锁也无法释放）
2. 在set时指定过期时间（推荐）

![](assets/Redis应用问题解决/625e44ad564b750a40033d51276a3002_MD5.png)



```java
@GetMapping("testLock")
public void testLock() {
    // 1. 获取锁，setne
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "111", 3, TimeUnit.SECONDS);
    // 其他代码....
}
```


再次测试：

num已经变为了2000，也没有什么问题

![](assets/Redis应用问题解决/33082ea0c8a943e533d3a5785314902f_MD5.png)



### 优化 UUID防误删

可能会释放其他服务器的锁。

场景：如果业务逻辑的执行时间是7s。执行流程如下：

1. index1业务逻辑没执行完，3秒后锁被自动释放。
2. index2获取到锁，执行业务逻辑，3秒后锁被自动释放。
3. index3获取到锁，执行业务逻辑。
4. index1业务逻辑执行完成，开始调用del释放锁，这时释放的是index3的锁，导致index3的业务只执行1s就被别人释放

最终等于没锁的情况。

解决：setnx获取锁时，设置一个指定的唯一值（例如：uuid），释放前获取这个值，判断是否自己的锁。

![](assets/Redis应用问题解决/2398fb848192bf0188059d171db61b14_MD5.png)


```java
@GetMapping("testLock")
public void testLock() {
    // 1. 获取锁，setne
    String uuid = UUID.randomUUID().toString();
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 3, TimeUnit.SECONDS);
    if (Boolean.TRUE.equals(lock)) {
        // 2. 获取锁成功，查询num的值
        Object value = redisTemplate.opsForValue().get("num");
        // 2.1 判断num为空return
        if (StringUtils.isEmpty(value)) {
            return;
        }
        // 2.2 有值就转成成int
        int num = Integer.parseInt(value + "");
        // 2.3 把redis的num加1
        redisTemplate.opsForValue().set("num", ++num);
        // 2.4 如果uuid一致再释放锁
        if (uuid.equals(redisTemplate.opsForValue().get("lock"))) {
            redisTemplate.delete("lock");
        }
    } else {
        // 3. 获取锁失败，每隔0.1秒再获取
        try {
            Thread.sleep(100);
            testLock();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

测试：

由2000变为了3000

![](assets/Redis应用问题解决/c0644fadd186b9ab8df7088371026f29_MD5.png)



### 优化 LUA脚本保证删除的原子性

1. index1执行删除时，查询到的lock值确实和uuid相等
   * uuid=v1
   * set(lock, uuid)
2. index1执行删除前(执行equals比较)，lock刚好过期时间已到，被redis自动释放，在redis中没有了lock，没有了锁
3. index2获取了lock，index2线程获取到了cpu的资源，开始执行方法
   * uuid=v2
   * set(lock, uuid)
4. index1执行删除，此时会把index2的lock删除，index1 因为已经在方法中了，所以不需要重新上锁，index1有执行的权限，index1已经比较(equals)完成了，这个时候，开始执行删除的index2的锁



```java
@GetMapping("/testLockLua")
public void testLockLua() {
    // 1. 声明一个uuid，做为value 放入key所对应的值中
    String uuid = UUID.randomUUID().toString();
    // 2. 定义一个锁：lua脚本可以使用同一把锁，来实现删除
    // 2.1 假设访问skuId为25号的商品
    String skuId = "25";
    // 2.2 锁住的是每个商品的数据
    String locKey = "lock:" + skuId;
    // 3. 获取锁
    Boolean lock = redisTemplate.opsForValue().setIfAbsent(locKey, uuid, 3, TimeUnit.SECONDS);
    if (Boolean.TRUE.equals(lock)) {
        // 3.1 获取锁成功，查询num的值
        Object value = redisTemplate.opsForValue().get("num");
        // 3.2 如果是空直接返回
        if (StringUtils.isEmpty(value)) {
            return;
        }
        // 3.3 不是空，如果说在这出现了异常，锁不会删除，锁永远存在
        int num = Integer.parseInt(value + "");
        // 3.4 使num+1，放入缓存
        redisTemplate.opsForValue().set("num", ++num);
        /*使用lua脚本来锁*/
        // 3.5 定义lua脚本
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        // 3.6 使用redis执行lua执行
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptText(script);
        // 3.7 设置返回值类型为Long，因为删除判断的时候返回是0。
        // 如果不封装默认返回String，返回字符串与0会有发生错误。
        redisScript.setResultType(Long.class);
        // 3.8 第一个要是 script 脚本 ，第二个需要判断的 key，第三个就是 key 所对应的值。
        redisTemplate.execute(redisScript, Collections.singletonList(locKey), uuid);
    } else {
        // 4.1 其他线程等待
        try {
            // 4.2 睡眠
            Thread.sleep(100);
            // 4.3 睡醒了之后，调用方法
            testLockLua();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```


测试：

![](assets/Redis应用问题解决/e744e88baba7f6b35aafbe63cf1800ad_MD5.png)



为了确保分布式锁可用，至少要确保锁的实现同时**满足以下四个条件**：

* 互斥性：在任意时刻，只有一个客户端能持有锁。
* 不会发生死锁：即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
* 解铃还须系铃人：加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。
* 加锁和解锁必须具有原子性。