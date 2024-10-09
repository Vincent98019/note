
Jedis所需要的jar包：

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```


连接Redis注意事项：

1. 禁用Linux的防火墙：Linux(CentOS7)里执行命令

```bash
systemctl stop/disable firewalld.service   
```

2. redis.conf中注释掉 `bind 127.0.0.1`
3. redis.conf中修改为 `protected-mode no`



## 快速入门

在pom.xml文件中添加依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```


```java
public static void main(String[] args) {
    // 1. 创建连接对象
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    // 2. 设置Redis的密码，如果无密码可以忽略
    jedis.auth("123456");
    // 3. 测试连接是否存在
    String pong = jedis.ping();
    System.out.println("连接成功：" + pong);

    // 4. 存入一个数据
    jedis.set("k1", "v1");
    // 5. 取出一个数据
    System.out.println(jedis.get("k1"));
    
    // 6. 关闭连接
    jedis.close();
}
```


## Jedis常用API

### 基本操作

```java
public static void main(String[] args) {
    // 设置连接参数
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    // 验证密码，如果没有设置密码这段代码省略
    // 验证密码的同时会进行connect操作
    jedis.auth("123456");
    // 连接
    jedis.connect();
    // 测试是否还能连接，是的话返回pong，测试的同时会进行connect操作
    String pong = jedis.ping();
    System.out.println("连接成功：" + pong);
    // 清空所有的key
    jedis.flushAll();
    // 释放资源
    jedis.close();
}
```


### key

```java
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    jedis.auth("123456");

    System.out.println("清空数据：" + jedis.flushDB());
    System.out.println("判断某个键是否存在：" + jedis.exists("username"));
    System.out.println("新增 <'username','aaa'> 的键值对：" + jedis.set("username", "aaa"));
    System.out.println("新增 <'password','123'> 的键值对：" + jedis.set("password", "123"));

    System.out.println("系统中所有的键如下：");
    Set<String> keys = jedis.keys("*");
    for (String key : keys) {
        System.out.println(key);
    }

    System.out.println("删除键 password:" + jedis.del("password"));
    System.out.println("判断键 password 是否存在：" + jedis.exists("password"));
    System.out.println("查看键 username 所存储的值的类型：" + jedis.type("username"));
    System.out.println("设置 username 的过期时间:"+jedis.expire("username",60));
    System.out.println("查看 username 的过期时间:"+jedis.ttl("username"));
    System.out.println("随机返回 key 空间的一个：" + jedis.randomKey());
    System.out.println("重命名 key：" + jedis.rename("username", "name"));
    System.out.println("取出改后的 name：" + jedis.get("name"));
    System.out.println("按索引查询：" + jedis.select(0));
    System.out.println("删除当前选择数据库中的所有 key：" + jedis.flushDB());
    System.out.println("返回当前数据库中 key 的数目：" + jedis.dbSize());
    System.out.println("删除所有数据库中的所有 key：" + jedis.flushAll());
}
```


### string

```java
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    jedis.auth("123456");

    jedis.flushDB();
    System.out.println("=========== 增加数据开始 ===========");
    System.out.println(jedis.set("key1", "value1"));
    System.out.println(jedis.set("key2", "value2"));
    System.out.println(jedis.set("key3", "value3"));
    System.out.println("删除键 key2:" + jedis.del("key2"));
    System.out.println("获取键 key2:" + jedis.get("key2"));
    System.out.println("修改 key1:" + jedis.set("key1", "newValue1"));
    System.out.println("获取 key1 的值：" + jedis.get("key1"));
    System.out.println("在 key3 后面加入值：" + jedis.append("key3", "key4"));
    System.out.println("key3 的值：" + jedis.get("key3"));
    System.out.println("增加多个键值对：" + jedis.mset("key01", "value01", "key02", "value02", "key03", "value03"));
    System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));
    System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03", "key04"));
    System.out.println("删除多个键值对：" + jedis.del("key01", "key02"));
    System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));
    System.out.println("=========== 增加数据结束 ===========");

    jedis.flushDB();
    System.out.println("=========== 新增键值对防止覆盖原先值开始 ==============");
    System.out.println(jedis.setnx("key1", "value1"));
    System.out.println(jedis.setnx("key2", "value2"));
    System.out.println(jedis.setnx("key2", "value2-new"));
    System.out.println(jedis.get("key1"));
    System.out.println(jedis.get("key2"));
    System.out.println("=========== 新增键值对防止覆盖原先值结束 ==============");

    System.out.println("=========== 新增键值对并设置有效时间开始 =============");
    System.out.println(jedis.setex("key3", 2, "value3"));
    System.out.println(jedis.get("key3"));
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(jedis.get("key3"));
    System.out.println("=========== 新增键值对并设置有效时间结束 =============");

    System.out.println("=========== 获取原值，更新为新值开始 ==========");
    System.out.println(jedis.getSet("key2", "key2GetSet"));
    System.out.println(jedis.get("key2"));
    System.out.println("获得 key2 的值的字串：" + jedis.getrange("key2", 2,4));
    System.out.println("=========== 获取原值，更新为新值结束 ==========");
}
```


