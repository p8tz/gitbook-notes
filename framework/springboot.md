## 加载指定配置文件

配置文件`person.properties`

```properties
person.name=JJZ
person.age=20
person.sons=LJY, SR
```

### 通过@PropertySource+@Value

```java
@Component("p1")
// 加载指定配置文件
@PropertySource(value = {"classpath:person.properties"})
public class Person {
    @Value("${person.name}")
    String name;
    @Value("${person.age}")
    int age;
    @Value("${person.sons}")
    String[] sons;
    // ...setters
}
```

### 通过@PropertySource+@ConfigurationProperties

```java
@Component("p1")
@PropertySource(value = {"classpath:person.properties"})
// 指定配置起始位置, 自动注入
@ConfigurationProperties(prefix = "person")
public class Person {
    String name;
    int age;
    String[] sons;
	// ...setters
}
```

**测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootTestApplicationTests {

    @Autowired
    @Qualifier("p1")
    Person person;

    @Test
    void contextLoads() {
        System.out.println(person);
        // Person{name='JJZ', age=20, sons=[LJY, SR]}
    }
}
```

### 注意

- `PropertySource`不支持`yml`读取
- 对于默认配置文件（`application.yml/properties`）不需要用`PropertySource`指定

## 条件注册Bean

需要用到`@Conditional`注解

场景：根据操作系统注册不同的Bean

需要按条件加载的`Bean`

```java
public class OS {
    String type;
    // ...
}
```

加载条件：需要实现`Condition`接口，重写`matches`方法，返回`true`加载，返回`false`不加载

```java
public class LinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        String os = environment.getProperty("os.name");
        return os != null && os.toLowerCase().startsWith("linux");
    }
}

public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        String os = environment.getProperty("os.name");
        return os != null && os.toLowerCase().startsWith("windows");
    }
}
```

```java
@Configuration
public class MainConfig {
    @Bean("linux")
    // 满足条件才进行注册Bean
    @Conditional({LinuxCondition.class})
    public OS linux() {
        return new OS("linux");
    }

    @Bean("windows")
    @Conditional({WindowsCondition.class})
    public OS windows() {
        return new OS("windows");
    }
}

@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootTestApplicationTests {

    @Autowired(required = false)
    @Qualifier("linux")
    OS linux;

    @Autowired(required = false)
    @Qualifier("windows")
    OS windows;

    @Test
    void contextLoads() {
        System.out.println(linux);		// null
        System.out.println(windows);	// OS{type='windows'}
        
        // 虚拟机参数加上 -Dos.name=linux 后
        // OS{type='linux'}
		// null
    }
}

```

## 日志配置

`SpringBoot`默认使用`slf4j + logback`

```yaml
# application.yml
logging:
  file:
    # 指定日志文件名
    name: log/xxx.log
    # 指定日志路径, 文件名为spring.log
