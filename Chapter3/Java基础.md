# 基础概念

## [1. Java 语言有哪些特点?](https://javaguide.cn/java/basis/java-basic-questions-01.html#java-语言有哪些特点)

1. 简单易学（语法简单，上手容易）；
2. 面向对象（封装，继承，多态）；
3. 平台无关性（ Java 虚拟机实现平台无关性）；
4. 支持多线程（ C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计，而 Java 语言却提供了多线程支持）；
5. 可靠性（具备异常处理和自动内存管理机制）；
6. 安全性（Java 语言本身的设计就提供了多重安全防护机制如访问权限修饰符、限制程序直接访问操作系统资源）；
7. 高效性（通过 Just In Time 编译器等技术的优化，Java 语言的运行效率还是非常不错的）；
8. 支持网络编程并且很方便；
9. 编译与解释并存；
10. 生态强大
11. 内存管理：Java有自己的垃圾回收机制，自动管理内存和回收不再使用的对象。这样，开发者不需要手动管理内存，从而减少内存泄漏和其他内存相关的问题。

## [2. Java SE vs Java EE](https://javaguide.cn/java/basis/java-basic-questions-01.html#java-se-vs-java-ee)

简单来说，Java SE （Standard Edition）是 Java 的基础版本，Java EE（Enterprise Edition）是 Java 的高级版本。

Java SE 更适合开发桌面应用程序或简单的服务器应用程序，Java EE 更适合开发复杂的企业级应用程序或 Web 应用程序。

## [3. JVM vs JDK vs JRE](https://javaguide.cn/java/basis/java-basic-questions-01.html#jvm-vs-jdk-vs-jre)

### 3.1 JVM

Java 虚拟机（Java Virtual Machine, JVM）是**运行 Java 字节码的虚拟机**。JVM 有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。字节码和不同系统的 JVM 实现是 Java 语言“一次编译，随处可以运行”的关键所在。

JVM 并不是只有一种！只要满足 JVM 规范，每个公司、组织或者个人都可以开发自己的专属 JVM。

![image-20250122103252536](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221032836.png)

### 3.2 JDK和JRE

JDK（Java Development Kit）是一个功能齐全的 **Java 开发工具包**，供开发者使用，用于**创建和编译 Java 程序**。

它包含了 **JRE（Java Runtime Environment）**，以及编译器 javac 和其他工具，如 javadoc（文档生成器）、jdb（调试器）、jconsole（监控工具）、javap（反编译工具）等。



JRE 是**运行已编译 Java 程序所需的环境**，主要包含以下两个部分：

1. **JVM** : 也就是我们上面提到的 Java 虚拟机。
2. **Java 基础类库（Class Library）**：一组标准的类库，提供常用的功能和 API（如 I/O 操作、网络通信、数据结构等）。



简单来说：JRE 只包含运行 Java 程序所需的环境和类库，而 JDK 不仅包含 JRE，还包括用于开发和调试 Java 程序的工具。

![image-20250122103813330](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221038359.png)

## [4.什么是字节码?采用字节码的好处是什么?](https://javaguide.cn/java/basis/java-basic-questions-01.html#什么是字节码-采用字节码的好处是什么)

**Java 是编译与解释共存的语言** 。

在 Java 中，**JVM 可以理解的代码就叫做字节码**（即扩展名为 **`.class`** 的文件），它不面向任何特定的处理器，只面向虚拟机。

Java 语言通过字节码的方式，在一定程度上**解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点**。

所以， Java 程序运行时相对来说还是高效的（不过，和 C、 C++，Rust，Go 等语言还是有一定差距的），而且，由于字节码并不针对一种特定的机器，因此，**Java 程序无须重新编译便可在多种不同操作系统的计算机上运行**。

有些方法和代码块是经常需要被调用的(也就是所谓的热点代码)，所以后面引进了 **JIT（Just in Time Compilation）** 编译器，而 JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。

------

著作权归JavaGuide(javaguide.cn)所有 基于MIT协议 原文链接：https://javaguide.cn/java/basis/java-basic-questions-01.html

![image-20250122104222701](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221042728.png)

## [5. 为什么说 Java 语言“编译与解释并存”？](https://javaguide.cn/java/basis/java-basic-questions-01.html#为什么说-java-语言-编译与解释并存)

