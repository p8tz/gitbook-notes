> 环境：JDK11

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
Integer[] boxed = {1, 2, 3};
int[] unboxed = Arrays.stream(boxed).mapToInt(Integer::intValue).toArray();
```

**基本类型数组转包装类型数组**

```java
int[] unboxed = {1, 2, 3};
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

## 四、集合特点概述

### ArrayList与Vector

- `ArrayList`线程不安全，`Vector`线程安全，通过方法加`synchronized`实现
- 初始容量都为10
- `ArrayList`不给初始容量则赋一个长度为0的空数组，第一次add后容量扩容为10；`Vector`直接初始化
- `ArrayList`扩容1.5倍，`Vector`可以指定每次扩容大小，不指定则为2倍
- `ArrayList`先扩容再添加元素

### HashMap与Hashtable

`HashMap`

- 初始容量16，扩容2倍
- 线程不安全
- 可以插入`null`值，但由于`null`值无法获取`hashCode`，因此对于`null`特判，放在数组索引0处
- 延迟初始化

- 1.7
  - 先扩容再添加
  - 数组 + 链表
- 头插法，多线程扩容链表死循环
  - 对`hashCode`多次异或，移位操作得到`hash`
  - 扩容不用`rehash`，重新**与**新数组长度-1得到下标

- 1.8
  
  - 先添加再扩容
  - 数组 + 链表 / 红黑树（数组长度达到64且链表长度达到8时转红黑树，长度变为6时转回链表），在负载因子为0.75时，链表长度达到8的概率为千万分之六
  - 在树化之前，先把单链表形成双向链表结构，方便扩容的时候遍历
  - 尾插法，解决死循环
  - 对`hashCode`高低16位异或得到`hash`
  - 扩容不用`rehash`，重新**与**原数组长度，决定是否偏移

`Hashtable`

  - 初始容量11，扩容2倍加1
  - 对数组长度取余得下标
  - 线程安全
  - 不支持`null`键值
  - 直接用`hashCode`作为计算角标的依据
  - 直接初始化

### CopyOnWriteArrayList

- `fail-safe`
- 读不加锁，使用迭代器遍历时，使用的是数据快照，对于新增的数据感知不到
- 写加可重入锁（`JDK8`是`ReentrantLock`，11是`synchronized`），复制一份原数组拷贝进行添加元素，然后改变原数组引用
- 不能保证读到的数据是最新的

### ConcurrentHashMap

多线程下HashMap的并发问题

- 链表成环，1.8中不存在这个问题
- `put`方法。A线程在判断一个桶为空后失去`CPU`，然后B线程在该桶`put`数据，这样A线程恢复`CPU`后就会覆盖B线程`put`的数据

1.7和1.8区别

- 1.7
  - 由`Segment`数组构成，每个`Segment`内都有一个真正存数据的`node`数组，所以定位一个元素分两步，先定位`Segment`，再定位`node`数组中的位置
  - `hash`前几位确定`segment`，后几位确定桶位置
  - `Segment+ReentrantLock+CAS+volatile`
  - 默认16个`Segment`，这个值被称为并发级别
  - `Segment`继承了`ReentrantLock`
  - `put()`方法，首先`tryLock()`，如果失败进入`scanAndLockForPut()`方法循环`tryLock()`，达到一定次数（处理器核心数大于1则为64次，否则为1次）后还没有获取锁后使用`lock()`
  - `get()`方法不加锁，但是获取元素时强制从主存中获取（`getObjectVolatile()`）
  - 每个`Segment`扩容只管自己，扩容时对于一个链表上的会连续计算节点新位置（直到新位置不一样），位置一样的整体复制过去
  - 初始化时只会初始化一个`Segment`，后面的15个只会在真正用到的时候以第一个为原型创建出来，但并不是原型模式（`Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);`）
  - `size()`方法，多次计算`size`（每个`Segment`的`count`累加），直到连续两次`modCount`相同（说明`size`没变，用`size`比较也行，但源码就是用`modCount`来判断的），最多重复计算3次，仍然不能得到结果后，全部`lock()`再计算
  
- 1.8
  - 取消`Segment`机制，核心：给每个桶的头节点加锁（使用`synchronized`），并发度更高。
  - 每次插入（更新）一个元素，如果桶处没有元素，则通过`CAS`插入，如果有元素，则`synchronized`第一个元素
  - 添加完后扩容
  - 如果初始化`table`的时候检测到已经有线程正在初始化，则`yield()`
  - `synchronized+CAS`
  - `put()`方法
    - 数组为空，初始化
    - 当前桶为空，通过`CAS`插入元素
    - 当前桶不为空，且正在扩容，则帮助它扩容
    - 当前桶不为空，加`synchronized`
  - 结构和1.8的`HashMap`基本相同，但是计算`hash`多了一步，把最高位置0，不知道有什么实际用途
  - 协助扩容。当一个线程正在对`table`进行扩容时，会把已经移动的元素的位置的插入一个`Forward`节点，其`hash`标为`MOVED`，这样其它线程想要迁移此处元素就会跳过，同时如果有元素在这里添加则会帮助其扩容。如果在其它地方添加，则添加结束后，还会帮助其扩容。
  - `size()`方法，计算`size`并不是简单的维护一个`size`变量，因为1.8的`CCHM`并发度更高，如果用一个变量计算`size`竞争会很激烈。
    1. 当需要修改元素数量时，线程会先去 `CAS `修改 `baseCount`加1，若成功即返回。
    2. 若失败，则线程被分配到某个 `CounterCell`，然后操作它的 `value`加1。若成功，则返回。
    3. 否则，给当前线程重新分配一个`CounterCell`，再尝试给 value 加1。（这里简略的说，实际更复杂）。
    4. 注：`CounterCell`会组成一个数组，也会涉及到扩容问题。统计`size`则是遍历 `counterCells`数组，得到每个对象中的`value`值进行累加，这只是获取`size`时的计算方法。计算过程中有个`fullAddCount()`方法，极为复杂
