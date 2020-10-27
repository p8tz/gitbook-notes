## AVL树

高度平衡的二叉搜索树

- 左子树和右子树的高度之差的不超过1
- 左子树和右子树都是AVL树

调整分为4种情况, `LL RR LR RL`, 有两种为对称情况

- LL: 插入节点在根节点左子树的左子树, 需要一次右旋
- LR: 插入节点在根节点左子树的右子树, 需要先左旋再右旋
- RR: 插入节点在根节点右子树的左子树, 需要一次左旋
- RL: 插入节点在根节点右子树的右子树, 需要先右旋再左旋

**LL调整具体步骤**

![image-20201024222639295](https://gitee.com/p8t/picbed/raw/master/imgs/20201024222640.png)

剩余三种情况调整算法的本质和LL都一样

**LR调整**

![image-20201024222726792](https://gitee.com/p8t/picbed/raw/master/imgs/20201024222727.png)

RR和RL是LL和LR的对称情况

## 红黑树

弥补了AVL树频繁调整的缺陷

- 节点是红色或黑色
- 根节点是黑色
- 所有叶子都是黑色
- 每个红色节点的两个子节点都是黑色（从每个叶子到根的所有路径上不能有两个连续的红色节点）
- 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点

这些约束强制了红黑树的关键性质: 左子树和右子树的高度之差不超过一倍

红黑树左旋右旋的步骤和AVL树一样, AVL是在高度差大于1就开始调整, 而红黑树则复杂的多

不是说高度差超过一倍就调整, 而是调整后的结果是高度差不超过一倍

**插入规则**

记插入元素为X, 默认标记为红色

1. 若X为根节点, 即第一次插入, 则把X标为黑色, 结束
2. 若X父节点为黑色, 结束
3. 若X父节点为红色, 父节点的兄弟节点为红色. 
   - 把这两个兄弟节点都变为黑色, 然后把它们的父节点, 即X的祖父节点变为红色
   - 这时可能破坏了红黑树的性质, 需要对X的祖父节点递归处理
   - 若递归最后根节点变成了红色, 则再转为黑色
4. 若X父节点为红色, 父节点的兄弟节点为黑色
   1. 若X是LL情况
      - 进行一次右旋
      - 交换父节点和祖父节点的颜色
   2. 若X是LR情况
      - 进行一次先左后右的双旋
      - 交换X和原祖父节点的颜色(左旋后, X的祖父节点变为了它的父节点)
   3. RR略
   4. RL略

**应用**

`C++ STL map`

`java.util.TreeMap`

## 堆

以数组形式存储的完全二叉树

最大堆: 任一节点的值大于等于两个子节点的值

最小堆: 任一节点的值小于等于两个子节点的值

从数组索引`0`开始存储的堆, `heap[i]`

- 左孩子为`heap[2 * i + 1]`
- 右孩子为`heap[2 * i + 2]`
- 父节点为`heap[(i - 1) / 2]`

从数组索引`1`开始存储的堆, `heap[i]`

- 左孩子为`heap[2 * i]`
- 右孩子为`heap[2 * i + 1]`
- 父节点为`heap[i / 2]`

下面以最大堆为例

一般来说, 堆顶元素是我们关注的, 因为它始终是最大值

`siftDown`: 从某一节点开始, 找出它和它两个儿子中的最大值, 若不是它本身, 则最大值上浮(交换), 对下浮的节点递归执行, 直到不再变化

`siftUp`: 从某一节点开始, 和它父节点比较, 若大于则上浮(交换), 递归执行, 直到不再变化

**应用**

`java.util.PriorityQueue`

`HeapSort`

`TopN`

## B树 / B+树

见MySQL

## trie树

### 特点

- 根节点不包含字符
- 除根节点外每一个节点都只包含一个字符
- 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串
- 每个节点的所有子节点包含的字符都不相同

### 实现

```java
public class Trie {
    class Node {
        Node[] next = new Node[127];
        boolean valid;

        boolean have(char c) {
            return next[c] != null;
        }

        Node get(char c) {
            return next[c];
        }

        void set(char c) {
            next[c] = new Node();
        }
    }

    Node root = new Node();

    // 插入单词
    void insert(String word) {
        char[] chs = word.toCharArray();

        Node node = root;
        for (char c : chs) {
            if (!node.have(c)) {
                node.set(c);
            }
            node = node.get(c);
        }
        node.valid = true;
    }
    
	// 搜索单词
    boolean search(String word) {
        Node n = helper(word);
        return n != null && n.valid;
    }
    
	// 查询前缀
    boolean startWith(String word) {
        Node n = helper(word);
        return n != null;
    }

    // 获取所有prefix开头的单词
    List<String> getAll(String prefix) {
        List<String> ans = new ArrayList<>();

        Node n = helper(prefix);
        if (n == null) return ans;

        dfs(n, prefix, ans);
        return ans;
    }

    void dfs(Node node, String prefix, List<String> ans) {
        if (node.valid) {
            ans.add(prefix);
        }

        for (int i = 0; i < node.next.length; i++) {
            if (node.next[i] == null) continue;

            dfs(node.next[i], prefix + (char) i, ans);
        }
    }
    
    // 前缀合法返回最后一个符号, 否则返回null
    Node helper(String prefix) {
        char[] chs = prefix.toCharArray();

        Node node = root;
        for (char c : chs) {
            if (!node.have(c)) {
                return null;
            }
            node = node.get(c);
        }
        return node;
    }
}
```

**应用**

搜索引擎下拉提示框

## 并查集

### 功能

- 查询元素X和元素Y是否属于同一组
- 合并元素X和元素Y所在的组

### 实现

两点优化, 降低树高

- 压缩路径
- 按秩合并

```java
public class UF {
    // 分组数量
    int count;
    // i 代表元素
    // parent[i] 代表元素i的父节点
    int[] parent;

    // 秩, 合并时秩小的合并到秩大的
    int[] rank;

    public UF(int count) {
        this.count = count;
        parent = new int[count];
        rank = new int[count];

        // 元素个自成一组
        for (int i = 0; i < parent.length; i++) {
            parent[i] = i;
        }
    }

    // 找到元素x的根节点, 根节点相同的元素在同一组
    int find(int x) {
        if (parent[x] == x)
            return x;

        // 压缩路径
        parent[x] = find(parent[x]);
        return parent[x];

        // return parent[x] == x ? x : (parent[x] = find(parent[x]));

        // while (parent[x] != x) {
        //     // 压缩路径
        //     parent[x] = parent[parent[x]];
        //
        //     x = parent[x];
        // }
        // return x;
    }

    void union(int x, int y) {
        int rX = find(x);
        int rY = find(y);

        if (rX == rY)
            return;

        // 按秩合并
        // 矮的树挂到高的上面, 高度不变
        if (rank[rX] < rank[rY]) {
            parent[rX] = rY;
        } else {
            parent[rY] = rX;
        }

        // rY 挂到 rX下面, rank[rX]加1
        if (rank[rX] == rank[rY]) {
            rank[rX]++;
        }

        count--;
    }
}
```