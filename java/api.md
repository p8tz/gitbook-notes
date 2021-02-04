## 文件

### `File`

构造函数

```java
// 如果写相对路径, 则以当前项目根目录为相对位置
File file = new File("1.jpg");	// project_dir/1.jpg
// 绝对路径
File file = new File("D:\\1.jpg");
// 自动拼接
File file = new File("dir", "file");
```

**下面三个都是`JDK1.7`新出的，用来取代`File`**

### `Paths`

```java
public static Path get(String first, String... more) {
    return Path.of(first, more);
}
public static Path get(URI uri) {
    return Path.of(uri);
}
```

### `Path`

#### 创建

```java
Path path = FileSystems.getDefault().getPath("D:\\1.jpg");
// 下面三个本质都是上面这个
Path path = new File("D:\\1.jpg").toPath();
Path path = Paths.get("D:\\1.jpg");
Path path = Path.of("D:\\1.jpg");	// JDK11
```

#### 常用方法

```java
Path.of("/a/b/c/1.jpg")

boolean isAbsolute() 
Path getFileName()	  // 1.jpg
Path getParent()      // /a/b/c
Path toAbsolutePath() // /a/b/c/1.jpg

// 注意是按路径匹配, 不是字符串
boolean endsWith(Path other) 
boolean endsWith(String other) 
boolean startsWith(Path other) 
boolean startsWith(String other) 
    
// 消除冗余: [/a/b///c//1.jpg] --> [/a/b/c/1.jpg]
Path normalize()

// 构造相对路径
Path relativize(Path other)
Path path = Path.of("/a/b/c/1.jpg");
path.relativize(Path.of("/a/b/d"));	  // ../../d
```

### `Files`

