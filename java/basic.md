## 数据类型

### 基本类型

关于boolean占多大空间的问题

官方没有明确说明。在`hotspot`虚拟机下，使用`jol`测试，显示占用一个字节

```java
class A {
    int a;
    boolean b;
    byte c;
    char d;
}
/*
OFFSET  SIZE          TYPE DESCRIPTION                               VALUE
    0    12            (object header)                                N/A
   12     4             int A.a                                       N/A
   16     2            char A.d                                       N/A
   18     1         boolean A.b                                       N/A
   19     1            byte A.c                                       N/A
...
*/
```

### 包装类型

从`JDK1.5`开始，出现了自动装箱 / 自动拆箱机制，原理就是自动在编译期自动加上对应的方法

```java
Integer x = 10;     // Integer x = Integer.valueOf(2)
int y = x;          // int y = x.intValue()
```

### 缓存池

除了浮点型的包装类，都存在缓存池

- `new Integer(10)`直接创建一个新对象
- `Integer.valueOf(10)`先从缓存池中拿，没有则创建

`Integer`的缓存池大小为`[-128, 127]`

```java
Integer a = Integer.valueOf(100);
Integer b = 100;    // 相当于上面的方式
Integer c = new Integer(100);
System.out.println(a == b); // true
System.out.println(a == c); // false

Integer x = 128;
Integer y = 128;
System.out.println(x == y); // false
```

只有`Integer`的缓存区是可以调整大小的，并且只能调整上界，通过虚拟机参数设置

## String

### 底层实现

1.8是`char[]`，1.9是`byte[]`

### 创建方式

```java
// 去字符串常量池找
//   已存在，则直接引用它
//   不存在，则在串池中创建一个并引用它
String s1 = "kotlin";

// 先去字符串常量池
//   已存在，则去堆中new一个并引用
//   不存在，则在串池中创建一个，再去堆中new一个并引用
String s2 = new String("kotlin");
```

### 字符串拼接

```java
// 字符串拼接操作时，只要有一边是变量就会
// 创建StringBuilder然后append以完成字符串的拼接
String s1 = "Hello";
String s2 = "Kotlin";
String s3 = s1 + s2;

// 因此多次分开拼接字符串会创建多个StirngBuilder，导致效率问题
```

### 编译期优化

```java
// 对于final修饰的字面量字符串，会在编译期进行一些优化
final String s1 = "Hello";
final String s2 = "Kotlin";
// 相当于 String s3 = "Hello" + "Kotlin";
String s3 = s1 + s2;

// 编译后s3，这样就不需要运行时创建StringBuilder了
String s3 = "HelloKotlin";
```

### intern()

`JDK1.7`之前

调用该方法时，会查询串池中是否有该字符串

- 如果有则返回串池中的引用
- **如果没有则在串池中创建并返回引用**

`JDK1.7`开始

调用该方法时，会查询串池中是否有该字符串

- 如果有则返回串池中的引用
- **如果没有则在串池中记录当前对象的引用并返回**，相当于不新建了，串池中的也指向堆区中的

### 几个题

```java
// 前提: 串池为空

// 1. 下面分别造了几个对象

// 两个, 串池中一个, 堆区一个
new String("ab");
// 可以认为是5个
// new StringBuilder()
// 串池中的 "a"
// 堆区中的 "a"
// 串池中的 "b"
// 堆区中的 "b"
// 更深入一点, toString()还会new一个"ab", 但是串池中没有"ab"
new String("a") + new String("b");

// 2. 针对上面说的看个比较难的题
String s1 = new String("1") + new String("1");
s1.intern();
String s2 = "11";
System.out.println(s1 == s2);  // jdk1.6为false, jdk1.7为true
// 原因: 
/*
	s1执行intern前, 串池中没有"11"
	s1执行intern后
		jdk1.6: 串池中新建"11"
		jdk1.7: 串池中保存指向堆区"11"的引用
	String s2 = "11"执行后
		jdk1.6: 指向的"11"和s1显然没关系
		jdk1.7: s2指向的"11", 实际上指向的是s1的引用
*/
```

