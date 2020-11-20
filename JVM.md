---
title: JVM
date: 2020-06-18 09:56:48
tags:
    - Java
categories:
    - JAVA
---

# 一、前言

## 1.1 字节码

* 平时说的Java字节码，指得是用Java语言编译成的字节码。准确的说任何能在JVM平台上执行的执行的字节码格式都是一样的。所有应该统称为：$\color{red}{JVM字节码}$

* 不同的编译器，可以编译出相同的字节码文件，字节码文件也可也在不同的JVM上运行
* Java虚拟机与Java语言并没有必然的联系，它只与特定的二进制文件格式-Class文件格式所关联，Class文件中包含了Java虚拟机指令集（或者称为字节码、Bytecodes）和符号表，还有一些其他辅助信息

## 1.2 多语言混合编程

* Java平台上的多语言混合编程正成为主流，通过特定领域的语言去解决特定领域的问题是当前软件开发应对日趋复杂的项目需求的一个方向

## 1.3 虚拟机

* 所谓虚拟机就是一台虚拟的计算机，它是一款软件，同来执行一系列虚拟计算机指令。大体上，虚拟机可以分为系统虚拟机和程序虚拟机

### 1.3.1 java虚拟机

* Java虚拟机是一台执行Java字节码的虚拟计算机，它拥有独立的运行机制，其运行的Java字节码也未必由Java语言编译而成
* JVM平台的各种语言可以共享Java虚拟机带来的跨平台性、优秀的垃圾回收器，以及可靠的即时编译器
* Java技术的核心就是Java虚拟机，因为所有的Java程序都运行在Java虚拟机内部
* Java虚拟机就是二进制字节码的运行环境，负责装载字节码到其内部，解释/编译为对应平台上的机器指令执行。每一条Java指令，Java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数





# 二、JVM架构模型

* Java编译器输入的指令流基本上是一种基于栈的==指令集架构==，另外一种指令集架构则是基于==寄存器的指令集架构==

* 两种架构之间的区别：

    1. 基于栈式结构的特点：
        * 设计和实现更简单，适用于资源受限的系统；
        * 避开了寄存器的分配难题：使用零地址指令方式分配
        * 指令流中的指令大部分是零地址指令，其执行过程依赖于操作栈。指令集更小，编译器容易实现
        * 不需要硬件支持，可移植性更好，更好实现跨平台

    2. 基于寄存器架构的特点：
        * 典型的应用是x86的二进制指令集：比如传统的PC以及Android的Davlik虚拟机
        * 指令集架构则完全依赖硬件，可移植性差
        * 性能优秀和执行更高效
        * 花费更少的指令取完成一项操作
        * 在大部分情况下，基于寄存器架构的指令集往往都以一地址指令，二地址指令和三地址指令为主，而基于栈架构的指令集却是以零地址指令为主



# 三、JVM发展历程

## 3.1 Sun Classic VM

* 早在1996年Java1.0版本的时候，Sun公司发布了一款名为Sun Classic VM的Java虚拟机，它同时是世界上第一款商用Java虚拟机，JDK1.4时完全被淘汰
* 这款虚拟机内部只提供解释器
* 如果使用JIT编译器，就需要进行外挂。但是一旦使用了JIT编译器，JIT就会接管虚拟机的执行系统。解释器就不再工作。解释器和编译器不能配合工作

## 3.2 Exact VM

* 为了解决上一个虚拟机问题，jdk1.2时，sun提供了此虚拟机	
* Exact Memory Mangement：准确式内存管理
    * 也可以叫Non-Conservative/Accurate Memory Management
    * 虚拟机可以知道内存中某个位置的数据具体是什么类型
* 具备现代高性能虚拟机的雏形
    * 热点探测
    * 编译器与解释器混合工作模式
* 只在Solaris平台短暂使用

## 3.3 HotSpot VM

* HotSpot历史：
    * 最初由一家名为“Longview Technologies”的小公司设计
    * 1997年，此公司被Sun收购；2009年，Sun公司被甲骨文收购
    * JDK1.3时，HotSpot VM成为默认虚拟机
* 目前HotSpot占有绝对的市场地位：
    * 不管是现在仍在广泛使用的JDK6， 还是使用比例较多的JDK8中，默认的虚拟机都是HotSpot
    * Sun/Oracle JDK和OpenJDK的默认虚拟机。相关机制也主要是指HotSpot的GC机制（比如其他两个虚拟机都没有方法区的概念）
* 名称中的HotSpot指的就是它的热点代码探测技术
    * 通过计数器找到最具编译价值的代码，触发即时编译或栈上替换
    * 通过编译器与解释器协同工作，在最优化的程序响应时间与最佳执行性能中取得平衡

## 3.4 BEA的JRockit

* 专注于服务器端应用：
    * 它可以不太关注程序启动速度，因此JRockit内部不包含解析器实现，全部代码都靠即时编译器编译后运行
* 大量的行业基准测试显示，JRockit JVM是世界上最快的JVM
* 优势：全面的Java运行时解决方案组合
    * JRockit面向延迟敏感性应用的解决方案JRockit Real Time提供以毫秒或微秒级的JVM响应时间
    * MissionControl服务套件，它是一组以极低的开销来监控、管理和分析生产环境中的应用程序的工具

## 3.5 IBM的J9

* 全称：IBM Technology for Java Vitual Machine， 简称IT4J，内部代号J9
* 市场定位与HotSpot接近，服务器端，桌面应用、嵌入式等多用途VM
* 广泛用于IBM的各种Java产品



# 四、类加载子系统

* 类加载器子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识
* ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定
* 加载的类信息存放于一块称为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）

![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%AD%90%E7%B3%BB%E7%BB%9F.png)

## 4.1 类的加载过程

![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/JVM.png)

### 4.1.1 加载

1. 通过一个类的全限定名获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. ==在内存中生成一个代表这个类的java.lang.Class对象==，作为方法区这个类的各种数据的访问入口
4. 补充：加载.class文件的方式
    * 从本地系统中直接加载
    * 通过网络获取，典型场景：Web Applet
    * 从zip压缩包中读取，成为日后jar、war格式的基础
    * 运行时计算生成，使用最多的是：动态代理技术
    * 由其他文件生成，典型场景：JSP应用
    * 从专有数据库中提取.class文件，比较少见
    * 从加密文件中获取，典型的防止Class文件被反编译的保护措施

### 4.1.2 链接

