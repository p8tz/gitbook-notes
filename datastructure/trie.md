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