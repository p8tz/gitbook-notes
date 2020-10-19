## 摘要

提供了一个统一的接口, 用来访问子系统中的一群接口,从而让子系统更容易使用。


```java
public class SystemA {
    public void printA() {
        System.out.println("system A");
    }
}
public class SystemB {
    public void printB() {
        System.out.println("system B");
    }
}
public class SystemC {
    public void printC() {
        System.out.println("system C");
    }
}
```

```java
// 中间加一层, 屏蔽复杂子系统对客户端的暴露
public class Facade {
    private SystemA a = new SystemA();
    private SystemB b = new SystemB();
    private SystemC c = new SystemC();

    public void printSystem() {
        a.printA();
        b.printB();
        c.printC();
    }
}
```

```java
public static void main(String[] args) {
    Facade facade = new Facade();
    facade.printSystem();
}
```