1. 验证
    * 目的在于确保class文件的字节流中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全
    * 主要包括四种验证，文件格式验证，元数据验证，字节码验证，符号引用验证
2. 准备
    * 为类变量分配内存并且设置该变量的默认初始值，即零值
    * 这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显式初始化
    * 这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中
3. 解析
    * 将常量池内的符号引用转换为直接引用的过程
    * 事实上，解析操作往往会伴随着JVM在执行完初始化之后再执行
    * 符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的Class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄
    * 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info等

### 4.1.3 初始化

1. 初始化阶段就是执行类构造器方法`<clinit>()`的过程
2. 此方法不需定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来
3. 构造器方法中指令按语句在源文件中出现的顺序执行
4. `<clinit>()`不同于类的构造器（关联：构造器是虚拟机视角下的`<init>()`）
5. 若该类具有父类，JVM会保证子类的`<clinit>()`执行前，父类的`<clinit>()`已经执行完毕
6. 虚拟机必须保证一个类的`<clinit>()`方法在多线程下被同步加锁

## 4.2 类加载器的分类

* JVM支持两种类型的类加载器，分别为==引导类加载器（Bootstrap ClassLoader）==和==自定义类加载器（User-Defined ClassLoader）==
* 从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是Java虚拟机规范却没有这么定义，而是将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器
* 无论类加载器的类型如何划分，在程序中最常见的类加载器始终只有3个

### 4.2.1 启动类加载器

* 启动类加载器（引导类加载器，Bootstrap ClassLoader）这个类加载使用C/C++语言实现的，嵌套在JVM内部
* 它用来加载Java的核心库（JAVA_HOME/jre/lib/rt.jar、resources.jar或sun.boot.class.path路径下的内容），用于提供JVM自身需要的类
* 并不继承自java.lang.ClassLoader，没有父加载器
* 加载扩展类和应用程序类加载器，并指定为他们的父类加载器
* 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类

### 4.2.2 扩展类加载器

* 扩展类加载器（Extension ClassLoader）由Java语言编写，由sun.misc.Launcher$ExtClassLoader实现，派生于ClassLoader类
* 父类加载器为启动类加载器
* 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录（扩展目录）下加载类库。如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载

### 4.2.3 应用程序类加载器

* 应用程序类加载器（系统类加载器 AppClassLoader）java语言编写，由sum.misc.Launcher$AppClassLoader实现，派生于ClassLoader类
* 父类加载器为扩展类加载器
* 它负责加载环境变量classpath或系统属性java.class.path指定路径下的类库
* ==该类加载是程序中默认的类加载器==，一般来说，Java应用的类都是由它来完成加载
* 通过ClassLoader#getSystemClassLoader()方法可以获取到该类加载器

## 4.3 关于ClassLoader

* ClassLoader类是一个抽象类，其后所有的类加载器都继承自ClassLoader（不包括启动类加载器）

    | 方法名称                                             | 描述                                                         |
    | ---------------------------------------------------- | ------------------------------------------------------------ |
    | getParent()                                          | 返回该类加载器的超类加载器                                   |
    | loadClass(String name)                               | 加载名称为name的类，返回结果为java.lang.Class类的实例        |
    | findClass(String name)                               | 查找名称为name的类，返回结果为java.lang.Class类的实例        |
    | findLoadedClass(String name)                         | 查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例 |
    | defineClass(String name, byte[] b, int off, int len) | 把字节数组b中的内容转换为以恶搞Java类，返回结果为java.lang.Class类的实例 |
    | resolveClass(Class<?> c)                             | 连接指定的一个Java类                                         |

* 获取ClassLoader的途径

    * 方式一：获取当前类的ClassLoader
        * `clazz.getClassLoader()`
    * 方式二：获取当前线程上下文的ClassLoader
        * `Thread.currentThread().getContextClassLoader()`
    * 方式三：获取系统的ClassLoader
        * `ClassLoader.getSystemClassLoader()`
    * 方式四：获取调用者的ClassLoader
        * `DriverManager.getCallerClassLoader()`



## 4.4 双亲委派机制

* Java虚拟机堆class文件采用的是==按需加载==的方式，也就是说当需要使用该类时才会将它的class文件加载到内存生成class对象。而且加载某个类的class文件时，Java虚拟机采用的时==双亲委派机制==，即把请求交给父类处理，它是一种任务委派模式

### 4.4.1 工作原理

1. 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行
2. 如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终到达顶层的启动类加载器
3. 如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式

* 优势：
    * 避免类的重复加载
    * 保护程序安全，防止核心API被随意篡改

## 4.5 其余内容

* 在JVM中表示两个class对象是否为同一个类存在两个必要条件：
    * 类的完整类名必须一致，包括包名
    * 加载这个类的ClassLoader（值classLoader实例对象）必须相同

### 4.5.1 对类加载器的引用：

* JVM必须知道一个类型是由启动加载器加载的还是由用户类加载器加载的。如果一个类型是由用户类加载器加载的，那么JVM会==将这个类加载器的一个引用作为类型信息的一部分保存在方法区中==。当解析一个类型到另一个类型的引用的时候，JVM需要保证这两个类型的类加载器是相同的

### 4.5.2 类的主动使用和被动使用

* Java程序对类的使用分为：主动使用和被动使用
* 主动使用又分为七种情况：
    1. 创建类的实例
    2. 访问某个类或接口的静态变量，或者对该静态变量赋值
    3. 调用类的静态方法
    4. 反射
    5. 初始化一个类的子类
    6. Java虚拟机启动时被标明为启动类的类
    7. JDK7开始提供的动态语言支持
* 除了以上其中情况，其他使用Java类的方式都被看作是对类的被动使用，都不会导致类的初始化

# 五、运行时数据区概述及线程

* 内存是非常重要的系统资源，是硬盘和CPU的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。JVM内存布局规定了Java在运行过程中内存申请、分配、管理的策略，保证了JVM的高效稳定运行。==不同的JVM对于内存的划分方式和管理机制存在着部分差异==。

<img src="https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.png" style="zoom:150%;" />

* Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程对应的数据区域会随着线程开始和结束而创建和销毁

>每个线程：独立包括程序计数器、栈、本地栈
>
>线程间共享：堆、堆外内存（永久代或元空间、代码缓存）
>
>灰色为单独线程私有的，红色的为多个线程共享的 

![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/%E6%A0%88.png)

## 5.1 线程