> 原文链接：[暹罗猫](https://blog.csdn.net/qq877728715/article/details/104499687/)

用于配合`Path`使用

#### 判断

```java
static boolean exists(Path path, LinkOption... options) 
static boolean notExists(Path path, LinkOption... options) 
static boolean isDirectory(Path path, LinkOption... options) 
static boolean isExecutable(Path path)
static boolean isHidden(Path path) // 只能判断文件
static boolean isReadable(Path path)
static boolean isWritable(Path path) 
```

#### 删除

```java
static boolean deleteIfExists(Path path) 
static void delete(Path path) 
```

#### 创建

```java
static Path createDirectories(Path dir, FileAttribute<?>... attrs) 
static Path createDirectory(Path dir, FileAttribute<?>... attrs) 
static Path createFile(Path path, FileAttribute<?>... attrs) 
```

#### 复制

```java
// 只能复制文件
static long copy(InputStream in, Path target, CopyOption... options) 
static long copy(Path source, OutputStream out) 
static Path copy(Path source, Path target, CopyOption... options) 
```

#### 移动和重命名

```java
static Path move(Path source, Path target, CopyOption... options) 
```

#### 遍历

```java
static Stream<Path> list(Path dir) 

// DFS列出全部
static Stream<Path> walk(Path dir)
```

## 日期

### 1、`Date`

获取一个指定的日期不是很友好，而且日期格式交互较差

```java
public static void main(String[] args) {
    Date date1 = new Date();
    System.out.println(date1);
    Date date2 = new Date(System.currentTimeMillis() + 60 * 60 * 1000);
    System.out.println(date2);
    // Fri Dec 11 22:01:33 CST 2020
    // Fri Dec 11 23:01:33 CST 2020
}
```

### 2、`SimpleDateFormat`

解决了`Date`类构造日期不方便以及显示格式的问题，但是构造一个相对日期还是不方便

```java
public static void main(String[] args) {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    // 构造日期对象
    Date date = sdf.parse("2020-12-11 22:04:26");
    System.out.println(date);

    // 格式化显示日期
    String f = sdf.format(new Date());
    System.out.println(f); // 2020-12-11 22:08:52
}
```

### 3、`Calendar`

方便构造一个相对日期，配合`SimpleDateFormat`使用，可以格式化显示一个相对时间

```java
public static void main(String[] args) {
    Calendar c = Calendar.getInstance();
    System.out.println(c.getTime());
    c.add(Calendar.DATE, 1);	// 当前时间基础上, 增加一天
    System.out.println(c.getTime());
    // Fri Dec 11 22:17:09 CST 2020
    // Sat Dec 12 22:17:09 CST 2020
}
```

### 4、`Java8`新增

`LocalDate、LocalTime、LocalDateTime`，分别表示日期，时间，日期时间

```java
public static void main(String[] args) {
    LocalDate ld = LocalDate.now();
    LocalTime lt = LocalTime.now();
    LocalDateTime ldt = LocalDateTime.now();

    System.out.println(ld);
    System.out.println(lt);
    System.out.println(ldt);
    // 2020-12-12
	// 09:21:28.751124700
	// 2020-12-12T09:21:28.751124700
    
    LocalDateTime ldt1 = LocalDateTime.now();
    System.out.println(ldt1.toLocalTime());
    LocalDateTime ldt2 = ldt1.plusHours(1);
    System.out.println(ldt2.toLocalTime());
    // 09:32:59.199525900
	// 10:32:59.199525900
}
```

## JWT

验证过程

![image-20201213091709699](https://gitee.com/p8t/picbed/raw/master/imgs/20201213091711.png)

用户不敏感信息可以写入`JWT`的`payload`区，比如用户id。如果需要放置敏感信息可以使用配合`redis`，或者只使用`redis`，这时候`token`只需要记录一个`UUID`，然后去`redis`取数据即可。

### 组成

- `header`：令牌类型，签名算法。使用base64编码

  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

- `payload`：存放非敏感数据。使用base64编码

  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

- `signature`：对`header + payload + salt`进行签名

### 使用

```java
class T {
    String SALT = "%#)@^%&3@43#BDS&^WE45UHBDC^DTFUBafysdfb7sgrfhb^%(";
    Algorithm ALGORITHM = Algorithm.HMAC256(SALT);

    @Test
    void contextLoads() {
        Calendar c = Calendar.getInstance();
        c.add(Calendar.MINUTE, 1);
        Date date = c.getTime();

        // 获取token  
        // 获取后写入响应体, 让浏览器保存在本地
        // 然后前端每次请求都携带这个token
        String token = JWT.create()
            // header不需要指定用了什么签名算法
            // 在签名时, 会把用到的签名算法自动填写进去, 也就是下面的sign()方法
            
            // payload
            .withClaim("userId", "123")
            .withExpiresAt(date)
            
            // signature
            .sign(ALGORITHM);

        // 验证token  验证失败会抛出异常
        // 通过拦截器对受限资源统一验证
        DecodedJWT decodedJWT = JWT.require(ALGORITHM)
            .build()
            .verify(token);

        // 获取payload信息
        // 对验证过的token, 可以从中获取用户信息
        String userId = decodedJWT.getClaim("userId").asString();
        System.out.println(userId);	// 123
    }
}
```

## 自定义异常

```java
public class CustomException extends RuntimeException {

    public CustomException() {
        super();
    }

    public CustomException(String message) {
        super(message);
    }
}
```

## JUnit 5

### 1、注解

| --              | --                                                        |
| --------------- | --------------------------------------------------------- |
| `@DisplayName`  | 标注测试类、方法的名称                                    |
| `@BeforeEach`   | 每一个单元测试之前都执行，一个`@Test`方法即是一个单元测试 |
| `@AfterEach`    | 每一个单元测试之后都执行                                  |
| `@BeforeAll`    | 所有单元测试之前执行一次，方法必须是静态方法              |
| `@AfterAll`     | 所有单元测试之后执行一次，方法必须是静态方法              |
| `@Disabled`     | 标注的单元测试不会执行                                    |
| `@Timeout`      | 规定方法超时时间，超出则抛异常                            |
| `@RepeatedTest` | 测试次数                                                  |

### 2、assert

一旦有断言失败，则后面的代码都不会执行

| --                    | --                               |
| --------------------- | -------------------------------- |
| `assertEquals()`      | 对象用`equals`判断，值用`==`判断 |
| `assertSame()`        | 判断是否为同一个对象             |
| `assertArrayEquals()` | 判断数组元素是否相等             |
| `assertAll()`         | 里面断言都成功才通过测试         |
| `assertNotNull()`     | 不为`null`                       |
| `assertTimeout()`     | 超时                             |
| `fail()`              | 执行到这个方法直接失败           |

### 3、assume

和断言类似，但是它只会终止方法，并不会报告测试失败

## Jackson

### JDK

```java
User user = new User();

// 序列化为字符串
ByteArrayOutputStream bao = new ByteArrayOutputStream();
ObjectOutputStream oos = new ObjectOutputStream(bao);
oos.writeObject(user);
String s = bao.toString(StandardCharsets.ISO_8859_1);
oos.close();

// 反序列化
ByteArrayInputStream bis = new ByteArrayInputStream(s.getBytes(StandardCharsets.ISO_8859_1));
ObjectInputStream ois = new ObjectInputStream(bis);
User user1 = (User) ois.readObject();
ois.close();

System.out.println(user1);
```

### 序列化

```java
ObjectMapper om = new ObjectMapper();
String s = om.writeValueAsString(new User());
System.out.println(s);
```

### 反序列化

不涉及泛型

```java
ObjectMapper om = new ObjectMapper();
String ser = om.writeValueAsString(new User());
User deser = om.readValue(ser, User.class);
System.out.println(deser);
```

涉及泛型

```java
ObjectMapper om = new ObjectMapper();
List<User> list = List.of(new User(), new User());
String ser = om.writeValueAsString(list);
List<User> deser = om.readValue(ser, new TypeReference<>(){});
System.out.println(deser);

Map<String, User> map = Map.of("a", new User(), "b", new User());
String ser = om.writeValueAsString(map);
Map<String, User> deser = om.readValue(ser, new TypeReference<>(){});
System.out.println(deser);

Set<User> set = Set.of(new User());
String ser = om.writeValueAsString(set);
Set<User> deser = om.readValue(ser, new TypeReference<>(){});
System.out.println(deser);
```

### 注解

| --                      | --                                                           |
| ----------------------- | ------------------------------------------------------------ |
| `@JsonProperty`         | 标在属性上。指定序列化后的key                                |
| `@JsonIgnore`           | 标在属性上。序列化时忽略该属性                               |
| `@JsonIgnoreProperties` | 标在类上。自动忽略无法序列化/反序列化的属性（无`Getter/Setter`），也可以指定忽略 |
| `@JsonFormat`           | 标在属性上。指定日期格式化                                   |





