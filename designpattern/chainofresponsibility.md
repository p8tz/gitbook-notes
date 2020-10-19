## 摘要

使多个对象都有机会处理请求，从而避免了请求的发送者和接 受者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处 理它为止。

## 实现

<img src="https://s1.ax1x.com/2020/10/19/0x37Hx.png" alt="0xlpD0.png" border="0" />

```java
public enum Level {
    TRACE, DEBUG, INFO, WARNING, ERROR;
    public boolean above(Level level) {
        return this.compareTo(level) >= 0;
    }
}

public class Request {
    private Level level;

    public Request(Level level) {
        this.level = level;
    }

    public Level getLevel() {
        return level;
    }
}

public class Response {
    private String message;

    public Response(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}
```

```java
public abstract class IHandler {
    private IHandler nextHandler;

    public final Response handle(Request request) {
        Response response = null;
        if (this.getHandlerLevel().above(request.getLevel())) {
            response = this.realHandle(request);
        } else {
            if (nextHandler != null) {
                this.nextHandler.handle(request);
            } else {
                System.out.println("无法处理此次请求");
            }
        }
        return response;
    }

    public void setNextHandler(IHandler nextHandler) {
        this.nextHandler = nextHandler;
    }

    protected abstract Level getHandlerLevel();

    protected abstract Response realHandle(Request request);
}
```

```java
public class HandlerA extends IHandler {
    private final Level LEVEL = Level.TRACE;

    @Override
    public Level getHandlerLevel() {
        return LEVEL;
    }

    @Override
    public Response realHandle(Request request) {
        Response response = new Response("HandlerA");
        System.out.println("处理级别: " + LEVEL.name());
        return response;
    }
}

public class HandlerB extends IHandler {
    private final Level LEVEL = Level.INFO;

    @Override
    public Level getHandlerLevel() {
        return LEVEL;
    }

    @Override
    public Response realHandle(Request request) {
        Response response = new Response("HandlerB");
        System.out.println("处理级别: " + LEVEL.name());
        return response;
    }
}

public class HandlerC extends IHandler {
    private final Level LEVEL = Level.WARNING;

    @Override
    public Level getHandlerLevel() {
        return LEVEL;
    }

    @Override
    public Response realHandle(Request request) {
        Response response = new Response("HandlerC");
        System.out.println("处理级别: " + LEVEL.name());
        return response;
    }
}
```
## 测试

```java
public static void main(String[] args) {
    IHandler handlerA = new HandlerA();
    IHandler handlerB = new HandlerB();
    IHandler handlerC = new HandlerC();

    handlerA.setNextHandler(handlerB);
    handlerB.setNextHandler(handlerC);

    handlerA.handle(new Request(Level.TRACE));   // 处理级别: TRACE
    handlerA.handle(new Request(Level.DEBUG));   // 处理级别: INFO
    handlerA.handle(new Request(Level.INFO));    // 处理级别: INFO
    handlerA.handle(new Request(Level.WARNING)); // 处理级别: WARNING
    handlerA.handle(new Request(Level.ERROR));   // 无法处理此次请求
}
```