* 线程是一个程序里的运行单元。JVM允许一个应用有多个线程并行的执行
* 在Hotspot JVM里，每个线程都与操作系统的本地线程直接映射
    * 当一个Java线程准备好执行以后，此时一个操作系统的本地线程也同时创建。Java线程执行终止后，本地线程也会回收
* 操作系统负责所有线程的安排调度到任何一个可用的CPU上。一旦本地线程初始化成功，他就会调用到Java线程中的run()方法

* 如果使用调试工具，都能看到在后台有许多线程在运行。这些后台线程不包括调用public static main的main线程以及所有这个main线程自己创建的线程
* 这些主要的后台系统线程在Hotspot JVM里主要是以下几个：
    * 虚拟机线程：这种线程的操作是需要JVM达到安全点才会出现，这些操作必须在不同的线程中发生的原因是他们都需要JVM达到安全点，这样堆才不会变化，这种线程的执行类型包括“stop-the-world”的垃圾收集；线程栈手机；线程挂起以及偏向锁撤销 
    * 周期任务线程：这种线程是时间周期事件的体现（比如中断）；他们一般用于周期性操作的调度执行
    * GC线程：这种线程对在JVM里不同种类的垃圾收集行为提供了支持
    * 编译线程：这种线程在运行时会将字节码编译成到本地代码
    * 信号调度线程：这种线程接受信号并发送给JVM；它在内部通过调用适当的方法进行处理

## 5.2 程序计数器

* JVM中的程序计数寄存器（Program Counter Register）中，Register的命名源于CPU的寄存器，寄存器存储指令相关的现场信息。CPU只有把数据装载到寄存器才能够运行

* 作用：PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码。由执行引擎读取下一条指令

    ![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/PCR.png)

* 它是一块很小的内存空间，几乎可以忽略不记，也是运行速度最快的存储区域
* 在JVM规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致
* 任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行native方法，则是未指定值（undefined）

* 它是程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成
* 字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令
* 它是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域

### 5.2.1 两个常见问题

1. 使用PC寄存器存储字节码指令地址有什么用？为什么使用PC寄存器记录当前线程的执行地址？

因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行

JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令

2. PC寄存器为什么会被设定为线程私有

所谓的多线程在一个特定的时间端内只会执行其中某一个线程的方法，CPU会不停地做任务切换，这样必然导致经常中断或恢复，如何保证分毫无差呢？为了能够准确的记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况

## 5.3 虚拟机栈

### 5.3.1 概述

* Java虚拟机栈，早期也叫Java栈。每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧，对应一次次的Java方法调用。是线程私有的
    * 作用：主管Java程序的运行，它保存方法的局部变量（8种基本数据类型、对象的引用地址）、部分结果，并参与方法的调用和返回
* 栈的特点：
    * 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器
    * JVM直接对Java栈的操作只有两个（$\color{red}{进栈和出栈}$）：
        1. 每个方法执行，伴随着进栈
        2. 执行结束后的出栈工作
    * 对于栈来说$\color{red}{不存在垃圾回收}$问题

* 栈中可能出现的异常：Java虚拟机规范允许Java栈的大小是动态的或者是固定不变的
    * 如果采用固定大小的Java虚拟机栈，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过Java虚拟机栈允许的最大容量，Java虚拟机栈将会抛出一个$\color{red}{StackOverflowError}$异常
    * 如果Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那Java虚拟机将会抛出一个$\color{red}{OutOfMemory}$异常
* 设置栈内存大小：
    * 使用参数-Xss 选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度

### 5.3.2 栈的存储单位

* $\color{blue}{每个线程都有自己的栈}$，栈中的数据都是以栈帧（Stack Frame）的格式存在
* 在这个线程上正在执行的每个方法都各自对应一个栈帧
* 栈帧是一个内存区块，是一个数据集，维系着方法执行过程种的各种数据信息

* JVM直接对Java栈的操作只有两个，就是对栈帧的$\color{red}{压栈和出栈}$，遵循$\color{blue}{先进后出}$的原则
* 在一条活动线程种，一个时间点上，只会有一个活动的栈帧。即只有当前正在执行的方法的栈帧（栈顶栈帧）是有效的，这个栈帧被称为$\color{red}{当前栈帧}$，与当前栈帧相对应的方法就是$\color{red}{当前方法}$，定义这个方法的类就是$\color{red}{当前类}$
* 执行引擎运行的所有字节码指令只针对当前栈帧进行操作
* 如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，称为新的当前帧。

### 5.3.3 栈帧的内部结构

每个栈帧中存储着：

* 局部变量表
* 操作数栈
* 动态链接（或指向运行时常量池的方法引用）
* 方法返回地址
* 一些附加信息

#### 局部变量表

* 局部变量表也被称之为局部变量数组或本地变量表
* $\color{red}{定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量}$这些数据类型包括各类基本数据类型、对象引用，以及returnAddress类型
* 由于局部变量表是建立在线程的栈上，是线程的私有数据，因此$\color{red}{不存在数据安全问题}$
* $\color{red}{局部变量表所需的容量大小是在编译器确定下来的}$，并保存在方法的Code属性的maximum local variables数据项中。在方法运行期间是不会改变局部变量表的大小的
* 关于Slot：
    * 参数值的存放总是在局部变量数组的index0开始，到数组长度-1的索引结束
    * 局部变量表，最基本的存储单元是Slot（变量槽）
    * 局部变量表中存放编译器可知的各种基本数据类型（8种），引用类型，returnAddress类型
    * 在局部变量表里，$\color{red}{32位以内的类型只占用一个slot（包括returnAddress类型），64位的类型（long和double)占用两个slot}$
* ![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/slot1.png)
* JVM会为局部变量表中的每一个slot都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值
* 当一个实例方法被调用的时候，它的方法参数和方法体内部定义局部变量将会按照顺序被复制到局部变量表中的每一个slot上
* 如果需要访问局部变量表中一个64bit的局部变量值时，只需要使用前一个索引即可
* 如果当前帧是由构造方法或者实例方法创建的，那么$\color{red}{该对象引用this将会存放在index为0的slot处}$，其余的参数按照参数表顺序继续排列

#### Slot的重复利用

* $\color{red}{栈帧中的局部变量表中的槽位是可以重用的}$，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变量就很有可能会复用过期局部变量的槽位，从而达到$\color{red}{节省资源的目的}$。

* ```java
    public class Test {
        public static void main(String[] args) {
            int a = 1;
            {
                int b = 2;
            }
            int c = 3;
            System.out.println("test...");
        }
    }
    ```

