## LRU

超出缓存容量后，删除最久没有使用的元素

### 实现

哈希表+双向链表

![image-20201114161855498](https://gitee.com/p8t/picbed/raw/master/imgs/20201114161856.png)

**新增的元素放在链表尾，访问的元素移到链表尾，删除时删掉表头元素即可**

```java
class LRUCache {
    int cap;
    // 两个虚拟节点简化操作
    Node head = new Node(-1, -1);
    Node tail = new Node(-1, -1);

    Map<Integer, Node> cache = new HashMap<>();

    class Node {
        int key;
        int value;
        Node next;
        Node prev;
        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    public LRUCache(int cap) {
        this.cap = cap;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        Node n = cache.get(key);
        if (n == null) {
            return - 1;
        }
        delNode(n);
        addToTail(n);
        return n.value;
    }

    public void put(int key, int value) {
        Node n = cache.get(key);
        if (n == null) {
            n = new Node(key, value);
            addToTail(n);
            cache.put(key, n);
            if (cache.size() > cap) {
                del(head.next.key);
            }
        } else {
            n.value = value;
            delNode(n);
            addToTail(n);
        }
    }

    public void del(int key) {
        Node n = cache.get(key);
        if (n == null) {
            return;
        }
        cache.remove(n.key);
        delNode(n);
    }

    private void addToTail(Node n) {
        tail.prev.next = n;
        n.prev = tail.prev;
        tail.prev = n;
        n.next = tail;
    }

    private void delNode(Node n) {
        n.prev.next = n.next;
        n.next.prev = n.prev;
        n.next = null;
        n.prev = null;
    }
}
```

