## 一、数组与集合转化

**数组转集合**

```java
// 引用类型数组转集合
String[] arr = {"1", "2", "3"};
List<String> list = new ArrayList<>(Arrays.asList(arr));
List<String> list = Arrays.stream(arr).collect(Collectors.toList());
// 基本类型数组转集合
int[] arr = {1, 2, 3};
List<Integer> list = Arrays.stream(arr).boxed().collect(Collectors.toList());
```

**集合转数组**

```java
// 集合转引用类型数组
List<String> list = new ArrayList<>(Arrays.asList("1", "2", "3"));
String[] arr = list.toArray(new String[0]);
// 集合转基本类型数组
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
int[] arr = Arrays.stream(list.toArray(new Integer[0])).mapToInt(Integer::intValue).toArray();
```

**包装类型数组转基本类型数组**

```java
Integer[] boxed = {1, 2 ,3};
int[] unboxed = Arrays.stream(boxed).mapToInt(Integer::intValue).toArray();
```

**基本类型数组转包装类型数组**

```java
int[] unboxed = {1, 2 ,3};
Integer[] boxed = Arrays.stream(unboxed).boxed().toArray(Integer[]::new);
```

## 二、单列集合

![image-20201113192053360](https://gitee.com/p8t/picbed/raw/master/imgs/20201113192054.png)

### ArrayDeque

1.6新增，禁止存`null`值

`ArrayDeque`是一个循环数组，仅支持头尾元素操作，因此天然适合栈和队列

```java
/**
 * Resizable-array implementation of the {@link Deque} interface.  Array
 * deques have no capacity restrictions; they grow as necessary to support
 * usage.  They are not thread-safe; in the absence of external
 * synchronization, they do not support concurrent access by multiple threads.
 * Null elements are prohibited.  This class is likely to be faster than
 * {@link Stack} when used as a stack, and faster than {@link LinkedList}
 * when used as a queue.
 */

public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    final Object[] es = elements;
    es[tail] = e;
    if (head == (tail = inc(tail, es.length)))
        grow(1);
}
```

### LinkedHashSet

支持按插入顺序遍历

### TreeSet

以红黑树实现的搜索树

## 三、双列集合

![image-20201113201450248](https://gitee.com/p8t/picbed/raw/master/imgs/20201113201451.png)

### Properties

用于配置文件