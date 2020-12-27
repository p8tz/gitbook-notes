## 注解

### 配置注解

`@Configuration`

- 加在类上，表明当前类是一个配置类，用于代替`XML`配置文件

`@Bean`

- 配合`@Configuration`使用，加在方法上，会在`IOC`容器中注册`Bean`对象。方法返回值作为`Bean`对象添加到容器中，`id`默认为方法名，可以给`Bean`显示指定`value`值作为`id`
- 方法参数自动从`IOC`容器中获取，默认不写`Autowired`

`@ComponentScan`
  - 配合`@Configuration`使用，指定扫描包路径。只有被扫描到的`Component`才会被注册到`IOC`容器
- 小结
  - `@Configuation`等价于`<Beans></Beans>`
  - `@Bean`等价于`<Bean></Bean>`
  - `@ComponentScan`等价于`<context:component-scan base-package="com.xxx"/>`

### 注册Bean

`@Component`

- 往`IOC`容器中注册`Bean`对象，`id`默认为类首字母小写。只有被扫描到才能注册

### 属性赋值

`@Value`

- 可以写基本类型
- 可以写`#{}`表达式
- 可以写`${}`，从外部配置文件中获取，通过`@PropertySource`加载配置文件

### 自动装配

`@Autowired`

- 根据属性类型注入，如果有多个则选取对象名去匹配
- 如果`required`指定为`false`，则找不到就不装配
- 标在方法上
  - `Spring`会在初始化`Bean`时调用这个方式，方法参数从`IOC`容器中获取
- 标在有参构造器上
  - 则容器初始化`Bean`时不会调用默认的无参构造，而是调用这个有参构造
- 标在方法参数上
- 无论怎么放，都是从`IOC`容器中装配

`@Qualifier`

- 根据属性名称注入，配合`Autowired`使用，可以指定注入对象的`id`

`@Primary`

- 在注入`Bean`的时候加上`Primary`，则`Autowired`会首选它装配。如果`Autowired`配合了`Qualifier`使用，则按`Qualifier`装配

`@Resource`

- `javax`包下的，相当于上面两个合起来，不支持`Primary`

`@Inject`

- `javax`包下的，需要导包，支持`Primary`

### Bean相关

`@Scope`

- 指定`Bean`作用域
  - `singleton`：单例（默认），`IOC`容器一启动就创建对象
  - `prototype`：多例，`IOC`容器启动后不会立刻创建对象，只有在获取时才创建
  - `request`
  - `session`

`@Lazy`

- 针对单例`Bean`，进行懒加载，即获取时才创建对象

## 组件注册方式

1、`@ComponentScan + @Component`，适用于给自己编写的类注册组件

2、`@Configuration + @Bean`，适用于注册第三方的组件

3、`@ImportResource`，导入`xml`配置文件

4、`@Import`，快速注册`Bean`。只要标注在配置方法上，然后`value`写上需要注册的类，就会自动调用无参构造进行注册

```java
@Import({User.class, Book.class}) // 快速注册User和Book, 组件名为全类名
@Configuration
public class ...
```

## AOP

实现AOP功能大致分为三步

- 业务类和切面类注册到`IOC`容器
- 切面类加上`@Aspect`注解
- 切面类编写切点表达式

```java
/*
 * 前置通知(@Before)
 * 后置通知(@After)
 * 返回通知(@AfterReturning)
 * 异常通知(@AfterThrowing)
 * 环绕通知(@Around)
 */

// 业务类
@Component
public class Calculator {
    public int divide(int i, int j) {
        return i / j;
    }
}

// 切面类
@Aspect
@Component
public class LogAspect {
    /*
     * JoinPoint用于获取方法信息
     * 必须写在第一个参数
     */
    
    // 用于抽取公共切点表达式
    // 其它类引用时需要使用全路径: cc.p8t.test.aspect.pointCut()
    @Pointcut("execution(public int cc.p8t.test.misc.Calculator.*(..))")
    public void pointCut() {}

    @Before("execution(public int cc.p8t.test.misc.Calculator.divide(int, int))")
    public void logStart(JoinPoint joinPoint) {
        Object[] args = joinPoint.getArgs();
        System.out.println("start...{ " + Arrays.asList(args) + " }");
    }

    @After("execution(public int cc.p8t.test.misc.Calculator.divide(int, int))")
    public void logEnd() {
        System.out.println("end");
    }

    // returning记录返回值
    @AfterReturning(value = "pointCut()", returning = "result")
    public void logResult(Object result) {
        System.out.println("result: { " + result + " }");
    }

    // throwing记录异常
    @AfterThrowing(value = "pointCut()", throwing = "exception")
    public void logException(JoinPoint joinPoint, Exception exception) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("exception: { " + methodName + " : " + exception + " }");
    }
}
```

## 声明式事务

使用`Mybatis`时，只需要给方法加上`@Transactional`注解即可

- 对于没有`try`的会自动回滚
- 对于`try`了的需要手动回滚

```java
TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
```

