# Java内存模型

## [1. 基础知识](https://javaguide.cn/java/jvm/memory-area.html#运行时数据区域)

**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区
- 直接内存 (非运行时数据区的一部分)

Java 虚拟机规范对于运行时数据区域的规定是相当宽松的。以堆为例：堆可以是连续空间，也可以不连续。堆的大小可以固定，也可以在运行时按需扩展 。虚拟机实现者可以使用任何垃圾回收算法管理堆，甚至完全不进行垃圾收集也是可以的。

![image-20250126113840492](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261138672.png)

![image-20250126113911449](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261139470.png)

## [2. 程序计数器](https://javaguide.cn/java/jvm/memory-area.html#程序计数器)

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。

另外，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
- 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

## [3. Java 虚拟机栈](https://javaguide.cn/java/jvm/memory-area.html#java-虚拟机栈)

栈绝对算的上是 JVM 运行时数据区域的一个核心，**除了一些 Native 方法**调用是通过**本地方法栈**实现的(后面会提到)，其他所有的 **Java 方法**调用都是通过**栈**来实现的（也需要和其他运行时数据区域比如程序计数器配合）。

**栈帧随着方法调用而创建，随着方法结束而销毁。无论方法正常完成还是异常完成都算作方法结束。**

栈空间虽然不是无限的，但一般正常调用的情况下是不会出现问题的。不过，如果函数调用陷入无限循环的话，就会导致栈中被压入太多栈帧而占用太多空间，导致栈空间过深。那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 `StackOverFlowError` 错误。

![image-20250126114243462](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261142485.png)

**局部变量表** 主要存放了编译期可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

**操作数栈** 主要作为方法调用的中转站使用，用于存放方法执行过程中产生的**中间计算结果**。另外，计算过程中产生的**临时变量**也会放在操作数栈中。

**动态链接** 主要服务一个方法需要调用其他方法的场景。Class 文件的常量池里保存有大量的符号引用比如方法引用的符号引用。当一个方法要调用其他方法，需要将常量池中指向方法的符号引用转化为其在内存地址中的直接引用。动态链接的作用就是为了将符号引用转换为调用方法的直接引用，这个过程也被称为 **动态连接** 。

## [4. 本地方法栈](https://javaguide.cn/java/jvm/memory-area.html#本地方法栈)

**虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

## [5. 堆](https://javaguide.cn/java/jvm/memory-area.html#堆)

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。

**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是**垃圾收集器管理的主要区域**，因此也被称作 **GC 堆（Garbage Collected Heap）**。

![image-20250126114958360](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261149383.png)

## [6. 方法区](https://javaguide.cn/java/jvm/memory-area.html#方法区)

方法区属于是 JVM 运行时数据区域的一块逻辑区域，是各个线程共享的内存区域。

当虚拟机要使用一个类时，它需要读取并解析 Class 文件获取相关信息，再将信息存入到方法区。

方法区会存储已被虚拟机加载的 **类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码缓存等数据**。

## [7. 运行时常量池（方法区中）](https://javaguide.cn/java/jvm/memory-area.html#运行时常量池)

用于存储编译期生成的字面量和符号引用。

字面量：

![image-20250126115536906](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261155936.png)

符号引用：

![image-20250126115657364](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261156394.png)

## 8. 字符串常量池（在堆中）

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了**避免字符串的重复创建。**

Java 程序中通常会有大量的被创建的字符串等待回收，将字符串常量池放到堆中，能够更高效及时地回收字符串内存。

## 9. 直接内存

直接内存是一种**特殊的内存缓冲区**，**并不在 Java 堆或方法区中分配的**，而是通过 JNI 的方式在本地内存上分配的。

## 10. 堆和栈的区别

![image-20250210220311270](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502102203316.png)

# [HotSpot 虚拟机对象探秘](https://javaguide.cn/java/jvm/memory-area.html#hotspot-虚拟机对象探秘)

## [1. 对象的创建](https://javaguide.cn/java/jvm/memory-area.html#对象的创建)

