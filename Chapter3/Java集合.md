# 集合概述

## 1. 集合框架图

![image-20250124115401899](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241154382.png)

只有Vector和HashTable和Stack是线程安全的，其他的都是线程不安全。

## [2. 集合框架底层数据结构总结](https://javaguide.cn/java/collection/java-collection-questions-01.html#集合框架底层数据结构总结)

### [2.1 List](https://javaguide.cn/java/collection/java-collection-questions-01.html#list)

- `ArrayList`：`Object[]` 数组。详细可以查看：[ArrayList 源码分析]()。
- `Vector`：`Object[]` 数组。
- `LinkedList`：双向链表(JDK1.6 之前为循环链表，JDK1.7 取消了循环)。详细可以查看：[LinkedList 源码分析]()。

### [2.2 Set](#set)

- `HashSet`(无序，唯一): 基于 `HashMap` 实现的，底层采用 `HashMap` 来保存元素，基于 `HashMap` 实现。它将元素存储在 `HashMap` 的**键**中，而 `HashMap` 的值则统一使用一个**静态的 `PRESENT` 对象**。
- `LinkedHashSet`: `LinkedHashSet` 是 `HashSet` 的子类，并且其内部是通过 `LinkedHashMap` 来实现的，它在 `HashMap` 的基础上维护了一个双向链表，用于记录元素的插入顺序。
- `TreeSet`(有序，唯一): 红黑树(自平衡的排序二叉树)。

### [2.3 Queue](#queue)

- `PriorityQueue`: `Object[]` 数组来实现小顶堆。详细可以查看：[PriorityQueue 源码分析]()。
- `DelayQueue`:`PriorityQueue`。详细可以查看：[DelayQueue 源码分析]()。
- `ArrayDeque`: 可扩容动态双向数组。

### [2.4 Map](#map)

- `HashMap`：JDK1.8 之前 `HashMap` 由**数组+链表**组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，**当链表长度大于阈值**（默认为 8）（将链表转换成红黑树前会判断，**如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树**）时，将链表转化为红黑树，以减少搜索时间。详细可以查看：[HashMap 源码分析]()。
- `LinkedHashMap`：`LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即**由数组和链表**或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条**双向链表**，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。详细可以查看：[LinkedHashMap 源码分析]()
- `Hashtable`：**数组+链表组成**的，数组是 `Hashtable` 的主体，链表则是主要为了解决哈希冲突而存在的。
- `TreeMap`：**红黑树**（自平衡的排序二叉树）。

# [List](https://javaguide.cn/java/collection/java-collection-questions-01.html#list-1)

## [1. ArrayList 和 Array（数组）的区别？](https://javaguide.cn/java/collection/java-collection-questions-01.html#arraylist-和-array-数组-的区别)

`ArrayList` 内部基于动态数组实现，比 `Array`（静态数组） 使用起来更加灵活：

`ArrayList`会根据实际存储的元素动态地扩容或缩容，而 `Array` 被创建之后就不能改变它的长度了。

`ArrayList` 允许你使用泛型来确保类型安全，`Array` 则不可以。

`ArrayList` 中只能存储对象。对于基本类型数据，需要使用其对应的包装类（如 Integer、Double 等）。`Array` 可以直接存储基本类型数据，也可以存储对象。

`ArrayList` 支持插入、删除、遍历等常见操作，并且提供了丰富的 API 操作方法，比如 `add()`、`remove()`等。`Array` 只是一个固定长度的数组，只能按照下标访问其中的元素，不具备动态添加、删除元素的能力。

`ArrayList`创建时不需要指定大小，而`Array`创建时必须指定大小。

## [2. ArrayList 和 Vector 的区别?（了解即可）](#arraylist-和-vector-的区别-了解即可)

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[]`存储，适用于频繁的查找工作，线程不安全 。
- `Vector` 是 `List` 的古老实现类，底层使用`Object[]` 存储，线程安全。

## [3. Vector 和 Stack 的区别?（了解即可）](https://javaguide.cn/java/collection/java-collection-questions-01.html#vector-和-stack-的区别-了解即可)

