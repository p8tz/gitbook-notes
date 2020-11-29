## LFU

超出缓存容量后，删除使用次数最少的元素，如果有相同的则删除最久未使用的

### 实现

和`LRU`差不多

```java
class LFUCache {
    int cap;
    Node head = new Node(-1, -1);
    Node tail = new Node(-1, -1);
    
    Map<Integer, Node> cache = new HashMap<>();
    
    class Node {
        int key;
        int value;
        int count = 0;
        Node next;
        Node prev;
        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
    public LFUCache(int cap) {
        this.cap = cap;
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        if (cap == 0) return -1;
        Node n = cache.get(key);
        if (n == null) {
            return - 1;
        }
        n.count++;
        delNode(n);
        moveToCorrect(n);
        return n.value;
    }
    
    public void put(int key, int value) {
        if (cap == 0) return;
        Node n = cache.get(key);
        if (n == null) {
            n = new Node(key, value);
            if (cache.size() >= cap) {
                del(head.next.key);
            }
            moveToCorrect(n);
            cache.put(key, n);
        } else {
            n.value = value;
            n.count++;
            delNode(n);
            moveToCorrect(n);
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
    
    private void moveToCorrect(Node n) {
        int c = n.count;
        Node h = head.next;
        while (h != tail && h.count <= c) {
            h = h.next;
        }
        h.prev.next = n;
        n.prev = h.prev;
        h.prev = n;
        n.next = h;
    }

    private void delNode(Node n) {
        n.prev.next = n.next;
        n.next.prev = n.prev;
        n.next = null;
        n.prev = null;
    }
}
```

