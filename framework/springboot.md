## 默认文件加载顺序

### 1、配置文件

```shell
classpath:/
classpath:/config/
file:./			# jar包当前目录
file:./config/*/
file:./config/
# 加载顺序从上往下, 下面的会覆盖上面的
```

### 2、静态资源文件

```shell
classpath:/static/
classpath:/public/
classpath:/resources/
classpath:/META-INF/resources/
classpath:/
```

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
- 对于默认配置文件（`application.yml/.properties`）不需要用`@PropertySource`指定

## 条件注册Bean

需要用到`@ConditionalOn*`系列注解，其中`@Conditional`是基础

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

非`SpringBoot`项目，自己引入`logback`配置

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.30</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
    <scope>compile</scope>
</dependency>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
	<!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) %cyan(%logger{35}) - %highlight(%msg%n)</pattern>
        </encoder>
    </appender>
    <!-- 文件输出 -->
    <appender name="FILEOUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 基于时间滚动，就是每天的日志输出到不同的文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 输出日志的目录文件名，window中默认分区为当前程序的硬盘分区，%d{yyyy-MM-dd}是当前日期 -->
            <fileNamePattern>utils/log/%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 最大保存99个文件，超出的历史文件会被删除 -->
            <maxHistory>99</maxHistory>
        </rollingPolicy>
        <!-- 按照日志级别进行过滤 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
        </filter>

        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="FILEOUT"/>
        <!--        <appender-ref ref="ERROR"/>-->
        <!--        <appender-ref ref="WARN"/>-->
        <!--        <appender-ref ref="INFO"/>-->
        <!--        <appender-ref ref="DEBUG"/>-->
        <!--        <appender-ref ref="TRACE"/>-->
    </root>
	<!-- 下面是把指定级别日志输出到文件 -->
    <!--
        <appender name="TRACE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>log/trace/%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>99</maxHistory>
            </rollingPolicy>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>TRACE</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
            </encoder>
        </appender>
        <appender name="DEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>log/debug/%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>99</maxHistory>
            </rollingPolicy>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>DEBUG</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
            </encoder>
        </appender>
        <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>log/info/%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>99</maxHistory>
            </rollingPolicy>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>INFO</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
            </encoder>
        </appender>
        <appender name="WARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>log/warn/%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>99</maxHistory>
            </rollingPolicy>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>WARN</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
            </encoder>
        </appender>
        <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>log/error/%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>99</maxHistory>
            </rollingPolicy>
            <filter class="ch.qos.logback.classic.filter.LevelFilter">
                <level>ERROR</level>
                <onMatch>ACCEPT</onMatch>
                <onMismatch>DENY</onMismatch>
            </filter>
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
            </encoder>
        </appender>
    -->
</configuration>
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

	  # 监控相关, 监控web显示需要stat-view-servlet的配置
      # filters: stat,wall,slf4j # 一键开启下面就不用enabled: true了
      filter:
        stat:
          enabled: true
          slow-sql-millis: 1000 # 超过1000ms的记为慢查询
          log-slow-sql: true    # 记录慢查询日志
          merge-sql: true
        wall:
          enabled: true
        slf4j:
          enabled: true
      web-stat-filter: # uri对应的sql以及sssion监控
        enabled: true
        # 下面两个都是默认值, 分别为监控的url和排除的url
        url-pattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'
      # Spring监控
      aop-patterns: com.example.*
      
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
        # StatView是否允许清空统计记录[多个重置按钮]
        reset-enable: true
      # https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE
      
      # 缺省多个DruidDataSource的监控数据是各自独立的，在Druid-0.2.17版本之后，支持配置公用监控数据
      use-global-data-source-stat: true
```

编码方式配置

```java
// 只需要注入一个javax.sql.DataSource的Bean即可
@Configuration
public class DruidConfig {
    
    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/db?serverTimezone=Asia/Shanghai");
        dataSource.setUsername("root");
        dataSource.setPassword("pwd");
        // ...
        return dataSource;
    }
}
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
public class RedisConfig /*extends CachingConfigurerSupport*/ {
    private final StringRedisSerializer keySerializer = new StringRedisSerializer();
	private final GenericJackson2JsonRedisSerializer valueSerializer = new GenericJackson2JsonRedisSerializer();
    // private final Jackson2JsonRedisSerializer<Object> serializer = getSerializer();