## 接口

```java
/*
JDK7接口可以包含
1. 静态常量
2. 抽象方法

JDK8增加了
3. 默认方法
4. 静态方法

JDK9增加了
5. 私有方法
*/

public interface Demo {
    // 1、静态常量
    [public static final] int N = 10;
	
    // 2、抽象方法
    [public abstract] void m1();
    
    // 3、默认方法
    [public] default void m2() {}
    
    // 4、静态方法
    [public] static void m3() {}
    
    // 5、私有方法 [只能在接口内部使用, 可用于封装重复代码的问题]
    // (1) 普通私有, 为默认方法提供服务
    private void m4() {}
    // (2) 静态私有, 为静态方法提供服务
    private static void m5() {}
}
```

- 接口没有静态代码块
- 接口没有构造方法
- 如果实现的多个接口有重复的**抽象方法**, 则只需覆盖重写一次
- 如果实现的多个接口有重复的**默认方法**, 则必须覆盖重写冲突的默认方法
- 如果一个类的父类方法和实现接口的默认方法冲突, **优先使用父类的方法**
- 接口可以多**继承**接口 如果默认方法冲突必须覆盖重写而且要有default关键字

## 异常

`Java`异常分为`Exception`和`Error`。`Error`表示虚拟机无法处理的异常，最常见的有`OOM`和栈溢出。

`Exception`可以分为`Checked Exception`和`Unchecked Exception`

- `Checked Exception`：必须处理的异常，比如`IOException`。除了`RuntimeException`及其子类外都是`Checked Exception`
- `Unchecked Exception`：可以不处理的异常，比如NPE

