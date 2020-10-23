## 摘要

当一个对象内在状态改变时允许其改变行为，这个对象看起来像改变了其原本的类。

## 类图

![image-20201023175431355](https://gitee.com/p8t/picbed/raw/master/imgs/20201023175433.png)

## 实现

电梯四种状态: run stop open close

```java
public abstract class LiftState {
    public abstract void open();
    public abstract void close();
    public abstract void run();
    public abstract void stop();

    protected Context context;

    public LiftState(Context context) {
        this.context = context;
    }
}

public class OpenState extends LiftState {
    public OpenState(Context context) {
        super(context);
    }

    @Override
    public void open() {
        System.out.println("Door is opened");
    }

    @Override
    public void close() {
        System.out.println("Preparing close the door...");
        System.out.println("Door is closed");
        super.context.setCurState(super.context.close);
    }

    @Override
    public void run() {
        System.out.println("Door is opened, can not run the lift");
    }

    @Override
    public void stop() {
        System.out.println("The lift has been stopped");
    }
}

public class CloseState extends LiftState {
    public CloseState(Context context) {
        super(context);
    }

    @Override
    public void open() {
        System.out.println("Preparing open the door...");
        System.out.println("Door is opened");
        super.context.setCurState(super.context.open);

    }

    @Override
    public void close() {
        System.out.println("The door has been closed");
    }

    @Override
    public void run() {
        System.out.println("Preparing run the lift...");
        System.out.println("Lift is running...");
        super.context.setCurState(super.context.run);

    }

    @Override
    public void stop() {
        System.out.println("The lift has been stopped");
        super.context.setCurState(super.context.stop);

    }
}

public class RunState extends LiftState {
    public RunState(Context context) {
        super(context);
    }

    @Override
    public void open() {
        System.out.println("Lift is running, can not open the door");
    }

    @Override
    public void close() {
        System.out.println("The door has been closed");
    }

    @Override
    public void run() {
        System.out.println("Lift is running");
    }

    @Override
    public void stop() {
        System.out.println("Preparing stop the lift...");
        System.out.println("Lift is stopped...");
        super.context.setCurState(super.context.stop);
    }
}

public class StopState extends LiftState {
    public StopState(Context context) {
        super(context);
    }

    @Override
    public void open() {
        System.out.println("Preparing open the door...");
        System.out.println("Door is opened");
        super.context.setCurState(super.context.open);

    }

    @Override
    public void close() {
        System.out.println("Door is closed");
        super.context.setCurState(super.context.close);
    }

    @Override
    public void run() {
        System.out.println("Preparing run the lift...");
        System.out.println("Lift is running...");
        super.context.setCurState(super.context.run);
    }

    @Override
    public void stop() {
        System.out.println("The lift has been stopped");
    }
}
```

```java
public class Context {
    public final LiftState run = new RunState(this);
    public final LiftState stop = new StopState(this);
    public final LiftState open = new OpenState(this);
    public final LiftState close = new CloseState(this);

    // default state
    private LiftState curState = stop;

    public void setCurState(LiftState curState) {
        this.curState = curState;
    }

    public void open() {
        printState();
        curState.open();
    }

    public void close() {
        printState();
        curState.close();
    }

    public void run() {
        printState();
        curState.run();
    }

    public void stop() {
        printState();
        curState.stop();
    }

    private void printState() {
        String s = curState.getClass().toString();
        System.out.println("current state: " + s.substring(s.lastIndexOf(".") + 1));
    }
}
```

## 测试

```java
public class Test {
    public static void main(String[] args) {
        Context context = new Context();
        context.setCurState(context.close);
        System.out.println("========================================");
        context.run();
        System.out.println("========================================");
        context.open();
        System.out.println("========================================");
        context.open();
        System.out.println("========================================");
        context.stop();
        System.out.println("========================================");
        context.open();
        System.out.println("========================================");
        context.close();
        System.out.println("========================================");
        context.run();
        System.out.println("========================================");
        context.open();
        System.out.println("========================================");
        context.open();
        System.out.println("========================================");
    }
}
/*
    ========================================
    current state: CloseState
    Preparing run the lift...
    Lift is running...
    ========================================
    current state: RunState
    Lift is running, can not open the door
    ========================================
    current state: RunState
    Lift is running, can not open the door
    ========================================
    current state: RunState
    Preparing stop the lift...
    Lift is stopped...
    ========================================
    current state: StopState
    Preparing open the door...
    Door is opened
    ========================================
    current state: OpenState
    Preparing close the door...
    Door is closed
    ========================================
    current state: CloseState
    Preparing run the lift...
    Lift is running...
    ========================================
    current state: RunState
    Lift is running, can not open the door
    ========================================
    current state: RunState
    Lift is running, can not open the door
    ========================================
*/
```

