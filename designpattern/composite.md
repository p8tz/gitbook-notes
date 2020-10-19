## 摘要

将对象组合成树形结构来表示“整体/部分”层次关系，允许用户以相同的方式处理单独对象和组合对象。


## 实现

一般来说文件结构由两部分组成: 文件夹和文件, 我们用叶子节点`LeafNode`代表文件, 分支节点`BranchNode`代表文件夹, 实现添加和删除的操作

<img src="https://s1.ax1x.com/2020/10/18/0XmQeJ.png" alt="0XmQeJ.png" border="0" />

```java
// 节点抽象类
public abstract class Node {
    protected String name;

    protected Node(String name) {
        this.name = name;
    }

    public abstract void printNode();

    public void addNode(Node node) {
        throw new UnsupportedOperationException();
    }

    public void delNode(Node node) {
        throw new UnsupportedOperationException();
    }

    public List<Node> getChildren() {
        throw new UnsupportedOperationException();
    }
}
```

```java
// 文件
public class LeafNode extends Node {
    // 文件有内容属性
    private String content;

    public LeafNode(String name) {
        super(name);
    }

    public LeafNode(String name, String content) {
        this(name);
        this.content = content;
    }

    @Override
    public void printNode() {
        System.out.println(name + ": " + content);
    }
}
```

```java
// 文件夹
public class BranchNode extends Node {
    private List<Node> children = new ArrayList<>();

    public BranchNode(String name) {
        super(name);
    }

    @Override
    public List<Node> getChildren() {
        return children;
    }

    @Override
    public void addNode(Node node) {
        this.children.add(node);
    }

    @Override
    public void delNode(Node node) {
        this.children.remove(node);
    }

    @Override
    public void printNode() {
        System.out.println(name);
    }
}
```

## 测试

```java
public static void main(String[] args) {
    Node root = new BranchNode("root-folder");
    Node folder1 = new BranchNode("folder1");
    Node folder2 = new BranchNode("folder2");
    Node file11 = new LeafNode("file11", "content11");
    Node file12 = new LeafNode("file12", "content12");
    Node file21 = new LeafNode("file21", "content21");
    Node file22 = new LeafNode("file22", "content22");
    Node file31 = new LeafNode("file31", "content31");
    Node file32 = new LeafNode("file32", "content32");

    root.addNode(file11);
    root.addNode(file12);
    root.addNode(folder1);
    folder1.addNode(file21);
    folder1.addNode(file22);
    folder1.addNode(folder2);
    folder2.addNode(file31);
    folder2.addNode(file32);

    printTree(root, 0);
    
    System.out.println("\n------------del folder1--------------\n");
    
    root.delNode(folder1);
    printTree(root, 0);
}

public static void printTree(Node root, int depth) {
    System.out.print("--".repeat(Math.max(0, depth)));
    root.printNode();
    if (root instanceof BranchNode) {
        BranchNode node = (BranchNode) root;
        for (Node n : node.getChildren()) {
            printTree(n, depth + 1);
        }
    }
}
/*
	root-folder
    --file11: content11
    --file12: content12
    --folder1
    ----file21: content21
    ----file22: content22
    ----folder2
    ------file31: content31
    ------file32: content32

    ------------del folder1--------------

    root-folder
    --file11: content11
    --file12: content12
*/
```

## 总结

1. 能让客户端以相同的方式处理叶子节点和分支节点