## 摘要

**耦合度排序**

依赖 => 关联 => 聚合 => 组合 => 泛化 => 实现


## 1. 依赖(Dependency)

只要A类用到B类就可以说A依赖B

<br>


## 2. 泛化(Generalization)

属于依赖. Java里对应extends


```java
class Dog extends Animal {
    public Dog() {
    }
}
```

<br>

## 3. 实现(Realization)

属于依赖. Java里对应implements

```java
class Phone implements USBC {
    public SmartPhone() {
    }
}
```

<br>

## 4. 关联(Association)

属于依赖. 体现为B类作为A类的成员属性

<br>

## 5. 聚合(Aggregation)

属于关联. 一般来说如果A类和B类存在关联关系, 则A与B是同一层次上的东西, 而聚合是整体和部分的关系, 并且可以整体与部分可以分离(不依赖整体的构造方法创建部分)

```java
class Computer {
    Mouse mouse;
    Keyboard keyboard;
    public Computer() {
    }
    // 不与整体共存亡
    public void setMouse(Mouse mouse) {
        this.mouse = mouse;
    }
    public void setKeyboard(Keyboard keyboard) {
        this.keyboard = keyboard;
    }
}
```

<br>

## 6. 组合(Composition)

属于关联. 在聚合基础上整体与部分同生共死(在构造方法中实例化对象)

```java
class Computer {
    Mouse mouse;
    Keyboard keyboard;
    // 整体与部分共存亡
    public Computer(Mouse mouse, Keyboard keyboard) {
        this.mouse = mouse;
        this.keyboard = keyboard;
    }
}
```