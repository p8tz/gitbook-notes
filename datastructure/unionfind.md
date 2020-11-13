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

