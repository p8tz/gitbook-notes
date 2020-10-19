## 摘要

原型模式解决的是创建大量重复对象的问题

通过类的实例对象克隆自身产生新的实例对象，无需使用new来创建

在Java中，实现原型模式非常简单，只需要实现Cloneable接口然后重写`Object类`的`clone()`方法就可以了

当然, 存在浅拷贝的问题, 下面举个例子说明如何解决

## 实现

```java
// 一棵树有一个枝条和许多叶子, 三者都是复杂类型
// 实现Object的clone()一定要实现Cloneable接口, 否则会抛CloneNotSupportedException
class Tree implements Cloneable {
    ArrayList<Leaf> leaves = new ArrayList<>();
    Branch branch = new Branch();

    @Override // 别忘了把protected扩展为public. 返回值改不改无所谓, 看你喜欢在哪强转
    public Tree clone() throws CloneNotSupportedException {
        // 先调用Object的clone方法简单的浅拷贝一个clone tree
        // 这一步很关键, 不需要new就能获得一个新的对象
        Tree cloneTree = (Tree) super.clone();

        // 以当前branch深拷贝一个branch
        cloneTree.branch = this.branch.clone();

        // 个人建议, 这个集合还是自己new比较好, 如果这样写
        // cloneTree.leaves = (ArrayList<Leaf>) this.leaves.clone();
        // 那么里面的叶子全都是浅拷贝, 想要深拷贝得先清空集合, 然后重新添加, 白白浪费了添加的操作
        // (叶子类重写clone()也没用, 原因见集合的clone()方法: 它也不知道泛型有没有重写clone方法, 默认protected是不能跨包调用的)
        ArrayList<Leaf> cloneLeaves = new ArrayList<>();
        for (Leaf leaf : this.leaves) {
            // 以当前leaves集合里面每一个leaf为原型clone
            cloneLeaves.add(leaf.clone());
        }
        // 最后改下cloneTree的叶子指向(不改的话, 还是原来的浅拷贝: cloneTree.leaves == this.leaves)
        cloneTree.leaves = cloneLeaves;
        return cloneTree;
    }
}

class Branch implements Cloneable {
    String id;

    // 作为其它类的属性时, 重写clone以帮助其深拷贝
    @Override
    public Branch clone() throws CloneNotSupportedException {
        return (Branch) super.clone();
    }
}

class Leaf implements Cloneable {
    String id;

    @Override
    public Leaf clone() throws CloneNotSupportedException {
        return (Leaf) super.clone();
    }
}
```

## 验证

```java
public static void main(String[] args) throws CloneNotSupportedException {
    Tree prototype = new Tree();
    prototype.leaves.add(new Leaf());
    prototype.leaves.add(new Leaf());
    Tree clone = prototype.clone();
    System.out.println(prototype == clone);                             // false
    System.out.println(prototype.branch == clone.branch);               // false
    System.out.println(prototype.leaves == clone.leaves);               // false
    System.out.println(prototype.leaves.get(0) == clone.leaves.get(0)); // false
    System.out.println(prototype.leaves.get(1) == clone.leaves.get(1)); // false
}
```

## 总结

1. 实现`cloneable`接口, 否则会抛`CloneNotSupportedException`
2. 重写`Object`类的`clone()`方法, 把权限修饰符改为public(也不是绝对的)
3. 如果该类字段全都是基本类型或String, 则只需要调用`super.clone()`即可
4. `Object`类的`clone()`是native方法, 直接复制对象数据, 不会`new`, 也就不会走构造方法
5. 由4可得, 它会破坏单例模式, 所以不要它们俩写一块去了