
### 配置国际化资源文件

1. **创建消息资源文件**  

   在`src/main/resources`目录下创建：
   - `messages.properties`（默认语言，如中文）
   - `messages_en.properties`（英文）

```properties
# messages.properties
NotBlank.message=该字段不能为空
```

```properties
# messages_en.properties
NotBlank.message=This field cannot be blank
```

2. **Spring Boot配置**  

   在`application.yaml`中启用国际化支持：
```yaml
spring:
  messages:
    basename: messages
    encoding: UTF-8
```


### 自定义`@NotBlank`消息

1. **在实体类中使用注解**  

通过`message`属性引用资源文件中的键：

```java
public class User {
	@NotBlank(message = "{NotBlank.message}")
    private String username;
}
```

2. **验证框架自动加载消息**  

Spring Validation会自动根据当前语言环境（`Locale`）选择对应的消息文本。



### AOP切面实现

通过切面自动注入语言参数：


```java
@Aspect
@Component
public class LanguageAspect {
    @Pointcut("execution(public * cn.healtrack.*.controller..*Controller.*(..))")
    public void controllerMethod() {}

    @Before("controllerMethod()")
    public void before(JoinPoint joinPoint) {
        ServletRequestAttributes attributes = 
            (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        String lang = request.getHeader("Accept-Language");
        // LocaleContextHolder.setLocale(Locale.forLanguageTag(lang));
        if ("en".equals(lang)) {  
		    LocaleContextHolder.setLocale(Locale.ENGLISH);  
		}
    }
}
```