- `Vector` 和 `Stack` 两者都是线程安全的，都是使用 `synchronized` 关键字进行同步处理。
- `Stack` 继承自 `Vector`，是一个后进先出的栈，而 `Vector` 是一个列表。

随着 Java 并发编程的发展，`Vector` 和 `Stack` 已经被淘汰，推荐使用并发集合类（例如 `ConcurrentHashMap`、`CopyOnWriteArrayList` 等）或者手动实现线程安全的方法来提供安全的多线程操作支持。

## [4. ArrayList 可以添加 null 值吗？](https://javaguide.cn/java/collection/java-collection-questions-01.html#arraylist-可以添加-null-值吗)

`ArrayList` 中可以存储任何类型的对象，包括 `null` 值。不过，不建议向`ArrayList` 中添加 `null` 值， `null` 值无意义，会让代码难以维护比如忘记做判空处理就会导致空指针异常。

## [5. ArrayList 插入和删除元素的时间复杂度？](https://javaguide.cn/java/collection/java-collection-questions-01.html#arraylist-插入和删除元素的时间复杂度)

对于插入：

- 头部插入：由于需要将所有元素都依次向后移动一个位置，因此时间复杂度是 O(n)。
- **尾部插入**：当 `ArrayList` 的容量未达到极限时，往列表末尾插入元素的时间复杂度是 O(1)，因为它只需要在数组末尾添加一个元素即可；**当容量已达到极限并且需要扩容时，则需要执行一次 O(n) 的操作将原数组复制到新的更大的数组中，然后再执行 O(1) 的操作添加元素。**
- 指定位置插入：需要将目标位置之后的所有元素都向后移动一个位置，然后再把新元素放入指定位置。这个过程需要移动平均 n/2 个元素，因此时间复杂度为 O(n)。

对于删除：

- 头部删除：由于需要将所有元素依次向前移动一个位置，因此时间复杂度是 O(n)。
- 尾部删除：当删除的元素位于列表末尾时，时间复杂度为 O(1)。
- 指定位置删除：需要将目标元素之后的所有元素向前移动一个位置以填补被删除的空白位置，因此需要移动平均 n/2 个元素，时间复杂度为 O(n)。

## [6. LinkedList 插入和删除元素的时间复杂度？](https://javaguide.cn/java/collection/java-collection-questions-01.html#linkedlist-插入和删除元素的时间复杂度)

- 头部插入/删除：只需要修改头结点的指针即可完成插入/删除操作，因此时间复杂度为 O(1)。
- 尾部插入/删除：只需要修改尾结点的指针即可完成插入/删除操作，因此时间复杂度为 O(1)。
- 指定位置插入/删除：需要先移动到指定位置，再修改指定节点的指针完成插入/删除，不过由于有头尾指针，可以从较近的指针出发，因此需要遍历平均 n/4 个元素，时间复杂度为 O(n)。

## [7. LinkedList 为什么不能实现 RandomAccess 接口？](https://javaguide.cn/java/collection/java-collection-questions-01.html#linkedlist-为什么不能实现-randomaccess-接口)

由于 `LinkedList` 底层数据结构是链表，内存地址不连续，只能通过指针来定位，不支持随机快速访问，所以不能实现 `RandomAccess` 接口。

注意：`RandomAccess` 接口只是标识（没定义功能），并不是说 `ArrayList` 实现 `RandomAccess` 接口才具有快速随机访问功能的！

## [8. ArrayList 与 LinkedList 区别?](https://javaguide.cn/java/collection/java-collection-questions-01.html#arraylist-与-linkedlist-区别)

- **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；

- **底层数据结构：**`ArrayList` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！）

- **插入和删除是否受元素位置的影响：**