**编译型**：[编译型语言](https://zh.wikipedia.org/wiki/編譯語言) 会通过[编译器](https://zh.wikipedia.org/wiki/編譯器)将源代码一次性翻译成可被该平台执行的机器码。一般情况下，编译语言的执行速度比较快，开发效率比较低。常见的编译性语言有 C、C++、Go、Rust 等等。

**解释型**：[解释型语言](https://zh.wikipedia.org/wiki/直譯語言)会通过[解释器](https://zh.wikipedia.org/wiki/直譯器)一句一句的将代码解释（interpret）为机器代码后再执行。解释型语言开发效率比较快，执行速度比较慢。常见的解释性语言有 Python、JavaScript、PHP 等等。



因为 Java 程序要经过先编译，后解释两个步骤，由 Java 编写的程序需要先经过编译步骤，生成字节码（`.class` 文件），这种字节码必须由 Java 解释器来解释执行。

## [6. Java 和 C++ 和 Python的区别?](https://javaguide.cn/java/basis/java-basic-questions-01.html#java-和-c-的区别)

- Java 不提供指针来直接访问内存，程序内存更加安全
- Java 的类是单继承的，C++ 支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承。
- Java 有自动内存管理垃圾回收机制(GC)，不需要程序员手动释放无用内存。
- C ++同时支持方法重载和操作符重载，但是 Java 只支持方法重载（操作符重载增加了复杂性，这与 Java 最初的设计思想不符）。



- Java是一种已编译的编程语言，Java编译器将源代码编译为字节码，而字节码则由Java虚拟机执行
- python是一种解释语言，翻译时会在执行程序的同时进行翻译。

# Java语法

## [1. 注释有哪几种形式？](https://javaguide.cn/java/basis/java-basic-questions-01.html#注释有哪几种形式)

1. **单行注释**：`//`通常用于解释方法内某单行代码的作用。
2. **多行注释**：`/*    */`通常用于解释一段代码的作用。
3. **文档注释**：``/**`    `*/` 结束`通常用于生成 Java 开发文档。

代码的注释不是越详细越好。实际上好的代码本身就是注释，我们要尽量规范和美化自己的代码来减少不必要的注释。

若编程语言足够有表达力，就不需要注释，尽量通过代码来阐述。

## [2. 标识符和关键字的区别是什么？](https://javaguide.cn/java/basis/java-basic-questions-01.html#标识符和关键字的区别是什么)

标识符：是用户为类、变量、方法、包等程序元素所取的名字，比如`Person`类。

关键字：是 Java 语言中具有特定含义和用途的保留字，它们被 Java 编译器赋予了特殊的意义，用于表示程序的结构、数据类型、控制流程等重要元素，不能用作其他用途，用户不能自定义与关键字相同的标识符。例如，`public`、`class`、`int`、`if`、`else`等都是关键字。

虽然 `true`, `false`, 和 `null` 看起来像关键字但实际上他们是字面值，同时你也不可以作为标识符来使用。

## [3. Java 语言关键字有哪些？](https://javaguide.cn/java/basis/java-basic-questions-01.html#java-语言关键字有哪些)

![image-20250122105843092](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221058117.png)

## [4. 自增自减](https://javaguide.cn/java/basis/java-basic-questions-01.html#自增自减运算符)

**前缀形式**（例如 `++a` 或 `--a`）：先自增/自减变量的值，然后再使用该变量，例如，`b = ++a` 先将 `a` 增加 1，然后把增加后的值赋给 `b`。

**后缀形式**（例如 `a++` 或 `a--`）：先使用变量的当前值，然后再自增/自减变量的值。例如，`b = a++` 先将 `a` 的当前值赋给 `b`，然后再将 `a` 增加 1。

## [5. 移位运算符](https://javaguide.cn/java/basis/java-basic-questions-01.html#移位运算符)

**使用移位运算符的主要原因**：

**高效**：移位运算符直接对应于处理器的移位指令。**现代处理器具有专门的硬件指令来执行这些移位操作**，这些指令通常在一个时钟周期内完成。相比之下，乘法和除法等算术运算在硬件层面上需要更多的时钟周期来完成。

**节省内存**：通过移位操作，**可以使用一个整数（如 `int` 或 `long`）来存储多个布尔值或标志位，从而节省内存**。



Java 中有三种移位运算符：

- `<<` :左移运算符，向左移若干位，高位丢弃，低位补零。`x << n`,相当于 x 乘以 2 的 n 次方(不溢出的情况下)。
- `>>` :带符号右移，向右移若干位，高位补符号位，低位丢弃。正数高位补 0,负数高位补 1。`x >> n`,相当于 x 除以 2 的 n 次方。
- `>>>` :无符号右移，忽略符号位，空位都以 0 补齐。

移位操作符实际上支持的类型只有`int`和`long`，编译器在对`short`、`byte`、`char`类型进行移位前，都会将其转换为`int`类型再操作。



**如果移位的位数超过数值所占有的位数会怎样？**

也就是说左移/右移 32 位相当于不进行移位操作（32%32=0），左移/右移 42 位相当于左移/右移 10 位（42%32=10）。

## [6. continue、break 和 return 的区别是什么？](https://javaguide.cn/java/basis/java-basic-questions-01.html#continue、break-和-return-的区别是什么)

1. `continue`：指跳出当前的这一次循环，继续下一次循环。
2. `break`：指跳出整个循环体，继续执行循环下面的语句。
3. `return` 用于跳出所在方法，结束该方法的运行。



## [7. 为什么 Java 不引入引用传递呢？](https://javaguide.cn/java/basis/why-there-only-value-passing-in-java.html#为什么-java-不引入引用传递呢)

Java中只有值传递。

Java 之父 James Gosling 在设计之初就看到了 C、C++ 的许多弊端，所以才想着去设计一门新的语言 Java。在他设计 Java 的时候就遵循了简单易用的原则，摒弃了许多开发者一不留意就会造成问题的“特性”，语言本身的东西少了，开发者要学习的东西也少了。

Java 中将实参传递给方法（或函数）的方式是 **值传递**：

- 如果参数是基本类型的话，很简单，传递的就是基本类型的字面量值的拷贝，会创建副本。
- 如果参数是引用类型，传递的就是实参所引用的对象在堆中地址值的拷贝，同样也会创建副本。

https://javaguide.cn/java/basis/why-there-only-value-passing-in-java.html#%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92%E6%98%AF%E6%80%8E%E4%B9%88%E6%A0%B7%E7%9A%84

# 基本数据类型

## [1. Java 中的几种基本数据类型了解么？](https://javaguide.cn/java/basis/java-basic-questions-01.html#java-中的几种基本数据类型了解么)

- 6 种数字类型：
  - 4 种整数型：`byte`、`short`、`int`、`long`
  - 2 种浮点型：`float`、`double`
- 1 种字符类型：`char`
- 1 种布尔型：`boolean`。

这八种基本类型都有对应的包装类分别为：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean` 。

在Java中char占两个字节，但是在C语言中，占一个字节。



Java 里使用 `long` 类型的数据一定要在数值后面加上 **L**，否则将作为整型解析。

Java 里使用 `float` 类型的数据一定要在数值后面加上 **f 或 F**，否则将无法通过编译。

`char a = 'h'`char :单引号，`String a = "hello"` :双引号。



![image-20250122112756408](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221127439.png)

在计算机中，整数通常以补码的形式存储和运算，

在 8 位二进制补码表示中，最高位为符号位，**0 表示正数，1 表示负数**。**对于正数，其补码与原码相同**；对于负数，其补码是原码除符号位外各位取反，然后末位加 1。

- **最小值**：`byte`类型能表示的最小值是 - 128，其补码表示为`10000000`。这是因为按照补码规则，`10000000`表示的是 - 128，而不是 - 0（在计算机中不存在 - 0 的概念）。
- **最大值**：`byte`类型能表示的最大值是 127，其补码表示为`01111111`，对应的十进制数就是 127。

## [2. 基本类型和包装类型的区别？](https://javaguide.cn/java/basis/java-basic-questions-01.html#基本类型和包装类型的区别)

**用途**：除了定义一些常量和局部变量之外，我们在其他地方比如方法参数、对象属性中很少会使用基本类型来定义变量。并且，包装类型可用于泛型，而基本类型不可以。

**存储方式**：基本数据类型的**局部变量**存放在 **Java 虚拟机栈中的局部变量表中**，基本数据类型的**成员变量**（未被 `static` 修饰 ）存放在 **Java 虚拟机的堆中**。**包装类型属于对象类型，我们知道几乎所有对象实例都存在于堆中**。

**占用空间**：相比于包装类型（对象类型）， 基本数据类型占用的空间往往非常小。

**默认值**：成员变量包装类型不赋值就是 `null` ，而基本类型有默认值且不是 `null`。

**比较方式**：对于基本数据类型来说，`==` 比较的是值。对于包装数据类型来说，`==` 比较的是对象的内存地址。所有整型包装类对象之间值的比较，全部使用 `equals()` 方法。



**基本数据类型存放在栈中是一个常见的误区！** 基本数据类型的存储位置取决于它们的作用域和声明方式。如果它们是局部变量，那么它们会存放在栈中；如果它们是成员变量，那么它们会存放在堆/方法区/元空间中。

## 3. 为什么要引入包装类，为什么要保留基本类

包装类：

- 对象封装有很多好处，可以把属性也就是数据跟处理这些数据的方法结合在一起，比如Integer就有parseInt()等方法来专门处理int型相关的数据。
- Java中绝大部分方法或类都是用来处理类类型对象的，如ArrayList集合类就只能以类作为他的存储对象，而这时如果想把一个int型的数据存入list是不可能的，必须把它包装成类，也就是Integer才能被List所接受。所以Integer的存在是很必要的。
- 在Java中，泛型只能使用引用类型，而不能使用基本类型

基本类型：

- 包装类是引用类型，对象的引用和对象本身是分开存储的，而对于基本类型数据，变量对应的内存块直接存储数据本身，因此基本类型数据在读写效率方面，要比包装类高效。
- 基本类型更省空间。

## [4. 包装类型的缓存机制了解么？](https://javaguide.cn/java/basis/java-basic-questions-01.html#包装类型的缓存机制了解么)

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `True` or `False`。

两种浮点数类型的包装类 `Float`,`Double` 并没有实现缓存机制。

![image-20250122120919968](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221209020.png)

~~~java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true

Float i11 = 333f;
Float i22 = 333f;
System.out.println(i11 == i22);// 输出 false

Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false


Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);//输出 false
/* Integer i1=40 这一行代码会发生装箱，也就是说这行代码等价于 Integer i1=Integer.valueOf(40) 。因此，i1 直接使用的是缓存中的对象。而Integer i2 = new Integer(40) 会直接创建新的对象。*/
~~~

## [5. 自动装箱与拆箱了解吗？原理是什么？](https://javaguide.cn/java/basis/java-basic-questions-01.html#自动装箱与拆箱了解吗-原理是什么)

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

装箱其实就是调用了 包装类的`valueOf()`方法，拆箱其实就是调用了 `xxxValue()`方法。

~~~java
Integer i = 10;  //装箱
int n = i;   //拆箱

Integer i = 10 等价于 Integer i = Integer.valueOf(10)
int n = i 等价于 int n = i.intValue();
~~~

如果频繁拆装箱的话，也会严重影响系统的性能。我们应该尽量避免不必要的拆装箱操作。

## [6. 为什么浮点数运算的时候会有精度丢失的风险？](https://javaguide.cn/java/basis/java-basic-questions-01.html#为什么浮点数运算的时候会有精度丢失的风险)

计算机是二进制的，而且计算机在表示一个数字时，宽度是有限的，无限循环的小数存储在计算机时，只能被截断，所以就会导致小数精度发生损失的情况。

~~~java
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.printf("%.9f",a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
~~~

## [7. 如何解决浮点数运算的精度丢失问题？](https://javaguide.cn/java/basis/java-basic-questions-01.html#如何解决浮点数运算的精度丢失问题)

`BigDecimal` 可以实现对浮点数的运算，不会造成精度丢失。

通常情况下，大部分需要浮点数精确运算结果的业务场景（比如涉及到钱的场景）都是通过 `BigDecimal` 来做的。

~~~java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("1.00");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(c);
BigDecimal y = b.subtract(c);

System.out.println(x); /* 0.2 */
System.out.println(y); /* 0.20 */
// 比较内容，不是比较值
System.out.println(Objects.equals(x, y)); /* false */
// 比较值相等用相等compareTo，相等返回0
System.out.println(0 == x.compareTo(y)); /* true */
~~~

## [8. 超过 long 整型的数据应该如何表示？](https://javaguide.cn/java/basis/java-basic-questions-01.html#超过-long-整型的数据应该如何表示)

使用`BigInteger` 内部使用 `int[]` 数组来存储任意大小的整形数据。

但相对于常规整数类型的运算来说，`BigInteger` 运算的效率会相对较低。

# 变量和方法

## [1. 成员变量与局部变量的区别？](https://javaguide.cn/java/basis/java-basic-questions-01.html#成员变量与局部变量的区别)

**语法形式**：从语法形式上看，成员变量是属于类的，而局部变量是在代码块或方法中定义的变量或是方法的参数；成员变量可以被 `public`,`private`,`static` 等修饰符所修饰，而局部变量不能被访问控制修饰符及 `static` 所修饰；但是，成员变量和局部变量都能被 `final` 所修饰。

**存储方式**：从变量在内存中的存储方式来看，如果成员变量是使用 `static` 修饰的，那么这个成员变量是属于类的，如果没有使用 `static` 修饰，这个成员变量是属于实例的。而对象存在于堆内存，局部变量则存在于栈内存。

**生存时间**：从变量在内存中的生存时间上看，成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动生成，随着方法的调用结束而消亡。

**默认值**：从变量是否有默认值来看，成员变量如果没有被赋初始值，则会自动以类型的默认值而赋值（一种情况例外:被 `final` 修饰的成员变量也必须显式地赋值），而局部变量则不会自动赋值。

**为什么成员变量有默认值？**

局部变量没赋值很好判断，可以直接报错。而成员变量可能是运行时赋值，无法判断，误报“没默认值”又会影响用户体验，所以采用自动赋默认值。

## [2. 静态变量有什么作用？](https://javaguide.cn/java/basis/java-basic-questions-01.html#静态变量有什么作用)

静态变量也就是被 `static` 关键字修饰的变量。它可以被类的所有实例共享，无论一个类创建了多少个对象，它们都共享同一份静态变量。也就是说，静态变量只会被分配一次内存，即使创建多个对象，这样可以节省内存。

通常情况下，静态变量会被 `final` 关键字修饰成为常量

## [3. 字符型常量和字符串常量的区别?](https://javaguide.cn/java/basis/java-basic-questions-01.html#字符型常量和字符串常量的区别)

字符常量是单引号引起的一个字符，字符串常量是双引号引起的 0 个或若干个字符。

**含义** : 字符常量相当于一个整型值( ASCII 值),可以参加表达式运算; 字符串常量代表一个地址值(该字符串在内存中存放位置)。

**占内存大小**：字符常量只占 2 个字节; 字符串常量占若干个字节。

## [4. 什么是方法的返回值?方法有哪几种类型？](https://javaguide.cn/java/basis/java-basic-questions-01.html#什么是方法的返回值-方法有哪几种类型)

获取到的某个方法体中的代码执行后产生的结果

**1、无参数无返回值的方法**

**2、有参数无返回值的方法**

**3、有返回值无参数的方法**

**4、有返回值有参数的方法**

## [5. 静态方法为什么不能调用非静态成员?](https://javaguide.cn/java/basis/java-basic-questions-01.html#静态方法为什么不能调用非静态成员)

静态方法是属于类的，在类加载的时候就会分配内存，可以通过类名直接访问。而非静态成员属于实例对象，只有在对象实例化之后才存在，需要通过类的实例对象去访问。

在类的非静态成员不存在的时候静态方法就已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作。

## [6. 静态方法和实例方法有何不同？](https://javaguide.cn/java/basis/java-basic-questions-01.html#静态方法和实例方法有何不同)

**1、调用方式**

在外部调用静态方法时，可以使用 `类名.方法名` 的方式，**调用静态方法可以无需创建对象** 。

**2、访问类成员是否存在限制**

静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），不允许访问实例成员（即实例成员变量和实例方法），而实例方法不存在这个限制。

## [7. 重载和重写有什么区别？](https://javaguide.cn/java/basis/java-basic-questions-01.html#重载和重写有什么区别)

重载：如果多个方法(比如 `StringBuilder` 的构造方法)有相同的名字、不同的参数， 便产生了重载。



**重写**：重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变。

重写需要：“两同”即方法名相同、形参列表相同；

“两小”指的是子类方法返回值类型应比父类方法返回值类型更小或相等，子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等；

“一大”指的是子类方法的访问权限应比父类方法的访问权限更大或相等。

区别：

![image-20250122134247171](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221342210.png)

注意：如果方法的返回类型是 void 和基本数据类型，则返回值重写时不可修改。但是如果方法的返回值是引用类型，重写时是可以返回该引用类型的子类的。

## [8. 什么是可变长参数？](https://javaguide.cn/java/basis/java-basic-questions-01.html#什么是可变长参数)

可变参数只能作为函数的最后一个参数，但其前面可以有也可以没有任何其他参数。

优先匹配固定参数的方法

Java 的可变参数编译后实际会被转换成一个数组

~~~java
public static void method2(String arg1, String... args) {
   //......
}
~~~

# 面向对象基础

## [1.面向对象和面向过程的区别](https://javaguide.cn/java/basis/java-basic-questions-02.html#面向对象和面向过程的区别)

- **面向过程编程（POP）**：面向过程把解决问题的过程拆成一个个方法，通过一个个方法的执行解决问题。
- **面向对象编程（OOP）**：面向对象会先抽象出对象，然后用对象执行方法的方式解决问题

相比较于 POP，OOP 开发的程序一般具有下面这些优点：

- **易维护**：由于良好的结构和封装性，OOP 程序通常更容易维护。
- **易复用**：通过继承和多态，OOP 设计使得代码更具复用性，方便扩展功能。
- **易扩展**：模块化设计使得系统扩展变得更加容易和灵活。

POP 的编程方式通常更为简单和直接，适合处理一些较简单的任务。

POP 和 OOP 的性能差异主要取决于它们的运行机制，而不仅仅是编程范式本身。因此，简单地比较两者的性能是一个常见的误区

## [2. 创建一个对象用什么运算符?对象实体与对象引用有何不同?](https://javaguide.cn/java/basis/java-basic-questions-02.html#创建一个对象用什么运算符-对象实体与对象引用有何不同)

new 运算符，new 创建对象实例（对象实例在堆内存中），对象引用指向对象实例（对象引用存放在栈内存中）。

- 一个对象引用可以指向 0 个或 1 个对象（一根绳子可以不系气球，也可以系一个气球）；
- 一个对象可以有 n 个引用指向它（可以用 n 条绳子系住一个气球）。

## [3. 对象的相等和引用相等的区别](https://javaguide.cn/java/basis/java-basic-questions-02.html#对象的相等和引用相等的区别)

- 对象的相等一般比较的是内存中存放的内容是否相等。
- 引用相等一般比较的是他们指向的内存地址是否相等。

~~~java
String str1 = "hello";
String str2 = new String("hello");
String str3 = "hello";
// 使用 == 比较字符串的引用相等
System.out.println(str1 == str2);//false
System.out.println(str1 == str3);//true
// 使用 equals 方法比较字符串的相等
System.out.println(str1.equals(str2));//true
System.out.println(str1.equals(str3));//true
~~~

## [4. 如果一个类没有声明构造方法，该程序能正确执行吗?](https://javaguide.cn/java/basis/java-basic-questions-02.html#如果一个类没有声明构造方法-该程序能正确执行吗)

如果一个类没有声明构造方法，也可以执行！因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法.

## [5. 构造方法有哪些特点？是否可被 override?](https://javaguide.cn/java/basis/java-basic-questions-02.html#构造方法有哪些特点-是否可被-override)

- **名称与类名相同**：构造方法的名称必须与类名完全一致。
- **没有返回值**：构造方法没有返回类型，且不能使用 `void` 声明。
- **自动执行**：在生成类的对象时，构造方法会自动执行，无需显式调用。

构造方法**不能被重写（override）**，但**可以被重载（overload）**。因此，一个类中可以有多个构造方法，这些构造方法可以具有不同的参数列表，以提供不同的对象初始化方式。

## [6. 面向对象三大特征](https://javaguide.cn/java/basis/java-basic-questions-02.html#面向对象三大特征)

封装：一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。

继承：通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间。

1. **子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，只是拥有**。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（以后介绍）。

多态：

- 重载
- 重写
- 接口多态：一个对象具有多种的状态，具体表现为父类的引用指向子类的实例（图形方法可以有三角形、圆形多个实例）

## [7. 接口和抽象类有什么共同点和区别？](https://javaguide.cn/java/basis/java-basic-questions-02.html#接口和抽象类有什么共同点和区别)

接口是更纯粹的抽象类，接口中只能定义静态变量和抽象方法，而抽象类可以定义成员变量和抽象方法。

[接口和抽象类的共同点](#接口和抽象类的共同点)

- **实例化**：接口和抽象类都不能直接实例化，只能被实现（接口）或继承（抽象类）后才能创建具体的对象。
- **抽象方法**：接口和抽象类都可以包含抽象方法。抽象方法没有方法体，必须在子类或实现类中实现。

[接口和抽象类的区别](https://javaguide.cn/java/basis/java-basic-questions-02.html#接口和抽象类的区别)

**设计目的**：接口主要用于对类的行为进行约束，你实现了某个接口就具有了对应的行为。抽象类主要用于代码复用，强调的是所属关系。

**继承和实现**：一个类只能继承一个类（包括抽象类），因为 Java 不支持多继承。但一个类可以实现多个接口，一个接口也可以继承多个其他接口。

**成员变量**：接口中的成员变量只能是 `public static final` 类型的（静态变量），不能被修改且必须有初始值。抽象类的成员变量可以有任何修饰符（`private`, `protected`, `public`），可以在子类中被重新定义或赋值。

**方法**： 

- Java 8 之前，接口中的方法默认是 `public abstract` ，（**因此可以省略**）也就是只能有方法声明。自 Java 8 起，可以在接口中定义 `default`（默认） 方法和 `static` （静态）方法。 自 Java 9 起，接口可以包含 `private` 方法。
- 抽象类可以包含抽象方法和非抽象方法。抽象方法没有方法体，必须在子类中实现。非抽象方法有具体实现，可以直接在抽象类中使用或在子类中重写。

## 9.为什么java8之后要有引入default？

可以在接口中添加带有默认实现的方法，实现该接口的类如果没有重写这个`default`方法，就会使用接口中提供的默认实现，这样就不会影响到现有的实现类，**使得接口可以在不破坏现有代码的情况下进行扩展。**

## [10. 深拷贝和浅拷贝区别了解吗？什么是引用拷贝？](https://javaguide.cn/java/basis/java-basic-questions-02.html#深拷贝和浅拷贝区别了解吗-什么是引用拷贝)

**浅拷贝**：浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，**浅拷贝会直接复制内部对象的引用地址，也就是说拷贝对象和原对象共用同一个内部对象**。

**深拷贝**：深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。

**引用拷贝**：引用拷贝就是两个不同的引用指向同一个对象。

![image-20250122141239432](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221412479.png)

## [11. java创建对象有哪些方式？](https://xiaolincoding.com/interview/java.html#java创建对象有哪些方式)

**使用new关键字**：通过new关键字直接调用类的构造方法来创建对象。

**使用Class类的newInstance()方法**：通过反射机制，可以使用Class类的newInstance()方法创建对象。

~~~java
public class MyClass {
    public MyClass() {
        // Constructor
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = MyClass.class;
        MyClass obj = (MyClass) clazz.newInstance();
    }
}
~~~

**使用Constructor类的newInstance()方法**：同样是通过反射机制，可以使用Constructor类的newInstance()方法创建对象。

**使用clone()方法**：如果类实现了Cloneable接口，可以使用clone()方法复制对象。

**使用反序列化**：通过将对象序列化到文件或流中，然后再进行反序列化来创建对象。

## [12. New出的对象什么时候回收？](https://xiaolincoding.com/interview/java.html#new出的对象什么时候回收)

Java的垃圾回收器（Garbage Collector）负责回收，垃圾回收器的工作是在程序运行过程中自动进行的，它会周期性地检测不再被引用的对象，并将其回收释放内存。	

某个对象的引用计数为0时，表示该对象不再被引用，可以被回收。

# [Object](https://javaguide.cn/java/basis/java-basic-questions-02.html#object)

## [1. Object 类的常见方法有哪些？](https://javaguide.cn/java/basis/java-basic-questions-02.html#object-类的常见方法有哪些)

~~~java
/**
 * native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
 */
public final native Class<?> getClass()
/**
 * native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
 */
public native int hashCode()
/**
 * 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
 */
public boolean equals(Object obj)
/**
 * native 方法，用于创建并返回当前对象的一份拷贝。
 */
protected native Object clone() throws CloneNotSupportedException
/**
 * 返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
 */
public String toString()
/**
 * native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
 */
public final native void notify()
/**
 * native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
 */
public final native void notifyAll()
/**
 * native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
 */
public final native void wait(long timeout) throws InterruptedException
/**
 * 多了 nanos 参数，这个参数表示额外时间（以纳秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 纳秒。。
 */
public final void wait(long timeout, int nanos) throws InterruptedException
/**
 * 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
 */
public final void wait() throws InterruptedException
/**
 * 实例被垃圾回收器回收的时候触发的操作
 */
protected void finalize() throws Throwable { }
~~~

## [2. == 和 equals() 的区别](https://javaguide.cn/java/basis/java-basic-questions-02.html#和-equals-的区别)

**`==`** 对于基本类型和引用类型的作用效果是不同的：

- 对于基本数据类型来说，`==` 比较的是值。
- 对于引用数据类型来说，`==` 比较的是对象的内存地址。

因为 Java 只有值传递，所以，对于 == 来说，不管是比较基本数据类型，还是引用数据类型的变量，其本质比较的都是值，只是引用类型变量存的值是对象的地址。



**`equals()`** 不能用于判断基本数据类型的变量，只能用来判断两个对象是否相等。`equals()`方法存在于`Object`类中，而`Object`类是所有类的直接或间接父类，因此所有的类都有`equals()`方法。

~~~java
//Object的equals方法。
public boolean equals(Object obj) {
    return (this == obj);
}
~~~

**`equals()` 方法存在两种使用情况：**

**类没有重写 `equals()`方法**：通过`equals()`比较该类的两个对象时，等价于通过“==”比较这两个对象，使用的默认是 `Object`类`equals()`方法。

**类重写了 `equals()`方法**：一般我们都重写 `equals()`方法来比较两个对象中的属性是否相等；若它们的属性相等，则返回 true(即，认为这两个对象相等)。

~~~java
String a = new String("ab"); // a 为一个引用
String b = new String("ab"); // b为另一个引用,对象的内容一样
String aa = "ab"; // 放在常量池中
String bb = "ab"; // 从常量池中查找
System.out.println(aa == bb);// true
System.out.println(a == b);// false
System.out.println(a.equals(b));// true
System.out.println(42 == 42.0);// true
~~~

## [3. hashCode() 有什么用？](https://javaguide.cn/java/basis/java-basic-questions-02.html#hashcode-有什么用)

`hashCode()` 的作用是**获取哈希码**（`int` 整数），也称为散列码。这个哈希码的作用是确定该对象在哈希表中的索引位置。

![image-20250122152241571](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221522632.png)

## [4. 为什么要有 hashCode？](https://javaguide.cn/java/basis/java-basic-questions-02.html#为什么要有-hashcode)

当你把对象加入 `HashSet` 时，`HashSet` 会先计算对象的 `hashCode` 值来判断对象加入的位置，同时也会与其他已经加入的对象的 `hashCode` 值作比较，如果没有相符的 `hashCode`，`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashCode` 值的对象，这时会调用 `equals()` 方法来检查 `hashCode` 相等的对象是否真的相同。如果两者相同，`HashSet` 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样我们就大大减少了 `equals` 的次数，相应就大大提高了执行速度。

有了 `hashCode()` 之后，**判断元素是否在对应容器中的效率会更高**（参考添加元素进`HashSet`的过程）！

**那为什么不只提供 `hashCode()` 方法呢？**

这是因为两个对象的`hashCode` 值相等并不代表两个对象就相等。

## [5. 为什么重写 equals() 时必须重写 hashCode() 方法？](https://javaguide.cn/java/basis/java-basic-questions-02.html#为什么重写-equals-时必须重写-hashcode-方法)

因为两个相等的对象的 `hashCode` 值必须是相等。也就是说如果 `equals` 方法判断两个对象是相等的，那这两个对象的 `hashCode` 值也要相等。

如果重写 `equals()` 时没有重写 `hashCode()` 方法的话就可能会导致 `equals` 方法判断是相等的两个对象，`hashCode` 值却不相等。

# String

## [1. String、StringBuffer、StringBuilder 的区别？](https://javaguide.cn/java/basis/java-basic-questions-02.html#string、stringbuffer、stringbuilder-的区别)

**可变性**

`String` 是不可变的（后面会详细分析原因）。

`StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串，不过没有使用 `final` 和 `private` 关键字修饰，最关键的是这个 `AbstractStringBuilder` 类还提供了很多修改字符串的方法比如 `append` 方法。

**线程安全性**

`String` 中的对象是不可变的，也就可以理解为常量，**线程安全**。

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

**性能**

每次对 `String` 类型进行改变的时候，都会生成一个新的 `String` 对象，然后将指针指向新的 `String` 对象。

`StringBuffer` 每次都会对 `StringBuffer` 对象本身进行操作，而不是生成新的对象并改变对象引用。

相同情况下使用 `StringBuilder` 相比使用 `StringBuffer` 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

## [2. String 为什么是不可变的?](https://javaguide.cn/java/basis/java-basic-questions-02.html#string-为什么是不可变的)

如何保证不可变：保存字符串的数组被 `final` 修饰且为私有的，并且`String` 类没有提供/暴露修改这个字符串的方法。	

为什么要这么设计：

为了安全，字符串经常被用作参数传递。由于字符串不可变，方法内部无法修改传入的字符串内容，从而保证了数据的安全性和一致性。

在多线程环境下，如果字符串是可变的，多个线程同时访问和修改同一个字符串对象时，就需要进行额外的同步操作来保证数据的完整性，这会增加编程的复杂性和性能开销。

## [3. 字符串拼接用“+” 还是 StringBuilder?](https://javaguide.cn/java/basis/java-basic-questions-02.html#字符串拼接用-还是-stringbuilder)

Java 语言本身并不支持运算符重载，“+”和“+=”是专门为 String 类重载过的运算符，也是 Java 中仅有的两个重载过的运算符。

字符串对象通过“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象，**是创建了一个新对象**！！！ 。

不过，在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个 `StringBuilder` 以复用，会导致创建过多的 `StringBuilder` 对象**。

## [4. String#equals() 和 Object#equals() 有何区别？](https://javaguide.cn/java/basis/java-basic-questions-02.html#string-equals-和-object-equals-有何区别)

`String` 中的 `equals` 方法是被重写过的，比较的是 String 字符串的值是否相等。 `Object` 的 `equals` 方法是比较的对象的内存地址。

## [5. 字符串常量池的作用了解吗？](https://javaguide.cn/java/basis/java-basic-questions-02.html#字符串常量池的作用了解吗)

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

~~~java
// 在字符串常量池中创建字符串对象 ”ab“
// 将字符串对象 ”ab“ 的引用赋值给 aa
String aa = "ab";
// 直接返回字符串常量池中字符串对象 ”ab“，赋值给引用 bb
String bb = "ab";
System.out.println(aa==bb); // true
~~~

## [6. String s1 = new String("abc");这句话创建了几个字符串对象？](https://javaguide.cn/java/basis/java-basic-questions-02.html#string-s1-new-string-abc-这句话创建了几个字符串对象)

字符串常量池中不存在 "abc"：会创建 2 个 字符串对象。一个在字符串常量池中，由 `ldc` 指令触发创建。一个在堆中，由 `new String()` 创建，并使用常量池中的 "abc" 进行初始化。

字符串常量池中已存在 "abc"：会创建 1 个 字符串对象。该对象在堆中，由 `new String()` 创建，并使用常量池中的 "abc" 进行初始化。

## [7. String#intern 方法有什么作用?](https://javaguide.cn/java/basis/java-basic-questions-02.html#string-intern-方法有什么作用)

1. **常量池中已有相同内容的字符串对象**：如果字符串常量池中已经有一个与调用 `intern()` 方法的字符串内容相同的 `String` 对象，`intern()` 方法会直接返回常量池中该对象的引用。
2. **常量池中没有相同内容的字符串对象**：如果字符串常量池中还没有一个与调用 `intern()` 方法的字符串内容相同的对象，`intern()` 方法会将当前字符串对象的引用添加到字符串常量池中，并返回该引用。

~~~java

// s1 指向字符串常量池中的 "Java" 对象
String s1 = "Java";
// s2 也指向字符串常量池中的 "Java" 对象，和 s1 是同一个对象
String s2 = s1.intern();
// 在堆中创建一个新的 "Java" 对象，s3 指向它
String s3 = new String("Java");
// s4 指向字符串常量池中的 "Java" 对象，和 s1 是同一个对象
String s4 = s3.intern();
// s1 和 s2 指向的是同一个常量池中的对象
System.out.println(s1 == s2); // true
// s3 指向堆中的对象，s4 指向常量池中的对象，所以不同
System.out.println(s3 == s4); // false
// s1 和 s4 都指向常量池中的同一个对象
System.out.println(s1 == s4); // true
~~~

## [8. String 类型的变量和常量做“+”运算时发生了什么？](https://javaguide.cn/java/basis/java-basic-questions-02.html#string-类型的变量和常量做-运算时发生了什么)

常量折叠：	

对于 `String str3 = "str" + "ing";` 编译器会给你优化成 `String str3 = "string";` 。

并不是所有的常量都会进行折叠，只有编译器在程序编译期就可以确定值的常量才可以：

- 基本数据类型( `byte`、`boolean`、`short`、`char`、`int`、`float`、`long`、`double`)以及字符串常量。
- `final` 修饰的基本数据类型和字符串变量

~~~java
String str1 = "str";
String str2 = "ing";
String str3 = "str" + "ing";//发生了常量折叠，指向常量池的string
String str4 = str1 + str2;//创建一个新的对象，实际上是通过 StringBuilder 调用 append() 方法实现的，拼接完成之后调用 toString() 得到一个 String 对象 。
String str5 = "string";
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
~~~

~~~java
final String str1 = "str";
final String str2 = "ing";
// 下面两个表达式其实是等价的
String c = "str" + "ing";// 常量池中的对象
String d = str1 + str2; // 常量池中的对象
System.out.println(c == d);// true
~~~

# [异常](https://javaguide.cn/java/basis/java-basic-questions-03.html#异常)

## 1.基础知识

![image-20250122160652976](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501221606039.png)

## [2. Exception 和 Error 有什么区别？](https://javaguide.cn/java/basis/java-basic-questions-03.html#exception-和-error-有什么区别)

- **`Exception`** :程序本身可以处理的异常，可以通过 `catch` 来进行捕获。`Exception` 又可以分为 Checked Exception (受检查异常，必须处理) 和 Unchecked Exception (不受检查异常，可以不处理)。
- **`Error`**：`Error` 属于程序无法处理的错误 ，不建议通过`catch`捕获 。例如 Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、类定义错误（`NoClassDefFoundError`）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

## [3. Checked Exception 和 Unchecked Exception 有什么区别？](https://javaguide.cn/java/basis/java-basic-questions-03.html#checked-exception-和-unchecked-exception-有什么区别)

**Checked Exception** 即 受检查异常 ，Java 代码在编译过程中，如果受检查异常没有被 `catch`或者`throws` 关键字处理的话，就没办法通过编译。

除了`RuntimeException`及其子类以外，其他的`Exception`类及其子类都属于受检查异常 。常见的受检查异常有：IO 相关的异常、`ClassNotFoundException`、`SQLException`...。

**Unchecked Exception** 即 **不受检查异常** ，Java 代码在编译过程中 ，我们即使不处理不受检查异常也可以正常通过编译。

`RuntimeException` 及其子类都统称为非受检查异常，常见的有（建议记下来，日常开发中会经常用到）：

- `NullPointerException`(空指针错误)
- `IllegalArgumentException`(参数错误比如方法入参类型错误)
- `NumberFormatException`（字符串转换为数字格式错误，`IllegalArgumentException`的子类）
- `ArrayIndexOutOfBoundsException`（数组越界错误）
- `ClassCastException`（类型转换错误）
- `ArithmeticException`（算术错误）
- `SecurityException` （安全错误比如权限不够）
- `UnsupportedOperationException`(不支持的操作错误比如重复创建同一用户)

## [4. Throwable 类常用方法有哪些？](https://javaguide.cn/java/basis/java-basic-questions-03.html#throwable-类常用方法有哪些)

- `String getMessage()`: 返回异常发生时的详细信息
- `String toString()`: 返回异常发生时的简要描述

## [5. try-catch-finally 如何使用？](https://javaguide.cn/java/basis/java-basic-questions-03.html#try-catch-finally-如何使用)

`try`块：用于捕获异常。其后可接零个或多个 `catch` 块，如果没有 `catch` 块，则必须跟一个 `finally` 块。

`catch`块：用于处理 try 捕获到的异常。

`finally` 块：无论是否捕获或处理异常，`finally` 块里的语句都会被执行。当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行。

**不要在 finally 语句块中使用 return!** 当 try 语句和 finally 语句中都有 return 语句时，try 语句块中的 return 语句会被忽略。

## [6. finally 中的代码一定会执行吗？](https://javaguide.cn/java/basis/java-basic-questions-03.html#finally-中的代码一定会执行吗)

就比如说 finally 之前虚拟机被终止运行的话，finally 中的代码就不会被执行。

## [7. 异常使用有哪些需要注意的地方？](https://javaguide.cn/java/basis/java-basic-questions-03.html#异常使用有哪些需要注意的地方)

不要把异常定义为静态变量，因为这样会导致异常栈信息错乱。每次手动抛出异常，我们都需要手动 new 一个异常对象抛出。

抛出的异常信息一定要有意义。

建议抛出更加具体的异常比如字符串转换为数字格式错误的时候应该抛出`NumberFormatException`而不是其父类`IllegalArgumentException`。

# 泛型

## [1.什么是泛型？有什么作用？](https://javaguide.cn/java/basis/java-basic-questions-03.html#什么是泛型-有什么作用)

泛型可以让代码更加灵活和可重用

## [2. 泛型的使用方式有哪几种？](https://javaguide.cn/java/basis/java-basic-questions-03.html#泛型的使用方式有哪几种)

**泛型类**、**泛型接口**、**泛型方法**。

见JavaGuide。

## [3. 项目中哪里用到了泛型？](https://javaguide.cn/java/basis/java-basic-questions-03.html#项目中哪里用到了泛型)

- 自定义接口通用返回结果 `CommonResult<T>` 通过参数 `T` 可根据具体的返回类型动态指定结果的数据类型

# 反射

## [1. 何谓反射？](https://javaguide.cn/java/basis/java-basic-questions-03.html#何谓反射)

Java 反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的**所有属性和方法和注解**，

对于任意一个对象，都能够调用它的任意一个方法（包括私有方法）和属性，包括修改；

这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。

## [2. 反射的优缺点？](https://javaguide.cn/java/basis/java-basic-questions-03.html#反射的优缺点)

反射让我们在运行时有了分析操作类的能力的同时，也增加了安全问题，比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）。另外，反射的性能也要稍差点，不过，对于框架来说实际是影响不大的。

## [3. 反射的应用场景？](https://javaguide.cn/java/basis/java-basic-questions-03.html#反射的应用场景)

这些框架中也大量使用了动态代理，而动态代理的实现也依赖反射。

为什么你使用 Spring 的时候 ，一个`@Component`注解就声明了一个类为 Spring Bean 呢？为什么你通过一个 `@Value`注解就读取到配置文件中的值呢？究竟是怎么起作用的呢？

这涉及到，Spring 使用反射机制调用类的构造方法创建 Bean 的实例。







我们的项目底层数据库有时是用mysql，有时用oracle，需要动态地根据实际情况加载驱动类，这个时候反射就有用了，假设 com.mikechen.java.myqlConnection，com.mikechen.java.oracleConnection这两个类我们要用。

这时候我们在使用 JDBC 连接数据库时使用 Class.forName()通过反射加载数据库的驱动程序，如果是mysql则传入mysql的驱动类，而如果是oracle则传入的参数就变成另一个了。

~~~java
//  DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
Class.forName("com.mysql.cj.jdbc.Driver");
~~~

## [4. 获取 Class 对象的四种方式](https://javaguide.cn/java/basis/reflection.html#获取-class-对象的四种方式)

**1. 知道具体类的情况下可以使用：**

~~~java
Class alunbarClass = TargetObject.class;
~~~

**2. 通过 `Class.forName()`传入类的全路径获取：**

~~~java
Class alunbarClass1 = Class.forName("cn.javaguide.TargetObject");
~~~

**3. 通过对象实例`instance.getClass()`获取：**

~~~java
TargetObject o = new TargetObject();
Class alunbarClass2 = o.getClass();
~~~

**4. 通过类加载器`xxxClassLoader.loadClass()`传入类路径获取:**

通过类加载器获取 Class 对象不会进行初始化，意味着不进行包括初始化等一系列步骤，静态代码块和静态对象不会得到执行

~~~java
ClassLoader.getSystemClassLoader().loadClass("cn.javaguide.TargetObject");
~~~

## [5. 反射的一些基本操作](https://javaguide.cn/java/basis/reflection.html#反射的一些基本操作)

见javaguide。

# [注解](https://javaguide.cn/java/basis/java-basic-questions-03.html#注解)

## [1. 能讲一讲Java注解的原理吗？](https://xiaolincoding.com/interview/java.html#能讲一讲java注解的原理吗)

注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。

我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，会最终调用注解处理器AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池。

## [2. 注解的解析方法有哪几种？](https://javaguide.cn/java/basis/java-basic-questions-03.html#注解的解析方法有哪几种)

- **编译期直接扫描**：编译器在编译 Java 代码的时候扫描对应的注解并处理，比如某个方法使用`@Override` 注解，编译器在编译的时候就会检测当前的方法是否重写了父类对应的方法。
- **运行期通过反射处理**：像框架中自带的注解(比如 Spring 框架的 `@Value`、`@Component`)都是通过反射来进行处理的。

# [序列化和反序列化](https://javaguide.cn/java/basis/java-basic-questions-03.html#序列化和反序列化)

## [1. 什么是序列化?什么是反序列化?](https://javaguide.cn/java/basis/java-basic-questions-03.html#什么是序列化-什么是反序列化)

如果我们需要持久化 Java 对象比如将 Java 对象保存在文件中，或者在网络传输 Java 对象，这些场景都需要用到序列化。

- **序列化**：将数据结构或对象转换成可以存储或传输的形式，通常是二进制字节流，也可以是 JSON, XML 等文本格式
- **反序列化**：将在序列化过程中所生成的数据转换为原始数据结构或者对象的过程



序列化和反序列化常见应用场景：

对象在进行网络传输（比如远程方法调用 RPC 的时候）之前需要先被序列化，接收到序列化的对象之后需要再进行反序列化；

将对象存储到文件之前需要进行序列化，将对象从文件中读取出来需要进行反序列化；

将对象存储到数据库（如 **Redis**）之前需要用到序列化，将对象从缓存数据库中读取出来需要反序列化；

将对象存储到内存之前需要进行序列化，从内存中读取出来之后需要进行反序列化



序列化协议属于 TCP/IP 协议**应用层**的一部分。

## [2. 常见序列化协议有哪些？](https://javaguide.cn/java/basis/java-basic-questions-03.html#常见序列化协议有哪些)

Hessian、**Kryo**、Protobuf、ProtoStuff，这些都是基于二进制的序列化协议。

**像 JSON 和 XML 这种属于文本类序列化方式。虽然可读性比较好，但是性能较差，一般不会选择。**

**不推荐用JDK自带的序列化。**

JDK 自带的序列化，只需实现 `java.io.Serializable`接口即可。

**不支持跨语言调用** : 如果调用的是其他语言开发的服务的时候就不支持了。

**性能差**：相比于其他序列化框架性能更低，主要原因是序列化之后的字节数组体积较大，导致传输成本加大。

**存在安全问题**：序列化和反序列化本身并不存在问题。但当输入的反序列化的数据可被用户控制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期的对象，在此过程中执行构造的任意代码。相关阅读：

# 代理模式

## [1. 代理模式](https://javaguide.cn/java/basis/proxy.html#_1-代理模式)

**代理模式的主要作用是扩展目标对象的功能，比如说在目标对象的某个方法执行前后你可以增加一些自定义的操作。**

![image-20250123115619299](https://acwhr.oss-cn-beijing.aliyuncs.com/typora_images/202501231156406.png)

## [2. 静态代理](https://javaguide.cn/java/basis/proxy.html#_2-静态代理)

1. 定义一个接口及其实现类；
2. 创建一个代理类同样实现这个接口
3. 将目标对象注入进代理类，然后在代理类的对应方法调用目标类中的对应方法。这样的话，我们就可以通过代理类屏蔽对目标对象的访问，并且可以在目标方法执行前后做一些自己想做的事情。

**非常不灵活（比如接口一旦新增加方法，目标对象和代理对象都要进行修改）且麻烦(需要对每个目标类都单独写一个代理类）。** 实际应用场景非常非常少，日常开发几乎看不到使用静态代理的场景。

## [3. 动态代理](https://javaguide.cn/java/basis/proxy.html#_3-动态代理)

Spring AOP、RPC 框架应该是两个不得不提的，它们的实现都依赖了动态代理。

**动态代理在我们日常开发中使用的相对较少，但是在框架中的几乎是必用的一门技术。学会了动态代理之后，对于我们理解和学习各种框架的原理也非常有帮助。**

### [3.1 JDK 动态代理机制](https://javaguide.cn/java/basis/proxy.html#_3-1-jdk-动态代理机制)

**在 Java 动态代理机制中 `InvocationHandler` 接口和 `Proxy` 类是核心。**

`Proxy` 类中使用频率最高的方法是：`newProxyInstance()` ，这个方法主要用来生成一个代理对象。

1. 定义一个接口及其实现类；
2. 自定义类实现`InvocationHandler`并重写`invoke`方法，在 `invoke` 方法中我们会调用原生方法（被代理类的方法）并自定义一些处理逻辑；
3. 通过 `Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)` 方法创建代理对象；

### [3.2. CGLIB 动态代理机制](https://javaguide.cn/java/basis/proxy.html#_3-2-cglib-动态代理机制)

**JDK 动态代理有一个最致命的问题是其只能代理实现了接口的类。**

1. 定义一个类；
2. 自定义 `MethodInterceptor` 并重写 `intercept` 方法，`intercept` 用于拦截增强被代理类的方法，和 JDK 动态代理中的 `invoke` 方法类似；
3. 通过 `Enhancer` 类的 `create()`创建代理类；

# IO

## [1. Java IO 流了解吗？](https://javaguide.cn/java/basis/java-basic-questions-03.html#java-io-流了解吗)

数据输入到计算机内存的过程即输入，反之输出到外部存储（比如数据库，文件，远程主机）的过程即输出。

- `InputStream`/`Reader`: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- `OutputStream`/`Writer`: 所有输出流的基类，前者是字节输出流，后者是字符输出流。

## [2. I/O 流为什么要分为字节流和字符流呢?](https://javaguide.cn/java/basis/java-basic-questions-03.html#i-o-流为什么要分为字节流和字符流呢)

**不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？**

- 字符流是由 Java 虚拟机将字节转换得到的，这个过程还算是比较耗时；
- 如果我们不知道编码类型的话，使用字节流的过程中很容易出现乱码问题。





- **字节流**：适用于处理任何类型的文件，特别是二进制文件，像图片、音频、视频等。常见的字节流类有 `FileInputStream`、`FileOutputStream`、`BufferedInputStream` 和 `BufferedOutputStream` 等。
- **字符流**：主要用于处理文本文件，如 `.txt`、`.java`、`.xml` 等。常见的字符流类有 `FileReader`、`FileWriter`、`BufferedReader` 和 `BufferedWriter` 等。