![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/slot%E9%87%8D%E7%94%A8.png)



#### 操作数栈

* 每一个独立的栈帧中除了包含局部变量表以外，还包含一个==后进先出==的操作数栈，也可以称之为表达式栈
* 操作数栈，在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据，即入栈/出栈
    * 某些字节码指令将值压入操作数栈，其余的字节码指令将操作数取出栈。使用它们后再把结果压入栈
    * 比如：执行复制、交换、求和等操作
* 作用和功能：
    * 主要用于$\color{red}{保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间}$
    * 操作数栈是JVM执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧也会随之被创建出来，$\color{red}{这个方法的操作数栈是空的}$
    * 每一个操作数栈都会拥有一个明确的栈深度用于存储数值，其所需的最大深度在编译器就定义好了，保存在方法的Code属性中，为max_stack的值
    * 栈中的任何一个元素都是可以为任意的Java数据类型
        * 32bit的类型占用一个栈单位深度
        * 64bit的类型占用两个栈单位深度
    * 操作数栈$\color{red}{并非采用访问索引的方式来进行数据访问}$的，而是只能通过标准的入栈和出栈操作来完成一次数据访问
    * $\color{red}{如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中}$，并更新PC寄存器中下一条需要执行的字节码指令
    * Java虚拟机的$\color{red}{解释引擎是基于栈的执行引擎}$，其中的栈指的就是操作数栈

代码追踪演示：

* ``` java
    public class Test {
        public static void main(String[] args) {
            // byte、short、char、boolean：都以int型来保存
            int i = 5;
            byte j = 13;
            int k = i + 5;
        }
    }
    ```

* ![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/%E6%93%8D%E4%BD%9C%E6%95%B0%E6%A0%88%E4%BB%A3%E7%A0%81%E6%BC%94%E7%A4%BA1.png)

#### 栈顶缓存技术

* 由于操作数栈是存储在内存中的，因此频繁地执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM的设计者提出了栈顶缓存（To， Top-of-Stack Cashing）技术，将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率

#### 动态链接

* 每一个栈帧内部包含了一个指向$\color{blue}{运行时常量池}$中$\color{red}{该栈帧所属方法的引用}$。包含这个引用的目的就是为了支持当前方法的代码能够实现$\color{red}{动态链接}$
* 在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用保存在class文件的常量池里
* 常量池的作用就是为了提供一些符号和常量，便于指令的识别

#### 方法的调用

* 在JVM中，将符号引用转换为调用方法的直接引用与方法的绑定机制相关。
* 静态链接：
    * 当一个字节码文件被装载进JVM内部时，如果被调用的目标方法在编译器可知，且运行期保持不变时，这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接
* 动态链接：
    * 如果被调用的方法在编译器无法被确定下来，也就是说，只能够在程序运行期间将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接
* 方法的绑定机制：早期绑定和晚期绑定。绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这只会发生一次
* 早期绑定：
    * 早期绑定就是指被调用的$\color{blue}{目标方法如果在编译器可知，且运行期保持不变时}$，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此就可以使用静态链接的方式将符号引用转换为直接引用
* 晚期绑定：
    * 如果$\color{blue}{被调用的方法在编译器无法被确定下来，只能够在程序运行期间根据实际的类型绑定相关的方法，这种绑定方式也就被称之为晚期绑定}$

#### 方法返回地址

* 存放调用该方法的pc寄存器的值
* 一个方法的结束有两种方式
    * 正常执行完成
    * 出现未处理的异常，非正式退出
* 无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。方法正常退出时，调用者的pc计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址。而通过异常退出的，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息

### 5.3.4 静态变量与局部变量的对比

* 参数表分配完毕之后，再根据方法体内定义的变量的顺序和作用域分配
* 类变量表有两次初始化的机会，第一次是在“$\color{red}{准备阶段}$”，执行系统初始化，对类变量设置零值，另一次则是在“$\color{red}{初始化}$”阶段，赋予程序员在代码中定义的初始值
* 和类变量初始化不同的是，局部变量表不存在系统初始化的过程，这意味着一旦定义了局部变量则必须人为的初始化，否则无法使用

``` java
public void test() {
    int i;
    System.out.println(i);
}
// 这样的代码是错误的，没有复制不能够使用
```

### 5.3.5 补充说明

* 在栈帧中，与性能调优关系最为密切的部分就是局部变量表，在方法执行时，虚拟机使用局部变量表完成方法的传递
* 局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收



## 5.4 虚拟机栈的五道面试题

1. 举例栈溢出的情况（StackOverflowError）
    1. 通过-Xss设置栈的大小
2. 调整栈大小，就能保证不出现溢出吗
    1. 不能保证
3. 分配的栈内存越大越好吗
    1. 不
4. 垃圾回收是否会涉及到虚拟机栈
    1. 不会涉及
5. 方法中定义的局部变量是否线程安全
    1. 具体问题具体分析

## 5.5 本地方法栈

* Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用
* 本地方法栈也是线程私有的
* 允许被实现固定或者可动态扩展的内存大小
    * 如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java虚拟机将会抛出一个StackOverflowError异常
    * 如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么Java虚拟机将会抛出一个OutOfMemory异常

# 六、本地方法库

## 6.1 本地方法接口

* 本地方法
    * 简单地将，一个Native Method就是一个Java调用非Java代码的接口。一个Native Method是这样一个Java方法：该方法的实现由非Java语言实现，比如C。这个特征并非Java所持有，很多其他的编程语言都有这一机制



# 七、堆

## 7.1 堆的核心概述

* 一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域。
* Java堆区在JVM启动的时候即被创建，其空间大小也被确定了。是JVM管理的最大一块内存空间。（堆内存的大小是可以调节的）
* 《Java虚拟机规范》规定，堆可以处于物理上不连续的内存空间中，但在逻辑上它应该被视为连续的
* 所有的线程共享Java堆，还可以划分为线程私有的缓冲区（Thread Local Allocation Buffer， TLAB）



## 7.2 设置内存大小与OOM

堆内存演示代码

