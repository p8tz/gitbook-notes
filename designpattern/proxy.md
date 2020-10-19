## 摘要

## 静态代理

代理者和被代理者实现同一个接口, 通过组合的方式扩展被代理者

```java
// 共同的接口
public interface IDogSeller {
    void sellDog();
}
// 被代理者
public class DogSeller implements IDogSeller {
    @Override
    public void sellDog() {
        System.out.println("sell a dog");
    }
}
// 代理者
public class PetStore implements IDogSeller{
    // 组合被代理者
    private DogSeller dogSeller;

    public PetStore(DogSeller dogSeller) {
        this.dogSeller = dogSeller;
    }

    // 扩展被代理者方法
    @Override
    public void sellDog() {
        System.out.println("PetStore...");
        dogSeller.sellDog();
    }
}

public static void main(String[] args) {
    PetStore petStore = new PetStore(new DogSeller());
    petStore.sellDog();
}
```

## 动态代理

JDK动态代理：基于接口

CGLib动态代理：基于类


### 1. JDK动态代理

- 基于接口实现，被代理者和代理者需要实现同一个接口。其中被代理者需要显示的实现接口，代理者则不需要，它是由Proxy类动态产生
- 通过反射调用方法
- 需要理解Proxy类和InvocationHandler接口

```java
// 顶层接口
public interface Shop {
    void sellDog();
    void sellCat();
}

// 被代理类
public class PetShop implements Shop {

    @Override
    public void sellDog() {
        System.out.println("sell a dog");
    }

    @Override
    public void sellCat() {
        System.out.println("sell a cat");
    }
}

// 处理增强方法的类, 封装了获取代理对象的方法
class ProxyHandler implements InvocationHandler {

    // 被代理对象
    private Object target;

    public ProxyHandler(Object target) {
        this.target = target;
    }

    // 获取代理类实例
    public Object getProxy() {
        // 获取代理对象需要
        //   目标类的类加载器
        //   目标类实现的接口: 可以得到需要增强的方法
        //   具体调用方法处理类
        Class<?> clazz = target.getClass();
        ClassLoader classLoader = clazz.getClassLoader();
        Class<?>[] interfaces = clazz.getInterfaces();
        // 动态生成代理类并返回实例, 继承了java.lang.reflect.Proxy
        return Proxy.newProxyInstance(classLoader, interfaces, this);
    }

    // 对被代理类方法的扩充, 也就是处理增强逻辑的核心所在
    // 代理类内部实际上就调用了这个方法
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before");
        // 调用被代理对象的原始方法, 没有返回值就返回null
        Object ret = method.invoke(target, args);
        System.out.println("after");
        return ret;
    }
}

// 测试
public static void main(String[] args) {
    Shop proxy = (Shop) new ProxyHandler(new PetShop()).getProxy();
    Class<?> superclass = proxy.getClass().getSuperclass();
    System.out.println(superclass); // java.lang.reflect.Proxy
}
```

### 2. CGlib动态代理

- 基于类实现
- `Enhancer`类会生成一个被代理类的增强子类作为代理类
- 与JDK动态代理的`InvocationHandler`接口类似，CGLib有一个接口`MethodInterceptor`，用来代理被代理者的方法，这个接口实例同样需要被传入`Enhancer`

引入依赖
```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

```java
// 被代理类, 没有实现任何接口
public class PetShop {
    public void sellDog() {
        System.out.println("sell a dog");
    }

    public void sellCat() {
        System.out.println("sell a cat");
    }
}


public class ProxyHandler implements MethodInterceptor {
    private Object target;

    public ProxyHandler(Object target) {
        this.target = target;
    }

    public Object getProxy() {
        // 类比于JDK动态代理中的Proxy类
        Enhancer enhancer = new Enhancer();
        // 设置代理类, 类比于JDK动态代理中获取代理类实现的接口
        enhancer.setSuperclass(target.getClass());
        // 设置处理增强方法的类
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("before");
        // 反射调用
        // Object ret = method.invoke(target, args);
        // 正常调用
        Object ret = proxy.invokeSuper(obj, args);
        System.out.println("after");
        return ret;
    }
}

public static void main(String[] args) {
    PetShop proxy = (PetShop) new ProxyHandler(new PetShop()).getProxy();
    proxy.sellDog();
}
```

## 总结
1. JDK代理基于接口
2. CGlib代理基于类