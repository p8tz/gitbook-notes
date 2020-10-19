## 摘要

动态地给一个对象添加一些额外的职责。就增加功能来说，装饰模式相比生成子类更为灵活

<img src="https://s1.ax1x.com/2020/10/16/0bFT8s.png" alt="0bFT8s.png" border="0" />

## 实现

```java
// Component顶层抽象
public interface ICoffee {
    void show();
}

// ConcreteComponent, 也就是被装饰的类
public class Coffee implements ICoffee {
    @Override
    public void show() {
        System.out.println("I am coffee");
    }
}
```

```java
// Decorator, 实现顶层抽象以达到任意装饰的目的
public abstract class CoffeeDecorator implements ICoffee {
    // 内部组合一个被装饰的类
    protected ICoffee coffee;
    public CoffeeDecorator(ICoffee coffee) {
        this.coffee = coffee;
    }
}

// ConcreteDecoratorA
public class Milk extends CoffeeDecorator {
    public Milk(ICoffee coffee) {
        super(coffee);
    }

    @Override
    public void show() {
        System.out.println("add some milk");
        coffee.show();
    }
}

// ConcreteDecoratorB
public class Sugar extends CoffeeDecorator {
    public Sugar(ICoffee coffee) {
        super(coffee);
    }

    @Override
    public void show() {
        System.out.println("add some sugar");
        coffee.show();
    }
}
```

## 使用

```java
public static void main(String[] args) {
    // 想要装饰什么就套什么, 但最里面一层一定是被装饰的类
    ICoffee coffee = new Sugar(new Milk(new Coffee()));
    coffee.show();
    /*
        add some sugar
        add some milk
        I am coffee
    */
}
```

## JDK源码

**看一个JDK IO流读取字节的例子吧**

<img src="https://s1.ax1x.com/2020/10/16/0bBKsA.png" alt="0bBKsA.png" border="0" />

```java
// Component
public abstract class InputStream {
    // 以读一个字节方法为例
    public abstract int read() throws IOException;
}
// ConcreteComponent
public class FileInputStream extends InputStream {
    // 读一个字节方法的实现
    public int read() throws IOException {
        return read0();
    }

    private native int read0() throws IOException;
}

// Decorator
public class FilterInputStream extends InputStream {
    protected InputStream in;

    protected FilterInputStream(InputStream in) {
        this.in = in;
    }
}
// ConcreteDecorator
public class DataInputStream extends FilterInputStream {
    public DataInputStream(InputStream in) {
        super(in);
    }

    // 装饰类根据读一个字节的方法, 稍加装饰, 实现了读boolean的方法(非0为true)
    public final boolean readBoolean() throws IOException {
        int ch = in.read();
        if (ch < 0)
            throw new EOFException();
        return (ch != 0);
    }
}
```

## 总结

1. 装饰类和被装饰类可以独立发展，而不会相互耦合。换句话说，`Component`类无须知道`Decorator`类，Decorator类是从外部来扩展`Component`类的功能，而`Decorator`也不用知道具体的构件
2. 装饰模式是继承关系的一个替代方案。我们看装饰类`Decorator`，不管装饰多少层，返回的对象还是`Component`，实现的还是`is-a`的关系
3. 可以解决子类排列组合的继承, 导致类爆炸的情况

