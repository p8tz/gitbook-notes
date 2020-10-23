## 摘要

用一个中介对象封装一系列的对象交互，中介者使各对象不需要显示地相互作用，从而使其耦合松散，而且可以独立地改变它们之间的交互

## 类图

![image-20201022170013978](https://gitee.com/p8t/picbed/raw/master/imgs/20201022170015.png)

## 实现

上下级之间通信, 上级发送的消息下级全都要收到, 下级发送请求只能上级收到

```java
public abstract class IMediator {
    public abstract void registerPosition(IPosition position);

    public abstract void transferMessage(String msg, IPosition from);
}

public class Mediator extends IMediator {
    // NPE
    private IPosition superior = new Superior(this);

    private List<IPosition> subordinates = new ArrayList<>();

    @Override
    public void registerPosition(IPosition position) {
        if (position.getClass() == Superior.class) {
            this.superior = position;
        } else {
            subordinates.add(position);
        }
    }

    @Override
    public void transferMessage(String msg, IPosition from) {
        if (from.getClass() == Superior.class) {
            subordinates.forEach(it -> it.receive(msg));
        } else {
            superior.receive(msg);
        }
    }
}
```

```java
public abstract class IPosition {
    protected IMediator mediator;

    public IPosition(IMediator mediator) {
        this.mediator = mediator;
    }

    public abstract void send(String msg);

    public abstract void receive(String msg);
}

public class Superior extends IPosition {
    public Superior(IMediator mediator) {
        super(mediator);
    }

    @Override
    public void send(String msg) {
        System.out.println("Superior sent message: " + msg);
        mediator.transferMessage(msg, this);
    }

    @Override
    public void receive(String msg) {
        System.out.println("Superior received message: " + msg);
    }
}

public class SubordinateA extends IPosition {
    public SubordinateA(IMediator mediator) {
        super(mediator);
    }

    @Override
    public void send(String msg) {
        System.out.println("SubordinateA sent message: " + msg);
        mediator.transferMessage(msg, this);
    }

    @Override
    public void receive(String msg) {
        System.out.println("SubordinateA received message: " + msg);
    }
}

public class SubordinateB extends IPosition {

    public SubordinateB(IMediator mediator) {
        super(mediator);
    }

    @Override
    public void send(String msg) {
        System.out.println("SubordinateB sent message: " + msg);
        mediator.transferMessage(msg, this);
    }

    @Override
    public void receive(String msg) {
        System.out.println("SubordinateB received message: " + msg);
    }
}
```

## 测试

```java
public static void main(String[] args) {
    IMediator mediator = new Mediator();

    IPosition superior = new Superior(mediator);
    IPosition subordinateA = new SubordinateA(mediator);
    IPosition subordinateB = new SubordinateB(mediator);

    mediator.registerPosition(superior);
    mediator.registerPosition(subordinateA);
    mediator.registerPosition(subordinateB);

    System.out.println("======================================");
    superior.send("hello");
    System.out.println("======================================");
    subordinateA.send("request A");
    System.out.println("======================================");
    subordinateB.send("request B");
    System.out.println("======================================");
}
/*
    ======================================
    Superior sent message: hello
    SubordinateA received message: hello
    SubordinateB received message: hello
    ======================================
    SubordinateA sent message: request A
    Superior received message: request A
    ======================================
    SubordinateB sent message: request B
    Superior received message: request B
    ======================================
*/
```

