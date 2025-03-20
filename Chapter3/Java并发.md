# 线程

## [1. 什么是线程和进程?](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#⭐️什么是线程和进程)

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

线程与进程相似，但线程是一个比进程更小的执行单位，线程是任务调度的基本单位。具体的见操作系统。

**一个 Java 程序的运行是 main 线程和多个其他线程同时运行**。

## [2. Java 线程和操作系统的线程有啥区别？](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#java-线程和操作系统的线程有啥区别)

**现在的 Java 线程的本质其实就是操作系统的线程**，一个 Java 线程对应一个**系统内核线程**。

## [3. 请简要描述线程与进程的关系,区别及优缺点？](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#⭐️请简要描述线程与进程的关系-区别及优缺点)

见操作系统和Javaguide。

[程序计数器为什么是私有的?](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#程序计数器为什么是私有的)

为了**线程切换后能恢复到正确的执行位置**。

[虚拟机栈和本地方法栈为什么是私有的?](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#虚拟机栈和本地方法栈为什么是私有的)

为了**保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的

![image-20250124162350257](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241623306.png)

堆和方法区是所有线程共享的资源。

其中堆是进程中最大的一块内存，主要用于存放**新创建的对象** (几乎所有对象都在这里分配内存)。

方法区主要用于存放已被加载的**类信息、常量、静态变量**、即时编译器编译后的代码等数据。

## [4. 如何创建线程？](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#如何创建线程)

`Java`创建线程有很多种方式啊，像实现`Runnable、Callable`接口、继承`Thread`类、创建线程池等等，不过这些方式并没有真正创建出线程，严格来说，`Java`就只有一种方式可以创建线程，那就是通过`new Thread().start()`创建。
而所谓的`Runnable、Callable……`对象，这仅仅只是线程体，也就是提供给线程执行的任务，并不属于真正的`Java`线程，它们的执行，最终还是需要依赖于`new Thread()`……



继承`Thread`类，重写`run`方法：

~~~java
public class ExtendsThread extends Thread {
    @Override
    public void run() {
        System.out.println("1......");
    }

    public static void main(String[] args) {
        new ExtendsThread().start();
    }
}
~~~

实现`Runnable`接口并重写`run`方法

~~~java
public class ImplementsRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("2......");
    }

    public static void main(String[] args) {
        ImplementsRunnable runnable = new ImplementsRunnable();
        new Thread(runnable).start();
    }
}
~~~

实现`Callable`接口并重写`call`方法，如下：

~~~java
public class ImplementsCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("3......");
        return "zhuZi";
    }

    public static void main(String[] args) throws Exception {
        ImplementsCallable callable = new ImplementsCallable();
        FutureTask<String> futureTask = new FutureTask<>(callable);
        new Thread(futureTask).start();
        System.out.println(futureTask.get());
    }
}
~~~

可以通过`Executors`创建线程池