![w95klD.png](https://gitee.com/p8t/picbed/raw/master/imgs/20201116194856.png)

出现`Exception`可以通过`try/catch`或`throws`来处理。其中`try/catch`是我们手动处理异常，而用`throws`则当前方法不处理，交给方法调用者来处理，如果到最后都没人处理则由虚拟机来处理，虚拟机中断线程并调用`e.printStackTrace()`

还有个关键字`throw`，用来手动抛出异常。如果`throw`一个`RuntimeException`或`Error`，我们可以不处理，默认交给JVM处理

## 反射

## 注解

## HashCode

**`HashCode`只有在用到HashMap，Set这类去重集合时才有实质性作用。**

对于这类集合，插入元素时：

- 首先通过`HashCode`计算出桶的位置
  - 如果桶为空，则插入
  - 如果桶非空，则通过`equals`一一比较，判断有相同的则插入失败。

因此如果只重写了`equals`，那么即使插入相同的元素（通过`equals`比较），它们也会被分配在不同的桶中（有极低的概率正好映射在一个桶），然后调用`equals`方法自然比较不到相同的元素，就会有重复的元素在集合中

**总之，重写`hashCode()`是为了保证相同的元素（通过`equals`比较）映射到同一个桶中。在重写`equals()`时也应当重写`hashCode()`**

下面代码如果只重写了`equals`，那么会打印2，如果`equals`和`hashCode`都重写了就打印1

```java
@Test
public void test() {
    Set<Person> set = new HashSet<>();
    set.add(new Person("Y"));
    set.add(new Person("Y"));
    System.out.println(set.size());
}
```

## 初始化代码块执行顺序

首先，静态代码块一定是最先加载并且只加载一次的，这是JVM类加载机制决定的（加载 => 验证 => 准备 => 解析 => 初始化，其中在初始化阶段会收集静态变量赋值语句以及`static`中的语句构成`<clinit>()`方法来执行，此后遇到`new`才轮到对象的初始化）。 其次，父类的构造先于子类构造执行，普通代码块先于构造执行。因此，正确的执行顺序如下

父类静态代码块 => 子类静态代码块 => 父类普通代码块 => 父类构造方法 => 子类普通代码块 => 子类构造方法

```java
class A {
    static {
        System.out.println("father : static");
    }
    
    {
        System.out.println("father : non-static");
    }

    public A() {
        System.out.println("father : constructor");
    }
}

class B extends A {
    static {
        System.out.println("son : static");
    }

    {
        System.out.println("son : non-static");
    }

    public B() {
        System.out.println("son : constructor");
    }
}

public class InitTest {
    @Test
    public void test() {
        new B();
    }
}
/*
    father : static
    son : static
    father : non-static
    father : constructor
    son : non-static
    son : constructor
*/
```



## 遍历HashMap

```java
@Test
public void test01() {
    Map<String, String> map = new HashMap<>(Map.of("k1", "v1", "k2", "v2", "k3", "v3"));

    // 1. 利用keySet() + 函数式接口
    map.keySet()
        .forEach(it -> System.out.println(it + " = " + map.get(it)));

    // 2. 利用keySet() + 迭代器
    Iterator<String> it = map.keySet().iterator();
    while (it.hasNext()) {
        String key = it.next();
        System.out.println(key + " = " + map.get(key));
    }

    // 语法糖版本
    for (String key : map.keySet()) {
        System.out.println(key + " = " + map.get(key));
    }

    // 3. map的key,value都是封装在一个个node对象中的, 
    //    可以直接获取这些node的set集合, 再用迭代器
    Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
    while (it.hasNext()) {
        Map.Entry<String, String> kv = it.next();
        System.out.println(kv.getKey() + " = " + kv.getValue());
    }

    // 语法糖版本
    for (Map.Entry<String, String> kv : map.entrySet()) {
        System.out.println(kv.getKey() + " = " + kv.getValue());
    }

    // 4. 获取所有的值,无法遍历key
    Iterator<String> it = map.values().iterator();
    while (it.hasNext()) {
        String value = it.next();
        System.out.println(value);
    }

    // 语法糖版本
    for (String value : map.values()) {
        System.out.println(value);
    }
}
```

## fail-fast

`fail-fast`机制导致了**使用迭代器遍历集合**的时候，如果集合结构被修改则抛出`ConcurrentModificationException`异常（并不一定抛，因为是多线程，可能改了又改回来而迭代器没有检测到），目的是及时止损。这也导致了单线程环境处理不当也会抛异常。

```java
@Test
public void test() {
    List<String> list = new ArrayList<>(Arrays.asList("1", "2", "3", "4", "5"));

    // 在使用迭代器遍历集合的时候, 不能对集合进行删除或添加操作
    // 否则会抛并发修改异常 ConcurrentModificationException
    for (String it : list) {
        if (it.equals("3")) {
            list.remove(it);
        }
    }
	   
    // 1. 使用for正序遍历修改, 需要修改索引
    for (int i = 0; i < list.size(); i++) {
        if (list.get(i).equals("3")) {
            list.remove(i);
            // 注意修改索引
            i--;
        }
    }

    // 2. 使用for倒序遍历, 不需要修改索引
    for (int i = list.size() - 1; i >= 0; i--) {
        if (list.get(i).equals("3")) {
            list.remove(i);
        }
    }

    // 3. 使用迭代器的remove方法
    //    原理是它会对expectedModCount重新赋值
    Iterator<String> it = list.iterator();
    while (it.hasNext()) {
        String val = it.next();
        if ("3".equals(val)) {
            it.remove();
        }
    }

    // 4. JDK1.8可以用removeIf()
    //    原理是对迭代器的封装
    list.removeIf(item -> "3".equals(item));
    list.removeIf("3"::equals);
}
```

## fail-safe

`fail-safe`用于多线程环境下，任何对集合结构的修改都是对原集合拷贝的修改。

典型的`CopyOnWriteArrayList`集合就用了fail-safe机制，当有线程对其调用`add()`方法时，会加锁并拷贝一份进行添加操作，最后改下数组指向，而其它线程正在遍历（通过迭代器）的集合则是旧集合，不会产生并发修改问题，但**读的数据不是最新的**

```java
// JDK8
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // 先加锁
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 对原集合拷贝一份
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 对新集合进行添加操作
        newElements[len] = e;
        // 改变指向
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
// JDK11
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}
```

