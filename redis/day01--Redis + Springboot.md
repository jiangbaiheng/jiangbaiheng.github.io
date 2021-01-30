# SpringBoot 整合 Redis

## 1 背景：

![](C:\Users\baiheng.jiang\Documents\GitHub\jiangbaiheng.github.io\img\redis\day01-1-background.png)

## 2 导入依赖：

```xml
<!-- redis依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 3 Redis配置文件：

+ SpringBoot 所有配置类，都有一个自动配置类
  - RedisAutoConfiguration
+ 自动配置类都会绑定一个properties 配置文件 
  - RedisProperties

```
#Redis配置
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

## 4 测试：

```java
@SpringBootTest
class RedisDemoApplicationTests {

    @Autowired
    private RedisTemplate redisTemplate;
    @Test
    void contextLoads() {

        //opsForValue 操作字符串 类似String
        //opsForList 操作字符串 类似List
        //opsForSet 操作字符串 类似Set
        //opsForHash
        //opsForZset

        //除了基本的操作，我们常用的方法可以直接通过redisTemplate操作，比如事务和基本的CRUD
        //获取redis连接对象

        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
        connection.flushDb();
        connection.flushAll();

        redisTemplate.opsForValue().set("mykey","狂神说Java");
        System.out.println(redisTemplate.opsForValue().get("mykey"));
    }
```

# SpringSession+Redis实现session共享及唯一登录

## 1 原理：

+ HttpSession的实现被Spring Session替换，操作HttpSession等同于操作redis中的数据。
+ 添加@EnableRedisHttpSession来开启spring session支持
+ 而@EnableRedisHttpSession这个注解是由spring-session-data-redis提供的
+ `@EnableRedisHttpSession` 这个注解创建了一个名为 springSessionRepositoryFilter 的 bean，负责替换 httpSession,同时由 redis 提供缓存支持。 
  `maxInactiveIntervalInSeconds`:设置Session失效时间。使用Redis Session之后，原Boot的server.session.timeout属性不再生效。

## 2 依赖：

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-redis</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.springframework.session</groupId>  
    <artifactId>spring-session-data-redis</artifactId>  
</dependency> 
```

## 3 添加配置类：

```java
@Configuration  
@EnableRedisHttpSession  
public class SessionConfig {  
} 
```