### list

```java
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    jedis.auth("123456");

    jedis.flushDB();
    System.out.println("===========添加一个 list===========");
    jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
    jedis.lpush("collections", "HashSet");
    jedis.lpush("collections", "TreeSet");
    jedis.lpush("collections", "TreeMap");
    //-1 代表倒数第一个元素，-2 代表倒数第二个元素，end 为 -1 表示查询全部
    System.out.println("collections 的内容：" + jedis.lrange("collections", 0, -1));
    System.out.println("collections 区间 0-3 的元素：" + jedis.lrange("collections", 0, 3));

    System.out.println("===============================");
    // 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后 add 进去的值先被删，类似于出栈
    System.out.println("删除指定元素个数：" + jedis.lrem("collections", 2, "HashMap"));
    System.out.println("collections 的内容：" + jedis.lrange("collections", 0, -1));
    System.out.println("删除下标 0-3 区间之外的元素：" + jedis.ltrim("collections", 0, 3));
    System.out.println("collections 的内容：" + jedis.lrange("collections", 0, -1));
    System.out.println("collections 列表出栈（左端）：" + jedis.lpop("collections"));
    System.out.println("collections 的内容：" + jedis.lrange("collections", 0, -1));
    System.out.println("collections 添加元素，从列表右端，与 lpush 相对应：" + jedis.rpush("collections", "EnumMap"));
    System.out.println("collections 的内容：" + jedis.lrange("collections", 0, -1));
    System.out.println("collections 列表出栈（右端）：" + jedis.rpop("collections"));
    System.out.println("collections 的内容：" + jedis.lrange("collections", 0, -1));
    System.out.println("修改 collections 指定下标 1 的内容：" + jedis.lset("collections", 1, "LinkedArrayList"));
    System.out.println("collections 的内容：" + jedis.lrange("collections", 0, -1));

    System.out.println("===============================");
    System.out.println("collections 的长度：" + jedis.llen("collections"));
    System.out.println("获取 collections 下标为 2 的元素：" + jedis.lindex("collections", 2));
    System.out.println("===============================");

    jedis.lpush("sortedList", "3", "6", "2", "0", "7", "4");
    System.out.println("sortedList 排序前：" + jedis.lrange("sortedList", 0, -1));
    System.out.println(jedis.sort("sortedList"));
    System.out.println("sortedList 排序后：" + jedis.lrange("sortedList", 0, -1));
}
```


### set

```java
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    jedis.auth("123456");

    jedis.flushDB();
    System.out.println("============向集合中添加元素（不重复）============");
    System.out.println(jedis.sadd("eleSet", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
    System.out.println(jedis.sadd("eleSet", "e6"));
    System.out.println(jedis.sadd("eleSet", "e6"));
    System.out.println("eleSet 的所有元素为：" + jedis.smembers("eleSet"));
    System.out.println("删除一个元素 e0：" + jedis.srem("eleSet", "e0"));
    System.out.println("eleSet 的所有元素为：" + jedis.smembers("eleSet"));
    System.out.println("删除两个元素 e7 和 e6：" + jedis.srem("eleSet", "e7", "e6"));
    System.out.println("eleSet 的所有元素为：" + jedis.smembers("eleSet"));
    System.out.println("随机的移除集合中的一个元素：" + jedis.spop("eleSet"));
    System.out.println("随机的移除集合中的一个元素：" + jedis.spop("eleSet"));
    System.out.println("eleSet 的所有元素为：" + jedis.smembers("eleSet"));
    System.out.println("eleSet 中包含元素的个数：" + jedis.scard("eleSet"));
    System.out.println("e3 是否在 eleSet 中：" + jedis.sismember("eleSet", "e3"));
    System.out.println("e1 是否在 eleSet 中：" + jedis.sismember("eleSet", "e1"));
    System.out.println("e5 是否在 eleSet 中：" + jedis.sismember("eleSet", "e5"));

    System.out.println("=================================");
    System.out.println(jedis.sadd("eleSet1", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
    System.out.println(jedis.sadd("eleSet2", "e1", "e2", "e4", "e3", "e0", "e8"));
    // 移到集合元素
    System.out.println("将 eleSet1 中删除 e1 并存入 eleSet3 中：" + jedis.smove("eleSet1", "eleSet3", "e1"));
    System.out.println("将 eleSet1 中删除 e2 并存入 eleSet3 中：" + jedis.smove("eleSet1", "eleSet3", "e2"));
    System.out.println("eleSet1 中的元素：" + jedis.smembers("eleSet1"));
    System.out.println("eleSet3 中的元素：" + jedis.smembers("eleSet3"));

    System.out.println("============集合运算=================");
    System.out.println("eleSet1 中的元素：" + jedis.smembers("eleSet1"));
    System.out.println("eleSet2 中的元素：" + jedis.smembers("eleSet2"));
    System.out.println("eleSet1 和 eleSet2 的交集:" + jedis.sinter("eleSet1", "eleSet2"));
    System.out.println("eleSet1 和 eleSet2 的并集:" + jedis.sunion("eleSet1", "eleSet2"));
    // eleSet1 中有，eleSet2 中没有
    System.out.println("eleSet1和eleSet2的差集:" + jedis.sdiff("eleSet1", "eleSet2"));
    // 求交集并将交集保存到 dstkey 的集合
    jedis.sinterstore("eleSet4", "eleSet1", "eleSet2");
    System.out.println("eleSet4 中的元素：" + jedis.smembers("eleSet4"));
}
```