#    path: log

  # 日志级别
  level:
    root: info
    # 指定包下级别
    cc.p8t.test: info

  # 这都是默认配置
  logback:
    rollingpolicy:
      # 自动按日压缩
      file-name-pattern: ${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz
      # 单日志文件最大大小
      max-file-size: 10MB
      # 保留天数
      max-history: 7
      
  # 引入自定义日志配置文件
#  config: classpath:logback-spring.xml
```

## druid配置

官网配置文档：[druid](https://github.com/alibaba/druid/wiki)

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>
```

```yaml
# application.yml
spring:
  datasource:
    # JDBC配置
    url: jdbc:mysql://127.0.0.1:3306/db?serverTimezone=Asia/Shanghai
    username: root
    password: pwd
    driver-class-name: com.mysql.cj.jdbc.Driver
    # 连接池配置
    druid:
      # 初始连接数量
      initial-size: 0
      # 最大连接池数量
      max-active: 10
      # 已经不再使用，配置了也没效果
      max-idle: 0
      # 最小连接池数量
      min-idle: 0
      # 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
      max-wait: 10000
      # 有两个含义：
      # 1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。
      # 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
      time-between-eviction-runs-millis: 60000
      # 连接保持空闲而不被驱逐的最小时间
      min-evictable-idle-time-millis: 300000
      # 用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。
      validation-query: select 'x'
      # 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
      test-while-idle: true
      # 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
      test-on-borrow: false
      # 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
      test-on-return: false
      # 是否缓存preparedStatement，在mysql下建议关闭。
      pool-prepared-statements: false
      # https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8

      # 监控统计用的filter:stat
      # 日志用的filter:log4j
      # 防御sql注入的filter:wall
      filters: stat, wall, log4j
      # 缺省多个DruidDataSource的监控数据是各自独立的，在Druid-0.2.17版本之后，支持配置公用监控数据
      use-global-data-source-stat: true
      connection-properties: druid.stat.slowSqlMillis=3000;druid.stat.logSlowSql=true;druid.stat.mergeSql=true
      # https://github.com/alibaba/druid/wiki/%E6%B3%A8%E8%A7%A3%E6%96%B9%E5%BC%8F%E9%85%8D%E7%BD%AE%E7%9B%91%E6%8E%A7
      # https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter

      # Druid内置提供了一个StatViewServlet用于展示Druid的统计信息。
      # 这个StatViewServlet的用途包括：
      #   提供监控信息展示的html页面
      #   提供监控信息的JSON API
      stat-view-servlet:
        enabled: true
        login-username: admin
        login-password: 123456
        # 由于匹配规则不支持IPV6，配置了allow或者deny之后，会导致IPV6无法访问。
        allow: 127.0.0.1
        deny: 192.168.75.128
        # StatView是否允许清空统计记录
        reset-enable: true
      # https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE
```

## 集成Redis和Redisson

在`SpringBoot`中，默认可以使用`RedisTemplate`和`StringRedisTemlate`操作`redis`

- `StringRedisTemlate`：操作字符串的方式操作`redis`
- `RedisTemplate`：操作对象的方式操作`redis`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.13.6</version>
</dependency>
```

`RedisTemplate`默认使用`JDK`序列化，在`redis`中存储后的数据可读性较差

下面是自定义序列化配置，使用`Jackson`

```java
@Configuration
public class RedisConfig extends CachingConfigurerSupport {
    /*
     * SpringBoot默认配置的redisTemplate加了如下注解, 表示如果已存在该bean则不加载
     * @ConditionalOnMissingBean(name = "redisTemplate")
     * 因此自定义的redisTemplate会覆盖默认的
     * 这里我没有选择覆盖, 因为Qualifier注入会爆红(能运行), 看着不爽。可能是我配置问题吧
     */
    @Bean("redis")									  // 如果引入了redisson, 必须指定这个Bean
    public RedisTemplate<String, Object> redisTemplate(@Qualifier("redissonConnectionFactory")RedisConnectionFactory connectionFactory) {
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

官网配置文档：[redisson](https://github.com/redisson/redisson/wiki/2.-%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95)

```yaml
# application.yml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password: pwd
    database: 0
    timeout: 60000
    jedis:
      pool:
        max-idle: 10
        min-idle: 0
        max-active: 10
        max-wait: 60000
    # redisson配置
    redisson:
      file: classpath:redisson.yml

# redisson.yml
# 单节点配置
singleServerConfig:
  # 节点地址
  address: redis://127.0.0.1:6379
  # 密码
  password: pwd
  # 数据库编号
  database: 0
  # 最小空闲连接数
  connectionMinimumIdleSize: 0
  # 连接池大小。在启用该功能以后，Redisson将会监测DNS的变化情况。
  connectionPoolSize: 64
  # DNS监测时间间隔，单位：毫秒
  dnsMonitoringInterval: 5000
  # 连接空闲超时
  idleConnectionTimeout: 10000
  # 连接超时。同节点建立连接时的等待超时。 
  connectTimeout: 10000
  # 命令等待超时。等待节点回复命令的时间。该时间从命令发送成功时开始计时。
  timeout: 3000
  # 命令失败重试次数,如果尝试达到retryAttempts仍然不能将命令发送至某个指定的节点时，将抛出错误。
  retryAttempts: 3
  # 命令重试发送时间间隔，单位：毫秒
  retryInterval: 1500
  #  # 重新连接时间间隔，单位：毫秒
  #  reconnectionTimeout: 3000
  #  # 执行失败最大次数
  #  failedAttempts: 3
  # 客户端名称
  clientName: null
  # 发布和订阅连接的最小空闲连接数
  subscriptionConnectionMinimumIdleSize: 1
  # 发布和订阅连接池大小
  subscriptionConnectionPoolSize: 50
  # 单个连接最大订阅数量
  subscriptionsPerConnection: 5
# 线程池数量, 默认值: 当前处理核数量 * 2
threads: 0
# Netty线程池数量, 默认值: 当前处理核数量 * 2
nettyThreads: 0
# 编码
codec: !<org.redisson.codec.JsonJacksonCodec> {}
# 传输模式
transportMode: NIO
```

使用

```java
@Autowired
@Qualifier("redis")
RedisTemplate<String, Object> redis;

@Autowired
@Qualifier("redisson")
RedissonClient redisson;
```

## 发邮件

### 依赖与配置

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  mail:
    host: smtp.qq.com
    port: 465
    protocol: smtp
    username: jljxvg@foxmail.com
    password: xxxxxxxxxxxxxxxx  # 邮箱授权码
    default-encoding: utf-8
    properties: {
      mail.smtp.auth: true,
      mail.smtp.starttls.enable: true,
      mail.smtp.starttls.required: true,
      mail.smtp.ssl.enable: true,
      mail.smtp.socketFactory.port: 465,
      smtp.socketFactory.class: javax.net.ssl.SSLSocketFactory
    }
```

### 发送静态页面

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootTestApplicationTests {
    
    // @Autowired
    // @Qualifier("templateEngine")
    // TemplateEngine templateEngine;

    @Autowired
    @Qualifier("mailSender")
    JavaMailSender mailSender;

    @Value("${spring.mail.username}")
    String from;
    @Value("${spring.mail.username}")
    String to;

    @Test
    void contextLoads() throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage);
        // 发件人
        helper.setFrom(from);
        // 收件人
        helper.setTo(to);
        // 主题
        helper.setSubject("springboot-mail-test");
        // 第二个参数表示是否启用html渲染, 默认false
        helper.setText("<h1 style='color:blue'>Hello JJZ</h1>", true);
        // 发送
        mailSender.send(mimeMessage);
    }
}
```

### 发送动态页面

使用`thymeleaf`动态填入数据

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootTestApplicationTests {
    
    @Autowired
    @Qualifier("templateEngine")
    TemplateEngine templateEngine;

    @Autowired
    @Qualifier("mailSender")
    JavaMailSender mailSender;

    @Value("${spring.mail.username}")
    String from;
    @Value("${spring.mail.username}")
    String to;

    @Test
    void contextLoads() throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject("springboot-mail-test");
        
		// Context存放与模板数据相关的内容
        Context context = new Context();// org.thymeleaf.context.Context;
        Map<String, Object> map = new HashMap<>();
        map.put("title", "Hello JJZ");
        map.put("color", "blue");
        context.setVariables(map);
        // 第一个参数为模板页面路径，第二个参数是Context
        // 这也就是把Context内容填入模板页面的过程
        String text = templateEngine.process("mail", context);
        
        helper.setText(text, true);
        mailSender.send(mimeMessage);
    }
}
```

`mail.html`

![image-20201121090305183](https://gitee.com/p8t/picbed/raw/master/imgs/20201121090306.png)

```html
<!DOCTYPE html>
<!--suppress ALL-->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>mail</title>
    </head>
    <body>
        <!-- &#58;表示 : -->
        <h1 th:style="'color&#58;'+${color}" th:text="${title}"></h1>
    </body>
</html>
```

### 发送附件

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootTestApplicationTests {

    @Autowired
    @Qualifier("mailSender")
    JavaMailSender mailSender;

    @Value("${spring.mail.username}")
    String from;
    @Value("${spring.mail.username}")
    String to;

    @Test
    void contextLoads() throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        // 发送附件需要把multipart设置为true
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject("springboot-mail-test");
        helper.setText("<h1>attachment</h1>", true);
		// 第一个参数为发过去显示的文件名, 第二个为要发送的文件
        helper.addAttachment("1.jpg", new File("C:\\Users\\JJZ\\Desktop\\1.jpg"));
        mailSender.send(mimeMessage);
    }
}
```

## 二维码

原文链接：[Java生成二维码](https://zhuanlan.zhihu.com/p/111099137)

```xml
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>core</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>com.google.zxing</groupId>
    <artifactId>javase</artifactId>
    <version>3.4.1</version>
</dependency>
```

```java
/**
 * 生成二维码，写到本地
 *
 * @param content  二维码包含的信息
 * @param width    二维码宽度
 * @param height   二维码高度
 * @param filePath 图片保存路径
 * @throws WriterException
 * @throws IOException
 */
public void generateQRCodeImage(String content, int width, int height, String filePath) throws WriterException, IOException {
    QRCodeWriter qrCodeWriter = new QRCodeWriter();
    BitMatrix bitMatrix = qrCodeWriter.encode(content, BarcodeFormat.QR_CODE, width, height);
    // Path path = FileSystems.getDefault().getPath(filePath);
	// Path path = Paths.get(filePath);
    Path path = Path.of(filePath);	// JDK11
    if (path.getParent() != null && !Files.exists(path.getParent())) {
        Files.createDirectories(path.getParent());
    }
    MatrixToImageWriter.writeToPath(bitMatrix, "PNG", path);
}

/**
 * 生成二维码，返回字节流
 *
 * @param content 二维码包含的信息
 * @param width   二维码宽度
 * @param height  二维码高度
 * @return
 * @throws WriterException
 * @throws IOException
 */
public byte[] getQRCodeImage(String content, int width, int height) throws WriterException, IOException {
    QRCodeWriter qrCodeWriter = new QRCodeWriter();
    BitMatrix bitMatrix = qrCodeWriter.encode(content, BarcodeFormat.QR_CODE, width, height);
    ByteArrayOutputStream pngOutputStream = new ByteArrayOutputStream();
    MatrixToImageWriter.writeToStream(bitMatrix, "PNG", pngOutputStream);
    return pngOutputStream.toByteArray();
}
```

