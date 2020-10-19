## 摘要

将抽象部分与它实现部分分离，使它们都可以独立地变化。

表现出来就是解决了多维度扩展的问题

## 问题

图形可以分为矩形，圆形等，然后每个图形又可以分为红色，蓝色等。我们在设计类的时候可以这样设计

![a9E92Q.png](https://s1.ax1x.com/2020/07/26/a9E92Q.png)

具体代码

顶层Shape接口

```java
public interface Shape {
    void show();
}
```

图形类

```java
public class Circle implements Shape {

    @Override
    public void show() {
        System.out.println("circle");
    }
}

public class Rectangle implements Shape {

    @Override
    public void show() {
        System.out.println("rectangle");
    }
}
```

带颜色的图形类(违反了里氏替换原则)

```java
public class BlueCircle extends Rectangle {
    @Override
    public void show() {
        System.out.println("blue circle");
    }
}

public class RedCircle extends Rectangle {
    @Override
    public void show() {
        System.out.println("red circle");
    }
}
public class BlueRectangle extends Rectangle {
    @Override
    public void show() {
        System.out.println("blue rectangle");
    }
}
public class RedRectangle extends Rectangle {
    @Override
    public void show() {
        System.out.println("red rectangle");
    }
}
```

照这样继承下去, 有m个图形, n个颜色, 就会产生m * n个子类

显然出现了类爆炸的情况

下面桥接模式就解决了这个问题

## 解决方案

把Color抽象出来，然后通过关联，为Shape提供颜色特征, 这样做子类数量为 m + n (md, 当时画图不知道箭头怎么改样式, 就写字了)

![a9VoXd.png](https://s1.ax1x.com/2020/07/26/a9VoXd.png)

代码

```java
// Shape抽象
public abstract class Shape {
    protected Color color;

    public abstract void show();

    // 颜色可选的话, 就不要构造传入了, 放到子类setter里面
    public Shape(Color color) {
        this.color = color;
    }
}

// Color抽象
public abstract class Color {
    public abstract void showColor();
}
```

```java
public class Blue extends Color {
    @Override
    public void showColor() {
        System.out.println("blue");
    }
}

public class Red extends Color {
    @Override
    public void showColor() {
        System.out.println("red");
    }
}
public class Circle extends Shape {
    public Circle(Color color) {
        super(color);
    }

    @Override
    public void show() {
        color.showColor();
        System.out.println("circle");
    }
}

public class Rectangle extends Shape {
    public Rectangle(Color color) {
        super(color);
    }

    @Override
    public void show() {
        color.showColor();
        System.out.println("rectangle");
    }
}
```

测试

```java
public static void main(String[] args) {
    Shape circle = new Circle(new Blue());
    circle.show();
}
/*
	blue
	circle
*/
```

## 总结

1. 分离抽象与实现, 对应例子中把颜色抽象出来单独实现
2. 解耦, 颜色和形状不再是`泛化`关系, 而是`组合`起来

<br>