~~~java
public class UseExecutorService {
    public static void main(String[] args) {
        ExecutorService poolA = Executors.newFixedThreadPool(2);
        poolA.execute(()->{
            System.out.println("4A......");
        });
        poolA.shutdown();

        // 又或者自定义线程池
        ThreadPoolExecutor poolB = new ThreadPoolExecutor(2, 3, 0,
                TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(3),
                Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
        poolB.submit(()->{
            System.out.println("4B......");
        });
        poolB.shutdown();
    }
}
~~~

## [5. 说说线程的生命周期和状态?](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#⭐️说说线程的生命周期和状态)

Java 线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态：

- NEW: 初始状态，线程被创建出来但没有被调用 `start()` 。
- RUNNABLE: 运行状态，线程被调用了 `start()`等待运行的状态。
- BLOCKED：阻塞状态，需要等待锁释放。
- WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）。
- TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。
- TERMINATED：终止状态，表示该线程已经运行完毕。

在操作系统层面，线程有 READY 和 RUNNING 状态；而在 JVM 层面，只能看到 RUNNABLE 状态

![image-20250124160612524](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241606564.png)

## [6. 什么是线程上下文切换?](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#什么是线程上下文切换)

线程切换意味着需要保存当前线程的上下文，留待线程下次占用 CPU 的时候恢复现场。

- 主动让出 CPU，比如调用了 `sleep()`, `wait()` 等。
- 时间片用完，因为操作系统要防止一个线程或者进程长时间占用 CPU 导致其他线程或者进程饿死。
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。

## [7. Thread#sleep() 方法和 Object#wait() 方法对比](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#thread-sleep-方法和-object-wait-方法对比)

**共同点**：两者都可以暂停线程的执行。

**区别**：

- **`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- `wait()` 通常被用于线程间交互/通信，`sleep()`通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。`sleep()`方法执行完成后，线程会自动苏醒，或者也可以使用 `wait(long timeout)` 超时后线程会自动苏醒。
- `sleep()` 是 `Thread` 类的静态本地方法，`wait()` 则是 `Object` 类的本地方法。为什么这样设计呢？下一个问题就会聊到。

为什么 wait() 方法不定义在 Thread 中？

每个对象（`Object`）都拥有对象锁，既然要释放当前线程占有的对象锁并让其进入 WAITING 状态，自然是要操作对应的对象（`Object`）而非当前的线程（`Thread`）。

为什么 `sleep()` 方法定义在 `Thread` 中？

因为 `sleep()` 是让当前线程暂停执行，不涉及到对象类，也不需要获得对象锁。

## [8. 可以直接调用 Thread 类的 run 方法吗？](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#可以直接调用-thread-类的-run-方法吗)

调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话会当成一个 main 线程下的普通方法去执行，不会以多线程的方式执行。

# 多线程

## [1. 并发与并行的区别](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#并发与并行的区别)

- **并发**：两个及两个以上的作业在同一 **时间段** 内执行。
- **并行**：两个及两个以上的作业在同一 **时刻** 执行。

## [2. 同步和异步的区别](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#同步和异步的区别)

- **同步**：发出一个调用之后，在没有得到结果之前， 该调用就不可以返回，一直等待。
- **异步**：调用在发出之后，不用等待返回结果，该调用直接返回。

## [3. 为什么要使用多线程?](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#⭐️为什么要使用多线程)

- **从计算机底层来说：** 线程可以比作是轻量级的进程，是程序执行的最小单位,线程间的切换和调度的成本远远小于进程。另外，多核 CPU 时代意味着多个线程可以同时运行，这减少了线程上下文切换的开销。
- **从当代互联网发展趋势来说：** 现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

## [4. 单核 CPU 支持 Java 多线程吗？](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#⭐️单核-cpu-支持-java-多线程吗)

单核 CPU 是支持 Java 多线程的。操作系统通过时间片轮转的方式，将 CPU 的时间分配给不同的线程。尽管单核 CPU 一次只能执行一个任务，但通过快速在多个线程之间切换，可以让用户感觉多个任务是同时进行的。

## [5. 单核 CPU 上运行多个线程效率一定会高吗？](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#⭐️单核-cpu-上运行多个线程效率一定会高吗)

对于单核 CPU 来说，如果任务是 CPU 密集型的，那么开很多线程会频繁的线程切换，影响效率；如果任务是 IO 密集型的，那么开很多线程会提高效率。

## [6. 使用多线程可能带来什么问题?](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#使用多线程可能带来什么问题)

内存泄漏、死锁、线程不安全（数据的正确性和一致性）等等。

# JMM

https://javaguide.cn/java/concurrent/jmm.html

## 1. 基础知识

- CPU 可以通过制定缓存一致协议（比如 [MESI 协议](https://zh.wikipedia.org/wiki/MESI协议)）来解决内存缓存不一致性问题。
- 为了提升执行速度/性能，计算机在执行程序代码的时候，会对指令进行重排序。 简单来说就是系统在执行代码的时候并不一定是按照你写的代码的顺序依次执行。**指令重排序可以保证串行语义一致，但是没有义务保证多线程间的语义也一致** ，所以在多线程下，指令重排序可能会导致一些问题。
- 你可以把 JMM 看作是 J**ava 定义的并发编程相关的一组规范**，除了抽象了线程和主内存之间的关系之外，其还规定了从 Java 源代码到 CPU 可执行指令的这个转化过程要遵守哪些和并发相关的原则和规范，其主要目的是为了简化多线程编程，增强程序可移植性的。

## 2. Java内存模型

![image-20250124172230664](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241722695.png)

从上图来看，线程 1 与线程 2 之间如果要进行通信的话，必须要经历下面 2 个步骤：

1. 线程 1 把本地内存中修改过的共享变量副本的值同步到主内存中去。
2. 线程 2 到主存中读取对应的共享变量的值。

也就是说，**JMM 为共享变量提供了可见性的保障。**

不过，多线程下，对主内存中的一个共享变量进行操作有可能诱发线程安全问题。举个例子：

1. 线程 1 和线程 2 分别对同一个共享变量进行操作，一个执行修改，一个执行读取。
2. 线程 2 读取到的是线程 1 修改之前的值还是修改后的值并不确定，都有可能，因为线程 1 和线程 2 都是先将共享变量从主内存拷贝到对应线程的工作内存中。

## [3. 并发编程三个重要特性](https://javaguide.cn/java/concurrent/jmm.html#再看并发编程三个重要特性)

### [3.1 原子性](https://javaguide.cn/java/concurrent/jmm.html#原子性)

一次操作或者多次操作，要么所有的操作全部都得到执行并且不会受到任何因素的干扰而中断，要么都不执行。

在 Java 中，可以借助**`synchronized`**、各种 `Lock` 以及各种原子类实现原子性。

`synchronized` 和各种 `Lock` 可以**指令重排序可以保证串行语义一致，但是没有义务保证多线程间的语义也一致** ，所以在多线程下，指令重排序可能会导致一些问题。

**各种原子类是利用 CAS (compare and swap) 操作**（可能也会用到 `volatile`或者`final`关键字）来保证原子操作。

### [3.2 可见性](https://javaguide.cn/java/concurrent/jmm.html#可见性)

**当一个线程对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。**

在 Java 中，可以借助`synchronized`、`volatile` 以及各种 `Lock` 实现可见性。

**锁确保线程在释放锁时，会将本地内存中的共享变量刷新到主内存；**

**在获取锁时，会从主内存中读取共享变量的最新值。**

如果我们将变量声明为 `volatile` ，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

### [3.3 有序性](https://javaguide.cn/java/concurrent/jmm.html#有序性)

由于指令重排序问题，代码的执行顺序未必就是编写代码时候的顺序。

我们上面讲重排序的时候也提到过：

> **指令重排序可以保证串行语义一致，但是没有义务保证多线程间的语义也一致** ，所以在多线程下，指令重排序可能会导致一些问题。

在 Java 中，`volatile` 关键字可以禁止指令进行重排序优化。

## [4. volatile 关键字](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#⭐️volatile-关键字)

1. 保证变量可见性
2. 禁止指令重排序

在 Java 内存模型（JMM）中，每个线程都有自己的工作内存，线程对变量的操作都是在工作内存中进行的，而变量的实际存储位置是主内存。当一个线程修改了主内存中变量的值后，其他线程的工作内存中该变量的副本可能还是旧值，这就会导致数据不一致的问题。

当一个变量被声明为 `volatile` 时，线程对该变量的写操作会立即刷新到主内存中，而读操作会直接从主内存中读取。这样一来，一个线程对 `volatile` 变量的修改能够及时被其他线程看到。



在 Java 中，为了提高程序的执行效率，编译器和处理器会对指令进行重排序。**指令重排序可以保证串行语义一致，但是没有义务保证多线程间的语义也一致** ，所以在多线程下，指令重排序可能会导致一些问题。`volatile` 关键字可以**禁止指令重排序**。

![image-20250124172534826](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241725857.png)

# 锁

## [1. 什么是悲观锁？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#什么是悲观锁)

认为共享资源每次被访问的时候就会出现问题，因此**共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程**。

像 Java 中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

~~~java
public void performSynchronisedTask() {
    synchronized (this) {
        // 需要同步的操作
    }
}

private Lock lock = new ReentrantLock();
lock.lock();
try {
   // 需要同步的操作
} finally {
    lock.unlock();
}
~~~

**高并发的场景下，激烈的锁竞争会造成线程阻塞，大量阻塞线程会导致系统的上下文切换，增加系统的性能开销**。并且，悲观锁还可能会存在死锁问题，影响代码的正常运行。

## [2. 什么是乐观锁？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#什么是乐观锁)

乐观锁总是假设最好的情况，认为共享资源每次被访问的时候不会出现问题，线程可以不停地执行，无需加锁也无需等待，只是在提交修改的时候去验证对应的资源（也就是数据）是否被其它线程修改了（具体方法可以使用**版本号机制**或 CAS 算法）。

在 Java 中`java.util.concurrent.atomic`包下面的**原子变量类**（比如`AtomicInteger`、`LongAdder`）就是使用了乐观锁的一种实现方式 **CAS** 实现的。

但是，如果**冲突频繁发生（写占比非常多的情况），会频繁失败和重试，这样同样会非常影响性能，导致 CPU 飙升。**

## 3. 如何选择乐观锁和悲观锁？

- 悲观锁通常多用于**写比较多的情况**（多写场景，竞争激烈），这样可以避免频繁失败和重试影响性能，悲观锁的开销是固定的。不过，如果乐观锁解决了频繁失败和重试这个问题的话（比如`LongAdder`），也是可以考虑使用乐观锁的，要视实际情况而定。
- 乐观锁通常多用于**写比较少的情况**（多读场景，竞争较少），这样可以避免频繁加锁影响性能。不过，乐观锁主要针对的对象是单个共享变量（参考`java.util.concurrent.atomic`包下面的原子变量类）。

## [4. 如何实现乐观锁？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#如何实现乐观锁)

**版本号机制**：一般是在数据表中加上一个数据版本号 `version` 字段，表示数据被修改的次数。当数据被修改时，`version` 值会加一。当线程 A 要更新数据值时，在读取数据的同时也会读取 `version` 值，在提交更新时，若刚才读取到的 version 值为当前数据库中的 `version` 值相等时才更新，否则重试更新操作，直到更新成功。

CAS：**Compare And Swap（比较与交换**），就是用一个预期值和要更新的变量值进行比较，两值相等才会进行更新

**CAS 是一个原子操作，底层依赖于一条 CPU 的原子指令。**

CAS 涉及到三个操作数：

- **V**：要更新的变量值(Var)
- **E**：预期值(Expected)
- **N**：拟写入的新值(New)

当且仅当 V 的值等于 E 时，CAS 通过原子方式用新值 N 来更新 V 的值。如果不等，说明已经有其它线程更新了 V，则当前线程放弃更新。

**举一个简单的例子**：线程 A 要修改变量 i 的值为 6，i 原值为 1（V = 1，E=1，N=6，假设不存在 ABA 问题）。

1. i 与 1 进行比较，如果相等， 则说明没被其他线程修改，可以被设置为 6 。
2. i 与 1 进行比较，如果不相等，则说明被其他线程修改，当前线程放弃更新，CAS 操作失败。

当多个线程同时使用 CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。

## [5. Java 中 CAS 是如何实现的？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#java-中-cas-是如何实现的)

在 Java 中，实现 CAS（Compare-And-Swap, 比较并交换）操作的一个关键类是`Unsafe`

`Unsafe`类中的 CAS 方法是`native`方法，确保比较和交换操作是**不可分割**的，即在一个CPU周期内完成。

- **无锁**：避免线程阻塞，提升并发性能。
- **原子性**：确保操作的不可分割性。

## [6. CAS 算法存在哪些问题？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#cas-算法存在哪些问题)

[ABA 问题](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#aba-问题)

如果一个变量 V 初次读取的时候是 A 值，并且在准备赋值的时候检查到它仍然是 A 值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回 A，那 CAS 操作就会误认为它从来没有被修改过。这个问题被称为 CAS 操作的 **"ABA"问题。**

ABA 问题的解决思路是在变量前面追加上**版本号或者时间戳**。

[循环时间长开销大](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#循环时间长开销大)

CAS 经常会用到自旋操作来进行重试，也就是**不成功就一直循环执行直到成功**。如果长时间不成功，会给 CPU 带来非常大的执行开销。

# [synchronized 关键字](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#synchronized-关键字)

## [1. synchronized 是什么？有什么用？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#synchronized-是什么-有什么用)

主要解决的是多个线程之间访问资源的同步性，可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。

## [2. 如何使用 synchronized？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#如何使用-synchronized)

修饰实例方法、修饰静态方法、修饰代码块。

见JAVAguide。

## [3. JDK1.6 之后的 synchronized 底层做了哪些优化？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#jdk1-6-之后的-synchronized-底层做了哪些优化-锁升级原理了解吗)

在 Java 6 之后， `synchronized` 引入了大量的优化如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销，这些优化让 `synchronized` 锁的效率提升了很多（JDK18 中，偏向锁已经被彻底废弃，前面已经提到过了）。

## [4. synchronized 和 volatile 有什么区别？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#⭐️synchronized-和-volatile-有什么区别)

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

- `volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。**可见性**：锁确保线程在释放锁时，会将本地内存中的共享变量刷新到主内存；在获取锁时，会从主内存中读取共享变量的最新值。
- `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。
-  `volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块 。

## 5. 原理

**Monitor 机制**，每个 Java 对象都与一个 Monitor 相关联。

Monitor 的核心组件

- **Owner**：当前持有锁的线程。
- **EntrySet**：等待获取锁的线程队列。
- **WaitSet**：调用了 `wait()` 方法而进入等待状态的线程队列。

1. **锁的获取**
   - 当线程尝试进入 `synchronized` 代码块时，JVM 会检查 Monitor 的 Owner 是否为空。
   - 如果 Owner 为空，当前线程成为 Owner，并进入临界区执行代码。
   - 如果 Owner 不为空，当前线程进入 EntrySet，进入阻塞状态，等待锁的释放。
2. **锁的释放**
   - 当线程执行完 `synchronized` 代码块时，会释放 Monitor，并将 Owner 设置为空。
   - JVM 会从 EntrySet 中唤醒一个线程，使其成为新的 Owner。
3. **`wait()` 和 `notify()`**
   - 当线程调用 `wait()` 时，它会释放锁并进入 WaitSet。
   - 当其他线程调用 `notify()` 或 `notifyAll()` 时，WaitSet 中的线程会被唤醒，重新进入 EntrySet，竞争锁。

## 6. 锁的升级过程

JVM 对 `synchronized` 进行了优化，引入了 **锁升级机制**：

1. **无锁状态**
   初始状态，没有线程竞争。
2. **偏向锁**
   - 当只有一个线程访问同步代码时，JVM 会将锁标记为偏向锁，避免 CAS 操作。
   - 偏向锁的目的是减少无竞争情况下的开销。
3. **轻量级锁**
   - 当有多个线程轻度竞争时，JVM 会将偏向锁升级为轻量级锁。
   - 轻量级锁通过 CAS 操作尝试获取锁，避免线程阻塞。
4. **重量级锁**
   - 当竞争激烈时，轻量级锁会升级为重量级锁。
   - 重量级锁依赖于操作系统的互斥量（Mutex），会导致线程阻塞和上下文切换。

# [ReentrantLock](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#reentrantlock)

## [1. ReentrantLock 是什么？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#reentrantlock-是什么)

`ReentrantLock` 实现了 `Lock` 接口，是一个**可重入且独占式的锁**，和 `synchronized` 关键字类似。不过，`ReentrantLock` 更灵活、更强大，增加了**轮询、超时、中断、公平锁和非公平锁**等高级功能。

## [2. 公平锁和非公平锁有什么区别？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#公平锁和非公平锁有什么区别)

- **公平锁** : 锁被释放之后，先申请的线程先得到锁。**性能较差一些**，因为公平锁为了保证时间上的绝对顺序，上下文切换更频繁。
- **非公平锁**：锁被释放之后，后申请的线程可能会先获取到锁，是随机或者按照其他优先级排序的。**性能更好，但可能会导致某些线程永远无法获取到锁。**

## [3. synchronized 和 ReentrantLock 有什么区别？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#⭐️synchronized-和-reentrantlock-有什么区别)

**二者都是可重入锁。**



`ReentrantLock` 就属于是可中断锁，`synchronized` 就属于是不可中断锁。

- **可中断锁**：获取锁的过程中可以被中断，不需要一直等到获取锁之后 才能进行其他逻辑处理。`ReentrantLock` 就属于是可中断锁。
- **不可中断锁**：一旦线程申请了锁，就只能等到拿到锁以后才能进行其他的逻辑处理。 `synchronized` 就属于是不可中断锁。



**可重入锁** 也叫递归锁，指的是线程可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果是不可重入锁的话，就会造成死锁。

~~~java
public class SynchronizedDemo {
    public synchronized void method1() {
        System.out.println("方法1");
        method2();
    }