- `ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`），时间复杂度就为 O(n)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。

  `LinkedList` 采用链表存储，所以在头尾插入或者删除元素不受元素位置的影响（`add(E e)`、`addFirst(E e)`、`addLast(E e)`、`removeFirst()`、 `removeLast()`），时间复杂度为 O(1)，如果是要在指定位置 `i` 插入和删除元素的话（`add(int index, E element)`，`remove(Object o)`,`remove(int index)`）， 时间复杂度为 O(n) ，因为需要先移动到指定位置再插入和删除。

- **是否支持快速随机访问：**`LinkedList` 不支持高效的随机元素访问，而 `ArrayList`（实现了 `RandomAccess` 接口） 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。

- **内存空间占用：**`ArrayList` 的空间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。

  

我们在项目中一般是不会使用到 `LinkedList` 的，需要用到 `LinkedList` 的场景几乎都可以使用 `ArrayList` 来代替，并且，性能通常会更好！就连 `LinkedList` 的作者约书亚 · 布洛克（Josh Bloch）自己都说从来不会使用 `LinkedList` 。

## [9. 说一说 ArrayList 的扩容机制吧](https://javaguide.cn/java/collection/java-collection-questions-01.html#说一说-arraylist-的扩容机制吧)

ArrayList在添加元素时，如果当前元素个数已经达到了内部数组的容量上限，就会触发扩容操作。

- 计算新的容量：一般情况下，新的容量会扩大为原容量的1.5倍（在JDK 10之后，扩容策略做了调整），然后检查是否超过了最大容量限制。
- 创建新的数组：根据计算得到的新容量，创建一个新的更大的数组。
- 将元素复制：将原来数组中的元素逐个复制到新数组中。
- 更新引用：将ArrayList内部指向原数组的引用指向新数组。
- 完成扩容：扩容完成后，可以继续添加新元素。

ArrayList的扩容操作涉及到数组的复制和内存的重新分配，所以在频繁添加大量元素时，扩容操作可能会影响性能。为了减少扩容带来的性能损耗，可以在初始化ArrayList时预分配足够大的容量，避免频繁触发扩容操作。

之所以扩容是 1.5 倍，是因为 1.5 可以充分利用移位操作，减少浮点数或者运算时间和运算次数。

**`int newCapacity = oldCapacity + (oldCapacity >> 1)`,所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）！** 奇偶不同，比如：10+10/2 = 15, 33+33/2=49。如果是奇数的话会丢掉小数.





**以无参数构造方法创建 `ArrayList` 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 10。** 下面在我们分析 `ArrayList` 扩容时会讲到这一点内容！

## 10. 为什么ArrayList不是线程安全的，具体来说是哪里不安全？

在高并发添加数据下，ArrayList会暴露三个问题;

- 部分值为null：当线程1走到了扩容那里发现当前size是9，而数组容量是10，所以不用扩容，这时候cpu让出执行权，线程2也进来了，发现size是9，而数组容量是10，所以不用扩容，这时候线程1继续执行，将数组下标索引为9的位置set值了，还没有来得及执行size++，这时候线程2也来执行了，又把数组下标索引为9的位置set了一遍，这时候两个先后进行size++，导致下标索引10的地方就为null了。
- 索引越界异常：线程1走到扩容那里发现当前size是9，数组容量是10不用扩容，cpu让出执行权，线程2也发现不用扩容，这时候数组的容量就是10，而线程1 set完之后size++，这时候线程2再进来size就是10，数组的大小只有10，而你要设置下标索引为10的就会越界（数组的下标索引从0开始）；
- size与我们add的数量不符：这个基本上每次都会发生，**这个理解起来也很简单，因为size++本身就不是原子操作**，可以分为三步：获取size的值，将size的值加1，将新的size值覆盖掉原来的，线程1和线程2拿到一样的size值加完了同时覆盖，就会导致一次没有加上，所以肯定不会与我们add的数量保持一致的；

# Set

## [1. 无序性和不可重复性的含义是什么](https://javaguide.cn/java/collection/java-collection-questions-01.html#无序性和不可重复性的含义是什么)

- 无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的。
- 不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，需要同时重写 `equals()` 方法和 `hashCode()` 方法。

