## 摘要

定义算法框架，并将一些步骤的实现延迟到子类。

通过模板方法，子类可以重新定义算法的某些步骤，而不用改变算法的结构。

## 类图

![image-20201020184227805](https://gitee.com/p8t/picbed/raw/master/imgs/20201020184325.png)

## 实现

```java
public abstract class IClassLoader {
    public abstract void load();
    public abstract void verify();
    public abstract void prepare();
    public abstract void resolve();
    public abstract void initialize();

    public final void loadClass() {
        load();
        verify();
        prepare();
        resolve();
        initialize();
    }
}
```

```java
public class ClassLoader extends IClassLoader {
    @Override
    public void load() {
        System.out.println("loading...");
        System.out.println("loaded");
    }

    @Override
    public void verify() {
        System.out.println("verifying...");
        System.out.println("verified");
    }

    @Override
    public void prepare() {
        System.out.println("preparing...");
        System.out.println("prepared");
    }

    @Override
    public void resolve() {
        System.out.println("resolving...");
        System.out.println("resolved");
    }

    @Override
    public void initialize() {
        System.out.println("initializing...");
        System.out.println("initialized");
    }
}
```

## 测试

```java
public static void main(String[] args) {
    IClassLoader loader = new ClassLoader();
    loader.loadClass();
}
```

## AQS

JUC包下的核心框架, ReentrantLock, CyclicBarrier都基于此实现

里面用到了大量模板方法
```java
// JDK11
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```