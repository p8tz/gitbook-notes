## 摘要

适配器模式，顾名思义，就是把原本不兼容的玩意，通过适配，使之兼容。

适配器模式一般分为三类：类适配器模式、对象适配器模式、接口适配器模式（缺省适配器模式）

以一个电压的例子来说明适配器模式。平时我们能直接使用的电压是220V的，但是我们的设备可能只能用5V，这时就需要适配器模式作为一个中间转换机构，把220转为5V

<br>

## 类适配器模式

被适配的类, 即电网给我们提供的220V电压, 不能直接拿来用

```java
public class Source {
    public int output220V() {
        return 220;
    }
}
```

目标接口, 包含我们所需要电压的方法

```java
public interface Target {
    int output5V();
}
```

目标接口具体实现, 也就是适配类

```java
public class Adapter extends Source implements Target {
    @Override
    public int output5V() {
        return output220V() / 44;
    }
}
```

测试

```java
public class Test {
    public static void main(String[] args) {
        Adapter adapter = new Adapter();
        System.out.println(adapter.output5V());
    }
}
```

<br>

## 对象适配器模式

对于类适配器模式，有一定的弊端: 耦合性太高，我们知道在类与类的六大关系中，泛化的耦合性是最高的，而这里适配类直接继承了被适配的类，不太好。因此我们可以使用对象适配器模式，把继承转为组合：将被适配的类作为属性，通过构造方法赋值

Source类和Target接口不变，只变Adapter

```java
public class Adapter implements Target {
    private Source source;

    public Adapter(Source source) {
        this.source = source;
    }

    @Override
    public int output5V() {
        return source.output220V() / 44;
    }
}
```

测试

```java
public class Test {
    public static void main(String[] args) {
        Adapter adapter = new Adapter(new Source());
        System.out.println(adapter.output5V());
    }
}
```
<br>

## 接口适配器模式

这东西减少了代码冗余和侧重点不明显的问题。有时候我们可能会遇到这么一种情况，一个接口有许多方法，但是用的时候只会用到其中一个或两个方法，又因为接口的特性，我们必须实现所有的方法，这样写不仅显得冗余，而且看不出来我们的侧重点。

因此，出现了接口适配器模式，我们可以在用一个抽象类空实现接口的所有方法，然后我们用的时候不再直接使用接口，而是使用抽象类，重写抽象类里面我们需要的方法即可

在Java源码里面有很多以Adapter结尾的类，这些类就是上面说的抽象类，空实现了接口的所有方法

比如在SpringMVC里的用来写配置的接口WebMvcConfigurer，里面有差不多二十个配置方法，我们往往需要什么才配置什么，不可能实现全部方法，因此Spring提供了一个抽象类WebMvcConfigurerAdapter，这个类实现了WebMvcConfigurer，我们可以通过它想配置什么就只写什么方法

```java
public interface WebMvcConfigurer {}
public abstract class WebMvcConfigurerAdapter 
    implements WebMvcConfigurer {}
```

不过，在JDK1.8开始就可以避免这种不得不实现接口全部方法的情况，因为1.8为接口加了一个default关键字，凡是被该关键字修饰的方法，其实现类可以不实现。上面例子里WebMvcConfigurer的方法就全变成了default方法，这也是我们在用SpringMVC配置类的时候，可以直接方便的使用WebMvcConfigurer的原因，而WebMvcConfigurerAdapter类也被加上了@Deprecated注解，官方不再推荐使用。

再看一个JDK源码接口适配的例子

```java
// 待实现的目标接口, 直接实现比较冗余
public interface WindowListener extends EventListener {
    public void windowOpened(WindowEvent e);
    public void windowClosing(WindowEvent e);
    public void windowClosed(WindowEvent e);
    public void windowIconified(WindowEvent e);
    public void windowDeiconified(WindowEvent e);
    public void windowActivated(WindowEvent e);
    public void windowDeactivated(WindowEvent e);
}
```



```java
// 中间层, 继承这个按需实现
public abstract class WindowAdapter
    implements WindowListener, WindowStateListener, WindowFocusListener {
    public void windowOpened(WindowEvent e) {}
    public void windowClosing(WindowEvent e) {}
    public void windowClosed(WindowEvent e) {}
    public void windowIconified(WindowEvent e) {}
    public void windowDeiconified(WindowEvent e) {}
    public void windowActivated(WindowEvent e) {}
    public void windowDeactivated(WindowEvent e) {}
    public void windowStateChanged(WindowEvent e) {}
    public void windowGainedFocus(WindowEvent e) {}
    public void windowLostFocus(WindowEvent e) {}
}
```

## 总结

1. 类适配器和对象适配器的区别就是对要适配的类进行泛化还是组合
2. 接口适配器作用直观, 中间套一层抽象类简化代码, 可以用`default`避免接口适配

<br>