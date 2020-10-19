## 摘要

工厂模式分为简单/静态工厂模式，工厂方法模式和抽象工厂模式，其主要目的是屏蔽了创建对象的复杂细节，注意这是对于复杂的创建对象而言，如果对象创建比较简单就没必要使用工厂模式，比如想要创建一个HashMap就不需要搞一个factory了

## 1. 简单/静态工厂模式

该模式对对象创建管理方式比较简单，通过向工厂传递类型来指定要创建的对象

假设有一个宠物店卖狗和猫

```java
// 动物接口
public interface Animal {
    void eat();
}
// 狗类
public class Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("狗吃肉");
    }
}
// 猫类
public class Cat implements Animal {
    @Override
    public void eat() {
        System.out.println("猫吃鱼");
    }
}
// 动物工厂
public class AnimalFactory {
    public static Animal create(Class type) {
        if (type == Cat.class) {
            return new Cat();
        } else if (type == Dog.class) {
            return new Dog();
        } else {
            throw new IllegalArgumentException();
        }
    }
}

public class Test {
    public static void main(String[] args) {
        Animal cat = AnimalFactory.create(Cat.class);
        cat.eat();
        
        Animal dog = AnimalFactory.create(Dog.class);
        dog.eat();
    }
}
```

问题

如果新增一个动物，就需要到AnimalFactory中修改原有的代码，违反了开闭原则

<br>

## 2.工厂方法模式

和简单工厂模式中相比，工厂方法模式将生产具体产品(对象)的任务分发给具体的产品工厂(类)

```java
// 动物接口
public interface Animal {
    void eat();
}
// 狗类
public class Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("狗吃肉");
    }
}
// 猫类
public class Cat implements Animal {
    @Override
    public void eat() {
        System.out.println("猫吃鱼");
    }
}
// 动物工厂接口
public interface AnimalFactory {
    Animal createAnimal();
}
// 猫工厂
public class CatFactory implements AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Cat();
    }
}
// 狗工厂
public class DogFactory implements AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Dog();
    }
}

public class Test {
    public static void main(String[] args) {
        AnimalFactory dogFactory = new DogFactory();
        Animal dog = dogFactory.createAnimal();
        dog.eat();

        AnimalFactory catFactory = new CatFactory();
        Animal cat = catFactory.createAnimal();
        cat.eat();
    }
}
```

</br>

## 3. 抽象工厂模式

上面两种模式不管工厂怎么拆分抽象，都只是针对一类产品Animal，现我们希望区分公母，也就是Dog类和Cat类都可以继续细分，这样就出现了两类产品。为了解决这个问题我们可以把它们拆分为两个工厂，这里可以使用抽象工厂模式

```java
// 动物顶层接口
public interface Animal {
    void eat();
    void gender();
}
// 狗的共性抽象类
public abstract class Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("狗吃肉");
    }
}
// 母狗
public class FemaleDog extends Dog {
    @Override
    public void gender() {
        System.out.println("I am a female dog");
    }
}
// 公狗
public class MaleDog extends Dog {
    @Override
    public void gender() {
        System.out.println("I am a male dog");
    }
}

// 猫类似就不贴代码了

// 动物工厂顶层接口
public interface AbstractAnimalFactory {
    Animal createDog();
    Animal createCat();
}

// 公狗/猫工厂   母狗/猫类似就不贴代码了
public class MaleAnimalFactory implements AbstractAnimalFactory {
    @Override
    public Animal createDog() {
        return new MaleDog();
    }

    @Override
    public Animal createCat() {
        return new MaleCat();
    }
}

public class Test {
    public static void main(String[] args) {
        AbstractAnimalFactory maf = new MaleAnimalFactory();
        Animal maleDog = maf.createDog();
        maleDog.eat();
        maleDog.gender();
    }
}
```

抽象工厂模式说到底就是多一层抽象，减少了工厂的数量

<br>







