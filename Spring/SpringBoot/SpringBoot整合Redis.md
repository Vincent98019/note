
在 Spring Boot 中一般使用 RedisTemplate 提供的方法来操作 Redis。

* JedisPoolConfig：配置连接池
* RedisConnectionFactory：是一个接口，配置连接信息，使用它的实现类，在 SpringDataRedis 方案中提供了以下四种工厂模型：
  * JredisConnectionFactory
  * JedisConnectionFactory
  * LettuceConnectionFactory
  * SrpConnectionFactory
* RedisTemplate：基本操作



## 搭建环境

在pom.xml文件中引入redis相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

> 在 Spring Boot 2.x 之后，原来使用的 jedis 被替换成 lettuce


在application.yml中添加配置信息：

```yaml
spring:
  redis:
    # Redis服务器地址
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis数据库索引（默认为0）
    database: 0
    # 连接超时时间（毫秒）
    timeout: 1800000
    lettuce:
      pool:
        # 连接池最大连接数（使用负值表示没有限制）
        max-active: 20
        # 最大阻塞等待时间(负数表示没限制)
        max-wait: -1
        # 连接池中的最大空闲连接
        max-idle: 5
        # 连接池中的最小空闲连接
        min-idle: 0
```


## Redis配置类

分析 RedisAutoConfiguration 自动配置类源码：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
		return new StringRedisTemplate(redisConnectionFactory);
	}

}
```

通过源码可以看出，Spring Boot 自动帮我们在容器中生成了一个 RedisTemplate 和一个 StringRedisTemplate。

因为有 `@ConditionalOnMissingBean` 注解，如果 Spring 容器中有 RedisTemplate 对象，这个自动配置的 RedisTemplate 不会实例化。因此可以自己配置 RedisTemplate。



```java
@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer<?> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        
        // ObjectMapper 指定在转成json的时候的一些转换规则
        ObjectMapper om = new ObjectMapper();
        // om.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);

        // 把自定义objectMapper设置到jackson2JsonRedisSerializer中（可以不设置，使用默认规则）
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setConnectionFactory(factory);

        // RedisTemplate默认的序列化方式使用的是JDK的序列化
        // 设置key采用String的序列化方式
        template.setKeySerializer(redisSerializer);
        // 设置value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // value hashmap序列化
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }

    /**
     * 缓存管理器
     */
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer<?> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        // 解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                // 配置序列化（解决乱码的问题），过期时间600秒
                .entryTtl(Duration.ofSeconds(600))
                // 设置 key为string序列化
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                // 设置value为json序列化
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                // 不缓存空值
                .disableCachingNullValues();
        return RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
    }
}
```