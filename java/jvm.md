> 如无特殊说明，下文针对的都是`HotSpot`虚拟机。测试环境：win10 + 16G内存

## 一、运行时数据区域

![image-20201204215958780](https://gitee.com/p8t/picbed/raw/master/imgs/20201204220000.png)

### 1、程序计数器

指向当前正在执行的字节码指令，如果正在执行本地方法，则为`undefined`

### 2、虚拟机栈

虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每一个方法被调用直至执行完毕的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

#### 栈帧

![image-20201204223649799](https://gitee.com/p8t/picbed/raw/master/imgs/20201204223651.png)

**局部变量表**

存放方法参数以及方法内部定义的变量，存放单位是槽（`slot`，`JVMS`没有强制规定为32位）。除了`double`和`long`占用两个`slot`之外，其它的都只占用一个`slot`。可见，对于boolean类型，虽然在对象中存储时占用一个字节，但是真正在用到时占用4个字节

**操作数栈**

字节码执行时需要的数据结构，因此Java解释执行引擎是基于栈的执行引擎，里面的“栈”就是操作数栈。

**动态连接**

指向运行时常量池该栈帧所属方法的引用

**方法返回地址**

方法退出分为两种情况，一种是正常退出，一种是异常退出。

无论采用何种退出方式，在方法退出之后，都必须返回到最初方法被调用时的位置，程序才能继续执行。 一般来说，方法正常退出时，主调方法的PC计数器的值就可以作为返回地址。而方法异常退出时，返回地址是要通过异常处理器表来确定的，栈帧中就一般不会保存这部分信息。

**附加信息**

`JVMR`允许虚拟机实现增加一些规范里没有描述的信息到栈帧之中，例如与调试、 性能收集相关的信息，这部分信息完全取决于具体的虚拟机实现。

### 3、本地方法栈

执行本地方法，`HotSpot`虚拟机直接将它与虚拟机栈合并

### 4、堆

对象分配的区域，G1收集器之前基于经典分代收集理论

### 5、方法区

方法区与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

### 运行时常量池

运行时常量池是方法区的一部分。

`Class`文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域。

除了在编译期生成的常量，还允许动态生成，例如`String`类的`intern()`。

###  直接内存

在`JDK1.4`中新引入了`NIO`类，它可以使用`Native`函数库直接分配堆外内存，然后通过`Java`堆里的`DirectByteBuffer`对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在堆内存和堆外内存来回拷贝数据。

### 堆和方法区的变化

永久代

- `JDK1.8`之前`HotSpot`对方法区的实现
- 在`JDK1.6`就有移出永久代的计划
- 在`JDK1.7`永久代里的部分内容已经被移到别的区域
- 在`JDK1.8`永久代这个概念不复存在，转而被元空间代替（`MetaSpace`）

字符串常量池与静态变量

- 在`JDK1.6`是在永久代（方法区）里的
- 在`JDK1.7`被移到了堆区

元空间

- `JDK1.8`开始，`HotSpot`对方法区的实现，代替了永久代
- 元空间不在JVM中，而是在本地内存里，所以默认情况下其大小不受`JVM`控制，但也可以设置参数来指定大小

## 二、对象内存布局与分配

### 对象内存布局

> [HotSpot Glossary of Terms](http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html)

![image-20201205104407172](https://gitee.com/p8t/picbed/raw/master/imgs/20201205104408.png)

![image-20201205104702272](https://gitee.com/p8t/picbed/raw/master/imgs/20201205104703.png)

#### 1、对象头（Object Header）

对象头包括三部分

- `mark word`：记录了`GC`分代和年龄，`synchronized`锁，`hashCode`，线程`ID`等信息
- `klass pointer`：指向该对象所属哪个类的引用，实际上指向的就是它的类对象
- 数组长度：非数组对象没有该部分

#### **mark word**

32位虚拟机占4字节，64位虚拟机占8字节

**klass pointer**

32位虚拟机占4字节，64位虚拟机占8字节。但是64位环境下默认开启了指针压缩，仍然占用4字节，开启的原因是4字节足够用了。使用下面参数显示开启或关闭

```bash
-XX:+UseCompressedOops
-XX:-UseCompressedOops
```

**数组长度**

无论32位还是64位环境都占4字节

#### 2、实例数据（Instance Data）

记录了对象非静态变量的信息

![image-20201205101630576](https://gitee.com/p8t/picbed/raw/master/imgs/20201205101632.png)

#### 3、对齐填充（Padding）

整个对象长度8字节对齐

### 对象内存分配

对于基于`Mark Compact`算法的垃圾收集器，对象分配通过“指针碰撞”的方式：用指针指向已分配内存和未分配内存的边界，当有新内存被分配时，只需移动指针即可。

对于`CMS`这种基于`Mark Sweep`算法的垃圾收集器，理论上就只能采用较为复杂的空闲列表来分配内存。强调“理论上”是因为在CMS的实现里面，为了能在多数情况下分配得更快，设计了一个叫作`Linear Allocation Buffer`的分配缓冲区，通过空闲列表拿到一大块分配缓冲区之后，在它里面仍然可以使用指针碰撞方式来分配。

除了上面如何分配的问题，还需要考虑线程安全问题，因为堆区是共享的，解决方式如下

- `CAS`：通过`CAS`保证从分配对象到移动碰撞指针的原子性
- `TLAB`：为每个线程在堆区预留一部分空间，用于对象内存分配

通过如下命令开启或关闭`TLAB`

```bash
-XX:+UseTLAB
-XX:-UseTLAB
```

## 三、垃圾收集

垃圾收集主要是针对堆和方法区进行。程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后就会消失，因此不需要对这三个区域进行垃圾回收。

### 判断对象是否可以被回收

#### 1、引用计数算法

为对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。

该算法简单，`Python`用的就是这个算法，但是存在循环引用时，无法回收，`JVM`没有使用这种算法

#### 2、可达性分析算法

以`GC Roots`为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收。

`JVM`使用该算法来判断对象是否可被回收，`GC Roots`一般包含以下内容：

- 栈帧中局部变量表中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象
- 本地方法栈中`JNI`中引用的对象
- 被`synchronized`持有的对象
- `JVM`内部的引用，比如一些常驻的异常对象，基本类型的`Class`对象

### 三色标记法

`JVM`判断对象是否存活用的是可达性分析算法，其具体实现为三色标记法

#### 实现

从`GC roots`开始，把遍历对象图过程中遇到的对象，按照“是否访问过”这个条件标记成以下三种颜色：

- 白色：表示对象尚未被垃圾收集器访问过。显然在可达性分析刚刚开始的阶段，所有的对象都是白色的，若在分析结束的阶段，仍然是白色的对象，即代表不可达。
- 黑色：表示对象已经被垃圾收集器访问过，且这个对象的所有引用都已经扫描过。黑色的对象代表已经扫描过，它是安全存活的，如果有其他对象引用指向了黑色对象，无须重新扫描一遍。黑色对象不可能直接（不经过灰色对象）指向某个白色对象。
- 灰色：表示对象已经被垃圾收集器访问过，但这个对象上至少存在一个引用还没有被扫描过。

#### 并发问题

对于并行的垃圾收集器（`CMS，G1`），它在对象图上标记颜色的同时，用户线程在修改引用关系——即修改对象图的结构，这样可能出现两种后果。

- 把原本消亡的对象错误标记为存活， 这其实是可以容忍的，只不过产生了一点逃过本次收集的浮动垃圾而已，下次收集清理掉就好。

![image-20201206181350483](https://gitee.com/p8t/picbed/raw/master/imgs/20201206181352.png)

- 把原本存活的对象错误标记为已消亡，这就是非常致命的后果了，程序肯定会因此
  发生错误。

![image-20201206181946818](https://gitee.com/p8t/picbed/raw/master/imgs/20201206181947.png)

#### 解决方法

观察上图，当且仅当以下两个条件同时满足时，会产生“对象消失”的问题，即原本应该是黑色的对象被误标为白色： 

1）、赋值器插入了一条或多条从黑色对象到白色对象的新引用；
2）、赋值器删除了全部从灰色对象到该白色对象的直接或间接引用。

只需要破坏上述一个条件即可

- 增量更新：破坏第一个条件。当黑色对象插入新的指向白色对象的引用关系时，就将这个新插入的引用记录下来，等并发扫描结束之后，再将这些记录过的引用关系中的黑色对象为根，重新扫描一次。这可以简化理解为，黑色对象一旦新插入了指向白色对象的引用之后，它就变回灰色对象 了。
- 原始快照：破坏第二个条件。当灰色对象要删除指向白色对象的引用关系时，就将这个要删除的引用记录下来，在并发扫描结束之后，再将这些记录过的引用关系中的灰色对象为根，重新扫描一次。这也可以简化理解为，无论引用关系删除与否，都会按照刚刚开始扫描那一刻的对象图快照来进行搜索。

**简单总结如下**

![image-20201206183114173](https://gitee.com/p8t/picbed/raw/master/imgs/20201206183115.png)

- 增量更新：有黑色节点插入白色节点的引用。则该黑色节点重新扫描。`CMS`使用
- 原始快照：有灰色节点删除白色节点的引用。则该灰色节点重新扫描。`G1、Shenandoah`使用

重新扫描时机为并发扫描结束后，统一通过写屏障完成

### 回收方法区

方法区的垃圾收集主要回收两部分内容：**废弃的常量**和**不再使用的类型**。回收废弃常量与回收
Java堆中的对象非常类似。举个常量池中字面量回收的例子，假如一个字符串“java”曾经进入常量池中，但是当前系统又没有任何一个字符串对象的值是“java”，换句话说，已经没有任何字符串对象引用常量池中的“java”常量，且虚拟机中也没有其他地方引用这个字面量。如果在这时发生内存回收，而且垃圾收集器判断确有必要的话，这个“java”常量就将会被系统清理出常量池。常量池中其他类（接口）、方法、字段的符号引用也与此类似。

类的卸载需要满足以下条件，但是满足了也不一定会回收，只是允许被回收 

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
- 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。
- 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

在大量使用反射、动态代理、CGLib等场景中，需要`JVM`具备类型卸载的能力，以避免方法区内存溢出。

### 垃圾收集算法

#### 标记清除

标记阶段，把存活的对象都标记出来

清除阶段，把未标记的死亡对象占用的内存释放，需要通过空闲链表记录哪些内存可以使用

- 效率高
- 会产生许多内存碎片，影响大对象分配

#### 标记压缩

把存活的对象按序移动到一边，这样剩下的另一边都是可用内存

- 效率低
- 不会产生内存碎片

#### 标记复制

把内存分为两部分，每次只使用一块，垃圾回收时，把一块的存活对象按序复制到另一块上

- 效率高
- 浪费一半内存空间

### HotSpot分代模型

以`JDK9`默认垃圾收集器`G1`为界限，之前的垃圾收集器都是服务于经典的分代模型。虽然`G1`保留了新生代和老年代的概念，但是内存布局和垃圾收集过程已经产生了革命的变化，不能以经典的分代模型看待，下面这个图就是经典的分代模型

![image-20201205163013525](https://gitee.com/p8t/picbed/raw/master/imgs/20201205163014.png)

### 垃圾收集器

对于不同的分代，有不同的垃圾收集器，在经典分代模型中，产生了6种垃圾收集器，三种工作在新生代，三种工作在老年代。

下图标注`JDK9`的部分表示该组合在`JDK9`已经不能使用。`CMS`和`Serial Old`连线表示在`CMS`工作过程中发现内存碎片不足以分配大对象或内存不够时，则会触发`Concurrent Mode Failure`，然后暂停用户线程，执行`Full GC`，该过程由`Serial Old`完成

![image-20201205163710597](https://gitee.com/p8t/picbed/raw/master/imgs/20201205163711.png)

单/多线程：指垃圾回收线程是一个还是多个

串/并行：指垃圾回收线程能否与用户线程并行执行

$$吞吐量 = \frac{执行有效任务时间}{执行有效任务时间 + 垃圾收集时间}$$

#### 1、Serial

基于标记复制的工作在新生代的单线程串行垃圾收集器

#### 2、ParNew

基于标记复制的工作在新生代的多线程串行垃圾收集器

`Serial`的多线程版本

#### 3、Parallel Scavenge

基于标记复制的工作在新生代的多线程串行垃圾收集器

和`ParNew`不同的是，它关注的是吞吐量

#### 4、Serial Old

基于标记整理的工作在老年代的单线程串行垃圾收集器

#### 5、Parallel Old

基于标记整理的工作在老年代的多线程串行垃圾收集器

搭配`Parallel Scavenge`使用，适用于注重吞吐量的场景

#### 6、CMS

基于**标记清除**的工作在老年代的多线程并行垃圾收集器

目标是低延迟

工作流程：

1）初始标记：串行，标记`GC Roots`能直接关联到的对象

2）并发标记：并行，可达性分析，递归遍历整个对象图

3）重新标记：串行，修正并发标记过程中的误标，使用增量更新方式

4）并发清除：并行

在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

缺点：

- 吞吐量低
- 会产生浮动垃圾：在并发标记，并发清除的过程中会产生无法标记的垃圾，只能等到下一次清除。因此，不能等老年代满了才开始执行垃圾回收，需要预留一些内存。如果回收赶不上分配（内存不够），则会出现`Concurrent Mode Failure`，触发`Full GC`，该过程需要暂停用户线程，由`Serial Old`来执行
- 会产生内存碎片：如果没有连续的内存足够分配给大对象，则会提前触发`Full GC`

#### 7、G1

> [详细信息](https://docs.oracle.com/en/java/javase/15/gctuning/garbage-first-g1-garbage-collector1.html#GUID-F1BE86FA-3EDC-4D4F-BDB4-4B044AD83180)

`Garbage First`收集器是垃圾收集器技术发展历史上的里程碑式的成果，它开创了收集器面向局部收集的设计思路和基于`Region`的内存布局形式。

`G1`把整个堆区划分为一个个等大小的`Region`，每一个`Region`都可以表示`Eden`区，`Survivor`区，`Tenured`区，此外还有`Humongous`区，专门用于存储大对象。当一个对象超过一个`Region`一半大小时就会被分配在`Humongous`区，如果一个`Humongous`不够分配，则会分配在连续的`Humongous Region`

![image-20201205222811593](https://gitee.com/p8t/picbed/raw/master/imgs/20201205222812.png)

``G1``是一个基于停顿时间模型的垃圾收集器，它会跟踪各个`Region`里面的垃圾堆积的“价值”大小，然后维护一个优先级列表，每次根据用户设定允许的收集停顿时间，优先处理回收价值收益最大的那些`Region`。 这种使用`Region`划分内存空间，以及具有优先级的区域回收方式，保证了`G1`收集器在有限的时间内获取尽可能高的收集效率。

`G1`垃圾收集分为`Young GC，Mixed GC`。如果回收赶不上分配，则会启动`Full GC`

> [详细信息](https://hllvm-group.iteye.com/group/topic/44381)

`Young GC`与以往垃圾收集器一样，都是新生代满了后触发，全程`STW`。回收`Eden`区和`Survivor`区

`Mixed GC`触发需要一个阈值，默认为老年代占全堆的45%，此时会开始并发标记过程，完成后把`Young GC`切换为`Mixed GC`，除了回收`Eden`和`Survivor`区外，根据设置的期望暂停时间，选择性的回收收益高的`old`区和`H`区

![image-20210106184823699](https://gitee.com/p8t/picbed/raw/master/imgs/20210106184825.png)

> [G1 垃圾收集器 Young GC 和 Mixed GC 有什么不同？](https://www.v2ex.com/t/682188)

如果我们不去计算用户线程运行过程中的动作（如使用写屏障维护记忆集的操作），`G1`收集器的 全局标记过程大致可划分为以下四个步骤： 

1）初始标记：串行，标记`GC Roots`能直接关联到的对象，这个阶段借用`YoungGC STW`时期完成

2）并发标记：并行，可达性分析，递归遍历整个对象图

3）最终标记：串行，修正并发标记过程中的误标，使用原始快照方式

4）筛选回收：串行，对各个`Region`的回收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划，然后把决定回收的那一部分`Region`的存活对象复制到空的`Region`中，再清理掉整个旧`Region`的全部空间。

`G1`从整体来看是基于“标记-整理”算法实现的收集器，但从局部（两个`Region`之间）上看又是基于“标记-复制”算法实现

### 垃圾收集过程

- `Minor GC`：单独收集新生代，`Serial，ParNew，Parallel Scavenge`
- `Major GC`：单独收集老年代，目前只有`CMS`存在该行为。**注意：大多数时候会把`Major GC`与`Full GC`混为一谈**
- `Mixed GC`：部分收集堆区，目前只有`G1`存在该行为
- `Full GC`：收集整个堆及方法区，`Serial Old，Parallel Old`

#### Minor GC

**触发条件：`Eden`区满了**

1）一个对象`new`出来以后，存放在`Eden`区。

2）当`Eden`区满了之后，触发`Minor GC`，把`Eden`区和`from`区存活的对象复制到`to`区（`Eden`区和`from`区清空）。

3）如果`to`区已满，则对象直接进入`Tenured`区，不会触发任何`GC`

**长期存活的对象进入老年代**

为对象定义年龄计数器，对象在 `Eden` 出生，每经过一次`Minor GC`并存活，年龄+1，达到`MaxTenuringThreshold`（默认15）则移动到老年代中。

**大对象直接进入老年代**

经过`Minor GC`后，`Eden`区一定是空的。如果这时还不够存放当前对象，则说明这个对象比整个`Eden`区还大，则该对象直接进入老年代。

可以设置大对象阈值，超过此阈值直接分配至老年代

**动态对象年龄判定** 

对象年龄不一定非要达到`MaxTenuringThreshold`（默认15）才能晋升至老年代，如果在`Survivor`空间中相同年龄的对象大小总和大于`Survivor`空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。

**空间分配担保**

在发生`Minor GC`之前，虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那这一次`Minor GC`可以确保是安全的。如果不成立，则虚拟机会先查看`-XX:HandlePromotionFailure`参数的设置值是否允许担保失败（`Handle Promotion Failure`）；如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次`Minor GC`，失败则重新`Full GC`，停顿时间更长；如果小于，或者`-XX:HandlePromotionFailure`设置不允许冒险，那这时就要改为进行一次`Full GC`。

在`JDK 6 Update 24`之后，`-XX:HandlePromotionFailure`参数不会再影响到虚拟机的空间分配担保策略，只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小，就会进行`Minor GC`，否则将进行`Full GC`。

#### Major GC

**触发条件：**

- 调用`System.gc()`，只是建议`JVM`执行，并不一定执行
- 老年代空间不足：大对象直接进入老年代，长期存活的对象进入老年代导致的空间不足
- 空间分配担保失败
- `Concurrent Mode Failure`：针对`CMS`的，并发清理垃圾时，如果内存不够分配，则需要暂停用户线程，由`Serial Old`执行`Full GC`

## 四、类加载系统

### 1、类生命周期

![image-20201206211324358](https://gitee.com/p8t/picbed/raw/master/imgs/20201206211325.png)

### 2、类加载过程

**1）加载**

在加载阶段，Java虚拟机需要完成以下三件事情：

- 通过一个类的全限定名来获取定义此类的二进制字节流。 
- 将这个字节流所代表的静态存储结构转化为**方法区的运行时数据结构**。 
- 在内存中生成一个代表这个类的`java.lang.Class`对象，作为方法区这个类的各种数据的访问入口。

**2）验证**

这一阶段的目的是确保`Class`文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。

使用纯粹的`Java`代码无法做到违反《Java虚拟机规范》的事情，如果尝试这样去做了，编译器会拒绝编译。但`Class`文件并不一定只能由`Java`源码编译而来，它可以使用包括靠键盘0和1直接在二进制编辑器中敲出`Class`文件在内的任何途径产生。

**3）准备**

准备阶段是正式为静态变量分配内存并设置初始零值的阶段，`JDK1.6`静态变量存在方法区，`JDK1.7`静态变量存在堆区

注意下面的`value`值经过准备阶段后，被赋为0值，而不是123

```java
static int value = 123;
```

但是下面的`value`值在编译时字段属性表就已经有对应的常量值，因此在准备阶段直接赋为123

```java
final static int value = 123;
```

**4）解析**

解析阶段是`Java`虚拟机将常量池内的符号引用替换为直接引用的过程

- 符号引用（`Symbolic References`）：在一个目标加载进内存之前，不可能知道它在内存中的位置，因此也就无法在`Class`文件中直接表明这个目标的地址，只能用逻辑上的“地址”来表示。这就是符号引用
- 直接引用（`Direct References`）：直接指向目标在内存中的位置。

**5）初始化**

直到初始化阶段，`Java`虚拟机才真正开始执行类中编写的`Java`程序代码，将主导权移交给应用程序。之前几个类加载的动作里，除了在加载阶段用户应用程序可以通过自定义类加载器的方式局部参与外，其余动作都完全由`Java`虚拟机来主导控制。

在准备阶段，静态变量已经赋过一次初始零值，而在初始化阶段，则会根据程序代码去初始化静态变量和其他资源。实际上，初始化阶段就是执行类构造器`<clinit>()`方法的过程。`<clinit>()`并不是在`Java`代码中直接编写的方法，它是**`Javac`编译器**的自动生成物。

- `<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态代码块（`static{}`块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问。
- `Java`虚拟机会保证在子类的`<clinit>()`方法执行前，父类的`<clinit>()`方法已经执行完毕。
- `Java`虚拟机必须保证一个类的`<clinit>()`方法在多线程环境中被正确地加锁同步，如果多个线程同时去初始化一个类，那么只会有其中一个线程去执行这个类的`<clinit>()`方法，其他线程都需要阻塞等 待，直到活动线程执行完毕`<clinit>()`方法。如果在一个类的`<clinit>()`方法中有耗时很长的操作，那就可能造成多个进程阻塞。

### 3、类加载时机

#### 主动引用

《Java虚拟机规范》中并没有强制约束何时进行类加载。但是对于初始化阶段，《Java虚拟机规范》 则是严格规定了**有且只有**六种情况必须立即对类进行“初始化”（而加载、验证、准备自然需要在此之前开始）：

**1）**遇到`new、getstatic、putstatic`或`invokestatic`这四条**字节码指令**时，如果类没有进行过初始 化，则需要先触发其初始化阶段。能够生成这四条指令的典型Java代码场景有： 

- 使用`new`关键字实例化对象的时候。
- 读取或设置一个类型的静态字段（被`final`修饰、已在编译期把结果放入常量池的静态字段除外）的时候。
- 调用一个类型的静态方法的时候。

**2）**`Class.forName("com.example.xxx")`反射调用时，如果类没有进行过初始 化，则需要先触发其初始化阶段。

**3）**当初始化类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。 

**4）**当虚拟机启动时，先初始化**`main()`方法**所在的那个类。

**5）**当使用JDK 7新加入的动态语言支持时，如果一个`java.lang.invoke.MethodHandle`实例最后的解析结果为`REF_getStatic、REF_putStatic、REF_invokeStatic、REF_newInvokeSpecial`四种类型的方法句柄，并且这个方法句柄对应的类没有进行过初始化，则需要先触发其初始化。

**6）**当一个接口中定义了`JDK8`新加入的**默认方法**时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。

#### 被动引用

除了以上六种场景外，所有引用类型的方式都不会触发初始化，称为被动引用。

**1）**通过子类引用父类的静态字段，不会导致子类初始化。

```java
public class Main {
    public static void main(String[] args) throws InterruptedException, ClassNotFoundException {
        System.out.println(B.a);	// 只会初始化A, 不会初始化B
    }
}
class A {
    static int a = 1;
}
class B extends A {
}
```

**2）**通过数组定义来引用类，不会触发此类的初始化。该过程会对数组类进行初始化，数组类是一个由虚拟机自动生成的、直接继承自 Object 的子类，其中包含了数组的属性和方法。

```java
A[] a = new A[10];	// 不会初始化A
```

3）常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

```java
public class Main {
    public static void main(String[] args) throws InterruptedException, ClassNotFoundException {
        System.out.println(A.a); // 不会初始化A
    }
}
class A {
    final static int a = 1;
}
```

### 4、类加载器分类

从`Java`虚拟机的角度来讲，只存在以下两种不同的类加载器：

- 启动类加载器（`Bootstrap ClassLoader`），使用`C++`实现，是虚拟机自身的一部分；
- 所有其它类的加载器，使用`Java`实现，独立于虚拟机，继承自抽象类 `java.lang.ClassLoader`。

从`Java`开发人员的角度看，类加载器可以划分得更细致一些：

- 启动类加载器（`Bootstrap ClassLoader`）此类加载器负责将存放在`<JRE_HOME>\lib`目录中的，或者被`-Xbootclasspath`参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如`rt.jar`，名字不符合的类库即使放在`lib`目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被`Java`程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用`null`代替即可。
- 扩展类加载器（`Extension ClassLoader`）这个类加载器是由 `ExtClassLoader`（`sun.misc.Launcher$ExtClassLoader`）实现的。它负责将 `<JAVA_HOME>/lib/ext` 或者被 `java.ext.dir` 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器。
- 应用程序类加载器（`Application ClassLoader`）这个类加载器是由 `AppClassLoader`（`sun.misc.Launcher$AppClassLoader`）实现的。由于这个类加载器是 `ClassLoader` 中的 `getSystemClassLoader()` 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（`ClassPath`）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

### 5、双亲委派模型

`JDK 9`之前的`Java`应用由上面三种类加载器互相配合来完成加载的，此外还可以加入自定义的类加载器。

下图展示了类加载器之间的层次关系，称为双亲委派模型（`Parents Delegation Model`）。该模型要求除了顶层的启动类加载器外，其它的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系（`Composition`）来实现，而不是继承关系（`Inheritance`）。

![image-20201217213027948](https://gitee.com/p8t/picbed/raw/master/imgs/20201217213029.png)

#### 工作过程

双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加 载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

#### 好处

使用双亲委派模型来组织类加载器之间的关系，一个显而易见的好处就是`Java`中的类随着它的类 加载器一起具备了一种带有优先级的层次关系。例如类`java.lang.Object`，它存放在`rt.jar`之中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的启动类加载器进行加载，因此`Object`类在程序的各种类加载器环境中都能够保证是同一个类。反之，如果没有使用双亲委派模型，都由各个类加载器自行去加载的话，如果用户自己也编写了一个名为`java.lang.Object`的类，并放在程序的`ClassPath`中，那系统中就会出现多个不同的`Object`类，`Java`类型体系中最基础的行为也就无从保证，应用程序将会变得一片混乱。如果尝试去写一个与`rt.jar`类库中已有类重名的`Java`类，将会发现它可以正常编译，但永远无法被加载运行。

#### 实现

```java
public abstract class ClassLoader {
    
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        // 父加载器不为空, 向上委托
                        c = parent.loadClass(name, false);
                    } else {
                        // 父加载器为空, 委托引导类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                    // 父类无法加载, 抛异常, 交给子类加载
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
    	}
	}
}
```

### 6、破坏双亲委派模型

#### 第一次破坏

双亲委派模型第一次被破坏发生在它出现之前。由上面可知，双亲委派模型的实现依赖于`java.lang.ClassLoader`的`loadClass()`方法，`JDK1.2`才实现该模型，但是类加载器的概念和抽象类`java.lang.ClassLoader`则在`JDK1.0`就已经存在，设计该类时也没有意识到将来会实现双亲委派模型，因此开放了用户**重写`loadClass()`**的权限。面对已经存在的用户自定义类加载器的代码，`Java`设计者们引入双亲委派模型时不得不做出一些妥协，为了兼容这些已有代码，无法再以技术手段避免`loadClass()`被子类覆盖的可能性，只能在`JDK1.2`之后的`java.lang.ClassLoader`中添加一个新的`protected`方法`findClass()`，并引导用户编写的类加载逻辑时尽可能去重写这个方法，而不是在`loadClass()`中编写代码。按照`loadClass()`方法的逻辑，如果父类加载失败，会自动调用自己的`findClass()`方法来完成加载，这样既不影响用户按照自己的意愿去加载类，又可以保证新写出来的类加载器是符合双亲委派规则的。

#### 第二次破坏

双亲委派模型的第二次被破坏是由这个模型自身的缺陷导致的。双亲委派很好地解决了各个类加载器协作时基础类型的一致性问题（越基础的类由越上层的加载器进行加载），基础类型之所以被称为“基础”，是因为它们总是作为被用户代码继承、调用的`API`存在。但也会有基础类型调用用户代码的情况出现。比如`JDBC`，显然这是一个上层类，在`JDK`中对数据库的连接操作只定义了一些接口，具体实现由数据库厂商完成。因此在真正建立数据库连接时，就会出现`JDBC`调用服务厂商实现类的代码的情况出现，而上层类加载器不可能认识这些实现类，这就出现了无法加载的问题。

为了解决这个困境，Java的设计团队只好引入了一个不太优雅的设计：线程上下文类加载器 （`Thread Context ClassLoader`）。这个类加载器可以通过`java.lang.Thread`类的`setContext-ClassLoader()`方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。

有了线程上下文类加载器，程序就可以做一些“舞弊”的事情了。上层使用这个线程上下文类加载器去加载所需的下层类，这是一种父类加载器去请求子类加载器完成类加载的行为，这种行为实际上是打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型的一般性原则，但也是无可奈何的事情。

```java
// 在java.sql.DriverManager的getConnection方法中可以看到通过线程上下文获取类加载器
private static Connection getConnection(
    // ...
	callerCL = Thread.currentThread().getContextClassLoader();
	// ...
}
```

#### 第三次破坏

OSGI模块化部署，类加载请求不再是简单的向上委托，而是分多种情况进行委托加载

#### 第四次破坏

### 7、破坏双亲委派模型实战

> [参考](https://www.bilibili.com/video/BV1CJ411g7U4)

1）前置知识：全盘负责机制，**如果一个类由A类加载器加载，在不特殊指定的情况下，它所依赖的所有类都由A类加载器来加载**

2）目的：在同一个`JVM`中加载两个全类名相同的类

3）操作步骤：

1、自定义类加载器

2、重写双亲委派模型核心方法`loadClass()`

3、`new`两个自定义类加载器来加载类

**环境准备**

现有两个jar包：`A.jar`和`B.jar`，结构如下。现在需要把两个`C.class`都加载进同一`JVM`，否则`A`和`B`总有一个不能工作

```yaml
A.jar
--com
----example
------A.class # 依赖当前目录下的C.class
------C.class # 版本为1.0

B.jar
--com
----example
------B.class # 依赖当前目录下的C.class
------C.class # 版本为2.0
```

```java
// A.jar
package com.example;
public class A {
    public A() {
        if ((new C()).version().equals("v1.0")) {
            System.out.println("A OK");
        } else {
            System.out.println("A ERROR");
        }
    }
}

package com.example;
public class C {
    public String version() {
        return "v1.0";
    }
}

// B.jar
package com.example;
public class B {
    public B() {
        if ((new C()).version().equals("v2.0")) {
            System.out.println("B OK");
        } else {
            System.out.println("B ERROR");
        }
    }
}

package com.example;
public class C {
    public String version() {
        return "v2.0";
    }
}
```

下面自定义类加载来加载分别加载A和B，进而引入不同的C，当然也可以直接加载两个C

```java
public class MyClassLoader extends ClassLoader {
    String jarDirName;
    String jarBaseName;
    // 加载过的类不再重新加载
    Map<String, Class<?>> cache;

    // 获取jar包绝对路径
    public MyClassLoader(String jarDirName, String jarBaseName) {
        this.jarDirName = jarDirName;
        this.jarBaseName = jarBaseName;
        cache = new HashMap<>();
    }

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        if (!name.startsWith("com.example")) {
            return super.loadClass(name);
        }
        if (cache.containsKey(name)) {
            return cache.get(name);
        }
        byte[] bytecode;
        try {
            // 获取字节码文件的字节数组
            bytecode = getBytes(name);
            // Converts an array of bytes into an instance of class {@code Class}.
            return defineClass(name, bytecode, 0, bytecode.length);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return super.loadClass(name);
    }

    // 获取类字节码, 忽略异常处理
    private byte[] getBytes(String className) throws IOException {
        // 把全类名改为路径形式
        String tmp = className.replaceAll("\\.", "/");
        JarFile jar = new JarFile(jarDirName + File.separator + jarBaseName);
        // 需要加载的.class文件
        JarEntry entry = jar.getJarEntry(tmp + ".class");
        InputStream is = jar.getInputStream(entry);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        int len;
        byte[] bytes = new byte[1024];
        while ((len = is.read(bytes)) != -1) {
            baos.write(bytes, 0, len);
        }
        byte[] data = baos.toByteArray();
        is.close();
        baos.close();
        return data;
    }
}
```

```java
public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException, MalformedURLException {
    String jarDirName = System.getProperty("user.dir") + File.separator + "lib";

    MyClassLoader cl1 = new MyClassLoader(jarDirName, "A.jar");
    MyClassLoader cl2 = new MyClassLoader(jarDirName, "B.jar");

    Class<?> a = cl1.loadClass("com.example.A");
    Class<?> b = cl2.loadClass("com.example.B");

    System.out.println(a.getClassLoader());
    System.out.println(b.getClassLoader());

    a.getConstructor().newInstance();
    b.getConstructor().newInstance();

    // 可以直接使用URLClassLoader, 这样就不用手动把字节码转数组了, 注意要把parent要指定为null
    // URLClassLoader cl3 = new URLClassLoader(new URL[]{new URL("file:" + jarDirName + File.separator + "A.jar")}, null);
    // URLClassLoader cl4 = new URLClassLoader(new URL[]{new URL("file:" + jarDirName + File.separator + "B.jar")}, null);
    //
    // Class<?> a = cl3.loadClass("com.example.A");
    // Class<?> b = cl4.loadClass("com.example.B");
    //
    // System.out.println(a.getClassLoader());
    // System.out.println(b.getClassLoader());
    //
    // a.getConstructor().newInstance();
    // b.getConstructor().newInstance();
}
```













## 五、JVM参数设置

> [详细信息](https://docs.oracle.com/en/java/javase/11/tools/java.html#GUID-BE93ABDC-999C-4CB5-A88B-1994AAAC74D5)

### 1、参数分类

```bash
-   : 标准参数
-X  : 非标准参数, 变化较小
-XX : 非标准参数, 变化较大, 可能被移除
```

### 2、杂项

**打印设置的参数信息**

- `-XX:+PrintCommandLineFlags`

**执行引擎设置**

- `-Xint `：仅使用解释器
- `-Xcomp`：仅使用JIT
- `-Xmixed`：混合模式（默认）

**打印GC信息**

- `-XX:+PrintGCDetails`：在`JDK11`已废弃
- `-Xlog:gc*`

**查看运行时信息**

- `jps`：查看当前运行中的`JVM`进程及`pid`
- `jinfo -flag <arg> pid`：查看某一参数信息
- `jinfo -flags pid`：查看所有参数信息（不全）
- `jstat -gc pid`：查看gc情况

**关闭指针压缩**

- `-XX:-UseCompressedOops`

### 3、堆区

- `-Xms?m`：初始堆空间大小（默认为物理内存/64）

  > `-XX:InitialHeapSize=?m`

- `-Xmx?m`：最大堆空间大小（默认为物理内存/4）

  > `-XX:MaxHeapSize=?m`

- `-Xmn?m`：设置新生代的大小

  > `-XX:MaxNewSize=?m`
  >
  > 经测试，优先级高于`-XX:NewRatio`，并且不会超出`-Xmx`的大小。其次老年代至少会有512k，也就是说`Xmn`最多只能分配到`Xmx-512k`的容量

- `-XX:NewRatio=?`：配置老年代与新生代内存大小比例（默认2）

- `-XX:SurvivorRatio=?`：设置新生代中Eden和S0/S1空间的比例（默认8）

  > Sets the ratio between eden space size and survivor space size. By default, this option is set to 8.
  > 官网说默认是8，但实际测试为6，即使关了自适应内存分配策略还是6，
  > 只能手动指定才能变为8

- `-XX:MaxTenuringThreshold=?`：设置新生代垃圾的最大年龄（默认15）

  > 年龄信息在对象头中，占4位，所以最大也只能设置为15

- `-XX:-UseAdaptiveSizePolicy`：关闭自适应内存分配策略
- `-XX:HandlePromotionFailure=?(boolean)`：是否启用空间分配担保

### 4、方法区

> 方法区在1.7的实现为永久代（PermGen），是堆区逻辑上的一部分。1.8实现为元空间（MetaSpace），用的是本地内存
>
> 名字变了，对应设置命令也变了

JDK1.7（win10）

- `-XX:PermSize=?m`：初始大小（默认`20.75M`）
- `-XX:PermMaxSize=?m`：最大大小（默认`82M`）

JDK1.8（win10）

- `-XX:MetaspaceSize=?m`：初始大小（默认`20.796875M`）
- `-XX:MaxMetaspaceSize=?m`：最大大小（默认$$2^{64}-65536$$）

### 5、虚拟机栈

- `-Xss:?k`：设置栈大小，初始值在64位的`Linux、Mac、Solaris`上都是`1024K`，在`windows`上依赖于`virtual memory`

  > `-XX:ThreadStackSize=?k`
  >
  > 
  >
  > 栈大小影响栈深
  >
  > 单线程环境下，不管是栈太深，还是栈太大，只会报`StackOverflowError`，不会报`OOM`
  >
  > 多线程也很难出现`OOM`

### 6、垃圾收集器

#### G1

> [详细信息](https://www.oracle.com/java/technologies/g1gc.html)

- `-XX:G1HeapRegionSize=?`：设置`region`大小，必须是1M到32M之间的2的幂

- `-XX:MaxGCPauseMillis=200`：设置期望最大暂停时间，默认200ms

- `-XX:G1NewSizePercent=5`：设置新生代最小占堆比例，默认5%

- `-XX:G1MaxNewSizePercent=60`：设置新生代最大占堆比例，默认60%

  > 这个设参数与`-XX:NewRatio`设置效果等价，且显示设置后者会覆盖前者
  >
  > 可以看出G1默认新生区可以很大，这是因为它的大小是动态的，它用不到的内存时可以被老年代使用

- `-XX:InitiatingHeapOccupancyPercent=45`：触发Mixed GC的阈值，默认值45%

- `-XX:ParallelGCThreads=?`：设置STW工作线程数，逻辑核心数小于等于8时，默认值等于逻辑核心数

- `-XX:ConcGCThreads=?`：设置并发标记工作线程数，应当（默认）设置为上面的1/4

- `-XX:G1MixedGCLiveThresholdPercent=65`：如果一个old region存活对象低于这个阈值，那么就会被纳入Mixed GC的目标（实际上应该会被放入一个待清理列表，是否清理取决于目标停顿时间），默认值65%

## 六、性能调优

> [参考](https://www.bilibili.com/video/BV1PJ411n7xZ)

### 性能优化步骤

1. 性能监控
2. 性能分析
3. 性能调优

### 性能指标

- 响应时间：从发出请求到收到响应所需要的时间
- 吞吐量：单位时间工作量
- 并发量
- 内存占用

### 监控工具

### 1、JDK自带

```shell
# 查看运行中的JVM进程
jps
	-l # 列出全类名
	-q # 只显示PID
	-m # 显示给main函数传递的参数
	-v # 显示虚拟机参数

# 查看JVM统计信息
jstat <option> <pid> [interval] [count] # 每隔interval查询一次信息, 共查询count次 单位ms
	-t # 加上时间戳, 即程序执行的时间 单位s
	-h # 每隔一定查询次数重新输出表头
	<option>
		-class # 查询类加载信息
		-compiler # 查询JIT编译信息
		-printcompilation # 查询JIT编译的方法
		-gc
		...
		https://docs.oracle.com/en/java/javase/11/tools/jstat.html

# 实时查看JVM参数
jinfo <option> <pid>
	-sysprops # 查看系统相关信息
	-flag <arg>
	-flags
	
# 堆区快照
jmap -dump:format=b,file=<file.hprof> <pid>
jmap -dump:live,format=b,file=<file.hprof> <pid> # 只保留存活的对象
# 自动生成dump文件
-XX:HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<file.hprof>

# 线程快照, 可以检测死锁
jstack <pid>

# 综合指令
jcmd <pid> help # 查询可以使用的参数
jcmd <pid> Thread.print # 相当于jsatck
jcmd <pid> GC.heap_dump <file.hprof> # 相当于jmap
jcmd <pid> VM.uptime # JVM运行时间 相当于jstat -t
jcmd <pid> VM.flags #
```