``` java
public class HeapDemo {
    public static void main(String[] args) {
        System.out.println("start");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

编译后添加启动时参数

```
-Xms10m -Xms10m
```

用`Java VisaulVM`查看

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E5%A0%86-JavaVisualVM%E6%9F%A5%E7%9C%8B%E5%A0%86%E5%86%85%E5%AD%98.png)

* 《Java虚拟机规范》中对Java堆的描述是：所有对象实例以及数组都应当在运行时分配在堆上
* 数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置
* 在方法结束后，堆中的对象不会马上被移出，仅仅在垃圾回收的时候才会被移除
* 堆是GC执行垃圾回收的重点区域



### 7.2.1 内存细分

现代垃圾回收器大部分都基于分代收集理论设计，堆空间细分为：

1. Java 7及之前，堆内存在逻辑上分为三部分：新生区+养老区+永久区，新生区又被划分为伊甸园和幸存者区
2. Java 8及其之后，堆内存在逻辑上分为三部分：新生区+养老区+元空间

### 7.2.2 OutOfMemory举例

* 设置JVM最大堆内存为600m`-Xms600m -Xmx600m`

``` java
public class OutOfMemory {
    public static void main(String[] args) {
        List<Picture> list = new ArrayList<>();
        while (true) {
            try {
                Thread.sleep(20);
            } catch (Exception e) {
                e.printStackTrace();
            }
            list.add(new Picture(new Random().nextInt(1024 * 1024)));
        }
    }
}

class Picture {
    private byte[] pixels;

    public Picture(int length) {
        this.pixels = new byte[length];
    }
}
```

* 运行结果

``` 
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.qiancijun.jvm.Picture.<init>(OutOfMemory.java:25)
	at com.qiancijun.jvm.OutOfMemory.main(OutOfMemory.java:16)