### hash

```java
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    jedis.auth("123456");
    jedis.flushDB();
    Map<String, String> map = new HashMap<>();
    map.put("key1", "value1");
    map.put("key2", "value2");
    map.put("key3", "value3");
    map.put("key4", "value4");
    //添加名称为hash（key）的hash元素
    jedis.hmset("hash", map);
    //向名称为hash的hash中添加key为key5，value为value5元素
    jedis.hset("hash", "key5", "value5");
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash")); //return Map<String,String>
    System.out.println("散列hash的所有键为：" + jedis.hkeys("hash")); //return Set<String>
    System.out.println("散列hash的所有值为：" + jedis.hvals("hash")); //return List<String>
    System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 6));
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
    System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 3));
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
    System.out.println("删除一个或者多个键值对：" + jedis.hdel("hash", "key2"));
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
    System.out.println("散列hash中键值对的个数：" + jedis.hlen("hash"));
    System.out.println("判断hash中是否存在key2：" + jedis.hexists("hash", "key2"));
    System.out.println("判断hash中是否存在key3：" + jedis.hexists("hash", "key3"));
    System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3"));
    System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3", "key4"));
}
```


### 事务

```java
public static void main(String[] args) {
    Jedis jedis = new Jedis("192.168.182.133", 6379);
    jedis.auth("123456");
    jedis.flushDB();

    Map<String, String> map = new HashMap<>();
    map.put("hello", "world");
    map.put("name", "java");
    String result = map.toString();

    // 开启事务
    Transaction multi = jedis.multi();
    try {
        // 向 redis 存入一条数据
        multi.set("map1", result);
        // 再存入一条数据
        multi.set("map2", result);

        // 这里引发了异常，用 0 作为被除数
        int i = 1 / 0;
        // 如果没有引发异常，执行进入队列的命令
        multi.exec();
    } catch (Exception e) {
        e.printStackTrace();
        // 如果出现异常，回滚
        multi.discard();
    } finally {
        System.out.println(jedis.get("map1"));
        System.out.println(jedis.get("map2"));
        // 最终关闭客户端
        jedis.close();
    }
}
```


## Jedis连接池

1. 创建JedisPool连接池对象  
2. 调用方法 `getResource()` 方法获取Jedis连接

```java
//0.创建一个配置对象
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(50);
config.setMaxIdle(10);

//1.创建Jedis连接池对象
JedisPool jedisPool = new JedisPool(config, "localhost", 6379);

//2.获取连接
Jedis jedis = jedisPool.getResource();

//3. 使用
jedis.set("hehe","heihei");

//4. 关闭 归还到连接池中
jedis.close();
```


连接池工具类：

```java
public class JedisPoolUtils {

    private static JedisPool jedisPool;

    static{
	//读取配置文件
	InputStream is = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
	//创建Properties对象
	Properties pro = new Properties();
	//关联文件
	try {
            pro.load(is);
	} catch (IOException e) {
	    e.printStackTrace();
	}
	//获取数据，设置到JedisPoolConfig中
	JedisPoolConfig config = new JedisPoolConfig();
	config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
	config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));
	//初始化JedisPool
	jedisPool = new JedisPool(config,pro.getProperty("host"),Integer.parseInt(pro.getProperty("port")));
    }
                  
    /**
     * 获取连接方法
     */
    public static Jedis getJedis(){
	return jedisPool.getResource();
    }
}
```
