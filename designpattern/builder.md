## 摘要

当一个类的构造函数参数过多, 而且有些参数在某些场合是**可选**的时候, 可以考虑使用建造者模式

## 实现(精简版)

从JDK11新版HttpClient类扒下来的一个例子, 很典型的建造者模式<br>

先看缩减版(不带有抽象层), 能清晰的理解建造者构建对象的方式
```java
class HttpClient {
    private String cookieHandler;
    private String connectTimeout;
    private String version;
    private String proxy;
    private String authenticator;

    // 构造过程相当于先给builder传参, 然后再给目标对象赋值
    private HttpClient(Builder builder) {
        this.cookieHandler = builder.cookieHandler;
        this.connectTimeout = builder.connectTimeout;
        this.version = builder.version;
        this.proxy = builder.proxy;
        this.authenticator = builder.authenticator;
    }
    
    // 静态方法提供一个builder
    // (套娃, 我先给你builder, 你再build我, 虽说应该先有builder再有目标对象, 但官方就是这么写的)
    public static Builder newBuilder() {
        return new Builder();
    }

    public static class Builder {
        // 定义和目标对象相同的字段
        private String cookieHandler;
        private String connectTimeout;
        private String version;
        private String proxy;
        private String authenticator;

        // 有些参数是必填的时候, 可以在这里加进来
        // 外部提供builder, 这里就可以私有化了, 如果不提供就public
        // 总之至少对外提供一个获取builder的接口, 不然拿头构建对象
        private Builder() {
        }

        // return this实现了链式调用, 一体化明显
        public Builder cookieHandler(String cookieHandler) {
            this.cookieHandler = cookieHandler;
            return this;
        }

        public Builder connectTimeout(String connectTimeout) {
            this.connectTimeout = connectTimeout;
            return this;
        }

        public Builder version(String version) {
            this.version = version;
            return this;
        }

        public Builder proxy(String proxy) {
            this.proxy = proxy;
            return this;
        }

        public Builder authenticator(String authenticator) {
            this.authenticator = authenticator;
            return this;
        }

        // new 一个对象, 把builder传进去
        // builder就包含了目标对象需要赋值的值
        public HttpClient build() {
            return new HttpClient(this);
        }
    }
}
```
## 使用
```java
public static void main(String[] args) {
    HttpClient httpClient = HttpClient.newBuilder()
            .version("HTTP/1.1")
            .connectTimeout("3000")
            .proxy("192.168.75.128")
            .cookieHandler("localStorage")
            .authenticator("jwt")
            .build();
    System.out.println(client); // HttpClient{cookieHandler='localStorage', connectTimeout='3000', version='HTTP/1.1', proxy='192.168.75.128', authenticator='jwt'}
}
```

<br>

## 带有抽象层的完整版
```java
// 定义了HttpClient结构的抽象类, 对应的builder放在内部
public abstract class HttpClient {
    // 把构造protected, 我认为目的是子类默认不能被外部new
    // 不能用private, 不然无法被继承
    protected HttpClient() {
    }

    // 静态方法提供一个builder (套娃, 我先给你builder, 你再build我)
    public static Builder newBuilder() {
        return new HttpClientBuilderImpl();
    }

    // 构造者接口
    public interface Builder {
        Builder cookieHandler(String cookieHandler);
        Builder connectTimeout(String connectTimeout);
        Builder version(String version);
        Builder proxy(String proxy);
        Builder authenticator(String authenticator);
        HttpClient build();
    }
}
```

```java
// HttpClient实现类
public class HttpClientImpl extends HttpClient {
    private String cookieHandler;
    private String connectTimeout;
    private String version;
    private String proxy;
    private String authenticator;
    
    // 构造私有化, 通过传入builder构造对象
    // 那么问题来了, 在构造私有的情况下, builder怎么new? 答案在下面那个方法
    private HttpClientImpl(HttpClientBuilderImpl builder) {
        this.cookieHandler = builder.cookieHandler;
        this.connectTimeout = builder.connectTimeout;
        this.version = builder.version;
        this.proxy = builder.proxy;
        this.authenticator = builder.authenticator;
    }

    // 静态方法提供一个对象, 即使别人通过这个方法直接获取HttpClient对象也需要传入builder
    // 把BuilderImpl放在HttpClientImpl内部就没必要这么做了
    public static HttpClientImpl create(HttpClientBuilderImpl builder) {
        return new HttpClientImpl(builder);
    }
}
```

```java
// HttpClient.Builder实现类
class HttpClientBuilderImpl implements HttpClient.Builder {
    // JDK没有用getter, 而是把BuilderImpl和HttpClientImpl放在同一个包内, 直接访问字段
    // 也可以把BuilderImpl放在HttpClientImpl内部, 这样结构更明显, 官方没用, 我这里就分开写了
    String cookieHandler;
    String connectTimeout;
    String version;
    String proxy;
    String authenticator;

    @Override
    public HttpClientBuilderImpl cookieHandler(String cookieHandler) {
        this.cookieHandler = cookieHandler;
        return this;
    }

    @Override
    public HttpClientBuilderImpl connectTimeout(String connectTimeout) {
        this.connectTimeout = connectTimeout;
        return this;
    }

    @Override
    public HttpClient.Builder version(String version) {
        this.version = version;
        return this;
    }

    @Override
    public HttpClient.Builder proxy(String proxy) {
        this.proxy = proxy;
        return this;
    }

    @Override
    public HttpClient.Builder authenticator(String authenticator) {
        this.authenticator = authenticator;
        return this;
    }

    // 调用HttpClientImpl静态方法传入build
    @Override
    public HttpClient build() {
        return HttpClientImpl.create(this);
    }
}
```

<br>

<h2 class="note note-primary">参考文章</h2>
<a class="btn" href="https://zhuanlan.zhihu.com/p/58093669">shusheng007</a>