```

* Picture会不断的被初始化，存储在堆空间中





## 7.3 年轻代与老年代

* 存储在JVM中的Java对象可以被划分为两类：
    * 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速
    * 另一类对象的生命周期非常长，在某些极端的情况下还能够与JVM的生命周期保持一致
* Java堆区进一步细分的话，可以划分为年轻代（YoungGen）和老年代（OldGen）
* 其中年轻代又可以划分为Eden空间、Survivor0空间和Survivor1空间（有时候也叫from区、to区）

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E5%A0%86-%E5%B9%B4%E8%BD%BB%E4%BB%A3%E4%B8%8E%E8%80%81%E5%B9%B4%E4%BB%A3.jpg.png)



* 配置新生代与老年代在堆结构的占比。
    * 默认`-XX:NewRatio=2`表示新生代占1，老年代占2，新生代占整个堆的1/3
    * 修改为`-XX:NewRatio=4`表示新生代占1，老年代占4，新生代占整个堆的1/5

> 这参数在开发中一般不会调

* 在HotSpot中，Eden空间和另外两个Survivor空间缺省所占的比例是8:1:1
* 开发人员可以通过选项`-XX:SurvivorRatio`调整这个空间比例。比如`-XX:SurvivorRation=8`
* $\color{red}{几乎所有的}$Java对象都是在Eden区被new出来的
* 绝大部分的Java对象的销毁都在新生代进行
    * IBM公司的研究表明，新生代中80%的对象都是“朝生夕死”的
* 使用选项`-Xmn`设置新生代最大内存大小（一般使用默认值）

## 7.4 对象分配过程

为新对象分配内存是一件非常严谨和复杂的任务，JVM的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还要考虑GC执行完内存回收后是否会在内存空间中产生碎片。

1. new的对象先放Eden区。此区有大小限制
2. 当Eden区的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对Eden区进行垃圾回收（Minor GC），将Eden区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到Eden区
3. 然后将Eden区中剩余的对象移动到幸存者0区。
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区，如果没有回收，就会放到幸存者1区
5. 如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区
6. 默认15次，对象才会被移入养老区（OldGen）
7. 当老年代内存不足时，再次触发GC，进行老年代区的内存清理
8. 若老年代执行了GC之后，依然无法进行存储对象，就会产生OOM异常(java.lang.OutOfMemoryError)
    * 可以设置参数：`-XX:MaxTenuringThreshold=<N>`

> 1. Eden区的数据会首先往to区放，同时把from区的数据也做GC看是否需要，如果还是需要也往to区放，此时的from区会转变成下一次GC的to区。
> 2. 只有当Eden区满的时候才会触发YGC，S0区满的时候不会触发YGC

* 总结：
    * 针对幸存者S0,S1区的总结：复制之后有交换，谁空谁是to
    * 关于垃圾回收：频繁在年轻代收集，很少在老年代收集，几乎不在元空间收集

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E5%A0%86-%E5%88%86%E9%85%8D%E5%AF%B9%E8%B1%A1%E5%86%85%E5%AD%98%E6%B5%81%E7%A8%8B.jpg.png)



## 7.5 MinorGC、MajorGC与FullGC

JVM在进行GC时，并非每次都对内存区域（年轻代、老年代、方法区）一起回收，大部分时候回收的都是指年轻代

针对HotSpot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（Partial GC），一种是整堆收集（Full GC）

* 部分收集：不是完整收集整个Java堆的垃圾收集。其中又分为：
    * 年轻代收集（Minor GC / Young GC）：只是年轻代的垃圾收集
    * 老年代收集（Major GC / Old GC）：只是老年代的垃圾收集
        * 目前，只有CMS GC会有单独收集老年代的行为
        * 很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收
    * 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集
        * 目前只有G1 GC会有这种行为
    * 整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集
* 年轻代GC（Minor GC）触发机制：
    * 当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是Eden区满，Survivor满不会引发GC。（每次Minor GC会清理年轻代的内存）
    * 因为Java对象$\color{red}{大多都具备朝生夕灭}$的特性，所以Minor GC非常频繁，一般回收速度也比较快
    * Minor GC会引发STW（Stop The World），暂停其他用户的线程，等垃圾回收结束，用户线程才恢复运行
* 老年代GC（Major GC / Full GC）触发机制：
    * 指发生在老年代的GC，对象从老年代消失时，就说“Major GC” 或 “Full GC”发生了
    * 出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就又有直接进行Major GC的策略选择过程）
        * 也就是在老年代空间不足时，会先尝试触发Minor GC。如果之后空间还不足，则触发Major GC
    * Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长
    * 如果Major GC后，内存还不足，就报OOM
* Full GC触发机制
    * 调用`System.gc()`时，系统建议执行Full GC，但是不必然执行
    * 老年代空间不足
    * 方法区空间不足
    * 通过Minor GC后进入老年代的平均大小大于老年代的可用空间
    * 有Eden区、Survivor space0区向Survivor space1区复制时，对象大小与Survivor space1可用内存，则把该对象转到老年代，且老年代的可用内存小于该对象大小

> 说明：Full GC是开发或调优中尽量要避免的。这样暂时时间会短一些



## 7.6 堆空间分代思想

不同对象的生命周期不同。70%-99%的对象都是临时对象

* 新生代：有Eden、两块大小相同的Survivor（又称from/to，s0/s1）构成，to总为空
* 老年代：存放新生代中经历多次GC仍然存活的对象

不分代完全可以，分代的唯一理由就是优化GC性能。如果没有分代，那所有的对象都在一块。GC的时候要找哪些对象没用，这样就会对堆的所有区域进行扫描。而很多对象都是朝生夕死的，如果分代的话，把新创建的对象放到某一地方，当GC的时候先把这块存储对象的区域进行回收，这样就会腾出很大的空间



## 7.7 内存分配总结

如果对象在Eden区出生并经过第一次MinorGC后仍然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并将对象年龄设为1。对象在Survivor区中每熬过一次MinorGC，年龄就增加1，当它的年龄增加到一定程度（默认15，其实每个JVM、每个GC都有所不同）时，就会被晋升到老年代中。

对象晋升老年代的年龄阈值，可以通过选项`-XX:MaxTenuringThreshold`来设置



### 7.7.1 对象提升原则

针对不同年龄段的对象分配原则如下所示：

* 优先分配到Eden
* 大对象直接分配到老年代
    * 尽量避免程序中出现过多的大对象
* 长期存活的对象分配到老年代
* 动态对象年龄判断
    * 如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无需等到MaxTenuringThreshold中要求的年龄
* 空间分配担保



## 7.8 为对象分配内存：TLAB

为什么需要TLAB（Thread Local Allocation Buffer）

* 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
* 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
* 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度

TLAB

* 从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，JVM为$\color{red}{每个线程分配了有个私有的缓存区域}$，它包含在Eden空间内
* 多个线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称之为$\color{red}{快速分配策略}$
* OpenJDK衍生出来的JVM都提供了TLAB的设计
* 尽管不是所有的对象实例都能够在TLAB中成功分配内存，但$\color{red}{JVM确实是将TLAB作为内存分配的首选}$
* 在程序中，开发人员可以通过选项`-XX:UserTLAB`设置是否开启TLAB空间
* 默认情况下，TLAB空间的内存非常小，$\color{red}{仅占有整个Eden空间的百分之1}$，可以通过选项`-XX:TLABWasteTargetPervent`设置TLAB空间所占用Eden空间的百分比大小
* 一旦对象在TLAB空间分配内存失败时，JVM就会成实者通过使用$\color{red}{加锁机制}$确保数据操作的原子性，从而直接在Eden空间中分配内存

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E5%A0%86-TLAB.jpg)



## 7.9 堆空间参数设置小结

1. 查看所有的参数的默认初始值
    * -XX:+PrintFlagsInitial
2. 查看所有的参数的最终值（可能会存在修改，不再是初始值）
    * -XX:+PrintFlagsFinal
3. 初始堆空间内存（默认为物理内存的1/64）
    * -Xms
4. 最大堆空间内存（默认为物理内存的1/4）
    * -Xmx
5. 设置新生代的大小（初始值及最大值）
    * -Xmn
6. 配置新生代与老年代在堆结构的占比
    * -XX:NewRatio
7. 具体查看某个参数的指令
    * jps：查看当前运行中的进程
    * jinfo -flag SurvivorRatio 进程id

在发生Minor GC之前，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象的总空间。

* 如果大于，则此次Minor GC是安全的
* 如果小于，则虚拟机会查看`-XX:HandlePromotionFailure`设置值是否允许担保失败
    * 如果`HandlePromotionFailure=true`，那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代的对象的平均大小
        * 如果大于，则尝试进行一次Minor GC，但这次Minor GC依然是有风险的
        * 如果小于，则改为进行一次Full GC
    * 如果`HandlePromotionFailure=false`，则改为进行一次Full GC

在JDK6 Update24之后，HandlePromotionFailure参数不会再影响到虚拟机的空间分配担保策略，OpenJDK中的源码变化，虽然源码中还定义了HandlePromotionFailure参数，但是在代码中已经不会再使用它。JDK6 Update24之后的规则变为$\color{red}{只要老年代的连续空间大于年轻代对象总大小}$或者$\color{red}{历次晋升的平均大小就会进行Minor GC}$，否则将进行Full GC。



## 7. 10 逃逸分析

* 堆是分配对象存储的唯一选择吗？

随着JIT编译期的发展与$\color{red}{逃逸分析技术}$逐渐成熟，$\color{red}{栈上分配}$、$\color{red}{标量替换优化技术}$将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么绝对了。

在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是经过逃逸分析后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配。这样就无需在对上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术



### 逃逸分析概述

* 如何将堆上的对象分配到栈，需要使用逃逸分析手段。这是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法
* 通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上
* 逃逸分析的基本行为就是分析对象动态作用域：
    * 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸
    * 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他方法中

> 如何快速的判断是否发生了逃逸分析：看new的对象是否有可能在外部被调用

* 参数设置：
    * 在JDK 6u23版本之后，HotSpot中默认就已经开启了逃逸分析
    * 如果使用的是较早的版本，可以通过选项`-XX: +DoEscapeAnalysis`显式开启逃逸分析
    * 通过选项`-XX: +PrintEscapeAnalysis`查看逃逸分析的筛选结果
* 结论：开发中能使用局部变量的，就不要使用在方法外定义的

### 代码优化
使用逃逸分析，编译器可以对代码做如下优化：
1. 栈上分配
将堆分配转换为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配
2. 同步省略
如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步
3. 分离对象或标量替换
有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

### 栈上分配
* JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无需进行垃圾回收了。
* 常见的栈上分配的场景
  * 在逃逸分析中，已经说明了。分别是给成员变量赋值、方法返回值、实例引用传递

演示代码：选项`-Xmx1G -Xms1G -XX:-DoEscapeAnalysis`
``` java
public class StackAllocation {
    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println(end - start);
        Thread.sleep(100000);
    }

    public static void alloc() {
        User user = new User();
    }
}

