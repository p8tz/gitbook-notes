## 文件

### File

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

### Paths

```java
public static Path get(String first, String... more) {
    return Path.of(first, more);
}
public static Path get(URI uri) {
    return Path.of(uri);
}
```

### Path

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

### Files

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

