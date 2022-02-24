### CPU相关

#### CPU概述

**中央处理器** （英语：**C**entral **P**rocessing **U**nit，缩写：**CPU**）是[计算机](https://zh.wikipedia.org/wiki/计算机)的主要设备之一，功能主要是解释计算机[指令](https://zh.wikipedia.org/wiki/指令)以及处理计算机[软件](https://zh.wikipedia.org/wiki/软件)中的[数据](https://zh.wikipedia.org/wiki/数据)。计算机的可编程性主要是指对中央处理器的[编程](https://zh.wikipedia.org/wiki/编程)。1970年代以前，中央处理器由多个独立单元构成，后来发展出由[集成电路](https://zh.wikipedia.org/wiki/集成电路)制造的中央处理器，这些高度收缩的器件就是所谓的[微处理器](https://zh.wikipedia.org/wiki/微处理器)，其中分出的中央处理器最为复杂的电路可以做成单一微小功能强大的单元，也就是所谓的核心。

出自：[中央处理器-wiki](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%A4%AE%E5%A4%84%E7%90%86%E5%99%A8) 

计算机主板上放置 `CPU` 的槽和 `CPU` 都被称作 `socket` ，根据语义的不同指不同的实体。在生产 `CPU` 时会将一大块晶圆 `Silicon Wafer` 分隔为很多小块，而这单个小块晶圆就称作为 `die` 。`CPU` 绿色的部分其实是一块 `PCB` 板，而 `die` 正是镶嵌在这块 `pcb` 上。

一块 `die` 上可以有多个核心 `core` ，而一块 `cpu` 的 `pcb` 上也可以有多个 `die` 。而 `die` 的大小对 `cpu` 的性能和成本也会有很大影响，`die` 越大则出错的几率也越大良品率下降成本增加，因为 `die` 内部的各部分组件直接通过片内总线相连对性能也有很大提升，例如 `Intel Xeon` 系列的高端处理器中整个 `cpu` 中只有一个 `die` 。反之，`die` 越小虽然良品率上升成本下降，也会因为 `die` 大小原因无法放下更多核心或多个 `die` 之间需要片外总线相连导致性能下降，例如 `AMD` 的 `EYPC CPU` 。

![image-20220211142800536](photo/53、CPU简单结构图(7).png)  

出自：[CPU DIE - zhihu](https://zhuanlan.zhihu.com/p/51354994) 

相关：[为什么晶圆都是圆的 - zhihu](https://zhuanlan.zhihu.com/p/30513730) 

#### CPU 缓存

##### 概述

计算机的储存硬件通常是固态硬盘 `ssd` 或者机械硬盘 `hdd` ，由于存储介质及其物理特性导致 `ssd` 硬盘比 `hdd` 硬盘存取速度甚至可以高出十几倍之多。虽然 `ssd` 硬盘的速度已经非常可观，但是和 `cpu` 的吞吐量相比差距十分巨大。所以就有了内存 `ram` 用作 `cpu` 和硬盘之间的桥梁，`cpu` 想要读取硬盘中的数据首先需要将数据加载到 `ram` 中再进行下一步操作。

##### 缓存与内存

虽然添加内存之后 `cpu` 的性能得到进一步的发挥，但是内存一般使用 `DRAM` 技术，使用 `DRAM` 储存一位数据需要一个电容加一个晶体管，数据储存在电容中，电容需要充放电才可以进行读写操作，这就导致在读写过程中产生较大的延迟。于是就需要 `cpu` 缓存技术，`cpu` 缓存一般是与核心一起封装在一个 `die` 中，这样的设计也会使性能大大增加。`cpu` 缓存一般采用 `SRAM` 技术，`SRAM` 储存一位数据需要六个晶体管性能方面较 `DRAM` 也有较大提升，因此 `cpu` 缓存的频率与 `cpu` 频率相近。而由于 `SRAM` 技术成本较高且缓存位于 `die` 中，因此 `cpu` 缓存一般只有几十 `KB` 到几十 `MB` 。

详见：[SRAM 与 DRAM](# SRAM 与 DRAM) 

##### CPU 中缓存的结构

现代 `cpu` 一般有三层缓存，呈金字塔状，最接近 `cpu` 的缓存（一般使用 `L1` 标记）容量最小性能也最高，其他缓存性能逐层递减容量也递增。当 `cpu` 查询某条数据时会先从 `L1` 缓存中查找若命中 `cache hit` 则直接使用，若未命中 `cache miss` 则从下一级缓存中查找，直到从内存中获取，最后经过缓存返回到 `cpu` 中。

对于一个完整的 `CPU` ，里面会有若干个核，比如四核八核等等，对于 `CPU` 多核心的场景下，每个核心的 `L1` , `L2` 缓存都只能被这个核访问，别的核无法访问，`L1` ，`L2` 是这个核的私有缓存。而相对应的，`L3` 缓存被以某种协议或者方式链接到 `CPU` 内的一种总线上，供所有核访问，所以，`L3` 为共享缓存。

> 注：在随机读取状态下，内存的读取速度会下降，因为随机读取会导致大量的 `cache miss` 致使性能下降。

##### 缓存性能

一般来说 `L1` 缓存作为最靠近 `cpu` 核心的部分，其频率是与核心频率相同的，当然，容量也是最小的，一般只有几十到几百 `KB` 。而 `L2` 和 `L3` 缓存容量逐级增大，性能也逐级下降。

某 `CPU` 性能表：

|                                 | 时钟周期 | 纳秒（ns） |
| ------------------------------- | -------- | ---------- |
| L1 一级缓存                     | 4        | 1.2~2.1    |
| L2 二级缓存                     | 10       | 3.0~5.3    |
| L3 三级缓存（核心独享）         | 40       | 12.0~21.4  |
| L3 三级缓存 （正在共享）        | 65       | 19.5~34.8  |
| L3 三级缓存（其他核心正在修改） | 75       | 22.5~40.2  |
| 内存                            |          | 60         |

出自：[CPU缓存(Cache)背后的运行逻辑1-缓存必知](https://zhuanlan.zhihu.com/p/353553358) 

英特尔 `skylake` 架构

![BW_GT3e](photo/52、Intel处理器skylake架构(7).png) 

可以看到上图中的 `L1、L2、L3` 三级缓存，以及最右侧的 `DRAM` 内存。

#### SRAM 与 DRAM

`SRAM` 与 `DRAM` 都会在断电后丢失数据，由于前者性能强于后者但成本大于后者，所以 `SRAM` 用作 `cpu` 缓存，而 `DRAM` 用作内存（内存条）。

##### SRAM 静态随机存取储存器

**静态随机存取存储器**（**S**tatic **R**andom **A**ccess **M**emory，**SRAM**）是[随机存取存储器](https://zh.wikipedia.org/wiki/随机存取存储器)的一种。所谓的“静态”，是指这种存储器只要保持[通电](https://zh.wikipedia.org/wiki/電)，里面存储的数据就可以恒常保持[[1\]](https://zh.wikipedia.org/wiki/静态随机存取存储器#cite_note-skorobogatov-1)。相对之下，[动态随机存取存储器](https://zh.wikipedia.org/wiki/動態隨機存取記憶體)（DRAM）里面所存储的数据就需要周期性地更新。然而，当电力供应停止时，SRAM存储的数据还是会消失（被称为[易失性存储器](https://zh.wikipedia.org/wiki/易失性存储器)），这与在断电后还能存储资料的[ROM](https://zh.wikipedia.org/wiki/ROM)或[闪存](https://zh.wikipedia.org/wiki/快閃記憶體)是不同的。

出自：[静态随机存取储存器 - wiki](https://zh.wikipedia.org/wiki/%E9%9D%99%E6%80%81%E9%9A%8F%E6%9C%BA%E5%AD%98%E5%8F%96%E5%AD%98%E5%82%A8%E5%99%A8) 

##### DRAM 动态随机存取储存器

**动态随机存取存储器**（**Dynamic Random Access Memory**，**DRAM**）是一种[半导体](https://zh.wikipedia.org/wiki/半导体)[存储器](https://zh.wikipedia.org/wiki/記憶體)，主要的作用原理是利用[电容](https://zh.wikipedia.org/wiki/電容)内存储[电荷](https://zh.wikipedia.org/wiki/電荷)的多寡来代表一个[二进制](https://zh.wikipedia.org/wiki/二进制)[比特](https://zh.wikipedia.org/wiki/位元)（bit）是1还是0。由于在现实中晶体管会有漏电电流的现象，导致电容上所存储的电荷数量并不足以正确的判别资料，而导致资料毁损。因此对于DRAM来说，周期性地充电是一个不可避免的条件。由于这种需要定时刷新的特性，因此被称为“动态”存储器。相对来说，静态存储器（[SRAM](https://zh.wikipedia.org/wiki/静态随机存取存储器)）只要存入资料后，纵使不刷新也不会丢失数据。

与SRAM相比，DRAM的优势在于结构简单——每一个比特的资料都只需一个电容跟一个[晶体管](https://zh.wikipedia.org/wiki/電晶體)来处理，相比之下在SRAM上一个比特通常需要六个晶体管。正因这缘故，DRAM拥有非常高的密度，单位体积的容量较高因此成本较低。但相反的，DRAM也有访问速度较慢，耗电量较大的缺点。

与大部分的[随机存取存储器](https://zh.wikipedia.org/wiki/隨機存取記憶體)（RAM）一样，由于存在DRAM中的资料会在电力切断以后很快消失，因此它属于一种[易失性存储器](https://zh.wikipedia.org/wiki/揮發性記憶體)（volatile memory）设备。

出自：[动态随机存取储存器 - wiki](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E9%9A%8F%E6%9C%BA%E5%AD%98%E5%8F%96%E5%AD%98%E5%82%A8%E5%99%A8) 

#### MESI 协议

因为 `IO` 操作的数据访问具有空间连续性（需要访问内存中的很多数据），但是 `IO` 操作较慢，而在 `IO` 操作时读一个字节和读多个字节的时间基本相同，所以内存中操作 `IO` 不是以字节为单位，而是以块为单位。即 `cpu` 缓存中最小的储存单元是缓存行 `cache line` ，而每一级的缓存都会被划分为多组 `cache line` 。目前主流的 `CPU` 的缓存行大小都为 `64B` ，也就是 `512bit` 。这是因为内存的一次传输的单位也是 `64` 字节。

`MESI` 协议即是对缓存行的定义，此协议定义了缓存行的四种状态，使用额外的两位来标识。

- `M` ：被修改状态 `Modified` 

		该缓存行只被缓存在该 `cpu` 中，并且是被修改过的即此缓存行中的数据与内存中的数据不一致。该缓存行中的数据需要在允许其他 `cpu` 读取内存中相应位置的数据前进行写回 `write back` ，当缓存行中的数据被写回后其状态会变为独享状态 `exclusive` 。

- `E` ：独享状态 `Exclusive` 

		该缓存行对应内存的数据只被该 `cpu` 缓存，该缓存行中的数据与对应内存中的数据相同。当其他 `cpu` 在读取对应内存中的数据后，此缓存行变为共享状态 `shared` 。当修改此缓存行中的数据后，此缓存行变为被修改状态 `modified` 。

- `S` ：共享状态 `Shared` 

		该缓存行中的数据被多个 `cpu` 缓存，且各个 `cpu` 缓存行中的数据与内存中的数据相同。当某一个 `cpu` 修改对应缓存行中的数据时，该 `cpu` 的该缓存行变为被修改状态 `modified` ，其他 `cpu` 的对象缓存行变为无效状态 `invalid` 。

- `I` ：无效状态 `Invalid` 

		该缓存行是无效状态，其他缓存了对应内存中数据的 `cpu` 修改了它的缓存行内容。

缓存行状态转换图

![4b4bfeaeec21d904d63e82d6a389e78d](photo/54、MESI协议缓存状态转换图(7).png) 



#### 伪共享

##### 概述

同时更新来自不同处理器的相同缓存代码行中的单个元素会使整个缓存代码行无效，即使这些更新在逻辑上是彼此独立的。每次对缓存代码行的单个元素进行更新时，都会将此代码行标记为**无效**。其他访问同一代码行中不同元素的处理器将看到该代码行已标记为**无效**。即使所访问的元素未被修改，也会强制它们从内存或其他位置获取该代码行的较新副本。这是因为基于缓存代码行保持缓存一致性，而不是针对单个元素的。因此，互连通信和开销方面都将有所增长。并且，正在进行缓存代码行更新的时候，禁止访问该代码行中的元素。

出自：[Oracle 伪共享](https://docs.oracle.com/cd/E19205-01/821-0393/aewcx/index.html) 

蓝色部分是指定 `cpu` 要使用的数据。 

![image-2022021109230149](photo/55、MESI协议伪共享01(7).png)  

当 `cpu1` 中缓存行其使用的元素进行修改时，会将 `cpu1` 的此缓存行标记为被修改，`cpu2` 的对应缓存行则会被标记为无效。

![image-20220211092058984](photo/56、MESI协议伪共享02(7).png) 

当 `cpu2` 需要使用后三个元素时，虽然其元素内容与内存中的内容相同，但由于 `cpu2` 的指定缓存行已经被标记为无效，所以还是需要重新获取最新的数据。

![image-20220211093506888](photo/57、MESI协议伪共享03(7).png) 

伪共享 `cpu` 结构图

![伪共享](photo/58、MESI协议伪共享结构图(7).png) 

可以看到在单 `cpu` 多核的情况下需要重新从 `L3` 缓存获取最新的数据，但是如果是多 `cpu` 甚至需要跨槽 `socket` 读取，需要从内存中加载最新值。

 ##### Java 中的伪共享

伪共享在 `java` 中也是一个实际存在的问题，尤其是在循环等情况下会产生较大的资源浪费。

```java
class MyObject{
    long a;
    long b;
    long c;
}
```

如上，对象中有三个 `long` 型变量，`jvm` 在堆中分配空间时这三个变量是在一起的。一个 `long` 占8个字节，而 `x86` 的缓存行为64个字节，因此这三个变量极有可能会在同一缓存行中。

使用以下代码来测试伪共享对性能的影响。

```java
public class FalseShareTest implements Runnable {
    // 使用的线程数量
    public static int NUM_THREADS = 5;
    // 数组中元素修改次数
    public final static long ITERATIONS = 500L * 1000L * 1000L;
    // 当前对象是第几个元素
    private final int arrayIndex;
    // 数组
    private static VolatileLong[] longs;
    // 运行10次的总时间
    public static long SUM_TIME = 0l;

    public FalseShareTest(final int arrayIndex) {
        this.arrayIndex = arrayIndex;
    }

    public static void main(final String[] args) throws Exception {
        Thread.sleep(10000);
        // 运行10次取均值
        for(int j=0; j<10; j++){
            System.out.println(j);
            // 读参
            if (args.length == 1) {
                NUM_THREADS = Integer.parseInt(args[0]);
            }
            // 新建数组
            longs = new VolatileLong[NUM_THREADS];
            for (int i = 0; i < longs.length; i++) {
                // 实例化对象赋值
                longs[i] = new VolatileLong();
            }
            final long start = System.nanoTime();
            // 启动线程并等待完成
            runTest();
            final long end = System.nanoTime();
            SUM_TIME += end - start;
        }
        // 取均值
        System.out.println("平均耗时："+SUM_TIME/10);
    }
    private static void runTest() throws InterruptedException {
        // 线程数组
        Thread[] threads = new Thread[NUM_THREADS];
        for (int i = 0; i < threads.length; i++) {
            // 实例化线程
            threads[i] = new Thread(new FalseShareTest(i));
        }
        // 运行
        for (Thread t : threads) {
            t.start();
        }
        // 等待完成
        for (Thread t : threads) {
            t.join();
        }
    }
    public void run() {
        // 循环给指定元素赋值
        long i = ITERATIONS + 1;
        while (0 != --i) {
            longs[arrayIndex].value = i;
        }
    }

    public final static class VolatileLong {
        public volatile long value = 0L;
        // 填充缓存行
        public long p1, p2, p3, p4, p5, p6;     //屏蔽此行
    }

}
```

上面代码使用了一个对象数组，这个对象 `VolatileLong` 中有7个 `long` 型变量，其中 `value` 使用 `volatile` 标记，以保证变量的可见性 详见：[Volatile 详解](# Volatile) 。线程 `run` 方法中的代码就是用不同的线程给数组中不同的元素 `VolatileLong` 对象的 `value` 赋值，使用 `for` 循环实例化对象并填充到数组中，此时，数组中的对象在内存中储存的位置应该彼此相邻。此时，整个数组就会在同一个缓存行中被加载到缓存中，不同线程循环不停赋值时就会出现伪共享现象。因此我们在 `VolatileLong` 对象中添加了6个额外的 `long` 型数据，用于撑大 `VolatileLong` 的大小，使数组中的元素不在同一个缓存行中避免伪共享。

关于对象储存大小的详细介绍请查看：[Java 对象模型](# Java 对象模型) 

一下的图表是填充空值和未填充空值时，分别1-5个线程下程序执行所用时间。

![](photo/59、MESI协议伪共享统计图(7).png)  

可以看到在无填充的情况下处理耗时的增长明显大于填充的情况，非常明显，在无填充时发生了伪共享问题而且这个问题在线程越多的情况下越严重。

测试方法及代码出自：[伪共享（false sharing），并发编程无声的性能杀手 ](https://www.cnblogs.com/cyfonly/p/5800758.html) 

##### 避免伪共享

首先第一种就是使用空白的变量进行填充，使不同的元素无法被保存在同一个缓存行中。第二种是使用 `sun.misc.Contended` 注解来解决此问题，此注解的原理也与空白填充的原理相同。在默认情况下，`＠Contended` 注解只用于 `Java` 核心类， 比如此包下的类。如果用户类路径下的类需要使用这个注解， 则需要添加 `JVM` 参数：`-XX:-RestrictContended` 。填充的宽度默认为前后128字节，要自定义宽度则可以设置 `-XX: ContendPaddingWidth` 参数。

伪共享是很隐蔽的，我们暂时无法从系统层面上通过工具来探测伪共享事件。其次，不同类型的计算机具有不同的微架构（如 32 位系统和 64 位系统的 `java` 对象所占自己数就不一样），如果设计到跨平台的设计，那就更难以把握了，一个确切的填充方案只适用于一个特定的操作系统。并且，缓存的资源是有限的，如果填充会浪费珍贵的 `cache` 资源，并不适合大范围应用。

### JVM 内存

`JVM` 内存分区是 `JVM` 在运行过程中数据区（内存）的分区。

`JMM` 内存模型是一个抽象的概念

#### JVM 内存分区

`JVM` 在执行 `Java` 程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则依赖用户线程的启动和结束而建立和销毁。

![jvm运行时数据区](photo/60、JVM运行时内存分区(7).png) 

`JVM` 内存分区是一个抽象概念，其中的堆、方法区和线程私有的数据，都有可能出现在物理机的内存、缓存和 `cpu` 寄存器中。

##### 程序计数器

程序计数器 `Program Counter Register` 用于储存当前线程所执行的字节码的地址，程序的分支、循环、跳转、异常、线程恢复等都依赖于计数器。当多线程执行当前线程没有获取到 `cpu` 资源或被其他线程获取，当前线程就需要一个单独的计数器，用于在获取到 `cpu` 资源后恢复到正确的位置继续执行。此计数器保存在线程私有的内存中，各线程之间独立储存互不干扰。

若当前线程执行的是 `Java` 方法，则计数器中储存虚拟机字节码的地址；若执行的是本地方法 `Native Method` ，则这个计数器值为空 `Undefined` 。

> 注：此内存空间是唯一一个在 `JVM` 中没有规定 `OutOfMemoryError` 错误的区域。

##### Java 虚拟机栈 Java Virtual Machine Stacks

`Java` 虚拟机栈 `Java Vitual Machine Stack` 是 `Java` 方法执行的内存模型，在栈中存放的是多个栈帧，每个栈帧对应一个被调用的方法，在栈帧中包含了方法的局部变量 `Local Variables` 、操作数栈 `Operand Stack` 、指向方法所属类的运行时常量池 `Reference to runtime constant pool` 、方法返回地址 `Return Address` 和一些额外信息。当线程执行一个方法，便会自动创建一个对应的栈帧，并将栈帧压入栈中。当方法执行完毕，便会将对应的栈帧出栈，而当前执行的方法会永远处于栈帧的顶部。

![image-20220211165207439](photo/61、JVM运行时内存分区-栈(7).png) 

##### 本地方法栈 Native Method Stacks

本地方法栈作用于 `Java` 栈类似，本地方法栈是用于执行本地方法 `Native Method` 。在JVM规范中，并没有具体规定本地方法栈的具体实现方法以及数据结构，而在 `HotSopt` 虚拟机中直接把本地方法栈和 `Java` 栈合并到了一起。

##### 堆 Heap

`Java` 中堆是用来储存对象的数据，在栈中储存的是对象的引用。在堆中的数据是被所有线程共享的，所有线程都可以访问到堆中的数据，且 `JVM` 中只有一个堆。

> 注：在 `Java` 中数组也是一个对象，所以，数组的数据也是储存在堆中。

##### 方法区 Method Area

方法区在与堆一样都是被线程共享的区域，在方法区中存储了每个类的信息（包括类的名称、方法信息、字段信息）、静态变量、常量以及编译器编译后的代码等。

在 `Class` 文件中除了类的字段、方法、接口等描述信息外，还有一项信息是常量池，用来存储编译期间生成的字面量和符号引用。

在方法区中有一个非常重要的部分就是运行时常量池，它是每一个类或接口的常量池的运行时表示形式，在类和接口被加载到 `JVM` 后，对应的运行时常量池就被创建出来。当然并非Class文件常量池中的内容才能进入运行时常量池，在运行期间也可将新的常量放入运行时常量池中，比如 `String` 的 `intern` 方法。

#### JMM 内存模型

`JMM` `Java Memory Model` ，是一种基于计算机内存模型（定义了共享内存系统中多线程程序读写操作行为的规范）。屏蔽了各种硬件和操作系统的访问差异，保证 `java` 程序在各平台下对内存的访问都能保证效果一致的规范，`JMM` 是一个抽象的概念并没有实际的物理实现。

![image-20220131155830443](photo/48、多线程共享变量操作(7).png) 

由此可见，多线程操作同一变量时其实是将变量复制到线程工作内存后再进行操作，线程不可以直接操作位于主内存中的变量。当然，线程在修改线程工作内存后，需要将工作内存中的变量同步到主内存中。

> 注意：每个线程的工作内存都是独立的，线程之间不可以访问其他线程的工作内存。

线程执行逻辑图

![image-20220131161846234](photo/49、多线程共享变量运行图(7).png) 

具体何时将数据同步到主内存中由 `JMM` 决定，`JMM` 规定了何时以及如何做线程工作内存与主内存之间的数据同步。

#### Java 对象模型

在 `HotSpot` 虚拟机中，设计了一个 `OOP-Klass Model` 。 `OOP（Ordinary Object Pointer）` 指的是普通对象指针，而 `Klass` 用来描述对象实例的具体类型。对象在内存中存储的内容可以分为三个部分：对象头 `Header` 、实例数据 `Instance Data` 和对齐填充 `Padding` 。

- 对象头 `Header` 
  - 标记字（32位 `JVM` 大小 `4B` ，64位 `JVM` 大小 `8B` ）
  - 类型指针（32位 `JVM` 大小 `4B` ，64位 `JVM` 大小 `8B` ）
  - 数组长度（如果这个对象是数组）
- 实例数据
  - 储存字段的内容，各字段的分配策略 `longs/doubles` 、`ints`、`shorts/chars`、`bytes/boolean`、`oops(ordinary object pointers)`，相同宽度的字段会分配到一起方便取出数据，父类定义的变量会在子类定义的变量之前。
- 对齐填充
  - 在64位虚拟机中，对象的大小必须是 `8B` 的整数倍，若不足8位则需要占位填充。