    /*
     * SpringBoot默认配置的redisTemplate加了如下注解, 表示如果已存在该bean则不加载
     * @ConditionalOnMissingBean(name = "redisTemplate")
     * 因此自定义的redisTemplate会覆盖默认的
     * 这里没有选择覆盖
     */
    @Bean("redis")									  // 如果引入了redisson, 必须指定这个Bean
    public RedisTemplate<String, Object> redisTemplate(@Qualifier("redissonConnectionFactory")RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // 对key使用String就行
        template.setKeySerializer(keySerializer);
        // 配置使用Jackson序列化对象
        template.setValueSerializer(valueSerializer);

        // 对hash的key/value执行同样策略 (其它4个统一)
        template.setHashKeySerializer(keySerializer);
        template.setHashValueSerializer(valueSerializer);

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

## 资源映射

> [static-content](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content)

对于静态资源，`SpringBoot`会自动去以下目录查找。当然，只有`Controller`不能处理时才会交给静态资源处理器处理，如果还找不到那就404

- `classpath:/static/`
- `classpath:/public/`
- `classpath:/resources/`
- `classpath:/META-INF/resources/`
- `classpath:/`

可以为静态资源加一个公共请求前缀，方便用于拦截器统一放行

```yaml
# 如果原来static目录下有一个文件叫1.jpg
# 不加上下面这个配置,通过protocol:ip:port/1.jpg访问
# 加上之后就是protocol:ip:port/resources/1.jpg
spring:
  mvc:
    static-path-pattern: /resources/**
```

但是有时候用户上传的文件可能不存在这里面，就需要自己配置查找路径，如果不配置，即使路径正确也不能访问，因为服务器不能对外暴露真实路径。

通过`Java`代码配置

```java
@Configuration
public class MainConfig {
    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                registry.addResourceHandler("/upload/**")
                		.addResourceLocations("file:D:/upload/");
            }
        };
    }
}
```

通过配置文件配置

```yaml
spring:
  mvc:
    static-path-pattern: /upload/**
  resources:
    static-locations:
      - classpath:/static/
      - classpath:/public/
      - classpath:/resources/
      - classpath:/META-INF/resources/
      - classpath:/
      # 服务器上的真实路径
      - file:D:/upload/
```

## 上传文件

前端页面

```html
<form th:action="@{upload}" method="post" enctype="multipart/form-data">
    点击上传: <input type="file" name="file" accept="image/*">
    <button type="submit">上传</button>
</form>
```

```java
@RequestMapping("upload")
public String addImage(@RequestParam("file") MultipartFile file) {
    if (file.isEmpty()) {
        return "index";
    }

    // 获取上传文件名和上传路径
    String fileName = file.getOriginalFilename();
    String filePath = fileUploadProperties.getUploadPath();

    // 构建上传uri
    String uri = filePath + fileName;
    File dest = new File(uri);

    // 上传文件
    file.transferTo(dest.getAbsoluteFile());
    return "index";
}
```

## 下载文件

下面用的是`a`标签`download`属性，只要把文件`URL`写在`href`里就行了，不需要代码实现下载

```html
<a th:href="@{'/upload/1.jpg'}" download="pic.jpg">download</a>
```

```java
@Configuration
public class MainConfig {
    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                registry.addResourceHandler("/upload/**")
                    	// 真实文件路径为D:/upload/1.jpg
                        .addResourceLocations("file:D:/upload/");
            }
        };
    }
}
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
<html lang="en" xmlns:th="https://www.thymeleaf.org">
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

## 生成二维码

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

## 数据校验

### 使用前置条件

**校验规则注解 + `@Validated` + `BindingResult / Errors`**

依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>2.3.4.RELEASE</version>
</dependency>
```

校验规则注解在`javax.validation.constraints`包下，常用如下

```java
@Null		// 必须为null
@NotNull	// 非null
@NotBlank	// 非null, 且String不能全为空白字符
@NotEmpty	// 非null, 且String、数组、集合、Map长度不能为0
@Digits		// 可null, 接受String数字, 可以指定数字长度以及小数位数
@Email		// 可null
@Size		// 可null, 对于String限制长度, 对于数组限制长度, 对于集合限制元素数量, 对于Map限制元素数量
@Length		// 这个来自hibernate-validator, 只能限制String长度
@Max 		// 可null, 只接受整数, 指定最大值
@Min		// 可null, 只接受整数, 指定最小值
@Future		// 可null, 未来时间
@Past		// 可null, 过去时间
@Pattern	// 可null, 满足正则表达式
```

一般情况下可以使用`javax`下的`@Valid`，但是它不支持分组校验，这时就需要`@Validated`

`BindingResult / Errors`用于获取校验信息，必须紧跟在校验参数后面

### 例子

```java
public class User {
    @NotBlank
    private String username;
    @NotBlank
    private String password;
}

@PostMapping("/login")
public Result<Object> checkUser(@Validated @RequestBody User user
                                BindingResult br) { // 校验信息自动写入到BindingResult
    if (br.hasFieldErrors()) {
        return new Result<>(CodeInfo.VALIDATED_ERROR);
    }
    // ...
}
```

一个实体在不同表单需要被校验的字段可能不一样，这时需要把校验的字段分组

```java
public class User {
    // UserInfo可以是任意一个空接口, 只要保证同时要求校验的字段声明同一个接口即可
    @NotBlank(groups = {UserInfo.class, UserLogin.class})
    private String username;
    @NotBlank(groups = {UserLogin.class})
    private String password;
}

@PostMapping("/user-info")	  // 此时只会校验username字段
public Result<Object> userInfo(@Validated(UserInfo.class) @RequestBody User user
                                BindingResult br) { // 校验信息自动写入到BindingResult
    if (br.hasFieldErrors()) {
        return new Result<>(CodeInfo.VALIDATED_ERROR);
    }
    // ...
}
```

## 缓存

### SpringBoot缓存

#### 1、开启缓存

`@EnableCaching`：加在任意一个`@Configuration`类上即可

#### 2、配置使用`Redis`作为缓存介质

只要自定义注入了`CacheManager`的`Bean`，`SpringBoot`就会使用其配置的缓存工具以及缓存配置参数

```java
@Configuration
public class RedisCacheConfig {
    private final StringRedisSerializer keySerializer = new StringRedisSerializer();
    private final GenericJackson2JsonRedisSerializer valueSerializer = new GenericJackson2JsonRedisSerializer();
    
    @Bean("redisCacheManager")
    @Primary
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        // Redis缓存配置
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                // 为key设置序列化器
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(keySerializer))
                // 为value设置序列化器
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(valueSerializer))
                // 缓存存活时间
                .entryTtl(Duration.ofSeconds(3600))
                // 缓存不允许null值
                .disableCachingNullValues();
        // 缓存管理器
        RedisCacheManager cm = RedisCacheManager.builder(connectionFactory)
                // 设置缓存配置
                .cacheDefaults(config)
                .build();
        return cm;
    }
}
```

#### 3、使用缓存

##### @Cacheable

加在方法上，方法执行前检测是否有满足的缓存。如果有则直接返回缓存内存，**不执行方法**；如果没有则执行方法，并把方法返回值加入缓存。

**例子**

```java
@RestController
public class CacheTestController {
    @Autowired
    UsrMapper usrMapper;

    @Cacheable(cacheNames = "user")
    @GetMapping("/cache/{id}")
    public Usr select(@PathVariable("id") int id) {
        System.out.println("查询用户, 没走缓存");
        return usrMapper.findById(id);
    }
}
```

![image-20201213200118677](https://gitee.com/p8t/picbed/raw/master/imgs/20201213200119.png)

在缓存有效期内，重复调用该接口只会执行一次方法

在`redis`中存储的数据格式如下，其中`user::1`是本次缓存内容的`key`，有效期60秒

![image-20201213200243689](https://gitee.com/p8t/picbed/raw/master/imgs/20201213200244.png)

猜测`key`由三部分组成，第一部分是必须定义的`cacheNames`，代表该缓存数据与`user`相关联，语义上可以认为把该数据放在`user`这个缓存中。第二部分不知道。第三部分代表该缓存数据`id`，一般需要自己指定。上面显示的1来自方法参数`id`

本质上，对于加了缓存注解的方法，会在调用该接口时根据注解信息以及方法参数映射为一个用于访问缓存数据库的`key`，比如上面的`user::1`。然后去查看缓存中是否有数据

**注解参数详解**

- `cacheNames`：指定数据缓存位置，可以指定多个，这样就会有多个缓存实例

- `key`：默认为方法参数的值。它是缓存`key`的第三部分，也就是上面`user::1`中的1。支持`SpEL`表达式

  ```java
  @Cacheable(cacheNames = "user", key = "'findById['+#id+']'")
  @GetMapping("/cache/{id}")
  public Usr select(@PathVariable("id") int id) {
      System.out.println("查询用户, 没走缓存");
      return usrMapper.findById(id);
  }
  // 参数id为1时, redis中的key为 user::findById[1]
  ```

- `keyGenerator`：自定义一个`key`生成器，和`key`的作用一样，只是生成策略是是单独的一个类，该类需要实现`KeyGenerator`

  ```java
  @Configuration
  public class CacheConfig {
      @Bean
      public KeyGenerator keyGenerator() {
          return new KeyGenerator(){
              @Override
              public Object generate(Object target, Method method, Object... params) {
                  return method.getName() + Arrays.asList(params).toString();
              }
          };
      }
  }
  
  @Cacheable(cacheNames = "user", keyGenerator = "keyGenerator")
  @GetMapping("/cache/{id}")
  public Usr select(@PathVariable("id") int id) {
      System.out.println("查询用户, 没走缓存");
      return usrMapper.findById(id);
  }
  ```

- `cacheManager`：指定使用哪一个缓存管理器，也就是缓存存在哪

- `condition`：满足条件才缓存，只能在方法执行前判断，也就是不能根据结果决定是否缓存

  ```java
  // 只有参数id大于1才缓存
  @Cacheable(cacheNames = "user", condition = "#id > 1")
  @GetMapping("/cache/{id}")
  public Usr select(@PathVariable("id") int id) {
      System.out.println("查询用户, 没走缓存");
      return usrMapper.findById(id);
  }
  ```

- `unless`：和`condition`判断条件相反，此外，它可以根据结果决定是否缓存

##### @CachePut

用于更新缓存，该缓存方法被调用时，一定会执行，并且在调用完成后把**方法返回值**写入缓存。

想要正确更新缓存，一定要保证缓存方法生成的`key`和更新方法生成的`key`一定要一致

##### @CacheEvict

用于删除缓存，根据生成的`key`删除

- `allEntries`：默认`false`。如果设置为`true`，则会删除其相关的所有缓存，相关指`key`的第一个字段。这种情况下也就不用指定`key`了
- `beforeInvocation`：默认`false`。是否在方法调用之前执行。针对的是方法出现异常的情况，如果方法未能成功执行，则不会删除。

##### @Caching

有时候一个方法需要多次销毁缓存或多次添加缓存，且它们的规则不一样，这时就需要组合注解

下面是官网的例子，方法调用会销毁两个地方的缓存

```java
@Caching(evict = { @CacheEvict("primary"), @CacheEvict(cacheNames="secondary", key="#p0") })
public Book importBooks(String deposit, Date date)
```

##### @CacheConfig

标在类上，指明该类缓存方法的公共信息

```java
@CacheConfig("books") 
public class BookRepositoryImpl implements BookRepository {
    @Cacheable
    public Book findBook(ISBN isbn) {...}
}
```

#### 4、小结

与缓存操作相关的三个注解，操作的都是`key`，添加、更新的数据则是方法返回值。

### Mybatis缓存

`Mybatis`缓存分为一级缓存和二级缓存

一级缓存：基于`sqlSession`，也就是只在同一个数据库连接下有效，默认开启。不可控

二级缓存：基于当前`mapper`，也就是在同一个`mapper`内有效，默认关闭。可控，默认使用一个`Map`缓存数据，可以实现`Mybatis`下的`Cache`接口，指定缓存介质

当二级缓存开启时，当前事务提交或者`sqlSession`关闭后，缓存数据会放入二级缓存。查询时，先查二级缓存，再查一级缓存

## 错误处理

### 1、全局异常处理

对于`Controller`层被`@RequestMapping`修饰的方法，`SpringBoot`提供了统一异常处理机制。

使用时需要额外编写一个`Controller`类，该类的性质和`Controller`层类的性质一样，需要加**`@ControllerAdvice`**注解，由于这个注解里面包含了`@Controller`注解，因此不需要额外添加它，然后根据需求决定是否添加`@ResponseBody`。

`Controller`类里面的方法都是添加`@RequestMapping`注解来匹配路由。**`ControllerAdvice`**类也一样，只不过它的方法不是匹配路由，而是匹配异常。用到`@ExceptionHandler`注解

#### 例子

下面是一个普通Controller类，可以认为里面会出现三类异常

- `ArithmeticException`
- `ArrayIndexOutOfBoundsException`
- 其它

```java
@RestController
public class ExceptionTestController {

    @GetMapping("/ex/{p1}/{p2}")
    public Result<Object> ex(@PathVariable("p1") int p1,
                             @PathVariable("p2") int p2) {
        int i = 1 / p1;	// ArithmeticException
        int[] j = new int[1];
        int k = j[p2];  // ArrayIndexOutOfBoundsException
        return new Result<>(CodeInfo.SUCCESS);
    }
}
```

下面是一个`ControllerAdvice`类，用来处理`Controller`层异常的类，针对上面三种异常都进行了相应处理

```java
@ResponseBody
@ControllerAdvice(basePackages = "cc.p8t.blog.controller") // 指明处理哪里的Controller异常
// 可以使用@RestControllerAdvice合并上面两个
public class ExceptionController {

    @ExceptionHandler(ArithmeticException.class)
    public Result<Object> ex1(ArithmeticException e) {
        System.out.println("出现异常: ArithmeticException.class");
        return new Result<>(CodeInfo.SERVER_METHOD_EXCEPTION_ERROR);
    }

    @ExceptionHandler(ArrayIndexOutOfBoundsException.class)
    public Result<Object> ex2(ArrayIndexOutOfBoundsException e) {
        System.out.println("出现异常: ArrayIndexOutOfBoundsException.class");
        return new Result<>(CodeInfo.SERVER_METHOD_EXCEPTION_ERROR);
    }

    @ExceptionHandler(Exception.class)
    public Result<Object> rootEx(Exception e) {
        System.out.println("出现异常: Exception.class");
        return new Result<>(CodeInfo.SERVER_METHOD_EXCEPTION_ERROR);
    }
}
```

当`ExceptionTestController`类的`RequestMapping`方法出现异常时，便会自动进入`ExceptionController`类进行异常匹配，然后执行相应的方法

### 2、404/5xx

对于经常遇到的404和5xx错误，只需要在特定位置下放上页面，出错时`SpringBoot`就会自动跳转过去

```sh
templates/error/404.html
templates/error/5xx.html
static/error/404.html
static/error/5xx.html
```

### 3、自定义异常错误信息

作用机制和第二个一样，使自定义异常匹配对应的状态码信息

```java
@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "CUSTOM EXCEPTION")
public class CustomException extends RuntimeException {

    public CustomException() {
        super();
    }

    public CustomException(String message) {
        super(message);
    }
}
```

```html
<body>
    <!-- 取的是上面异常code值 -->
	<h1 th:text="${status}">X</h1>
    <!-- 取的是上面异常reason值 -->
    <p th:text="${message}">X</p>
</body>
```

## lombok

```java
@Data // getter setter toString equals hashCode
@AllArgsConstructor
@NoArgsConstructor
@Slf4j
```

## 获取请求参数

```shell
@PathVariable
# 如果一个URL有多个路径变量可以用一个Map<String, String>统一接收

@RequestHeader
1、加参数获取指定请求头信息
2、不加参数获取所有请求头信息, 后面参数可选如下
    -HttpHeaders
    -Map<String,String>
```

## 多Profile

- `application.yml`一定会加载，因此把相同的配置写在这里面
- 同名配置以选取的`profile`优先

```yaml
application.yml
application-dev.yml
application-pro.yml

# 1、在application.yml激活要使用的profile
spring:
  profiles:
    active: dev
    
# 2、以命令行方式激活profile
java -jar xxx.jar --spring.profiles.active=dev
```

对于使用代码编写的配置，使用`@Profile`选择加载

可以标注`@Profile`的注解

- `@Component`
- `@Configuration`
- `@ConfigurationProperties`

```java
@Configuration
public class DruidConfig {

    @Profile("dev")
    @Bean("dataSource")
    public DataSource devDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/db?serverTimezone=Asia/Shanghai");
        dataSource.setUsername("root");
        dataSource.setPassword("pwd");
        return dataSource;
    }
    @Profile("pro")
    @Bean("dataSource")
    public DataSource proDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://10.2.5.251:3306/db?serverTimezone=Asia/Shanghai");
        dataSource.setUsername("root");
        dataSource.setPassword("pwd");
        return dataSource;
    }
}
```

## 表单REST支持

配置文件开启支持

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true
```

对表单不能发送的`post`和`delete`请求，填写一个隐藏输入框，`value`指定请求类型

```html
<form action="/user" method="get">
</form>
<form action="/user" method="post">
</form>
<form action="/user" method="post">
    <input name="_method" value="PUT" type="hidden">
</form>
<form action="/user" method="post">
    <input name="_method" value="DELETE" type="hidden">
</form>
<form action="/user" method="post">
    <input name="_method" value="PATCH" type="hidden">
</form>
```

原理：通过过滤器，判断是否为正常的`post`请求，然后获取`_method`参数的值，根据值对请求方式进行修改，然后放行

## 过滤器

过滤器是原生`Servlet`的功能，而拦截器是`Spring`提供的功能

```java
@Order(1)	// 多个过滤器时, 该数字越小越先执行
@Component	// 只要注册Bean, SpringBoot就会自动调用
public class FilterOrder1 implements javax.servlet.Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
        System.out.println("filter 1");
        chain.doFilter(servletRequest, servletResponse);
    }
}
```