## [2. 比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同](https://javaguide.cn/java/collection/java-collection-questions-01.html#比较-hashset、linkedhashset-和-treeset-三者的异同)

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都不是线程安全的。
- `HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同。`HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）。`LinkedHashSet` 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。`TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。
- 底层数据结构不同又导致这三者的应用场景不同。`HashSet` 用于不需要保证元素插入和取出顺序的场景，`LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景，`TreeSet` 用于支持对元素自定义排序规则的场景。
- `HashSet`、`LinkedHashSet` 和 `TreeSet` 插入删除复杂度分别为 O (1)、 O (1)、 O (log n)

## [3.HashSet 如何检查重复?](https://javaguide.cn/java/collection/java-collection-questions-02.html#hashset-如何检查重复)

当你把对象加入`HashSet`时，`HashSet` 会先计算对象的`hashcode`值来判断对象加入的位置，同时也会与其他加入的对象的 `hashcode` 值作比较，如果没有相符的 `hashcode`，`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashcode` 值的对象，这时会调用`equals()`方法来检查 `hashcode` 相等的对象是否真的相同。如果两者相同，`HashSet` 就不会让加入操作成功。



也就是说，在 JDK1.8 中，实际上无论`HashSet`中是否已经存在了某元素，`HashSet`都会直接插入，只是会在`add()`方法的返回值处告诉我们插入前是否存在相同元素。

# [Queue](https://javaguide.cn/java/collection/java-collection-questions-01.html#queue-1)

## [1. Queue 与 Deque 的区别](https://javaguide.cn/java/collection/java-collection-questions-01.html#queue-与-deque-的区别)

## [2. ArrayDeque 与 LinkedList 的区别](https://javaguide.cn/java/collection/java-collection-questions-01.html#arraydeque-与-linkedlist-的区别)

`ArrayDeque` 和 `LinkedList` 都实现了 `Deque` 接口，两者都具有队列的功能，但两者有什么区别呢？

- `ArrayDeque` 是基于**可变长的数组和双指针**来实现，而 `LinkedList` 则通过链表来实现。
- `ArrayDeque` 不支持存储 `NULL` 数据，但 `LinkedList` 支持。
- `ArrayDeque` 是在 JDK1.6 才被引入的，而`LinkedList` 早在 JDK1.2 时就已经存在。
- `ArrayDeque` 插入时可能存在扩容过程, 不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。

从性能的角度上，选用 `ArrayDeque` 来实现队列要比 `LinkedList` 更好。此外，`ArrayDeque` 也可以用于实现栈。

## [3. 说一说 PriorityQueue](https://javaguide.cn/java/collection/java-collection-questions-01.html#说一说-priorityqueue)

`PriorityQueue` 利用了**二叉堆**的数据结构来实现的，底层使用**可变长的数组**来存储数据

`PriorityQueue` 通过堆元素的上浮和下沉，实现了在 O(logn) 的时间复杂度内插入元素和删除堆顶元素。

`PriorityQueue` 是非线程安全的，且不支持存储 `NULL` 和 `non-comparable` 的对象。

`PriorityQueue` 默认是小顶堆，但可以接收一个 `Comparator` 作为构造参数，从而来自定义元素优先级的先后。

## [4. 什么是 BlockingQueue？](https://javaguide.cn/java/collection/java-collection-questions-01.html#什么是-blockingqueue)

`BlockingQueue` （阻塞队列）是一个接口，继承自 `Queue`。`BlockingQueue`阻塞的原因是其支持当队列没有元素时一直阻塞，直到有元素；还支持如果队列已满，一直等到队列可以放入新元素时再放入。

![image-20250124125026010](/Users/wanghaoran/Library/Application Support/typora-user-images/image-20250124125026010.png)

# Map

## [1. HashMap 和 Hashtable 的区别](https://javaguide.cn/java/collection/java-collection-questions-02.html#hashmap-和-hashtable-的区别)

