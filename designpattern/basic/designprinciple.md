## 摘要

- 里氏替换原则（Liskov Substitution Principle）
- 单一职责原则（Single Responsibility Principle）
- 依赖倒置原则（Dependence Inversion Principle）
- 接口隔离原则（Interface Segregation Principle）
- 迪米特法则（Law of Demeter）
- 开闭原则（Open Closed Principle）

取首字母联合起来简称为**SOLID**

<br>

## 1. 里氏替换原则

在程序中将任意的基类替换为其子类，程序不受任何影响

简单点说，就是子类继承父类时，可以增加自己独有的方法，但是不能重写父类**已经实现**的方法

举个例子

```java
class A {
    public void calA(int x, int y) {
        System.out.println("x + y = " + (x + y));
    }
}

class B extends A {
    @Override
    public void calA(int x, int y) {
        System.out.println("x - y = " + (x - y));
    }

    public void calB(int x, int y) {
        System.out.println("x * y = " + (x * y));
    }
}
```

可以看到B实现A并且重写了A已经实现的方法cal，显然违反了里氏替换原则

<br>

## 2. 单一职责原则

所谓职责是指类变化的原因。如果一个类有多于一个的动机被改变，那么这个类就具有多于一个的职责。而单一职责原则就是指一个类或者模块应该有且只有一个改变的原因

简单点说，就是控制类的粒度大小，降低类的复杂度，一个类只负责一项职责

<br>

## 3. 依赖倒置原则

高层次的模块不应该依赖于低层次的模块，二者都应该依赖于其抽象；抽象不应该依赖于具体，具体应该依赖于抽象。

简单点说，就是要面向接口编程，实体类之间不直接发生依赖关系，其依赖关系是通过接口或抽象产生的

<br>

## 4. 接口隔离原则

客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上

也就是说，设计接口的时候，不要把一大堆方法写倒一个接口里面，应该根据功能不同拆分为多个接口

举个例子

```java
interface Animal {
    void eat();
    void run();
    void fly();
}

class Lion implements Animal {
    @Override
    public void eat() {
        System.out.println("狮子吃");
    }
    
    @Override
    public void fly() {

    }
    
    @Override
    public void run() {
        System.out.println("狮子跑");
    }
}
```

我们可以看到，由于我们把动物的三个方法写到一个接口里，导致了狮子不会飞也要实现fly方法，显然不合理。我们可以把Animal拆分为IEat，IFly，IRun三个接口让Lion选择性的实现

```java
interface IEat {
    void eat();
}

interface IFly {
    void fly();
}

interface IRun {
    void run();
}

class Lion implements IEat, IRun {
    @Override
    public void eat() {
        System.out.println("狮子吃");
    }

    @Override
    public void run() {
        System.out.println("狮子跑");
    }
}
```

<br>

## 5. 迪米特法则

一个对象应当对其他对象有尽可能少的了解。就是说要尽量降低类之间的耦合度，提高类的独立性，这样当一个类修改的时候，对其他类的影响也会降到最低

简单点说，就是一个类无论内部多复杂，对外只需要暴露出调用的方法即可

<br>

## 6. 开闭原则

对扩展开放，对修改关闭

前面5个原则的最终目的就是实现开闭原则

