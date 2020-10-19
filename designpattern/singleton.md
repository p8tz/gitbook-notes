## 实现方式

1. 构造函数私有化

2. 在类的内部创建实例

3. 提供获取唯一实例的方法

## 1. 饿汉式

```java
public class Singleton {
    // 构造私有化, 不可通过new创建对象
    private Singleton() {}
    // 类内部实例化
    private static Singleton instance = new Singleton();
    // 提供获取唯一实例的方法
    public static Singleton getInstance() {
        return instance;
    }
}
```
- 线程安全
- 如果从始至终未使用该类的实例, 则会造成内存浪费

## 2. 懒汉式
延时初始化, 解决内存浪费的问题

### 2.1 简单懒汉式
```java
public class Singleton {
    // 构造私有化, 不可通过new创建对象
    private Singleton() {}
    // 先不实例化
    private static Singleton instance = null;
    // 提供获取唯一实例的方法
    public static Singleton getInstance() {
        // 如果未实例化, 则创建对象
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
- 线程不安全

  

可以简单的加一个锁synchronized
```java
public class Singleton {
    // 构造私有化, 不可通过new创建对象
    private Singleton() {}
    // 先不实例化
    private static Singleton instance = null;
    // 提供获取唯一实例的方法 加一个锁
    public static synchronized Singleton getInstance() {
        // 如果未实例化, 则创建对象
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
- 线程安全
- 锁范围较大, 以后每一次获取instance都需要加锁，效率低, 需要缩小锁的范围

### 2.2 双检锁懒汉式(DCL Double-Check Locking)

```java
public class Singleton {
    // 构造私有化, 不可通过new创建对象
    private Singleton() {}
    // 加 volatile 禁止指令重排序, 防止返回未初始化完成的实例
    private static volatile Singleton instance = null;
    public static Singleton getInstance() {
        // 只要实例创建出来, 以后每次获取都进不去这个if, 也不会存在抢锁这个消耗性能的动作上
        if (instance == null) {
            // 减小锁的范围
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```



## 3. 静态内部类懒汉式

```java
public class Singleton {
    private Singleton() {}
    // 使用静态内部类
    private static class SingletonHolder {
        private static final Singleton instance = new Singleton();
    }
    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
}
```
- 线程安全
- 内部类的加载是延时的. 只有在第一次使用才会加载, 很好的契合了单例模式的延时加载需求

## 4. 枚举实现

```java
public enum Singleton {
    INSTANCE;
}
```

<br>

<div class="note note-primary">参考文章</div>
<a class="btn" href="https://mp.weixin.qq.com/s/dU_Mzz76h-qQZvrgeSe44g">Java3y</a>