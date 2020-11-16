## 集成Redis

在`SpringBoot`中，默认可以使用`RedisTemplate`和`StringRedisTemlate`操作`redis`

- `StringRedisTemlate`：操作字符串的方式操作`redis`
- `RedisTemplate`：操作对象的方式操作`redis`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>${version}</version>
</dependency>
```

`RedisTemplate`默认使用JDK序列化，在`redis`中存储的数据的可读性较差

下面是自定义序列化配置，使用`Jackon`

```java
@Configuration
public class RedisConfig extends CachingConfigurerSupport {
    /*
     * SpringBoot默认配置的redisTemplate加了如下注解, 表示如果已存在该bean则不加载
     * 因此自定义的redisTemplate会覆盖默认的
     * @ConditionalOnMissingBean(name = "redisTemplate")
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // 对key使用String就行
        template.setKeySerializer(new StringRedisSerializer());

        // 配置使用Jackson序列化对象
        Jackson2JsonRedisSerializer<Object> serializer = getSerializer();
        template.setValueSerializer(serializer);

        // 对hash的key/value执行同样策略 (其它4个统一)
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);

        // 收尾工作, 保证都设置了序列化方式
        template.afterPropertiesSet();
        return template;
    }

    private Jackson2JsonRedisSerializer<Object> getSerializer() {
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);
        // 记录序列化对象的具体类型, 因为泛型是Object, 如果不配置不可能直接映射为entity
        // 具体配置原理是: 把全类名写入JSON
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(om);
        return serializer;
    }
}
```