- **线程是否安全：** `HashMap` 是非线程安全的，`Hashtable` 是线程安全的,因为 `Hashtable` 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）；
- **效率：** 因为线程安全的问题，`HashMap` 要比 `Hashtable` 效率高一点。另外，`Hashtable` 基本被淘汰，不要在代码中使用它；
- **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
- **初始容量大小和每次扩充容量大小的不同：** ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` **默认的初始化大小为 16**。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 `Hashtable` 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 **2 的幂次方大小**（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
- **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间（后文中我会结合源码对这一过程进行分析）。`Hashtable` 没有这样的机制。
- **哈希函数的实现**：`HashMap` 对哈希值进行了高位和低位的混合扰动处理以减少冲突，而 `Hashtable` 直接使用键的 `hashCode()` 值。

## [2. HashMap 和 HashSet 区别](https://javaguide.cn/java/collection/java-collection-questions-02.html#hashmap-和-hashset-区别)

`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

## [3. HashMap 和 TreeMap 区别](https://javaguide.cn/java/collection/java-collection-questions-02.html#hashmap-和-treemap-区别)

哈希表、红黑树

无序、有序

O(1)、O(logn)

## [4. HashMap 的底层实现](https://javaguide.cn/java/collection/java-collection-questions-02.html#hashmap-的底层实现)

HashMap 通过 key 的 `hashcode` 经过**扰动函数**处理过后得到 hash 值，然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过**拉链法**解决冲突。

`HashMap` 中的扰动函数（`hash` 方法）是用来优化哈希值的分布。通过对原始的 `hashCode()` 进行额外处理，扰动函数可以减小由于糟糕的 `hashCode()` 实现导致的碰撞，从而提高数据的分布均匀性。

JDK1.8之前：

![image-20250124135357482](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241353522.png)

JDK1.8之后

![image-20250124135410743](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241354774.png)

## [5. HashMap 的长度为什么是 2 的幂次方](https://javaguide.cn/java/collection/java-collection-questions-02.html#hashmap-的长度为什么是-2-的幂次方)

得到hash值之后，用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。

1. 位运算效率更高：位运算(&)比取余运算(%)更高效。当长度为 2 的幂次方时，`hash % length` 等价于 `hash & (length - 1)`。
2. 可以更好地保证哈希值的均匀分布：扩容之后，在旧数组元素 hash 值比较均匀的情况下，新数组元素也会被分配的比较均匀，最好的情况是会有一半在新数组的前半部分，一半在新数组后半部分。
3. 扩容机制变得简单和高效：扩容后只需检查哈希值高位的变化来决定元素的新位置，要**么位置不变（高位为 0），要么就是移动到新位置（高位为 1，原索引位置+原容量）。**

~~~txt
假设有一个元素的哈希值为 10101100

旧数组元素位置计算：
hash        = 10101100
length - 1  = 00000111
& -----------------
index       = 00000100  (4)

新数组元素位置计算：
hash        = 10101100
length - 1  = 00001111
& -----------------
index       = 00001100  (12)

看第四位（从右数）：
1.高位为 0：位置不变。
2.高位为 1：移动到新位置（原索引位置+原容量）。
~~~

## [6. 列举HashMap在多线程下可能会出现的问题？](https://javaguide.cn/java/collection/java-collection-questions-02.html#hashmap-多线程操作导致死循环问题)

- JDK1.7中的 HashMap 使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环。因此，JDK1.8使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。
- 多线程同时执行 put 操作，如果计算出来的索引位置是相同的（比如发生了哈希冲突），那会造成前一个 key 被后一个 key 覆盖，从而导致元素的丢失。此问题在JDK 1.7和 JDK 1.8 中都存在

## [7. ConcurrentHashMap 和 Hashtable 的区别](https://javaguide.cn/java/collection/java-collection-questions-02.html#concurrenthashmap-和-hashtable-的区别)

HashTable底层是数组+链表，它对整个 `Hashtable` 对象进行加锁，效率十分低。

ConcurrentHashMap，在 JDK1.7 的时候， **采用分段的数组+链表**，但是每一把锁只锁容器其中一部分数据（下面有示意图），多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。

![image-20250124141513032](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241415068.png)

