## 摘要

利用共享的方式来支持大量细粒度的对象

## 实现

一盘棋有很多不同颜色的棋子, 对于相同颜色的棋子复用一个就够了

<img src="https://s1.ax1x.com/2020/10/18/0XrLin.png" alt="0XrLin.png" border="0" />

```java
// 棋子抽象类
public abstract class IChess {
    protected String color;

    public IChess(String color) {
        this.color = color;
    }

    public abstract void printColor();
}
```
```java
// 棋子实现类
public class Chess extends IChess {
    public Chess(String color) {
        super(color);
    }

    @Override
    public void printColor() {
        System.out.println(color);
    }
}
```

```java
// 生产棋子的工厂
public class ChessFactory {
    private Map<String, IChess> chessPool = new HashMap<>();

    private ChessFactory() {
    }

    private static class Helper {
        private static final ChessFactory instance = new ChessFactory();
    }

    public static ChessFactory getInstance() {
        return Helper.instance;
    }

    public IChess getChess(String color) {
        if (!chessPool.containsKey(color)) {
            IChess chess = new Chess(color);
            chessPool.put(color, chess);
            return chess;
        }
        return chessPool.get(color);
    }
}
```

## 测试

```java
public static void main(String[] args) {
    ChessFactory factory = ChessFactory.getInstance();
    IChess black1 = factory.getChess("black");
    IChess black2 = factory.getChess("black");
    System.out.println(black1 == black2);   // true
}
```

## 总结

1. 池化思想, 复用具有相同特性的资源
2. 字符串常量池就是典型的享元模式
    `String s1 = "a"; String s2 = "a";` s1和s2指向字符串常量池中同一个`"a"`