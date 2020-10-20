## 摘要

定义一系列算法，封装每个算法，并使它们可以互换。

策略模式可以让算法独立于使用它的客户端。

## 类图

![image-20201020182411768](https://gitee.com/p8t/picbed/raw/master/imgs/20201020182412.png)

## 实现

```java
public interface Transport {
    void go();
}

public class Bus implements Transport {
    @Override
    public void go() {
        System.out.println("attend by bus");
    }
}

public class Bike implements Transport {
    @Override
    public void go() {
        System.out.println("attend by bike");
    }
}
```

```java
public class Attend {
    private Transport transport;

    public Attend(Transport transport) {
        this.transport = transport;
    }

    public void go() {
        transport.go();
    }

    public void setTransport(Transport transport) {
        this.transport = transport;
    }
}
```

## 测试

```java
public static void main(String[] args) {
    Attend attend = new Attend(new Bus());
    attend.go();

    attend.setTransport(new Bike());
    attend.go();
}
```