到了 JDK1.8 的时候，直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作，像是优化过且线程安全的 `HashMap`。



![image-20250124141649277](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241416311.png)

## [8. ConcurrentHashMap 线程安全的具体实现方式/底层具体实现](https://javaguide.cn/java/collection/java-collection-questions-02.html#concurrenthashmap-线程安全的具体实现方式-底层具体实现)

[JDK1.8 之前](https://javaguide.cn/java/collection/java-collection-questions-02.html#jdk1-8-之前-1)

一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组，`Segment` 的个数一旦**初始化就不能改变**。 `Segment` 数组的大小默认是 16，也就是说默认可以同时支持 **16 个**线程并发写。

也就是说，对同一 `Segment` 的并发写入会被阻塞，不同 `Segment` 的写入是可以并发执行的。

[JDK1.8 之后](https://javaguide.cn/java/collection/java-collection-questions-02.html#jdk1-8-之后-1)

Java 8 中，**锁粒度更细**，`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，就不会影响其他 Node 的读写，效率大幅提升。

## [9. ConcurrentHashMap 能保证复合操作的原子性吗？](https://javaguide.cn/java/collection/java-collection-questions-02.html#concurrenthashmap-能保证复合操作的原子性吗)

如果线程 A 和 B 的执行顺序是这样：

1. 线程 A 判断 map 中不存在 key
2. 线程 B 判断 map 中不存在 key
3. 线程 B 将 (key, anotherValue) 插入 map
4. 线程 A 将 (key, value) 插入 map

那么最终的结果是 (key, value)，而不是预期的 (key, anotherValue)。这就是复合操作的非原子性导致的问题。

提供了一些原子性的**复合操作**，如 `putIfAbsent`、`compute`、`computeIfAbsent` 、`computeIfPresent`、`merge`等。这些方法都可以接受一个函数作为参数，根据给定的 key 和 value 来计算一个新的 value，并且将其更新到 map 中。

~~~java
// 线程 A
map.putIfAbsent(key, value);
// 线程 B
map.putIfAbsent(key, anotherValue);
~~~

## 10. ConcurrentHashMap已经用了synchronized，为什么还要用CAS（乐观锁）呢？

ConcurrentHashMap使用这两种手段来保证线程安全主要是一种权衡的考虑，在某些操作中使用synchronized，还是使用CAS，主要是根据锁竞争程度来判断的。

比如：在putVal中，如果计算出来的hash槽没有存放元素，那么就可以直接使用CAS来进行设置值，这是因为在设置元素的时候，因为hash值经过了各种扰动后，造成hash碰撞的几率较低，那么我们可以预测使用较少的自旋来完成具体的hash落槽操作。

当发生了hash碰撞的时候说明容量不够用了或者已经有大量线程访问了，因此这时候使用synchronized来处理hash碰撞比CAS效率要高，因为发生了hash碰撞大概率来说是线程竞争比较强烈。

## 11. 为什么HashMap要用红黑树而不是平衡二叉树？

与平衡树不同的是，红黑树在插入、删除等操作，**不会像平衡树那样，频繁着破坏红黑树的规则，所以不需要频繁着调整**，这也是我们为什么大多数情况下使用红黑树的原因。

##  12. HashMap的扩容机制介绍一下

hashMap默认的负载因子是0.75，即如果hashmap中的元素个数超过了总容量75%，则会触发扩容，扩容分为两个步骤：

- **第1步**是对哈希表长度的扩展（2倍）
- **第2步**是将旧哈希表中的数据放到新的哈希表中。

## 13. 往hashmap存20个元素，会扩容几次？

当插入 20 个元素时，HashMap 的扩容过程如下：

**初始容量**：16

- 插入第 1 到第 12 个元素时，不需要扩容。
- 插入第 13 个元素时，达到负载因子限制，需要扩容。此时，HashMap 的容量从 16 扩容到 32。

**扩容后的容量**：32

- 插入第 14 到第 24 个元素时，不需要扩容。

因此，总共会进行一次扩容。