    public synchronized void method2() {
        System.out.println("方法2");
    }
}
~~~

由于 `synchronized`锁是可重入的，同一个线程在调用`method1()` 时可以直接获得当前对象的锁，执行 `method2()` 的时候可以再次获取这个对象的锁，不会产生死锁问题。假如`synchronized`是不可重入锁的话，由于该对象的锁已被当前线程所持有且无法释放，这就导致线程在执行 `method2()`时获取锁失败，会出现死锁问题。

[synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#synchronized-依赖于-jvm-而-reentrantlock-依赖于-api)

[ReentrantLock 比 synchronized 增加了一些高级功能](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#reentrantlock-比-synchronized-增加了一些高级功能)

- **可实现公平锁** : `ReentrantLock`可以指定是公平锁还是非公平锁。而`synchronized`只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock`默认情况是非公平的，可以通过 `ReentrantLock`类的`ReentrantLock(boolean fair)`构造方法来指定是否是公平的。
- **支持超时** ：`ReentrantLock` 提供了 `tryLock(timeout)` 的方法，可以指定等待获取锁的最长等待时间，如果超过了等待时间，就会获取锁失败，不会一直等待。
- **等待可中断** : `ReentrantLock`提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说当前线程在等待获取锁的过程中，如果其他线程中断当前线程「 `interrupt()` 」，当前线程就会抛出 `InterruptedException` 异常，可以捕捉该异常进行相应处理。

## [4. 可中断锁和不可中断锁有什么区别？](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#可中断锁和不可中断锁有什么区别)

- **可中断锁**：获取锁的过程中可以被中断，不需要一直等到获取锁之后才能进行其他逻辑处理。`ReentrantLock` 就属于是可中断锁。
- **不可中断锁**：一旦线程申请了锁，就只能等到拿到锁以后才能进行其他的逻辑处理。 `synchronized` 就属于是不可中断锁。



## 5. 原理

**AQS（AbstractQueuedSynchronizer）**，AQS 是一个用于构建锁和同步器的框架，它通过一个 **FIFO 队列** 来管理等待锁的线程。

**AQS 的核心组件**

1. **state 状态变量**
   - 用于表示锁的状态。
   - 对于 `ReentrantLock`，`state` 表示锁的持有次数：
     - `state = 0`：锁未被持有。
     - `state > 0`：锁被持有，且值为重入次数。
2. **FIFO 等待队列**
   - 用于存储等待锁的线程。
   - 每个线程会被封装成一个 `Node` 对象，加入队列中。
3. **CAS 操作**
   - 通过 CAS（Compare-And-Swap）操作来修改 `state` 的值，确保线程安全。

非公平锁的获取过程

1. 线程尝试通过 CAS 操作将 `state` 从 0 改为 1，如果成功，则获取锁。
2. 如果失败，检查当前线程是否是锁的持有者（重入锁），如果是，则增加 `state` 的值。
3. 如果以上都失败，线程会被封装成 `Node` 对象，加入等待队列，并进入阻塞状态。

公平锁的获取过程

1. 检查等待队列中是否有其他线程在等待锁。
2. 如果有，当前线程直接加入等待队列。
3. 如果没有，尝试通过 CAS 操作获取锁。

锁的释放过程

1. 将 `state` 的值减 1。
2. 如果 `state` 变为 0，表示锁完全释放，唤醒等待队列中的下一个线程。

# [ThreadLocal](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#threadlocal)

## [1. ThreadLocal 有什么用？用法？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#threadlocal-有什么用)

**`ThreadLocal` 类允许每个线程绑定自己的值**，用于存储私有数据，确保不同线程之间的数据互不干扰。

~~~java
public class MyDemo {

    private static ThreadLocal<String> tl = new ThreadLocal<>();

    private String content;

    private String getContent() {
        return tl.get();
    }

    private void setContent(String content) {
         tl.set(content);
    }
}
~~~

## [2. ThreadLocal 原理了解吗？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️threadlocal-原理了解吗)

ThreadLocalMap 理解为ThreadLocal 类实现的定制化的 HashMap。

**最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 可以理解为只是`ThreadLocalMap`的封装，传递了变量值。**

 `ThrealLocal` 类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。

~~~java
public void set(T value) {
    //获取当前请求的线程
    Thread t = Thread.currentThread();
    //取出 Thread 类内部的 threadLocals 变量(哈希表结构)
    ThreadLocalMap map = getMap(t);
    if (map != null)
        // 将需要存储的值放入到这个哈希表中
        map.set(this, value);//this就是ThreadLocal的对象
    else
        createMap(t, value);
}
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
~~~

**每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`可以存储以`ThreadLocal`为 key ，Object 对象为 value 的键值对。**

比如我们在同一个线程中声明了两个 `ThreadLocal` 对象的话， `Thread`内部都是使用仅有的那个`ThreadLocalMap` 存放数据的，`ThreadLocalMap`的 key 就是 `ThreadLocal`对象，value 就是 `ThreadLocal` 对象调用`set`方法设置的值。

![](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241836071.png)

## [3. ThreadLocal 内存泄露问题是怎么导致的？如何避免？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️threadlocal-内存泄露问题是怎么导致的)

ThreadLocal原理：

- **key 是弱引用**：`ThreadLocalMap` 中的 key 是 `ThreadLocal` 的弱引用 (`WeakReference<ThreadLocal<?>>`)。 这意味着，如果 `ThreadLocal` 实例不再被任何强引用指向，垃圾回收器会在下次 GC 时回收该实例，导致 `ThreadLocalMap` 中对应的 key 变为 `null`。
- **value 是强引用**：`ThreadLocalMap` 中的 value 是强引用。 即使 key 被回收（变为 `null`），value 仍然存在于 `ThreadLocalMap` 中，被强引用，不会被回收。

**归根到底，由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，与key是弱引用无关，反而弱引用多了一层保障。**

具体的说：`ThreadLocal` 实例不再被强引用，由于是弱引用，它会被垃圾回收，导致键变为null，但值Value是强引用，不会被回收；并且，线程持续存活，导致 `ThreadLocalMap` 长期存在。此外虽然 `ThreadLocalMap` 在 `get()`, `set()` 和 `remove()` 操作时会尝试清理 key 为 null 的 entry，但这种清理机制是被动的，并不完全可靠。

**如何避免？**

1. 在使用完 `ThreadLocal` 后，务必调用 `remove()` 方法。 这是最安全和最推荐的做法。 `remove()` 方法会从 `ThreadLocalMap` 中显式地移除对应的 entry，彻底解决内存泄漏的风险。 即使将 `ThreadLocal` 定义为 `static final`，也强烈建议在每次使用后调用 `remove()`。
2. 在线程池等线程复用的场景下，使用 `try-finally` 块可以确保即使发生异常，`remove()` 方法也一定会被执行。

## 4.为什么使用弱引用？

​	事实上，在ThreadLocalMap中的set/getEntry方法中，会对key为null（也即是ThreadLocal为null）进行判断，如果为null的话，那么是会对value置为null的。

​	这就意味着使用完ThreadLocal，CurrentThread依然运行的前提下，就算忘记调用remove方法，**弱引用比强引用可以多一层保障**：弱引用的ThreadLocal会被回收，对应的value在下一次ThreadLocalMap调用set,get,remove中的任一方法的时候会被清除，从而避免内存泄漏。



## [5. 如何跨线程传递 ThreadLocal 的值？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#如何跨线程传递-threadlocal-的值)

- `InheritableThreadLocal` ：`InheritableThreadLocal` 是 JDK1.2 提供的工具，继承自 `ThreadLocal` 。使用 `InheritableThreadLocal` 时，会在创建子线程时，令子线程继承父线程中的 `ThreadLocal` 值，但是无法支持线程池场景下的 `ThreadLocal` 值传递。
- `TransmittableThreadLocal` ： `TransmittableThreadLocal` （简称 TTL） 是阿里巴巴开源的工具类，继承并加强了`InheritableThreadLocal`类，可以在线程池的场景下支持 `ThreadLocal` 值传递。项目地址：https://github.com/alibaba/transmittable-thread-local。

# 线程池

## [1. 什么是线程池?](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#什么是线程池)

线程池就是管理一系列线程的资源池。当有任务要处理时，直接从线程池中获取线程来处理，处理完之后线程并不会立即被销毁，而是等待下一个任务。

## [2. 为什么要用线程池？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️为什么要用线程池)

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控

## [3. 如何创建线程池？常用方法](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#如何创建线程池)

**方式一：通过`ThreadPoolExecutor`构造函数来创建（推荐）。**

![image-20250124194438082](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241944131.png)

**方式二：通过 `Executor` 框架的工具类 `Executors` 来创建。**极其不推荐

- `FixedThreadPool`：固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
- `SingleThreadExecutor`： 只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。
- `CachedThreadPool`： 可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。
- `ScheduledThreadPool`：给定的延迟后运行任务或者定期执行任务的线程池。



如果你只需要提交不需要返回结果的任务，并且不关心任务执行过程中的异常情况，可以使用 `execute` 方法；

如果需要获取任务的执行结果，或者需要处理任务执行过程中的异常，建议使用 `submit` 方法。

![image-20250124195250916](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241952960.png)

## [4. 为什么不推荐使用内置线程池？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️为什么不推荐使用内置线程池)

阿里巴巴不允许使用 `Executors` 去创建，而是通过 `ThreadPoolExecutor` 构造函数的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

`Executors` 返回线程池对象的弊端如下：

- `FixedThreadPool` 和 `SingleThreadExecutor`:使用的是有界阻塞队列是 `LinkedBlockingQueue` ，其**任务队列的最大长度**为 `Integer.MAX_VALUE` ，可能堆积大量的请求，从而导致 OOM。
- `CachedThreadPool`:使用的是同步队列 `SynchronousQueue`, 允许创建的线程数量为 `Integer.MAX_VALUE` ，如果任务数量过多且执行速度较慢，可能会创建大量的线程，从而导致 OOM。
- `ScheduledThreadPool` 和 `SingleThreadScheduledExecutor` :使用的无界的延迟阻塞队列 `DelayedWorkQueue` ，任务队列最大长度为 `Integer.MAX_VALUE` ，可能堆积大量的请求，从而导致 OOM。

## [5. 线程池常见参数有哪些？如何解释？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️线程池常见参数有哪些-如何解释)

七个。

核心的是：核心线程池、最大线程数量、任务队列。

![image-20241012155713780](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241948072.png)

## [6. 线程池的核心线程会被回收吗？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#线程池的核心线程会被回收吗)

`ThreadPoolExecutor` 默认不会回收核心线程，即使它们已经空闲了。

## [7. 线程池的拒绝策略有哪些？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️线程池的拒绝策略有哪些)

![image-20241012171253118](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241950080.png)

## [8. 如果不允许丢弃任务任务，应该选择哪个拒绝策略？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#如果不允许丢弃任务任务-应该选择哪个拒绝策略)

CallerRunsPolicy

## [9. CallerRunsPolicy 拒绝策略有什么风险？如何解决？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#callerrunspolicy-拒绝策略有什么风险-如何解决)

可能会导致主线程阻塞，影响程序的正常运行。

## [10. 线程池常用的阻塞队列有哪些？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#线程池常用的阻塞队列有哪些)

- 容量为 `Integer.MAX_VALUE` 的 `LinkedBlockingQueue`（有界阻塞队列）：`FixedThreadPool` 和 `SingleThreadExecutor` 。`FixedThreadPool`最多只能创建核心线程数的线程（核心线程数和最大线程数相等），`SingleThreadExecutor`只能创建一个线程（核心线程数和最大线程数都是 1），二者的任务队列永远不会被放满。
- `SynchronousQueue`（同步队列）：`CachedThreadPool` 。`SynchronousQueue` 没有容量，不存储元素，目的是保证对于提交的任务，如果有空闲线程，则使用空闲线程来处理；否则新建一个线程来处理任务。也就是说，`CachedThreadPool` 的最大线程数是 `Integer.MAX_VALUE` ，可以理解为线程数是可以无限扩展的，可能会创建大量线程，从而导致 OOM。
- `DelayedWorkQueue`（延迟队列）：`ScheduledThreadPool` 和 `SingleThreadScheduledExecutor` 。`DelayedWorkQueue` 的内部元素并不是按照放入的时间排序，而是会按照延迟的时间长短对任务进行排序，内部采用的是“堆”的数据结构，可以保证每次出队的任务都是当前队列中执行时间最靠前的。`DelayedWorkQueue` 添加元素满了之后会自动扩容，增加原来容量的 50%，即永远不会阻塞，最大扩容可达 `Integer.MAX_VALUE`，所以最多只能创建核心线程数的线程。
- `ArrayBlockingQueue`（有界阻塞队列）：底层由数组实现，容量一旦创建，就不能修改

## [11. 线程池处理任务的流程了解吗？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️线程池处理任务的流程了解吗)

![image-20250124195710768](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501241957811.png)

**线程池在提交任务前，可以提前创建线程吗？**

答案是可以的！`ThreadPoolExecutor` 提供了两个方法帮助我们在提交任务之前，完成核心线程的创建，从而实现线程池预热的效果：

- `prestartCoreThread()`:启动一个线程，等待任务，如果已达到核心线程数，这个方法返回 false，否则返回 true；
- `prestartAllCoreThreads()`:启动所有的核心线程，并返回启动成功的核心线程数。

## [12. 线程池中线程异常后，销毁还是复用？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️线程池中线程异常后-销毁还是复用)

简单来说：

使用`execute()`时，未捕获异常导致线程终止，线程池创建新线程替代；

使用`submit()`时，异常被封装在`Future`中，线程继续复用。

## [13. 如何设定线程池的大小？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#如何设定线程池的大小)

- **CPU 密集型任务(N+1)：** 这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1。比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。
- **I/O 密集型任务(2N)：** 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。

CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。但凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。

## [14. 如何设计一个能够根据任务的优先级来执行的线程池？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️如何设计一个能够根据任务的优先级来执行的线程池)

使用 `PriorityBlockingQueue` （优先级阻塞队列）作为任务队列。

## 15. 如何知道是否要销毁线程？

1. **线程进入空闲状态**：
   - 当线程执行完任务后，会尝试从任务队列中获取新任务。
   - 如果任务队列为空，线程会调用`BlockingQueue.poll(keepAliveTime, TimeUnit)`方法等待新任务。
2. **线程超时**：
   - 如果线程在`keepAliveTime`时间内没有获取到新任务，`poll`方法会返回`null`。
   - 线程池会将该线程标记为可销毁。
3. **线程销毁**：
   - 线程池会检查当前线程数是否大于`corePoolSize`。
   - 如果大于`corePoolSize`，线程池会调用线程的`interrupt()`方法，终止线程的执行；否则，线程会继续等待任务。
   - 核心线程是线程池中始终存在的线程，即使它们处于空闲状态也不会被销毁（除非设置了`allowCoreThreadTimeOut(true)`）

## 16. 线程池区分临时线程和核心线程吗，销毁的一定是后来创建的临时线程吗?

销毁的线程 **不一定是后来创建的临时线程**，而是销毁线程在`keepAliveTime`时间内没有在任务队列获取到新任务的线程。

线程池在内部 **并不显式区分核心线程和临时线程**，临时线程的数量由`maximumPoolSize - corePoolSize`决定。

# [Future](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#future)

## [1. Future 类有什么用？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#future-类有什么用)

当我们执行某一耗时的任务时，可以将这个耗时任务交给一个子线程去**异步执行**，同时我们可以干点其他事情，不用傻傻等待耗时任务执行完成。

~~~java
// V 代表了Future执行的任务返回值的类型
public interface Future<V> {
    // 取消任务执行，成功取消返回 true，否则返回 false
    boolean cancel(boolean mayInterruptIfRunning);
    // 判断任务是否被取消
    boolean isCancelled();
    // 判断任务是否已经执行完成
    boolean isDone();
    // 获取任务执行结果
    V get() throws InterruptedException, ExecutionException;
    // 指定时间内没有返回计算结果就抛出 TimeOutException 异常
    V get(long timeout, TimeUnit unit)

        throws InterruptedException, ExecutionException, TimeoutExceptio

}
~~~

## [2. Callable 和 Future 有什么关系？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#callable-和-future-有什么关系)

`FutureTask` 提供了 `Future` 接口的基本实现，`ExecutorService.submit()` 方法返回的其实就是 `Future` 的实现类 `FutureTask` 。

`FutureTask` 有两个构造函数，可传入 `Callable` 或者 `Runnable` 对象。实际上，传入 `Runnable` 对象也会在方法内部转换为`Callable` 对象。

## [3.  一个任务需要依赖另外两个任务执行完之后再执行，怎么设计？](https://javaguide.cn/java/concurrent/java-concurrent-questions-03.html#⭐️一个任务需要依赖另外两个任务执行完之后再执行-怎么设计)

1:使用 `CountDownLatch`

~~~java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        // 创建一个 CountDownLatch，计数为 2
        CountDownLatch latch = new CountDownLatch(2);
        // 第一个依赖任务
        Thread task1 = new Thread(() -> {
            try {
                System.out.println("任务 1 开始执行");
                Thread.sleep(2000); // 模拟耗时操作
                System.out.println("任务 1 执行完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                // 任务完成，计数减 1
                latch.countDown();
            }
        });
        // 第二个依赖任务
        Thread task2 = new Thread(() -> {
            try {
                System.out.println("任务 2 开始执行");
                Thread.sleep(3000); // 模拟耗时操作
                System.out.println("任务 2 执行完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                // 任务完成，计数减 1
                latch.countDown();
            }
        });
        // 启动两个依赖任务
        task1.start();
        task2.start();
        // 主线程等待两个任务完成
        latch.await();
        // 执行依赖于前两个任务的任务
        System.out.println("开始执行依赖任务");
        System.out.println("依赖任务执行完毕");
    }
}
~~~

2.`CompletableFuture`

~~~java

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class CompletableFutureExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 第一个依赖任务
        CompletableFuture<Void> task1 = CompletableFuture.runAsync(() -> {
            try {
                System.out.println("任务 1 开始执行");
                Thread.sleep(2000); // 模拟耗时操作
                System.out.println("任务 1 执行完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 第二个依赖任务
        CompletableFuture<Void> task2 = CompletableFuture.runAsync(() -> {
            try {
                System.out.println("任务 2 开始执行");
                Thread.sleep(3000); // 模拟耗时操作
                System.out.println("任务 2 执行完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 等待两个任务完成
        CompletableFuture<Void> allTasks = CompletableFuture.allOf(task1, task2);

        // 执行依赖于前两个任务的任务
        CompletableFuture<Void> dependentTask = allTasks.thenRun(() -> {
            System.out.println("开始执行依赖任务");
            System.out.println("依赖任务执行完毕");
        });

        // 等待依赖任务完成
        dependentTask.get();
    }
}
~~~

3.使用线程池和 `Future`

~~~java
import java.util.concurrent.*;

public class FutureExample {
    public static void main(String[] args) {
        // 创建一个固定大小的线程池
        ExecutorService executor = Executors.newFixedThreadPool(2);

        // 第一个依赖任务
        Future<?> task1 = executor.submit(() -> {
            try {
                System.out.println("任务 1 开始执行");
                Thread.sleep(2000); // 模拟耗时操作
                System.out.println("任务 1 执行完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 第二个依赖任务
        Future<?> task2 = executor.submit(() -> {
            try {
                System.out.println("任务 2 开始执行");
                Thread.sleep(3000); // 模拟耗时操作
                System.out.println("任务 2 执行完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        try {
            // 等待两个任务完成
            task1.get();
            task2.get();
            // 执行依赖于前两个任务的任务
            System.out.println("开始执行依赖任务");
            System.out.println("依赖任务执行完毕");
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            // 关闭线程池
            executor.shutdown();
        }
    }
}
~~~