[Step1:类加载检查](https://javaguide.cn/java/jvm/memory-area.html#step1-类加载检查)

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在**常量池**中定位到这个类的**符号引用**，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

[Step2:分配内存](https://javaguide.cn/java/jvm/memory-area.html#step2-分配内存)

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 **Java 堆**中划分出来。

- **指针碰撞（Bump the Pointer）**：如果Java堆内存是规整的（即已使用的内存和未使用的内存之间有明确的分界指针），那么分配内存仅仅是将指针向未使用的内存区域移动一段与对象大小相等的距离。
- **空闲列表（Free List）**：如果Java堆内存不是规整的，JVM会维护一个空闲列表，记录哪些内存块是可用的，然后从列表中找到一个足够大的空间分配给对象。

**内存分配并发问题（补充内容，需要掌握）**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

[Step3:初始化零值](https://javaguide.cn/java/jvm/memory-area.html#step3-初始化零值)

内存分配完成后，虚拟机需要将分配到的**内存空间都初始化为零值**（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以**不赋初始值就直接使用**，可以直接使用默认值（如 `0`、`false`、`null` 等），程序能访问到这些字段的数据类型所对应的零值。

![](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261935907.png)

[Step4:设置对象头](https://javaguide.cn/java/jvm/memory-area.html#step4-设置对象头)

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是**哪个类的实例**、如何才能找到类的元数据信息、对象的**哈希码**、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

[Step5:执行 init 方法](https://javaguide.cn/java/jvm/memory-area.html#step5-执行-init-方法)

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

## 2. 对象的内存布局

对象在内存中的布局可以分为 3 块区域：**对象头（Header）**、**实例数据（Instance Data）\**和\**对齐填充（Padding）**。

对象头包括两部分信息：

1. 标记字段（Mark Word）：用于存储对象自身的运行时数据， 如哈希码（HashCode）、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。
2. 类型指针（Klass pointer）：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

**实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

**对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。** 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 **8 字节的整数倍**。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

## [3. 对象的访问定位](https://javaguide.cn/java/jvm/memory-area.html#对象的访问定位)

见javaguide。句柄和没有句柄。

# [内存分配和回收原则](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#内存分配和回收原则)

**Eden 区与两个 Survivor 区**：空间大小比例是 8:1:1, 大因为大部分新创建的对象在短时间内就会变为垃圾，因此可以将更多的空间分配给 Eden 区，以减少 Minor GC 的频率。

**新生代与老年代**：默认情况下，新生代和老年代的比例为 1:2。原因：新创建的对象大部分都是短生命周期的。

![image-20250126114958360](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261228586.png)

## [1. 对象优先在 Eden 区分配](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#对象优先在-eden-区分配)

大多数情况下，对象**在新生代中 Eden 区分配**。当 Eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC。

- **Minor GC** 是指对新生代（Young Generation）的垃圾回收。
- 它的主要目标是清理 **Eden 区** 和 **Survivor 区** 中的**无用对象**，并将存活的对象移动到**另一个 Survivor 区或老年代**（**分配担保机制** 把新生代的对象提前转移到老年代中去）。
- Minor GC 通常比 **Major GC**（针对老年代的垃圾回收）更频繁，因为新生代中的对象生命周期较短，大部分对象在 Minor GC 时就会被回收。

## [2. 大对象直接进入老年代](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#大对象直接进入老年代)

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。

大对象直接进入老年代的行为是由虚拟机动态决定的，它与具体使用的垃圾回收器和相关参数有关。

大对象直接进入老年代是一种优化策略，旨在避免将大对象放入新生代，从而**减少新生代的垃圾回收频率和成本**。

## [3. 长期存活的对象将进入老年代](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#长期存活的对象将进入老年代)

虚拟机给每个对象一个对象年龄（Age）计数器。

对象都会首先在 Eden 区域分配。如果对象在 Eden 出生并经过第一次 **Minor GC** 后仍然能够存活，并且能被 Survivor 容纳的话，将被移动到 **Survivor 空间（s0 或者 s1**）中（轮转），并将对象年龄设为 1(Eden 区->Survivor 区后对象的初始年龄变为 1)。

对象在 Survivor 中**每熬过一次 MinorGC,年龄就增加 1 岁**，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

## [4. 进行 gc 的区域和时机](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#主要进行-gc-的区域)

- 新生代收集（Minor GC / Young GC）：只对新生代进行垃圾收集；Eden 区空间不足触发Minor GC。
- 老年代收集（Major GC / Old GC）：只对老年代进行垃圾收集。需要注意的是 Major GC 在有的语境中也用于指代整堆收集；老年代空间不足触发。
- 混合收集（Mixed GC /full gc）：对整个新生代和部分老年代进行垃圾收集。

## [5. 空间分配担保](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#空间分配担保)

在 Minor GC 之前，JVM 会检查老年代是否有足够的连续空间来存放所有可能从新生代晋升的对象。如果没有足够的空间，JVM 会先触发一次 **Full GC**，清理老年代的空间，然后再进行 Minor GC。这个过程就是空间分配担保。

如果 Full GC 后仍然没有足够的空间，JVM 会抛出 `OutOfMemoryError`。

# [死亡对象判断方法](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#死亡对象判断方法)

## [1. 引用计数法](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#引用计数法)

给对象中添加一个引用计数器：

- 每当有一个地方引用它，计数器就加 1；
- 当引用失效，计数器就减 1；
- 任何时候计数器为 0 的对象就是不可能再被使用的。

**这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间循环引用的问题。**

![image-20250126135923478](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261359513.png)

## [2. 可达性分析算法](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#可达性分析算法)

这个算法的基本思想就是通过一系列的称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的，需要被回收。

下图中的 `Object 6 ~ Object 10` 之间虽有引用关系，但它们到 GC Roots 不可达，因此为需要被回收的对象。

![image-20250126140017822](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261400858.png)

**哪些对象可以作为 GC Roots 呢？**

- **虚拟机栈中的局部变量**
- 方法区中静态变量
- 方法区中常量
- 所有被同步锁持有的对象
- 虚拟机内部引用：类加载器、Class对象

**对象可以被回收，就代表一定会被回收吗？**

即使在可达性分析法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑阶段”，要真正宣告一个对象死亡，至少要经历

两次标记过程；可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行 `finalize` 方法。当对象没有覆盖 `finalize` 方法，或 `finalize` 方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。

被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收。

## [3. 引用类型总结](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#引用类型总结)

在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为**软引用可以加速 JVM 对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生**。

### 3.1 强引用

如果一个对象具有强引用，那就类似于**必不可少的生活用品**，垃圾回收器绝不会回收它。当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题

### 3.2 软引用

如果一个对象只具有软引用，那就类似于**可有可无的生活用品**。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

### 3.3 弱引用

如果一个对象只具有弱引用，那就类似于**可有可无的生活用品**。

弱引用与软引用的区别在于：只具有**弱引用的对象拥有更短暂的生命周期**。在垃圾回收器线程扫描它所管辖的内存区域的过程中，**一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。**

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

### 3.4 虚引用

"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，**那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。**

## [4. 如何判断一个常量是废弃常量？](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#如何判断一个常量是废弃常量)

假如在字符串常量池中存在字符串 "abc"，如果当前没有任何 String 对象引用该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池了。

## [5. 如何判断一个类是无用的类？](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#如何判断一个类是无用的类)

- 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
- 加载该类的 `ClassLoader` 已经被回收。
- 该类对应的 `java.lang.Class` 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

# [垃圾收集算法](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#垃圾收集算法)

## [1. 标记-清除算法](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#标记-清除算法)

分为两个过程：标记和清除。

1. **效率问题**：标记和清除两个过程效率都不高。
2. **空间问题**：标记清除后会产生大量不连续的内存碎片。

![image-20250126141151562](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261411605.png)

## [2. 复制算法](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#复制算法)

- **可用内存变小**：可用内存缩小为原来的一半。
- **不适合老年代**：如果存活对象数量比较大，复制性能会变得很差。

![image-20250126141353755](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261413794.png)

## [3. 标记-整理算法](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#标记-整理算法)

由于多了整理这一步，因此效率也不高，适合**老年代这种垃圾回收频率不是很高**的场景。

![image-20250126141500432](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261415471.png)

## [4. 分代收集算法( HotSpot 为什么要分为新生代和老年代？)](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#分代收集算法)

当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。

一般将 Java 堆分为**新生代和老年代**，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。



比如在**新生代**中，每次收集都会有大量对象死去，所以可以选择“复制”算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。

而**老年代**的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。

# [垃圾收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#垃圾收集器)

**如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现**

## 1. 基础知识

**如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。**

直到现在为止还没有最好的垃圾收集器出现，更加没有万能的垃圾收集器，**我们能做的就是根据具体应用场景选择适合自己的垃圾收集器**。

- JDK 8: Parallel Scavenge（新生代）+ Parallel Old（老年代）
- JDK 9 ~ JDK22: G1

其他知识见javaguide。

![img](https://cdn.xiaolincoding.com//picgo/1712649527581-d6aee0bf-35ab-4406-8a26-270b35ae8771.png)

## [2. Serial 收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#serial-收集器)

Serial（串行）收集器是最基本、历史最悠久的垃圾收集器了。

它的 **“单线程”** 的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ **"Stop The World"** ），直到它收集结束。

**新生代采用标记-复制算法，老年代采用标记-整理算法。**

![Serial 收集器](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502072216891.png)

## [3. Serial Old 收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#serial-old-收集器)

**Serial 收集器的老年代版本**，它同样是一个单线程收集器。它主要有两大用途：一种用途是在 JDK1.5 以及以前的版本中与 Parallel Scavenge 收集器搭配使用，另一种用途是作为 CMS 收集器的后备方案。

![Serial 收集器](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502072225224.png)

## [4. ParNew 收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#parnew-收集器)

ParNew 收集器其实就是 Serial 收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和 Serial 收集器完全一样。

**但是会暂停所有用户线程。**

**新生代采用标记-复制算法，老年代采用标记-整理算法。**

![ParNew 收集器 ](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502072217720.png)

## [5. Parallel Scavenge 收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#parallel-scavenge-收集器)

它看上去几乎和 ParNew 都一样。所谓吞吐量就是 CPU 中用于运行用户代码的时间与 CPU 总消耗时间的比值。

Parallel Scavenge 收集器关注点是**吞吐量**（高效率的利用 CPU）。

CMS 等垃圾收集器的关注点更多的是用户线程的**停顿时间**（提高用户体验）。

 Parallel Scavenge 收集器提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，如果对于收集器运作不太了解，手工优化存在困难的时候，使用 Parallel Scavenge 收集器配合自适应调节策略，把内存管理优化交给虚拟机去完成也是一个不错的选择。

![Parallel Old收集器运行示意图](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502072220630.png)



## [6. Parallel Old 收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#parallel-old-收集器)

**Parallel Scavenge 收集器的老年代版本**。使用多线程和“标记-整理”算法。在注重吞吐量以及 CPU 资源的场合，都可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

![Parallel Old收集器运行示意图](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502072226081.png)

## [7. CMS 收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#cms-收集器)

**CMS（Concurrent Mark Sweep）收集器是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。**

CMS 收集器是一种 **“标记-清除”算法**实现的，它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。整个过程分为四个步骤：

- **初始标记：** 短暂停顿，标记**直接与 root 相连的对象**（根对象）；
- **并发标记：** 同时开启 GC 和用户线程，用一个闭包结构去记录**可达对象**。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
- **重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
- **并发清除：** 开启用户线程，同时 GC 线程开始**对未标记的区域做清扫**。

![CMS 收集器](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502072231747.png)

**并发收集、低停顿**

- **对 CPU 资源敏感；**
- **无法处理浮动垃圾；**
- **它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。**

**CMS 垃圾回收器在 Java 9 中已经被标记为过时(deprecated)，并在 Java 14 中被移除。**

## [8. G1 收集器](https://javaguide.cn/java/jvm/jvm-garbage-collection.html#g1-收集器)

G1 (Garbage-First) 是一款**面向服务器**的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以**极高概率满足 GC 停顿时间**要求的同时,还具备**高吞吐量**性能特征。

**从 JDK9 开始，G1 垃圾收集器成为了默认的垃圾收集器。**

优点：

- **G1 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region(这也就是它的名字 Garbage-First 的由来)** ，保证了 G1 收集器在有限时间内可以尽可能高的收集效率（**把内存化整为零**）。
- **并行与并发**：G1 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 java 程序继续执行。
- **空间整合**：与 CMS 的“标记-清除”算法不同，**G1 从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的，产生的内存碎片少**
- **可预测的停顿**：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能**建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒**。
- **分代收集**：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。

G1 收集器的运作大致分为以下几个步骤：

- **初始标记**： 短暂停顿（Stop-The-World，STW），标记从 GC Roots 可**直接引用**的对象，即标记所有直接可达的活跃对象
- **并发标记**：与应用并发运行，标记**所有可达对象**。 这一阶段可能持续较长时间，取决于堆的大小和对象的数量。
- **最终标记**： 短暂停顿（STW），处理并发标记阶段结束后残留的少量**未处理的引用变更**。
- **筛选回收**：根据标记结果，选择**回收价值高的区域**，复制存活对象到新区域，回收旧区域内存。这一阶段**包含一个或多个停顿**（STW），具体取决于回收的复杂度。

![G1 收集器](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202502072234629.png)

# 类

## 1. 类文件结构

![image-20250126191702305](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261917346.png)

[1.1 魔数（Magic Number）](https://javaguide.cn/java/jvm/class-file-structure.html#魔数-magic-number)

每个 Class 文件的头 4 个字节称为魔数（Magic Number）,它的唯一作用是**确定这个文件是否为一个能被虚拟机接收的 Class 文件**。

Java 规范规定魔数为固定值：0xCAFEBABE。如果读取的文件不是以这个魔数开头，Java 虚拟机将拒绝加载它。

[1.2 Class 文件版本号](https://javaguide.cn/java/jvm/class-file-structure.html#class-文件版本号-minor-major-version)

第 5 和第 6 个字节是**次版本号**，第 7 和第 8 个字节是**主版本号**。

每当 Java 发布大版本（比如 Java 8，Java9）的时候，主版本号都会加 1。你可以使用 `javap -v` 命令来快速查看 Class 文件的版本号信息。

高版本的 Java 虚拟机可以执行低版本编译器生成的 Class 文件，但是低版本的 Java 虚拟机不能执行高版本编译器生成的 Class 文件。所以，我们在实际开发的时候要确保开发的的 JDK 版本和生产环境的 JDK 版本保持一致。

[1.3 常量池（Constant Pool）](https://javaguide.cn/java/jvm/class-file-structure.html#常量池-constant-pool)

字面量和符号引用。字面量比较接近于 Java 语言层面的的常量概念，如文本字符串、声明为 final 的常量值等。而符号引用则属于编译原理方面的概念。包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

[1.4 访问标志(Access Flags)](https://javaguide.cn/java/jvm/class-file-structure.html#访问标志-access-flags)

这个 Class 是类还是接口，是否为 `public` 或者 `abstract` 类型，如果是类的话是否声明为 `final` 等等。

[1.5 当前类（This Class）、父类（Super Class）、接口（Interfaces）索引集合](https://javaguide.cn/java/jvm/class-file-structure.html#当前类-this-class-、父类-super-class-、接口-interfaces-索引集合)

类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。

接口索引集合用来描述这个类实现了哪些接口，这些被实现的接口将按 `implements` (如果这个类本身是接口的话则是`extends`) 后的接口顺序从左到右排列在接口索引集合中。

[1.6 字段表集合（Fields）](https://javaguide.cn/java/jvm/class-file-structure.html#字段表集合-fields)

字段表（field info）用于描述接口或类中声明的变量。字段包括类级变量以及实例变量，但不包括在方法内部声明的局部变量。

![image-20250126192253830](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261922865.png)

[1.7 方法表集合（Methods）](https://javaguide.cn/java/jvm/class-file-structure.html#方法表集合-methods)

![image-20250126192321338](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261923372.png)

[1.8 属性表集合（Attributes）](https://javaguide.cn/java/jvm/class-file-structure.html#属性表集合-attributes)

属性表集合的限制稍微宽松一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写 入自己定义的属性信息，Java 虚拟机运行时会忽略掉它不认识的属性。

## 2. 类加载过程

![image-20250126192624604](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261926636.png)

### [2.1 加载](https://javaguide.cn/java/jvm/class-loading-process.html#加载)

由类加载器完成，具体是哪个类加载器加载由 **双亲委派模型** 决定。

1. 通过全类名获取定义此类的二进制字节流。
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构。
3. 在内存中生成一个代表该类的 `Class` 对象，作为方法区这些数据的访问入口。

### [2.2 验证](https://javaguide.cn/java/jvm/class-loading-process.html#验证)

确保 Class 文件的字节流中包含的信息符合《Java 虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。

### [2.3 准备](https://javaguide.cn/java/jvm/class-loading-process.html#准备)

**准备阶段是正式为类变量（静态变量）分配内存并设置类变量初始值的阶段**，这些内存都将在方法区中分配。

![image-20250126193544969](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261935010.png)

### [2.4 解析](https://javaguide.cn/java/jvm/class-loading-process.html#解析)

将常量池内的符号引用替换为直接引用的过程。

### [2.5 初始化](https://javaguide.cn/java/jvm/class-loading-process.html#初始化)

执行类的初始化代码，包括静态变量赋值（static int i= 3）和静态代码块。

- **触发条件**：
  - 创建类的实例（`new`）。
  - 访问类的静态变量或静态方法。
  - 使用反射（`Class.forName()`）。
  - 初始化子类时，父类会被初始化。
  - JVM 启动时指定的主类（包含 `main` 方法的类）。

### [2.6 类卸载](https://javaguide.cn/java/jvm/class-loading-process.html#类卸载)

该类的 Class 对象被 GC。

1. 该类的所有的实例对象都已被 GC，也就是说堆不存在该类的实例对象。
2. 该类没有在其他任何地方被引用
3. 该类的类加载器的实例已被 GC

## 3. 类加载器

### [3.1 类加载器介绍](https://javaguide.cn/java/jvm/classloader.html#类加载器介绍)

类加载器的主要作用就是加载 Java 类的字节码（ `.class` 文件）到 JVM 中，在内存中生成一个代表该类的 `Class` 对象。

- 类加载器是一个负责加载类的对象，用于实现类加载过程中的加载这一步。
- 每个 Java 类都有一个引用指向加载它的 `ClassLoader`。
- 数组类不是通过 `ClassLoader` 创建的（数组类没有对应的二进制字节流），是由 JVM 直接生成的。

### [3.2 类加载器加载规则](https://javaguide.cn/java/jvm/classloader.html#类加载器加载规则)

JVM 启动的时候，并不会一次性加载所有的类，而是根据需要去**动态加载**。也就是说，大部分类在具体用到的时候才会去加载，这样对内存更加友好。

对于已经加载的类会被放在 `ClassLoader` 中。在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。也就是说，对于一个类加载器来说，相同二进制名称的类只会被加载一次。

### [3.3 类加载器总结](https://javaguide.cn/java/jvm/classloader.html#类加载器总结)

JVM 中内置了三个重要的 `ClassLoader`：

1. **`BootstrapClassLoader`(启动类加载器)**：最顶层的加载类，由 C++实现，通常表示为 null，并且没有父级，主要用来加载 JDK 内部的核心类库（ `%JAVA_HOME%/lib`目录下的 `rt.jar`、`resources.jar`、`charsets.jar`等 jar 包和类）以及被 `-Xbootclasspath`参数指定的路径下的所有类。
2. **`ExtensionClassLoader`(扩展类加载器)**：主要负责加载 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类以及被 `java.ext.dirs` 系统变量所指定的路径下的所有类。
3. **`AppClassLoader`(应用程序类加载器)**：面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。

除了这三种类加载器之外，用户还可以加入自定义的类加载器来进行拓展，以满足自己的特殊需求。

![类加载器层次关系图](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261951002.png)

除了 `BootstrapClassLoader` 是 JVM 自身的一部分之外，其他所有的类加载器都是在 JVM 外部实现的，并且全都继承自 `ClassLoader`抽象类。这样做的好处是用户可以自定义类加载器，以便让应用程序自己决定如何去获取所需的类。

每个 `ClassLoader` 可以通过`getParent()`获取其父 `ClassLoader`，如果获取到 `ClassLoader` 为`null`的话，那么该类是通过 `BootstrapClassLoader` 加载的。

## [4. 双亲委派模型](https://javaguide.cn/java/jvm/classloader.html#双亲委派模型)

### [4.1 双亲委派模型介绍](https://javaguide.cn/java/jvm/classloader.html#双亲委派模型介绍)

类加载器有很多种，当我们想要加载一个类的时候，具体是哪个类加载器加载呢？这就需要提到双亲委派模型了。

- `ClassLoader` 类使用委托模型来搜索类和资源。
- 双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。
- `ClassLoader` 实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。

### [4.2 双亲委派模型的执行流程](https://javaguide.cn/java/jvm/classloader.html#双亲委派模型的执行流程)

**向上委派**：如果当前类已经被类加载器加载过（缓存中有），就直接返回；否则委派给父类加载器完成，直到最顶层的`BootstrapClassLoader` 中。

**向下加载**：不同的类加载器有不同的加载路径，如果最顶层的路径中没有，就让子加载器去加载，直到回到最初的加载器，如果此时还没有，就返回`ClassNotFoundException` 异常。

![类加载器层次关系图](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501261957261.png)

### [4.3 双亲委派模型的好处](https://javaguide.cn/java/jvm/classloader.html#双亲委派模型的好处)

**安全性**：如果没有双亲委派模型，用户可以自定义一个 `java.lang.String` 类，并替换 JVM 核心类库中的 `String` 类，导致安全问题。双亲委派模型确保核心类库（如 `java.lang.*`）由**启动类加载器加载**（`BootstrapClassLoader`），用户无法通过自定义类加载器替换核心类库。

**避免类的重复加载**：如果没有双亲委派模型，不同的类加载器可能会加载同一个类，导致类的重复加载和内存浪费。双亲委派模型通过委派机制，确保类只会被加载一次。父类加载器加载的类，子类加载器不会重复加载。

### 4.4 JVM判定两个 Java 类是否相同的具体规则

JVM 不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。