class User{}
```
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90-%E6%A0%88%E4%B8%8A%E5%88%86%E9%85%8D1.png)

由于没有开启逃逸分析，所以所有的`User`对象都被分配在了堆空间上，尽管`alloc`方法没有发生逃逸。
减小虚拟机堆内存空间，打印GC日志`-Xmx256m -Xms256m -XX:-DoEscapeAnalysis -XX:+PrintGCDetails`
输出结果：
```
[0.006s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.021s][info   ][gc,heap] Heap region size: 1M
[0.032s][info   ][gc     ] Using G1
[0.033s][info   ][gc,heap,coops] Heap address: 0x00000000f0000000, size: 256 MB, Compressed Oops mode: 32-bit
[0.210s][info   ][gc,start     ] GC(0) Pause Young (Normal) (G1 Evacuation Pause)
[0.211s][info   ][gc,task      ] GC(0) Using 6 workers of 8 for evacuation
[0.213s][info   ][gc,phases    ] GC(0)   Pre Evacuate Collection Set: 0.0ms
[0.213s][info   ][gc,phases    ] GC(0)   Evacuate Collection Set: 1.4ms
[0.213s][info   ][gc,phases    ] GC(0)   Post Evacuate Collection Set: 0.2ms
[0.213s][info   ][gc,phases    ] GC(0)   Other: 0.9ms
[0.213s][info   ][gc,heap      ] GC(0) Eden regions: 24->0(152)
[0.213s][info   ][gc,heap      ] GC(0) Survivor regions: 0->1(3)
[0.213s][info   ][gc,heap      ] GC(0) Old regions: 0->0
[0.213s][info   ][gc,heap      ] GC(0) Humongous regions: 0->0
[0.213s][info   ][gc,metaspace ] GC(0) Metaspace: 807K->807K(1056768K)
[0.213s][info   ][gc           ] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 24M->1M(256M) 2.558ms
[0.213s][info   ][gc,cpu       ] GC(0) User=0.00s Sys=0.00s Real=0.00s
140
```
由于对象全都被分配了到了堆空间中，导致堆内存满，会引发GC。
开启逃逸分析`-Xmx256m -Xms256m -XX:+DoEscapeAnalysis -XX:+PrintGCDetails`
输出结果：
```
[0.008s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.022s][info   ][gc,heap] Heap region size: 1M
[0.034s][info   ][gc     ] Using G1
[0.034s][info   ][gc,heap,coops] Heap address: 0x00000000f0000000, size: 256 MB, Compressed Oops mode: 32-bit
8
```
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90-%E6%A0%88%E4%B8%8A%E5%88%86%E9%85%8D%E4%BC%98%E5%8C%96.png)
没有引发GC，并且对象的数量明显小于没有开启逃逸分析的情况

### 同步省略
* 线程同步的代价是相当高的，同步的后果是降低并发性和性能
* 在动态编译同步块的时候，JIT编译器可以借助逃逸分析来$\color{red}{判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程}$。如果没有，那么JIT编译器在编译这个同步块的时候就会取消这部分代码的同步。这样就能大大提高并发现和性能。这个取消同步的过程就叫同步省略，也叫$\color{red}{锁消除}$

实例代码：
``` java
public void f() {
    Object hollis = new Object();
    synchronized (hollis) {
        System.out.println(hollis);
    }
}
```
代码中对`hollis`这个对象进行加锁，但是`hollis`对象的生命周期旨在`f()`方法中，并不会被其他线程所访问到，所以在JIT编译阶段就会被优化掉。优化成：
``` java
public void f() {
    Object hollis = new Object();
    System.out.println(hollis);
}
```
字节码文件依旧有`synchronized`，只有在运行的时候才会考虑去除
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90-%E5%90%8C%E6%AD%A5%E7%9C%81%E7%95%A5.png)

### 分离对象或标量替换
* 标量：是指一个无法再分解成更小的数据的数据。Java中的原始数据类型就是标量。
* 聚合量：相对标量，那些还可以分解的数据叫做聚合量，Java中的对象就是聚合量，因为他可以分解成其他聚合量和标量。

在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。这个过程就是标量替换。

示例代码：
``` java
public static void main(String[] args) {
    alloc();
}
private static void alloc() {
    Point point = new Point(1, 2);
    System.out.println(point.x + ' ' + point.y);
}
class Point {
    private int x;
    private int y;
}
```
以上代码，经过标量替换后，就会变成：
``` java
private static void alloc() {
    int x = 1;
    int y = 1;
    System.out.println(x + ' '+ y);
}
```
Point这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个标量了。标量替换可以大大减少堆内存的使用。因为一旦不需要创建对象了，那么就不再需要分配堆内存了。

* 参数 `-XX: +EliminateAllocations`开启标量替换（默认打开），允许将对象打散分配在栈上。

## 7.11 代码优化及堆小结
参数小结：
* `-server`：启动Server模式，因为在Server模式下，才可以启用逃逸分析
* `-XX:+DoEscapeAnalysis`：启用逃逸分析
* `-Xmx10m`：指定了堆空间最大为10MB
* `-XX:+PrintGC`：打印GC日志
* `-XX:+EliminateAllocations`：开启标量替换，默认大考，允许将对象打散分配在栈上。比如对象拥有id和name两个字段，那么这两个字段将会被视为两个独立的局部变量进行分配

![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90-Server.png)
通过`java -version`可以发现，64位操作系统下，默认启用Server模式，不用调用-server模式

* 关于逃逸分析的论文在1999年就已经发表了，但直到JDK1.6才有实现，而且这项技术到如今也并不是十分成熟。其根本原因就是，无法保证逃逸分析的性能小号一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、锁消除。但是逃逸分析自身也是需要进行一些列复杂的分析，这其实也是一个相对耗时的过程。
* 举一个非常极端的例子，经过逃逸分析后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。
* 虽然这项技术并不十分成熟，但是它也是$\color{red}{是即时编译器优化技术中一个十分重要的手段}$
* 有一些观点认为，通过逃逸分析，JVM会在栈上分配那些不会逃逸的对象，这在理论上是可行的，但是取决于JVM设计者的选择。在HotSpot JVM中并没有那么做，这一点逃逸分析相关的文档里已经说明，所以可以明确所有的对象实例都是创建在堆上。
* intern字符串的缓存和静态变量曾经都被分配在永久代上，而永久代已经被元空间取代。但是，intern字符串缓存和静态变量并不是被转移到了元空间，而是直接在堆上分配，所以这一点同样符合前面一点的结论：$\color{red}{对象实例都是分配在堆上}$。

# 八、 方法区

## 8.1 栈、堆、方法区的交互关系

从内存结构角度来看运行时数据区
![](https://qiancijun.coding.net/p/Image/d/Image/git/raw/master/Java/JVM/%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.png)
从线程共享与否的角度来看运行时数据区
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E6%96%B9%E6%B3%95%E5%8C%BA-%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.png)
> 虚拟机栈和本地方法栈有异常，但是没有GC；程序计数器既没有异常，也没有GC；堆空间和元空间既有异常，也有GC。

栈、堆、方法区的交互关系
![](https://qiancijun-images.oss-cn-beijing.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%9B%BE%E7%89%87/JavaEE/JVM/%E6%96%B9%E6%B3%95%E5%8C%BA-%E6%A0%88%E3%80%81%E5%A0%86%E3%80%81%E6%96%B9%E6%B3%95%E5%8C%BA%E7%9A%84%E4%BA%A4%E4%BA%92%E5%85%B3%E7%B3%BB.jpg)





## 8.2 方法区概述

《Java虚拟机规范》中明确说明：“尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会去选择进行垃圾收集或者进行压缩。”但对于HotSpotJVM而言，方法区还有一个别名叫做Non-Head，目的就是要和堆分开。

所以，$\color{red}{方法区看作是一块独立于Java堆的内存空间。}$

### 方法区的基本理解

* 方法区（Method Area）与Java堆一样，是各个线程共享的内存区域。
* 方法区在JVM启动的时候被创建，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的。
* 方法区的大小，跟堆空间一样，可以选择固定大小或者扩展
* 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样回抛出内存溢出错误：`java.lang.OutOfMemoryEorror:PermGen Space`或者`java.lang.OutOfMemoryEorror:Metaspace`
* 关闭JVM就会释放这个区域的内存



### HotSpot中方法区的演进

* 在jdk7以前，习惯上把方法区称为永久代。jdk8以后，使用元空间取代了永久代
* 本质上，方法区和永久代并不等价，仅是对Hotspot而言的。《Java虚拟机规范》对如何实现方法区，不做统一要求。例如：BEA JRockit / IBM J9中不存在永久代的概念
* 而到了JDK8，完全废弃了永久代的概念，改用与JRockit、J9一样在本地内存中实现的元空间来代替
* 元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代最大的区别在于：$\color{red}{元空间不在虚拟机设置的内存中，而是使用本地内存}$
* 永久代、元空间并不只是名字不同，内部结构也不相同
* 根据《Java虚拟机规范》的规定，如果方法区无法满足新的内存分配需求时，将抛出OOM异常

## 8.3 设置方法区大小与OOM

* 方法区的大小不必是固定的，jvm可以根据应用的需要动态调整
* JDK7及以前：
    * 通过`-XX:PermSize`来设置永久代初始分配空间。默认值是20.75M
    * `-XX:MaxPermSize`来设定永久代最大可分配空间。32位机器默认是64M，64位及机器默认是82M
    * 当JVM加载的类信息容量超过了这个值，会报异常OutOfMemoryError:PermGen Space
* JDK8及以后：
    * 元空间大小可以使用参数`-XX:MetaspaceSize`和`-XX:MaxMetaspaceSize`指定，替代上述原有的两个参数
    * 默认值依赖于平台。windowsw下，默认值为21M，没有上限
    * 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据区发生溢出，虚拟机一样会抛出OutOfMemoryError:Metaspace
    * `-XX:MetaspaceSize`：设置初始的元空间大小。对于一个64位的服务器端JVM来说，其默认的`-XX:MetaspaceSize`值为21M.这就是初始的高水位线，一旦超过限制，Full GC将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个限制会重置。新的限制的值取决于GC后释放了多少元空间。如果释放的空间不足，那么在不超过MaxMetaspaceSize时，适当提高该值。如果释放空间过多，则适当降低该值。
    * 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到Full GC多次调用。为了避免频繁的发生GC，建议将`-XX:MetaspaceSize`设置为一个相对较高的值。

### 如何解决OOM

1. 要解决OOM异常或者heap space的异常，一般的手段是首先通过内存映像分析工具对dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清除到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）
2. 如果是内存泄漏，可进一步通过工具查看泄露对象到GC Roots的引用链。于是就能找到泄露对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄露对象的类型星系，以及GC Roots引用链的信息，就可以比较准确的定位出泄露代码的位置
3. 如果不存在内存泄露，换句话说就是内存中的对象确实都还必须存活者，那就应当检查虚拟机的堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调打，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗

## 8.4 方法区的内部结构

它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译后的代码缓存等

1. 类型信息：堆每个加载的类型（类class、接口interface、枚举enum、注解annotation），JVM必须在方法区存储以下类型信息
    * 这个类型的完整有效名称（全名=包名.类名）
    * 这个类型直接父类的完整有效名（对于interface或是java.lang.Object，都没有父类）
    * 这个类型的修饰符（public，abstract，final的某个子继）
    * 这个类型直接接口的一个有序列表
2. 域信息（成员变量）
    * JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序
    * 域的相关信息包括：域名称、域类型、域修饰符
3. 方法信息：JVM必须保存所有方法的以下信息，包括声明顺序
    * 方法名称
    * 方法的返回类型
    * 方法参数的数量和类型
    * 方法的修饰符
    * 方法的字节码
    * 异常表（抽象方法和本地方法除外）

### non-final的类变量

* 静态变量和类关联在一起，随着类的加载而加载，它们成为类数据在逻辑上的一部分
* 类变量被类的所有实例共享，即使没有类实例时你也可以访问它们

``` java
public class NonFinal {
    public static void main(String[] args) {
        Test t = null;
        System.out.println(t.count);
        t.hello();
    }
}

class Test {
    static int count = 1;

    public static void hello() {
        System.out.println("hello");
    }
}
```

* 被声明为final的类变量的处理方法则不同，每个全局常量在编译的时候就会被分配了。



### 运行时常量池

* 方法区，内部包含了运行时常量池
* 字节码文件，内部包含了常量池
* 一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述信息外，还包含一项信息那就是常量池表，包括各种字面量和对类型、域和方法的符号引用
* 一个Java源文件中的类、接口，编译后产生一个字节码文件。而Java中的字节码文件需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候会用到运行时常量池

``` java
public class SimpleClass {
    public void sayHello() {
        System.out.println("hello");
    }
}
```

* 虽然只有194字节，但是里面却使用了String、System、PrintStream、Object等结构。这里代码量其实已经很小了，如果代码多，引用到的结构会更多，就需要使用常量池了
* 小结：常量池可以看做是一张表，虚拟机指令根据这张表找到要执行的类名、方法名、参数类型、字面量等类型