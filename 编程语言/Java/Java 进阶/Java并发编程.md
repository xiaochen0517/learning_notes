## 概述

### 并发与并行

#### 并发是什么

并发是指当有多个线程在操作时,如果系统同时之能执行一个线程，它只能把CPU运行时间划分成若干个时间段,再将时间 段分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状。这种方式称之为并发(`Concurrent`)。

#### 并行是什么

并行是当系统可同时执行的线程大于等于需要执行的线程数时，这时所有的线程都可以在同一时间同时执行，而这时程序的执行就被称为并行(`parallel`)。

### 进程和线程

#### 进程是什么

进程可以认为是一个执行单元，每个进程都有独立的内存空间和资源，进程和进程之间相互独立。一个进程无法去访问另一个进程的数据。进程间的资源如内存和CPU时间等资源都是由操作系统来分配。

#### 线程是什么

进程可以拥有多个线程，进程下的线程可以访问进程的资源及线程之间的数据。每个线程都有独自的缓存，如果线程读取了某个数据，那么它将会将这些数据存放到自己的缓存中以供使用。因此多线程程序最重要的一个目标就是要保证数据在多线程下的安全性。

> 一个 `JAVA` 程序默认以一个进程的形式运行，在一个进程中使用不同的多个线程一起完成并行运行算或实现异步操作。

#### 进程与线程的关系

一个进程包含一个或以上的线程，若只包含一个线程则这个线程被称为主线程。 `CPU` 的调度单位（也就是 `CPU` 真正执行的单位是线程），当 `CPU` 在执行多个线程时会通过时间片轮转来调度每个线程（如下图），当线程的数量小于 `CPU` 核数时才会“并行”执行。

![image-20220119162307176](photo\32、多线程运行逻辑(7).png)

> 线程解决了多任务同时执行的问题（例如边打游戏边听歌），当然线程也是一种受限的系统资源，系统不可能无限制的创建线程。当线程在创建和销毁时都会有相应的开销，为了解决线程频繁的创建和销毁带来的性能问题通常会采用线程池技术，即在线程池总会缓存一定数量的线程，通过这种方法来避免线程的频繁创建和销毁带来的性能问题。

## Java 中的多线程

### 主线程

我们在编写 `Java` 程序时首先要编写一个 `main` 方法，而这个方法会在当前运行的 `Java` 进程的主线程中执行，可以使用以下的代码获取到当前执行的线程名称。

```java
public static void main(String[] args) {
    System.out.println(Thread.currentThread().getName()); // 输出：main
}
```

当程序需要进行一些需要耗费大量时间进行的操作时（例如网络请求、 `I/O` 操作等），当这些操作依旧直接在主线程中执行时就会使主线程停止运行，直到操作完成程序才会继续运行，于是就需要多线程来执行这些操作防止主线程阻塞，将耗时操作放到子线程中执行。

### 线程状态

#### 状态转换图

![image-20220128173943816](photo\42、Thread生命周期转换图(7).png) 

主要方法介绍 [Thread类主要方法](# Thread 类) 

#### 代码实现

##### NEW：初始状态

```java
Thread thread1 = new Thread(() -> {
    // nothing...
});
LOGGER.info("线程状态：{}", thread1.getState()); // NEW
```

当 `Thread` 对象不为空时，线程当前的状态为初始化状态。

##### RUNNABLE：可运行状态

```java
Thread thread1 = new Thread(() -> {
    // nothing...
});
thread1.start();
LOGGER.info("线程状态：{}", thread1.getState()); // RUNNABLE
```

当执行 `start()` 方法时线程为可运行状态。

##### BLOCKED：阻塞状态

```java
final int[] i = {0};

new Thread(()->{
    synchronized (i){
        while (true){
            i[0] = 1;
        }
    }
}).start();

Thread thread4 = new Thread(() -> {
    synchronized (i){
        i[0] = 1;
    }
});
thread4.start();
Thread.sleep(500L);
LOGGER.info("线程状态：{}", thread4.getState()); // BLOCKED
```

当线程在等待同步锁时，为阻塞状态。

##### WAITING：等待状态

```java
Thread thread2 = new Thread(() -> {
    try {
        testThread.waitRun();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
thread2.start();
Thread.sleep(500L);
LOGGER.info("线程状态：{}", thread2.getState()); // WAITING
// TestThread#waitRun()
public synchronized void waitRun() throws InterruptedException {
    this.wait();
}
```

执行 `wait()` 方法后，线程进入等待状态。

##### TIMED_WAITING：有超时的等待状态

```java
Thread thread3 = new Thread(() -> {
    try {
        Thread.sleep(1000L);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
thread3.start();
Thread.sleep(500L);
LOGGER.info("线程状态：{}", thread3.getState()); // TIMED_WAITING
```

进入超时等待状态有两种方法，一种是使用 `sleep` 方法，另一种是使用 `wait(long time)` 方法。

##### TERMINATED：结束状态

```java
Thread thread5 = new Thread(() -> {});
thread5.start();
Thread.sleep(500L);
LOGGER.info("线程状态：{}", thread5.getState()); // TERMINATED
```

正常执行完成的线程进入结束状态。

### 使用多线程

在Java中最重要的三个和线程相关的类是 `Thread` `Runnable` `Callable` ，其中 `Thread` 是真正创建和运行线程的类而其余两个只是接口，用于向 `Thread` 类中传入对象引用用于执行方法的。以下源代码中可以看到 `Runnable` 和 `Callable` 类并没有实质性的方法，真正创建线程和执行的是 `Thread` 类。Thread 类是线程类而 `Runnable` 和 `Callable` 是任务类，任务类只是将需要执行的任务进行封装后传入线程类来真正执行。

#### Thread 类

在 `Thread` 类中，真正创建线程和执行的方法是本地方法 `start0` 

```java
private native void start0();
```

类中有一个名为 `target` 的变量，此变量就是需要执行的 `Runnable` 对象，而这个参数会在 `Thread` 类实例化时传入或者赋值为空。

```java
/* What will be run. */
private Runnable target;
```

##### run方法

真正在线程中执行的代码是 `run` 方法中的代码，`Thread` 实现了 `Runnable` 接口同样也实现了 `run` 方法，以下是 `Thread` 类中实现的 `run` 方法的源代码。

```java
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

在执行线程时判断是否有传入指定的 `Runnable` 对象，若存在则执行传入对象的 `run` 方法，否则直接结束。

`Thread` 类中的 `run` 方法调用是由 `start` 方法启动之后，当线程获取到 `CPU` 执行时间时会自动调用。

##### start方法

`start` 方法作用是将线程从可运行状态转换为可运行状态，内部调用了一个 `native` 方法用于初始化线程。

```java
public synchronized void start() {

    if (threadStatus != 0) // 如果当前线程的状态不是新建状态，则抛出错误
        throw new IllegalThreadStateException();

    group.add(this); // 将此Thread添加到ThreadGroup中

    boolean started = false; // 是否成功启动
    try {
        start0(); // native方法
        started = true; // 启动成功
    } finally {
        try {
            if (!started) { // 如果启动失败
                group.threadStartFailed(this); // 将当前Thread标记为启动失败
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
            /* 什么也不做。如果start0抛出一个Throwable，
            	那么它将被向上传递到调用堆栈 */
        }
    }
}
```

方法会判断需要启动线程的状态，不为新建状态则直接抛出错误，不进行启动操作。

##### sleep方法

`sleep` 方法有两个重载方法，启动 `sleep(time)` 是 `native` 方法，`sleep(millis, nanos)` 最终将参数处理后也是交给 `sleep(time)`  来执行。

```java
public static native void sleep(long millis) throws InterruptedException;

public static void sleep(long millis, int nanos) throws InterruptedException {
    if (millis < 0) { // 毫秒不可小于0
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) { // 纳秒不可小于0大于999999
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }

    // 当纳秒大于50万或者纳秒大于0且毫秒为0
    if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
        millis++; // 将毫秒加一
    }
    sleep(millis);
}
```

`sleep` 方法作用就是将线程睡眠，空出 `CPU` 执行时间用于执行其他线程。从第二个重载方法源码中可以看出，本质上 `sleep` 只能以毫秒为单位进行睡眠，`nanos` 参数并没有实际的效果。

> 注意：`sleep` 方法不会释放锁，若当前线程拥有一个对象的锁进入 `sleep` 状态时，其他需要这个对象锁的线程依旧无法拿到锁。

##### yield方法

`yield` 方法是一个 `native` 方法，作用和 `sleep` 方法相同都是空出 `CPU` 执行时间用于执行其他线程，同样， `yield` 方法也不会释放锁。唯一与 `sleep` 方法不同的是 `yield` 方法无法控制空出的时间，并且 `yield` 方法只能让拥有同样优先级的线程有获取 `CPU` 执行时间的机会。

> 注意：此方法不会将线程进入阻塞状态，只会将线程重新置为可运行状态 `RUNNABLE` 。

##### join方法

`join` 方法有三个重载，此方法的作用是等待线程完成运行。

```java
public final void join() throws InterruptedException {
    join(0);
}
```

此方法是 `join` 的主要方法啊，其他的重载方法都是在调用此方法。

```java
public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis(); // 当前系统时间
    long now = 0;

    if (millis < 0) { // 指定的等待时间小于0，抛出参数异常错误
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) { // 指定的等待时间为0
        while (isAlive()) { // 当线程在存活状态时循环调用wait方法
            wait(0); // 使当前线程进入等待状态
        }
    } else {
        while (isAlive()) { // 若此线程存活
            // 将需要等待的时间减去已等待的时间得出还需等待的时间
            long delay = millis - now;
            // 当等待时间已经清零，直接退出循环
            if (delay <= 0) {
                break;
            }
            // 将调用此方法的线程进入等待状态
            wait(delay);
            // 将运行等待后的时间减去等待前的时间，得出已经等待的时间
            now = System.currentTimeMillis() - base; 
        }
    }
}
```

`wait` 方法是 `Object` 类中的方法，作用是将调用此方法的线程进入等待状态，`timeout` 为超时时间若超时时间为零，则一直等待知道通知唤醒。

```java
public final native void wait(long timeout) throws InterruptedException;
```

此方法的逻辑与 `sleep` 的同参方法逻辑相同，实际只有毫秒进入方法，纳秒没有实际使用。

```java
public final synchronized void join(long millis, int nanos) throws InterruptedException {
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
                            "nanosecond timeout value out of range");
    }
    if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
        millis++;
    }
    join(millis);
}
```

##### interrupt方法

此方法会设置线程的中断标志位，如果线程在 `sleep` 、`wait` 、`join` 处于阻塞状态时，线程会定时检查中断标志位若发现中断则会抛出 `InterruptedException` 异常，并在异常抛出后将中断标志位清除。若线程在运行状态或获取锁 `synchronized` `lock` 等，则会忽略中断。

```java
public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();

    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           // Just to set the interrupt flag
            b.interrupt(this);
            return;
        }
    }
    interrupt0();
}
```

```java
public final void checkAccess() {
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkAccess(this);
    }
}
```

##### suspend和resume方法

使用 `suspend` 挂起线程使用 `resume` 唤醒，由于此方法容易发生死锁，自 `jdk7` 已标记为弃用。

#### Callable 类

`Callable` 类和 `Runnable` 类一样是一个接口类，不同之处在于 `Callable` 的 `call` 方法拥有返回值，而 `Runnable` 的 `run` 方法没有返回值。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

#### Runnable 类

`Runnable` 类：

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```



#### 代码实例

##### 使用 Thread 类创建线程

```java
Thread thread = new Thread() {
    @Override
    public void run() {
        System.out.println("new thread with rewrite run method");
    }
};
thread.start();
```

此方法使用匿名内部类来继承 `Thread` 并重写其中的 `run` 方法，因为 `Thread` 是实现 `Runnable` 接口类，所以重写 `run` 方法后，使用 `start` 方法启动线程依旧可以实现相同的效果。

##### 使用 Runnable 传入 Thread 

```java
new Thread(new Runnable(){
    @Override
    public void run() {
        System.out.println("new Thread with runnable");
    }
}).start();
// lambda 简写
new Thread(() -> {
    System.out.println("new Thread with runnable");
}).start();
```

最普通的写法，使用 匿名内部类实例化 `Runnable` 接口，并将实例化的对象引用传入 `Thread` 对象中，最后调用 `start` 方法启动线程。


##### 使用线程池和 Callable

```java
ExecutorService service = Executors.newSingleThreadExecutor();
// 子线程执行内容
Future<String> result = service.submit(() -> {
    System.out.println("new thread with executors");
    Thread.sleep(2000L);
    return "return data";
});
// 获取返回值
String resultString = result.get();
System.out.println(resultString);
// 关闭线程池
service.shutdown();
```

`Callable` 和 `Runnable` 是不同的，而 `Thread` 中是无法传入 `Callable` 对象的，这时就需要用到一个新技术线程池，使用 `Executors` 新建一个线程池，最后使用线程池的 `submit` 方法传入 `Callable` 对象进行线程的创建和执行。关于线程池相关请查看[Java线程池](#线程池) 

## 线程安全及锁

### 线程安全

#### Java 内存模型

在 `java` 内存模型中内存分为两种，一种是主内存其中储存着所有线程共享的资源，另一种是线程工作内存，其中储存着线程需要用到的主存中的资源副本。

![48、多线程共享变量操作(7)](photo/48、多线程共享变量操作(7).png) 

在 `Java` 内存模型中规定，所有的资源都会被存放在主存中，如果线程需要使用指定的资源，就会将资源复制到指定线程的工作内存中，线程操作的也是自己的工作内存。

关于 `Java` 内存相关请查看： [JVM 内存](/编程语言/Java/JVM%20虚拟机/JVM%20内存) 

#### 线程安全三要素

##### 可见性

可见性就是要保证每一个线程在获取值时，必须获取到最新的值。

当一个线程修改其工作内存中某个资源副本时，并不会直接更新到主存中，此时主存中的值已经是旧值。这时另一个线程想要使用同样的资源进行操作是，无论是其工作内存中还是拷贝主存中的值，其获取的都是旧值。新的值只在第一个修改此资源的线程的工作内存中存在，其他线程是无法及时获取到最新的值。

解决方法：[Volatile 关键字](# Volatile) 

##### 有序性

程序的执行顺序需要按照代码顺序执行，因为在多线程环境下 `JMM` 为了性能优化，编译器和处理器会对无数据依赖的指令进行重排。

```java
int a = 1;
int b = 2;
int c = a + b;
```

上面的代码中 `c` 依赖 `a` 和 `b` 的值，但是 `a` 和 `b` 没有互相依赖，在单线程状态下1/2行的顺序并不影响计算结果，及有可能出现指令重排。

##### 原子性

原子性的含义就是在执行一系列的操作时，要么全部执行，要么不执行，不可以被中断或只执行一部分。

以自增举例，将 `m++` 反编译为字节码。

```
12: getfield      #2                  // Field b:I
15: iconst_1
16: iadd
17: putfield      #2                  // Field b:I
```

反编译代码后可以看到，`m++` 并不属于一个原子性操作，它的操作有4步：获取值压入栈、相加数压入栈、二者相加最后赋值。

```java
private static Integer m = 0;

public static void main(String[] args) throws InterruptedException {
    Thread[] threads = new Thread[100];

    for (int i = 0; i < threads.length; i++) {
        threads[i] = new Thread(()->{
            for (int i1 = 0; i1 < 100; i1++) {
                m++;
            }
        });
    }

    for (Thread thread : threads) {
        thread.start();
    }

    for (Thread thread : threads) {
        thread.join();
    }

    LOGGER.info("m的值：{}", m);
}
```

使用100个线程并发修改同一个值 `m` ，每个线程将 `m` 自增100次若程序正常执行最后 `m` 的值应该为10000。但是在实际执行中，通常 `m` 的值无法达到10000。

```shell
2022-01-31 17:09:35.071 [main] INFO  java.lang.Thread - m的值：8456
2022-01-31 17:11:42.616 [main] INFO  java.lang.Thread - m的值：9413
2022-01-31 17:11:50.083 [main] INFO  java.lang.Thread - m的值：9235
2022-01-31 17:12:00.287 [main] INFO  java.lang.Thread - m的值：9305
```

可以看到每次执行的结果都不相同，且结果并没有达到10000。

![image-20220131172820107](photo/50、多线程安全运行图(7).png) 

由上图可知，当线程 `AB` 从主内存中获取变量拷贝到线程内存中，此时在线程 `AB` 中的变量都是为0。此时线程 `A` 将 `m` 自增两次后同步到主内存，主内存中 `m` 为2。此时线程 `B` 也将 `m` 增加了三次，也将 `m` 同步到了主内存中，此时线程 `A` 同步的 `m` 将会被线程 `B` 的同步操作覆盖，也就相当于最后的值少了2，这也是上面代码执行结果最后总数小于10000的原因。

### 关键字

由 `Java` 封装好底层功能用于解决线程安全，实现线程安全三要素的相关关键字。

#### Synchronized

##### 概述

`Synchronized` 关键字底层是由系统的 `Mutex Lock` 实现的，其可对类和对象进行加锁，保证方法或者代码块中资源的互斥访问，即在同一时间及同一 `Monitor` 监视的代码，最多只能有一个线程在访问。

```java
class A{
    public void synchronized f1(){}
    public static void synchronized f2(){}
}
```

相当于

```java
class B{
    public void f1(){
        synchronized(this){}
    }
    public static void f2(){
        synchronized(B.class){}
    }
}
```

此关键字对于非静态方法，锁定的是对象实例。而对于静态方法锁定的是类，当然 `B.class` 也是一个对象（所有类都是 `Class` 类的实例化对象）。由于这两个方法锁定的对象不同，一个是 `B` 的实例另一个是 `Class` 的实例，所以，当分别被不同的线程调用并不会互斥。

##### 解析

```java
public class SyncTestClass {

    // 普通方法无同步
    public void TestMethod1(){
        int a = 10;
        System.out.println(a);
    }

    // 在方法上添加关键字
    public synchronized void TestMethod2(){
        int a = 10;
        System.out.println(a);
    }

    // 同步代码块
    public void TestMethod3(){
        synchronized (SyncTestClass.class){
            int a = 10;
            System.out.println(a);
        }
    }

}
```

将上面的代码的 `class` 文件使用 `javap -c <filename>` 命令反编译。

```assembly
public class com.mochen.advance.juc.sync.SyncTestClass {
  public com.mochen.advance.juc.sync.SyncTestClass();
    Code:
       0: aload_0
       1: invokespecial #1    // Method java/lang/Object."<init>":()V
       4: return


  public void TestMethod1();
    Code:
       0: bipush        10
       2: istore_1
       3: getstatic     #2    // Field java/lang/System.out:Ljava/io/PrintStream;
       6: iload_1
       7: invokevirtual #3    // Method java/io/PrintStream.println:(I)V
      10: return

  public synchronized void TestMethod2();
    Code:
       0: bipush        10
       2: istore_1
       3: getstatic     #2    // Field java/lang/System.out:Ljava/io/PrintStream;
       6: iload_1
       7: invokevirtual #3    // Method java/io/PrintStream.println:(I)V
      10: return

  public void TestMethod3();
    Code:
       0: ldc           #4    // class com/mochen/advance/juc/sync/SyncTestClass
       2: dup
       3: astore_1
       4: monitorenter
       5: bipush        10
       7: istore_2
       8: getstatic     #2    // Field java/lang/System.out:Ljava/io/PrintStream;
      11: iload_2
      12: invokevirtual #3    // Method java/io/PrintStream.println:(I)V
      15: aload_1
      16: monitorexit
      17: goto          25
      20: astore_3
      21: aload_1
      22: monitorexit
      23: aload_3
      24: athrow
      25: return
    Exception table:
       from    to  target type
           5    17    20   any
          20    23    20   any
}
```

可以看到 `TestMethod2` 在 `TestMethod1` 基础上增加了 `synchronized` 关键字，使用代码块方式则是使用 `monitorenter` 和 `monitorexit` 来实现同步功能。

关于同步方法和同步代码块的解释 来源：[JVM doc Synchronization](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.14) 

同步的实现方式有两种，一种是显式的使用 `monitorenter` 和 `monitorexit` 指令实现，另一种是隐式的使用方法的调用和返回来实现。

对于用Java编程语言编写的代码，可能最常见的同步形式是 `synchronized` 方法。 `synchronized` 方法通常不会使用 `monitorenter` 和 `monitorexit` 实现。它只是在运行时常数池中由`ACC_SYNCHRONIZED`标志来区分，该标志由方法调用指令检查。 可以把执行 `monitorenter` 指令理解为加锁，执行 `monitorexit` 理解为释放锁。

在Java的对象头中，有一块数据叫标记字 `Mark Word` 。在64位机器上，`Mark Word` 是8字节（64位）的，这64位中有2个重要字段：锁标志位和占用该锁的 `thread ID` 。且因为不同版本的 `JVM` 实现，对象头的数据结构会有各种差异。这个标记字就是 `synchronized` 关键字的实现原理，他可以存储该对象是否被锁定，以及当前占用资源的线程。

##### 特点

保证方法或代码块操作的原子性

在同步关键字的代码块或方法中内部资源互斥访问，同一时间只有一个线程在访问共享资源，从而保障资源操作的原子性。

保证监视资源的可见性

保证多线程环境下对监视资源的数据同步，即任何线程在获取到锁后的第一时间，会将共享内存中的数据复制到自己的工作内存中；并在线程释放锁时，会将自身私有内存中的数据复制到共享内存中。并且只有一个线程执行被锁定的代码，其他线程在此线程释放锁后获取到的资源一定是最新的。

> `synchronized` 无法解决有序性，无法阻止指令重排序。

#### Volatile 

上面使用 `synchronized` 关键字也可以解决资源的可见性，但是使用锁对性能损耗过大。当一个线程获取锁之后，其他线程必须等待其释放锁，这样的解决方案太过于笨重。所以，`synchronized` 通常也被称作重量级锁。

`volatile` 可以在减少性能损耗的前提下解决可见性问题，使用此关键字修饰的变量对其的修改会对其他线程保持可见。可以简单理解为，经过 `volatile` 修饰的关键字线程在操作时会直接修改主存中的值，且会将其他线程工作内存中的该变量标记为无效（ `MESI` 协议），其他线程需要用到此变量时会从主存中获取最新的值。

`volatile` 同时可以阻止指令重排，被其修饰的变量在写 `volatile` 变量时，写入之前的指令不会被重排到写入之后；读 `volatile` 变量时，读之后的操作不会被重排到读之前。

```java
private volatile int a = 0;
```

> `volatile` 并不能解决原子性问题。

### CAS

#### 概述

上面介绍了 `m++` 其实分为三步（读、计算和写）可以使用 `synchronized` 关键字解决，但是`synchronized` 缺点是性能消耗较大，而`volatile` 是无法解决原子性问题的。由于这两个原因 `JDK` 提供了 `Unsaft` 类用作高效解决读改写的原子性问题。

`CAS` 全称 `Compare and Swap` 比较和交换，用作对资源的读取和赋值。通常一个 `CAS` 方法有四个参数，分别是指定对象 `Object` 、对象中的变量 `valueOffset` 、变量的预期值 `expect` 和要更新的值 `update` 组成。

> `CAS` 又称无锁操作，是一种乐观锁设计。在多线程访问下共享变量不会被加锁，线程之间不会阻塞排队，也不会被挂起。使用循环对比的方法，直到成功访问修改。看似这种方法消耗资源，其实 `CAS` 底层全部由本地方法实现，效率极高，且并没有同步锁使用系统的 `Mutex Lock` 时，用户态转内核态的资源消耗，因此 `CAS` 是一种非常高效的解决原子性问题的方法。

#### Unsafe 类方法

`JDK` 的 `rt.jar` 包中的 `Unsafe` 类提供了系统级别的原子性操作，因为其是直接调用本地方法实现，即 `native` 方法，使用 `JNI` 的方式调用本地 `C++` 函数。

- `long objectFieldOffset（Field field）` 方法：返回指定的变量在所属类中的内存偏移地址，该偏移地址仅仅在该 `Unsafe` 函数中访问指定字段时使用。
- `int arrayBaseOffset（Class arrayClass）` 方法：获取数组中第一个元素的地址。
- `int arrayIndexScale（Class arrayClass）` 方法：获取数组中一个元素占用的字节。
- `boolean compareAndSwapLong（Object obj, long offset, long expect, long update）` 方法：比较对象 `obj` 中偏移量为 `offset` 的变量的值是否与 `expect` 相等，相等则使用 `update` 值更新，然后返回 `true` ，否则返回 `false` 。
- `public native long getLongvolatile（Object obj, long offset）` 方法：获取对象 `obj` 中偏移量为 `offset` 的变量对应 `volatile` 语义的值。
- `void putLongvolatile（Object obj, long offset, long value）`方法：设置 `obj` 对象中 `offset` 偏移的类型为 `long` 的 `field` 的值为 `value` ，支持 `volatile` 语义。
- `boolean compareAndSwapLong（Object obj, long offset, long expect, long update）`方法：比较对象 `obj` 中偏移量为 `offset` 的变量的值是否与 `expect` 相等，相等则使用 `update` 值更新，并返回 `true` ，否则返回 `false` 。
- `public native long getLongvolatile（Object obj, long offset）`方法：获取对象 `obj` 中偏移量为 `offset` 的变量对应 `volatile` 语义的值。
- `void putLongvolatile（Object obj, long offset, long value）`方法：设置 `obj` 对象中 `offset` 偏移的类型为 `long` 的 `field` 的值为 `value` ，支持 `volatile` 语义。
- `void putOrderedLong（Object obj, long offset, long value）` 方法：设置 `obj` 对象中 `offset` 偏移地址对应的 `long` 型 `field` 的值为 `value` 。这是一个有延迟的 `putLongvolatile` 方法，并且不保证值修改对其他线程立刻可见。只有在变量使用 `volatile` 修饰并且预计会被意外修改时才使用该方法。
- `void park（boolean isAbsolute, long time）` 方法：阻塞当前线程，其中参数 `isAbsolute` 等于 `false` 且 `time` 等于0表示一直阻塞。 `time` 大于0表示等待指定的 `time` 后阻塞线程会被唤醒，这个 `time` 是个相对值，是个增量值，也就是相对当前时间累加 `time` 后当前线程就会被唤醒。如果 `isAbsolute` 等于 `true` ，并且 `time` 大于0，则表示阻塞的线程到指定的时间点后会被唤醒，这里 `time` 是个绝对时间，是将某个时间点换算为 `ms` 后的值。另外，当其他线程调用了当前阻塞线程的 `interrupt` 方法而中断了当前线程时，当前线程也会返回，而当其他线程调用了 `unPark` 方法并且把当前线程作为参数时当前线程也会返回。
- `void unpark（Object thread）` 方法：唤醒调用 `park` 后阻塞的线程。

`JDK8` 新增内容：

- `long getAndSetLong（Object obj, long offset, long update）` 方法：获取对象 `obj` 中偏移量为 `offset` 的变量 `volatile` 语义的当前值，并设置变量 `volatile` 语义的值为 `update` 。

```java
public final long getAndSetLong(Object var1, long var2, long var4) {
    long var6;
    do {
        var6 = this.getLongVolatile(var1, var2);
    } while(!this.compareAndSwapLong(var1, var2, var6, var4));
    return var6;
}
```

首先获取指定 `offset` 的变量内容，再使用 `compareAndSwapLong` 来修改值。

> 使用 `while` 循环，在失败时重复尝试。

- `long getAndAddLong（Object obj, long offset, long addValue）` 方法：获取对象 `obj` 中偏移量为 `offset` 的变量 `volatile` 语义的当前值，并设置变量值为原始值 `+addValue` 。

其他类似的操作例如 `getAndSetInt` 等也与以上方法相同。

#### 使用 Unsafe

在 `Unsafe` 类中有 `getUnsafe` 方法，可以获取到 `Unsafe` 实例，但在运行时会报错。

```java
Exception in thread "main" java.lang.SecurityException: Unsafe
```

```java
@CallerSensitive
public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
        throw new SecurityException("Unsafe");
    } else {
        return theUnsafe;
    }
}
```

可以看到 `getUnsafe` 方法中检查了调用此方法类的类加载器，判断其是否为 `Bootstrap` 类加载器加载的类。但是我们在调用时使用的是自己编写的类，是由 `AppClassLoader` 加载的。所以，此处在调用 `getUnsafe` 时会报错。

由于 `Unsafe` 类中的方法都是直接操作内存的，所以它就像它的名字一样是不安全的。所以，在设计时要避免开发人员使用此类，而是只有 `rt.jar` 核心类才可以使用这些方法。

使用万能的反射来重新编码之前多线程安全的问题代码。

```java
public class BasicTestClass implements BasicLogger {

    private static volatile int m = 0; // 将m使用volatile修饰保证可见性
    private static Unsafe unsafe = null; // unsafe实例

    public static void main(String[] args) throws Exception {
        Thread[] threads = new Thread[100]; // 100个线程
        // 获取m的field对象
        Field field = BasicTestClass1.class.getDeclaredField("m");
        // 使用反射回去theUnsafe字段
        Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
        theUnsafe.setAccessible(true); // 解除安全检查
        unsafe = (Unsafe) theUnsafe.get(null);
        // 获取静态变量m在对象中的偏移值
        long offset = unsafe.staticFieldOffset(field);

        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int i1 = 0; i1 < 100; i1++) {
                    // m++
                    unsafe.getAndAddInt(BasicTestClass1.class, offset, 1);
                }
            });
        }
        for (Thread thread : threads) {
            thread.start();
        }
        for (Thread thread : threads) {
            thread.join();
        }
        LOGGER.info("m的值：{}", m);
    }
}
```

使用 `Unsafe` 类中的方法和 `volatile` 关键字，完美解决了之前出现的问题。

#### ABA 问题

##### 原因

在使用 `CAS` 时不免会有一个问题，如果一个线程 `A` 获取到了变量的值，此时线程 `B` 将值从1修改为2，然后在 `A` 进行下一步操作之前又再次将变量的值从2修改回1。那么此时的线程 `A` 再去比对时，变量的值看似没有变化，但是已经被修改了两次。

##### 解决方法

如何避免出现 `ABA` 问题，这时就要用到 `AtomicStampedReference` 类。此类就相当于在对象上添加了一个标记戳，用此来判断其是否有过修改。

基础方法

```java
//构造方法, 传入引用和戳
public AtomicStampedReference(V initialRef, int initialStamp)
//返回引用
public V getReference()
//返回版本戳
public int getStamp()
//如果当前引用 等于 预期值并且 当前版本戳等于预期版本戳, 将更新新的引用和新的版本戳到内存
public boolean compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp)
//如果当前引用 等于 预期引用, 将更新新的版本戳到内存
public boolean attemptStamp(V expectedReference, int newStamp)
//设置当前引用的新引用和版本戳
public void set(V newReference, int newStamp)
```

测试

```java
AtomicStampedReference<String> reference = new AtomicStampedReference<String>("First", 1);
LOGGER.info("原值：{}", reference.getReference());
reference.compareAndSet("First", "Second", reference.getStamp(), reference.getStamp() + 1);
LOGGER.info("首次修改值：{}", reference.getReference());
boolean editStampResult = reference.attemptStamp("Second", reference.getStamp() + 1);
LOGGER.info("修改标记是否成功：{}", editStampResult);
LOGGER.info("修改后的标记：{}", reference.getStamp());
boolean endEditResult = reference.compareAndSet("Second", "Over", 4, reference.getStamp() + 1);
LOGGER.info("最后修改是否成功：{}", endEditResult);
LOGGER.info("最后修改值：{}", reference.getReference());
```

结果

```
2022-02-14 14:18:02.124 [main] INFO  java.lang.Thread - 原值：First
2022-02-14 14:18:02.128 [main] INFO  java.lang.Thread - 首次修改值：Second
2022-02-14 14:18:02.128 [main] INFO  java.lang.Thread - 修改标记是否成功：true
2022-02-14 14:18:02.128 [main] INFO  java.lang.Thread - 修改后的标记：3
2022-02-14 14:18:02.128 [main] INFO  java.lang.Thread - 最后修改是否成功：false
2022-02-14 14:18:02.128 [main] INFO  java.lang.Thread - 最后修改值：Second
```

除了 `AtomicStampedReference` 类，还有 `AtomicMarkableReference` 只可以使用布尔值，表示是否被修改过。

> 注意：`AtomicMarkableReference` 并不可以完全避免 `ABA` 问题出现，只能降低出现的几率。

#### Atomic 包

##### 概述

`atomic` 包位于 `java.util.concurrent.atomic` 之下，其中有许多具有原子性的类。例如 `AtomicLong` 、`AtomicInteger` 等类，与 `synchronized` 和锁的原理不同，这些原子类都是由 `CAS` 实现。

![image-20220214144738653](photo/49、Atomic包内容(7).png) 

> 注：上方的 `AtomicStampedReference` 和 `AtomicMarkableReference` 都是 `Atomic` 中的一员。

##### 解析

打开 `AtomicInteger` 源码，可以看到成员变量 `unsafe` 。

```java
// Unsafe 对象
private static final Unsafe unsafe = Unsafe.getUnsafe();
// value 变量在此对象中的偏移值
private static final long valueOffset;
// 真正储存数据的对象
private volatile int value;
```

初始化 `value` 的偏移值。

```java
static {
    try {
        // 获取value的Field对象，使用Field对象获取到value的偏移值
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
```

构造方法，可以赋默认值。

```java
public AtomicInteger(int initialValue) {
    value = initialValue;
}
```

以下方法等都是使用 `CAS` 方法实现。

```java
public final int getAndSet(int newValue) {
    return unsafe.getAndSetInt(this, valueOffset, newValue);
}

public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}

//...
```

##### 使用

使用原子类来解决线程安全问题示例代码。

```java
public class AtomicTestClass implements BasicLogger {

    private static AtomicInteger m = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        Thread[] threads = new Thread[100];
        for (int i = 0; i < 100; i++) {
            Thread thread = new Thread(() -> {
                for (int j = 0; j < 10000; j++) {
                    m.incrementAndGet();
                }
            });
            threads[i] = thread;
        }
        for (int i = 0; i < 100; i++) {
            threads[i].start();
        }
        for (int i = 0; i < 100; i++) {
            threads[i].join();
        }
        LOGGER.info("结果：{}", m);
    }
}
```

结果

```
2022-02-14 15:08:39.809 [main] INFO  java.lang.Thread - 结果：1000000
```

可以看到使用原子类后结果没有问题。

### AQS

`AQS` 全称 `AbstractQueuedSynchronizer` （抽象队列同步器），`AQS` 定义了一套多线程访问共享资源的同步器框架，许多同步类实现都基于 `AQS` 的理念，例如 `ReentrantLock` 等。

#### 概述

**`CLH` 队列锁** 

`CLH` 锁是有由 `Craig` ,  `Landin` ,  `Hagersten` 这三个人发明的锁，取了三个人名字的首字母，所以叫 `CLH` 锁。

`CLH` 队列锁也是一种基于**双向链表**的可扩展、高性能、公平的自旋锁，申请线程仅仅在本地变量上自旋，它不断轮询前驱的状态，发现前驱释放了锁就结束自旋。

`AQS` 类只是一个框架，展示了队列锁的设计方法。其中使用一个 `volatile int state` 来表示共享资源，多线程通过争抢此资源阻塞时进入队列来实现队列锁的效果。

其中，关于 `sate` 变量的修改都是使用 `CAS` 的方式，从而保证其在多线程下的安全性。

#### 源码详解

##### waitStatus属性

```java
volatile int waitStatus;
```

下面是已经定义好的状态

```java
/** waitStatus value to indicate thread has cancelled */
static final int CANCELLED =  1;
/** waitStatus value to indicate successor's thread needs unparking */
static final int SIGNAL    = -1;
/** waitStatus value to indicate thread is waiting on condition */
static final int CONDITION = -2;
/** waitStatus value to indicate the next acquireShared should unconditionally propagate */
static final int PROPAGATE = -3;
```

- `CANCELLED 1` ：表示当前结点已取消调度。当timeout或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的结点将不会再变化。	
- `SIGNAL -1` ：表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为 `SIGNAL` 。
- `CONDITION -2` ：表示结点等待在 `Condition` 上，当其他线程调用了 `Condition` 的 `signal()` 方法后， `CONDITION` 状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
- `PROPAGATE -3` ：共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
- `0` ：新结点入队时的默认状态。

> 负值为节点处于有效状态，正值则表示节点已经取消。

##### Node类中其他属性

```java
volatile Node prev; // 上一个node
volatile Node next; // 下一个node
volatile Thread thread; // 当前节点储存的线程，实例化Node时传入，使用后置空。
```

由此可见， `Node` 其实是一个双向链表。

![image-20220125095028713](photo/45、CLH队列Node图示1(7).png)

##### AQS中的head和tail属性

```java
private transient volatile Node head; // 指向线程链表的第一个node
private transient volatile Node tail; // 指向线程链表的最后一个node
```

![image-20220125104832047](photo/46、CLH队列Node图示2(7).png)

##### acquire(int)方法详解

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && // 直接尝试获取资源
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt(); // 中断当前线程
}
```

在独占模式下获取资源，若获取到资源直接返回线程，否则进入等待队列直到获取到资源后为止，整个过程忽略中断的影响。

###### tryAcquire方法


```java
// 此方法用户尝试以独占模式获取，成功为true
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

使用此方法直接去获取资源，成功返回 `true` 。此处因为 `AQS` 只是一个框架，所以此处没有真正的实现，需要在自定义同步器中去实现。

> 既然 `AQS` 类只是一个框架，为什么没有被定义为 `abstract` ？
>
> 因为独占模式只需要实现 `tryAcquire` `tryRelease` ，共享模式只需要实现 `tryAcquireShared` `tryReleaseShared` ，如果定义为抽象类每个模式都需要去实现另一个模式下的接口，所以为了避免一些不必要的代码所以定义为一个普通类。

###### addWaiter方法


```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode); // 包装为node
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) { // 队列尾部不为空
        node.prev = pred; // 当前node中prev指向目前的队尾node
        if (compareAndSetTail(pred, node)) { // 将tail替换为当前node
            pred.next = node; // 将之前的队尾node的next指向当前的node
            return node; // 返回当前node
        }
    }
    enq(node); // 自旋，直到成功添加node
    return node; // 返回当前node
}
```

将当前的线程加入等待队列的末尾，并返回当前线程生产的节点。传入的 `mode` 是一个空值，表示当前的模式为独占模式，在此方法中 `Node` 类中的 `nextWaiter` 参数会等于空。

###### enq方法

```java
private Node enq(final Node node) {
    for (;;) { // 自旋
        Node t = tail;
        if (t == null) { // 如果当前尾部指向为null，则表示没有初始化
            if (compareAndSetHead(new Node())) // 将head和tail设置为同一个空node
                tail = head;
        } else { // 以下代码与addWaiter中添加节点部分代码相同
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

自旋循环，若 `tail` 指向的尾部为空时，将头和尾都设置为同一个空的 `node` ，若 `tail` 指向的 `node` 不为空，则执行与 `addWaiter` 中添加节点相同的代码进行添加，最后返回当前的 `node` 。

###### acquireQueued方法


```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true; // 是否成功拿到资源
    try {
        boolean interrupted = false; // 等待过程中是否有中断
        for (;;) {
            final Node p = node.predecessor(); // 获取当前节点的上一个节点
            // 当前节点的上一个节点是头部节点并且尝试去获取资源
            if (p == head && tryAcquire(arg)) { 
                setHead(node); // 获取到资源后，将当前节点设置为头部节点
                p.next = null; // 将之前的头部节点进行置空方便垃圾回收
                failed = false; // 已经成功拿到资源了，重新设置一下标记
                return interrupted; // 将是否有中断返回
            }
            if (shouldParkAfterFailedAcquire(p, node) && // 线程应该阻塞是为true
                parkAndCheckInterrupt()) // 当线程需需要阻塞时，将线程阻塞进入等待状态
                interrupted = true; // 将中断设置为true
        }
    } finally {
        if (failed) // 获取资源失败
            cancelAcquire(node); // 取消节点在队列的等待
    }
}
```

进入这一步的线程已经使用 `addWaiter` 方法将当前线程放入了等待队列中，此方法需要做的就是等待队列中当前线程之前的线程执行完毕后唤醒自己。所以，代码中使用了自旋，不断获取当前线程所属节点的上一个节点，如果上一个节点变成了头结点，说明当前节点时等待队列中的而第二个节点。随即使用 `tryAcquire` 方法去尝试获取资源，若获取到资源则将 `head` 修改为当前节点，并返回是否存在中断。如果当前节点不是第二个节点，或者当前线程没有获取到资源，则会使用 `shouldParkAfterFailedAcquire` 方法清理当前节点前已经失效的节点，并使用 `parkAndCheckInterrupt` 阻塞线程并返回是否存在中断。当目前节点的上一个节点状态处于 `SIGNAL` （需要唤醒下一级节点），并且线程存在中断时，会将 `interrupted` 设置为 `true` 后继续循环。

1. 节点加入队尾后，检查状态并使用 `park` 阻塞线程等待唤醒。
2. 被唤醒后，尝试获取资源。
   1. 获取成功，将 `head` 指向此节点，并返回线程是否存在中断。
   2. 获取失败，继续循环执行阻塞，等待唤醒。

###### shouldParkAfterFailedAcquire方法

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus; // 获取当前节点上一级节点的状态
    if (ws == Node.SIGNAL) // 当上一级节点的状态，标记为需要唤醒当前节点时直接返回true
        return true;
    if (ws > 0) { // 上一级节点已经为取消状态
        do {
            node.prev = pred = pred.prev; // 将取消状态的节点从链表中删除
        } while (pred.waitStatus > 0); // 循环删除知道遇到不为取消状态的节点为止
        pred.next = node; // 将删除节点的上一级节点中的next指向当前节点
    } else { // 小于等于0不定于-1时
        // 更新上一级节点中的waitStatus的值为-1，安全修改，以防上一级节点刚刚取消
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

如果上一个节点的状态不为 `SIGNAL` ，则会将上一级节点修改为 `SIGNAL` 。如果上一级节点状态为已取消，则会将节点删除到非取消状态的节点。

`parkAndCheckInterrupt` 方法：

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this); // 将线程进入等待状态
    return Thread.interrupted(); // 如果被唤醒，检查是否为中断
}
```

在被 `park` 方法进入 `waiting` 状态下的线程中，可以被 `unpark` 和 `interrupt` 方法唤醒。

> 需要注意的是， `Thread.interrupted()` 会清除当前线程的中断标记位，若中断的线程在第一调用时返回 `true` 在第二次调用时则会返回 `false` 。

###### cancelAcquire方法

```java
private void cancelAcquire(Node node) {
    if (node == null) // 忽略空值
        return;

    node.thread = null; // 将节点中储存的线程置空 help GC

    Node pred = node.prev;
    while (pred.waitStatus > 0) // 删除已经成为取消状态的上级节点
        node.prev = pred = pred.prev;

    Node predNext = pred.next;
    node.waitStatus = Node.CANCELLED; // 将节点设置为取消状态

    // 当尾结点是传入的节点时，将tail设置为传入node的上一级node
    if (node == tail && compareAndSetTail(node, pred)) {
        compareAndSetNext(pred, predNext, null); // 设置尾结点成功后，将尾结点的next指向空
    } else {
        
        int ws;
        if (pred != head && // 传入节点的上一级节点不是head
            ((ws = pred.waitStatus) == Node.SIGNAL || // 上一级节点的状态为-1
             // 节点状态不为取消，且将上一级节点的waitStatus装填修改为-1成功
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) { // 上一级节点中储存的线程不为空
            Node next = node.next;
            // 传入node的下一级不为空且状态不为取消状态
            if (next != null && next.waitStatus <= 0)
                // 将上一级node的next指向传入node的next指向的节点，也就是将传入节点从链表中删除
                compareAndSetNext(pred, predNext, next);
        } else { // 传入节点的上一级节点是head时
            unparkSuccessor(node);
        }

        node.next = node; // help GC
    }
}
```

此方法用于取消删除自旋锁非正常结束且没有正常获取到资源的节点，并清除传入节点之前所有为已取消状态的节点。

###### unparkSuccessor方法

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus; // 获取传入节点的状态
    if (ws < 0)
        // 节点为取消状态时，将节点状态设置为新节点默认状态
        compareAndSetWaitStatus(node, ws, 0);
    Node s = node.next; // 获取到下一级节点
    // 当下一级节点为空或者状态为取消时
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 初始变量：t为尾结点；循环条件：t不为空且不为当前传入节点；结束操作：将t指定为当前遍历节点的上一个节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0) // 如果遍历节点状态不为取消
                s = t;
    }
    if (s != null)
        // 若传入node的下一级node不为空且状态正常，或者传入node之后有不为空且状态正常的node
        // 则将这个node中的线程唤醒
        LockSupport.unpark(s.thread);
}
```

###### 总结acquire方法

1. 使用 `tryAcquire` 尝试获取资源，若获取到直接返回。
2. 使用 `addWaiter` 方法将当前线程包装为节点加入等待队列中，并标记为独占模式。
3. 使用 `acquireQueued` 方法使线程进入等待状态，并在唤醒后尝试获取资源，获取到资源后返回线程是否存在中断。
4. 若存在中断，则使用 `selfInterrupt` 进行中断线程。

> 当线程在等待状态中被中断，线程是不会响应的，只有在 `acquireQueued` 获取到了资源将是否存在中断返回后，才会由 `acquire` 方法中的 `selfInterrupt` 方法执行中断。

流程图：

![image-20220125172503874](photo/47、acquire方法流程图(7).png)

##### release方法

此方法时独占模式下线程释放共享资源的方法，他会释放指定量的资源彻底释放则将 `state=0` ，并唤醒等待队列中的其他线程来获取资源。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

> 此方法是使用返回来确定该线程是否已经完成对资源的释放。

###### tryRelease方法

尝试释放资源，需要自定义同步器来实现，一般情况下此线程都会成功释放资源，因为在执行 `release` 方法时线程已经获取到了资源。

```java
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

###### unparkSuccessor方法

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus; // 获取当前节点状态
    if (ws < 0) // 状态不为取消和新创建
        compareAndSetWaitStatus(node, ws, 0); // 将节点状态设置为新创建

    Node s = node.next; // 获取下一级节点
    if (s == null || s.waitStatus > 0) { // 节点为空或状态为已取消时
        s = null; //将当前节点置空
        // 从后向前查找不为空且不可为当前节点的节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0) // 状态不为已取消时
                s = t;
    }
    if (s != null) // 当获取到当前节点之后的节点不为空时
        LockSupport.unpark(s.thread); // 将线程恢复运行
}
```

在此方法正常执行后，唤醒的节点就会继续执行 [acquireQueued方法](# acquireQueued方法) ，此时当唤醒的线程是队列中的第二个线程时，`acquireQueued` 方法中的获取资源和将唤醒的节点置为 `head` 的操作就会执行。若是从后到前查找并唤醒的节点不为当前队列的第二个节点时，唤醒的线程就会在 `parkAndCheckInterrupt` 方法处继续阻塞。

##### acquireShared(int)方法

`acquireShared` 方法和独享模式的 `acquire` 方法流程基本相同，唯一不同的是当前资源已经被之前线程使用一部分后剩下的资源依旧足够下一个线程使用，则会唤醒下一个线程。这也是独享模式和共享模式最大个区别，独享模式同时只有 `head` 的线程在运行，而共享模式可能会同时运行多个线程。

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0) // 获取资源
        doAcquireShared(arg); // 添加到等待队列
}
```

第一步依旧是尝试获取资源，不过与独显模式不同的是 `tryAcquireShared` 方法返回的是一个 `int` 值。负数表示获取资源失败，0表示获取成功但是没有多余的资源，正数表示获取成功并且有剩余的资源其他线程还可以获取资源。第二步则是线程获取资源失败后将线程加入等待队列。

###### tryAcquireShared方法

以共享模式获取资源的方法依旧是需要自定义同步器去实现。

```java
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}
```

###### doAcquireShared方法

此方法用于等待和被唤醒后获取资源。

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED); // 将线程包装为node添加到队尾
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) { // 自旋
            final Node p = node.predecessor(); // 获取当前node上一个node
            if (p == head) { // 上一级node是否为head
                int r = tryAcquireShared(arg); // 如果当前node为第二个node则尝试获取资源
                if (r >= 0) { // 若获取成功
                    setHeadAndPropagate(node, r); // 将当前node设置为head
                    p.next = null; // help GC
                    if (interrupted) // 如果存在中断
                        selfInterrupt(); // 中断线程
                    failed = false;
                    return;
                }
            }
            // 此方法内容参见上方解释，删除当前node前失效node
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 阻塞并返回是否有中断
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed) // 非正常结束
            cancelAcquire(node); // 删除当前及之前的无效节点
    }
}
```

使用自旋保证每一次被唤醒后都检查节点状态，当节点为第二个节点时尝试去获取资源，若不满足条件会清除无效节点继续进入阻塞等待状态以便下一次唤醒，当自旋时非正常结束且没有正常获取到资源时，则会将本节点及之前的无效节点全部删除。

###### setHeadAndPropagate方法

此方法用于设置head指向的节点，在资源还有剩余时唤醒下一个节点

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // 获取head
    setHead(node); // 将head指向当前节点，且将thread和prev置空
    // 剩余资源数大于0 或 head不为空 或 head状态不为新建和取消
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next; // 获取到当前节点的下一级节点
        // 下一级节点为空或节点为共享模式
        if (s == null || s.isShared())
            doReleaseShared(); // 继续唤醒下一级节点
    }
}
```

首先先将传入的节点设置为头结点，并删除其中保存的线程和上一个 `node` 引用删除，并检查剩余资源唤醒下一节点。

##### releaseShared方法

释放线程占用的资源，其中的 `tryReleaseShared` 方法依旧需要具体实现类来实现。

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

###### doReleaseShared方法

此方法用于释放共享资源。

```java
private void doReleaseShared() {	
    for (;;) {
        Node h = head; // 获取head
        // head不为空且不为队尾，即队列中不是只有一个节点
        if (h != null && h != tail) {
            int ws = h.waitStatus; // 获取头部状态
            if (ws == Node.SIGNAL) { // 当head状态为SIGNAL -1
                // 将状态从SIGNAL替换为新建状态 0
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h); // 替换成功，唤醒队列中head后面的线程
            }
            // head状态为新建，且将head状态从0替换到PROPAGATE时失败，continue重启循环
            else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        // 若head未变化则中断循环
        if (h == head)                   // loop if head changed
            break;
    }
}
```

获取到 `head` 当状态满足时间队列中后继节点唤醒

#### ReentrantLock

##### 概述

`ReentrantLock` 是一个可重入的互斥锁，可重入是指一个线程可以多次获取同一个锁，互斥锁指当一个线程获取到锁之后，其他的线程必须等待获取到锁的线程释放锁。`ReentrantLock` 锁与 `synchronized` 关键字的作用相同，但 `ReentrantLock` 在更灵活可以响应中断、超时、尝试性获取锁等操作；可以通过构造传参来使用公平锁或者非公平锁；且可以关联多个条件队列。

##### 公平锁与非公平锁

###### 公平锁

公平锁可以简单理解为排队，最先进入的线程会获取到资源，而在之后进入的线程必须进入等待队列。当拥有资源的线程释放资源时，会唤醒在其后进入队列的线程，让其获取到资源。当拥有资源的线程在释放资源后，若此时有线程进入检测到可以获取资源，但是队列中有等待的线程时，刚进入的线程就会进入队列尾并阻塞等待唤醒。

下面是公平锁的图解，首先线程A进入，`state` 为0可以获取资源，队列中也没有线程，直接获取资源。

![image-20220215090207197](photo/51、公平锁流程01(7).png) 

线程B进入，`state` 无法获取资源，线程进入队列中阻塞等待。

![image-20220215090358355](photo/52、公平锁流程02(7).png) 

线程A释放资源，并唤醒队列中的线程B，让你尝试获取资源。

![image-20220215090723248](photo/53、公平锁流程03(7).png) 

此时一个新的线程C进入，`state` 为0可以获取资源，但此时队列中还有一个等待的线程B，于是线程C也进入等待队列。

![image-20220215090907835](photo/54、公平锁流程04(7).png) 

被唤醒的线程B开始获取资源，获取资源后将其从等待队列中删除。

![image-20220215091039479](photo/55、公平锁流程05(7).png) 

这就是公平锁的运行流程，具体 `ReentrantLock` 是如何实现公平锁与非公平锁，详见：[ReentrantLock 公平锁与非公平锁](# ReentrantLock 公平锁与非公平锁) 。

###### 非公平锁

非公平锁可以简单理解为新加入的线程都可以插一次队，当最先进入的线程获取到资源之后，再次有线程进入，此时该线程会尝试获取资源。若无法获取到资源则会进入等待队列队尾并阻塞，当拥有资源的线程释放线程后会唤醒在队列中最前的线程，让唤醒的线程尝试获取资源。但是在被唤醒的线程尝试获取资源之前，又有一个新的线程进入此时新的线程就会插队，新进入的线程若获取到资源，被唤醒的线程则会被继续阻塞。若被唤醒的线程获取到资源，新进入的线程则会进入等待队列队尾并阻塞。

线程A进入，检查 `state` 之后直接获取资源。

![image-20220215163233573](photo/57、非公平锁流程01(7).png) 

线程B进入，发现资源已经被获取，进入队列中等待。

![image-20220215163406742](photo/58、非公平锁流程02(7).png) 

线程A释放资源，并唤醒队列中的线程B，等待其获取资源。

![image-20220215163511621](photo/59、非公平锁流程03(7).png) 

此时，线程A已经释放资源，而线程B还没有获取到资源时，线程C进入，检查 `state` 之后直接获取资源，进行插队操作，而线程B发现资源无法获取继续在队列中进入阻塞状态。

![image-20220215163709493](photo/60、非公平锁流程04(7).png) 

##### 源码解析

###### ReentrantLock 公平锁与非公平锁

首先，在实例化 `ReentrantLock` 对象时可以通过传参指定公平锁或非公平锁。

```java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

默认实例化的是非公平锁，可以通过传 `boolean` 值来判断是否为公平锁。

我们先看公平锁 `FairSync` 类，此类是 `ReentrantLock` 中的一个静态内部类，非公平锁 `NonfairSync` 也是如此。其继承了 `Sync` 对象，下图是这些类的继承关系。

![image-20220215092252307](photo/56、ReentrantLock继承关系(7).png) 

可以看到本质获取资源等方法是调用了 `AQS` 类中已经定义好的方法，在 `FairSync` 和 `NonfairSync` 只是定义了 `AQS` 类中没有实现的方法。

**公平锁** 

首先先查看一下公平锁的 `lock` 方法，`ReentrantLock` 的获取锁操作就是 `lock` 方法。

```java
final void lock() { tes
    acquire(1);
}
```

`lock` 方法中调用了 `AQS` 类中的 `acquire` 方法，此方法在之前已经分析过，详见：[acquire方法](# acquire(int)方法详解) 。其中首先调用了 `tryAcquire` 方法，用于尝试获取资源，`tryAcquire` 方法的具体实现就在 `FairSync` 方法中。

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread(); // 获取当前线程对象
    int c = getState(); // 获取state的值
    if (c == 0) { // 可以获取到资源
        // 查看队列中是否有等待的线程，如果没有等待线程直接获取资源
        if (!hasQueuedPredecessors() &&
            // 修改state的值，获取资源
            compareAndSetState(0, acquires)) {
            // 将当前线程设置为拥有资源的线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) { // 拥有资源的线程就是当前线程
        // 此处的方法是实现锁的可重入，当一个线程多次获取资源时state将自增
        int nextc = c + acquires;
        // int溢出之后会变成负值，所以需要判断是否溢出
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        // 设置state
        setState(nextc);
        return true;
    }
    // 无法获取到资源，或者可以获取资源但有等待的线程
    return false;
}
```

公平锁的 `tryAcquire` 方法中，无论是否可以获取资源，只要线程中有等待的线程就会返回 `false` 。并由之后的 `addWaiter` 方法和 `acquireQueued` 方法，将线程加入队列并阻塞等待唤醒。

**非公平锁** 

非公平锁的 `tryAcquire` 方法调用了 `Sync` 类中的 `nonfairTryAcquire` 方法来实现。

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread(); // 获取当前线程
    int c = getState(); // 获取state资源值
    if (c == 0) { // 可以获取到资源
        // 直接获取资源
        if (compareAndSetState(0, acquires)) {
            // 设置拥有资源线程为当前线程
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 可重入设计，与公平锁相同
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // 未获取到资源，之后进入等待队列
    return false;
}
```

非公平锁与公平锁的区别就是在 `state` 为0时，公平锁会检查队列中是否存在等待的线程，而非公平锁则不会这样做，直接插队尝试去获取资源。

> 可重入锁：即可以对资源多次加锁，可以在 `tryAcquire` 方法中看到，即是对 `state` 进行自增操作。同时，在释放锁时也需要多次释放，因为 `state` 不为0，则其他线程无法获取到锁。

由上面的介绍可以得出，使用公平锁所有的线程都会按照先来先得的顺序依次获取到资源，没有线程会一直获取不到资源处于阻塞，但是线程的唤醒操作是比较消耗性能的，因此在线程数较多的情况下会因为不停的唤醒线程导致性能和吞吐量下降。非公平锁可以进行插队操作，在一个线程释放锁时另一个线程刚好进入，此时新进入的线程会直接去获取锁。由此减少了唤醒的消耗，同时也增加了吞吐量，但是在不断有新线程进入的情况下，队列中的线程有可能长时间无法获取锁而导致其一直处于阻塞状态。




## FutureTask

### FutureTask 是什么

要了解 `FutureTask` 是什么首先要了解这个类是为了解决什么问题，现在有一个场景，我们需要请求一个接口而这个网络请求是耗时操作，我们需要用到多线程技术，在请求发出后的一段时间之内我们需要在主线程中知道这个接口是否返回了数据，是否成功等情况。这时就需要用到 `FutureTask` 来实现这个需求，在此类中有很多方法可以让我们获取到线程的状态以及去操作线程。

![image-20220120101305193](photo\36、FutureTask方法列表(7).png)

### 使用 FutureTask

```java
public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
    FutureTask<String> futureTask = new FutureTask<>(new Callable<String>() {
        @Override
        public String call() throws Exception {
            LOGGER.info("start thread");
            Thread.sleep(2000L);
            return "This is result data";
        }
    });
    new Thread(futureTask).start();
    LOGGER.info("Mission Is Done : {}", futureTask.isDone());
    String result = futureTask.get();
    // String result = futureTask.get(1, TimeUnit.SECONDS); // 在指定的时间超时后会抛出 TimeoutException
    LOGGER.info("Mission Is Done : {}", futureTask.isDone());
    LOGGER.info("Result Data : {}", result);
}
```

- 在实例化 `FutureTask` 时传入 `Callable` 对象
- 将 `FutureTask` 对象传入 Thread 并执行线程
- 使用 `isDone` 方法检查是否执行完成
- 使用 `get` 方法获取到返回的内容

> `get` 方法有两个重载方法，一个无参数另一个需要传入超时时间，超时后抛出 `TimeoutException` 异常。
>
> 在执行 `get` 方法时，主线程会被阻塞，知道获取到返回结果或者超时。

### 解析源码

#### 实例化 FutureTask

`FutureTask` 拥有两个构造方法，一个直接传入 `Callable` 对象，另一个传入 `Runnable` 对象和一个默认的返回值，因为 `FutureTask` 类默认是存在可以获取返回值的。

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

在当传入的对象是 `Callable` 时，`FutureTask` 会直接将 `Callable` 赋值给属性。当传入的 `Runnable` 时会使用到线程池对象 `Executors` 的 `callable` 方法，将 `Runnable` 转换为 `Callable`。

```java
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

// RunnableAdapter
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

可以看到，`Runnable` 和 传入的返回值被传入到 `RunnableAdapter` 对象中，此对象实现 `Callable` 接口并在 `call` 方法中调用了 `Runnable` 的 `run` 方法，并将传入的返回值进行返回。至此，使用传入的 `Runnable` 对象和默认返回值转换为 `Callable` 对象。

#### 传入到 Thread

从下图可知，`FutureTask` 实现了 `RunnableFuture` 而 `RunnableFuture` 接口类继承了 `Runnable` 和 `Future` ，因此 `FutureTask` 本质上还是 `Runnable` ，所以可以直接传入 `Thread` 类中。

![image-20220120112001460](photo\37、FutureTask继承结构图(7).png) 

在 `FutureTask` 中的 `run` 方法最后还是执行的是 `Callable` 的 `call` 方法。

#### FutureTask 如何包装 Runnable 和 Callable

![image-20220120170609195](photo\38、FutureTask包装任务类(7).png) 

#### get 方法

`FutureTask` 使用以下状态用来区分当前线程的状态

```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6; 
```

- `NEW` ：新建
- `COMPLETING` ：即将完成
- `NORMAL` ：完成
- `EXECEPTIONAL` ：抛异常
- `CANCELLED` ：任务取消
- `INTERRRUPTING` ：任务即将被打断
- `INTERRUPTED` ：任务被打断

`get` 方法源码

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state; // 获取当前
    if (s <= COMPLETING) // 状态为新建或即将完成
        s = awaitDone(false, 0L);
    return report(s);
} 
```

`get` 方法会阻塞线程等待返回，阻塞部分的核心代码就是 `awaitDone` 方法。

```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
} 
```

关于阻塞部分，方法中使用了 `LockSupport` 这是一个阻塞线程工具类，提供了多种方法用来阻塞及唤醒线程。

```java
public static void park(Object blocker); // 暂停当前线程
public static void parkNanos(Object blocker, long nanos); // 暂停当前线程，不过有超时时间的限制
public static void parkUntil(Object blocker, long deadline); // 暂停当前线程，直到某个时间
public static void park(); // 无期限暂停当前线程
public static void parkNanos(long nanos); // 暂停当前线程，不过有超时时间的限制
public static void parkUntil(long deadline); // 暂停当前线程，直到某个时间
public static void unpark(Thread thread); // 恢复当前线程
public static Object getBlocker(Thread t);
```

#### LockSupport 简单使用

使用 `LockSupport` 阻塞线程后唤醒。

```java
public static void main(String[] args) throws InterruptedException {
    List<Thread> threadList = new ArrayList<>();

    for (int i = 0; i < 5; i ++){
        int finalI = i;
        Thread thread = new Thread(() -> {
            LOGGER.info("启动线程：{}", finalI);
            LockSupport.park();
            LOGGER.info("线程恢复：{}", finalI);
        });
        thread.setName("线程--"+finalI);
        thread.start();
        threadList.add(thread);
    }

    Thread.sleep(2000L);
    LOGGER.info("主线程恢复");
    for (Thread thread : threadList) {
        LockSupport.unpark(thread);
    }
}
```

结果

```java
2022-01-21 09:43:59.993 [线程--0] INFO  java.lang.Thread - 启动线程：0
2022-01-21 09:43:59.993 [线程--2] INFO  java.lang.Thread - 启动线程：2
2022-01-21 09:43:59.993 [线程--1] INFO  java.lang.Thread - 启动线程：1
2022-01-21 09:43:59.993 [线程--3] INFO  java.lang.Thread - 启动线程：3
2022-01-21 09:43:59.993 [线程--4] INFO  java.lang.Thread - 启动线程：4
2022-01-21 09:44:02.007 [main] INFO  java.lang.Thread - 主线程恢复
2022-01-21 09:44:02.007 [线程--0] INFO  java.lang.Thread - 线程恢复：0
2022-01-21 09:44:02.007 [线程--4] INFO  java.lang.Thread - 线程恢复：4
2022-01-21 09:44:02.007 [线程--3] INFO  java.lang.Thread - 线程恢复：3
2022-01-21 09:44:02.007 [线程--2] INFO  java.lang.Thread - 线程恢复：2
2022-01-21 09:44:02.007 [线程--1] INFO  java.lang.Thread - 线程恢复：1
```

## 线程间通信

### 什么是线程间通信

同进程下的线程是可以访问共享数据的，线程间通信的定义为针对同一资源的操作有不同种类的线程。即为多个不同作用的线程操作同一资源，最经典的实例就是生产者消费者。

### 生产者与消费者

生产者消费者顾名思义就是生产产品的对象和消费产品的对象，在消费者消费时发现产品已经没有了，此时就要通知生产者进行生产。反之生产者生产出产品也需要通知消费者进行消费。

#### 代码实现

- 轮询实现

```java
public class WhileQueueTestClass extends BasicLogger {

    public static void main(String[] args) {
        // 队列
        WhileQueue<String> queue = new WhileQueue<>();

        // 生产者
        new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    queue.put("消息" + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        // 消费者
        new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    /**
     * 轮询版本
     */
    public static class WhileQueue<T> {
        // 容器，用来装东西
        private final LinkedList<T> queue = new LinkedList<>();

        public void put(T resource) throws InterruptedException {
            while (queue.size() >= 1) {
                // 队列满了，不能再塞东西了，轮询等待消费者取出数据
                System.out.println("生产者：队列已满，无法插入...");
                TimeUnit.MILLISECONDS.sleep(1000);
            }
            System.out.println("生产者：插入" + resource + "!!!");
            queue.addFirst(resource);
        }

        public void take() throws InterruptedException {
            while (queue.size() <= 0) {
                // 队列空了，不能再取东西，轮询等待生产者插入数据
                System.out.println("消费者：队列为空，无法取出...");
                TimeUnit.MILLISECONDS.sleep(1000);
            }
            System.out.println("消费者：取出消息!!!");
            queue.removeLast();
            TimeUnit.MILLISECONDS.sleep(5000);
        }

    }
}
```

轮询状态线程并没有处于等待或休眠状态，而是一直处于运行状态，因此这种方法会极大程度的消耗性能，造成很多无所谓的消耗。

- 等待唤醒 `wait/notify` 

使用 `wait` 方法可以使线程进入等待状态，使用 `notify` 随机唤醒一个等待中的线程，使用 `notifyAll` 唤醒所有等待中的线程。

```java
public class ThreadQueueTestClass {

    public static void main(String[] args) {
        // 队列
        WaitNotifyQueue<String> queue = new WaitNotifyQueue<>();

        // 生产者
        new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    queue.put("消息" + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "生产者--1").start();

        // 消费者
        new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "消费者--1").start();

    }

    public static class WaitNotifyQueue<T> extends BasicLogger {
        // 容器队列，用于放置生产的产品
        private final LinkedList<T> queue = new LinkedList<>();

        public synchronized void put(T resource) throws InterruptedException {
            while (queue.size() >= 1) {
                // 队列中已经存在内容
                LOGGER.info("生产者：队列已满，无法继续");
                this.wait();
            }
            LOGGER.info("生产者：生产 {} ", resource);
            queue.addFirst(resource);
            this.notifyAll();
        }

        public synchronized void take() throws InterruptedException {
            while (queue.size() <= 0) {
                // 队列无内容
                LOGGER.info("消费者：队列为空，无法继续");
                this.wait();
            }
            LOGGER.info("消费者：使用产品");
            queue.removeLast();
            this.notifyAll();
        }
    }
}
```

> - 为什么使用 `notifyAll` 
>
>   - 当使用的方法为 `notify` 时，此方法会随机唤醒一个线程，当生产者线程为多个时，有可能因为两次唤醒的都是生产者线程导致程序阻塞。
>
> - 为什么 `LinkedList` 是 `final` 状态
>
>   - 因为两个方法执行时是不同的线程，在子线程中只可以访问常量，所以需要使用 `final` 修饰。
>
> - 为什么方法中需要使用 `syncronized` 同步锁
>
>   - 因为 ` queue` 是生产者线程和消费者线程都需要操作的对象，所以需要保证此对象的原子性，即此对象不可以同时被多个线程修改，所以需要方法上添加 `syncronized` 同步锁，保证同一时间只有一个线程在修改此对象。
>
> 注：线程在执行 `wait` 方法后进入 `WAITING` 状态，此状态必须使用 `notify` 唤醒之后才可以继续执行，否则将会一直处于 `WAITING` 状态导致程序阻塞，这也是为什么要使用 `notifyAll` 的原因。

- 等待唤醒 `condition` 

```java
public class ConditionQueueTestClass extends BasicLogger {

    public static void main(String[] args) {
        ConditionQueue<String> queue = new ConditionQueue<>();
        new Thread(()->{
            try {
                for (;;){
                    queue.put("resource");
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }, "生产者-1").start();

        new Thread(()->{
            try {
                for (;;){
                    queue.take();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }, "消费者-1").start();

    }

    public static class ConditionQueue<T>{
        private final LinkedList<T> queue = new LinkedList<>();

        // 显式锁
        private final ReentrantLock lock = new ReentrantLock();
        private final Condition producerCondition = lock.newCondition();
        private final Condition consumerCondition = lock.newCondition();

        public void put(T resource) throws InterruptedException {
            lock.lock();
            try {
                while (queue.size() >= 1){
                    LOGGER.info("线程：{} ；队列已满", Thread.currentThread().getName());
                    producerCondition.await();
                }
                LOGGER.info("线程：{} ；生产产品", Thread.currentThread().getName());
                queue.push(resource);
                consumerCondition.signal();
            }finally {
                lock.unlock();
            }
        }

        public void take() throws InterruptedException {
            lock.lock();
            try {
                while (queue.size() <= 0){
                    LOGGER.info("线程：{} ；没有产品", Thread.currentThread().getName());
                    consumerCondition.await();
                }
                LOGGER.info("线程：{} ；消费产品", Thread.currentThread().getName());
                queue.pop();
                producerCondition.signal();
            }finally {
                lock.unlock();
            }
        }
    }
}
```

`condition` 使用 `ReentrantLock` 对象的 `newCondition` 方法创建，可以将 `condition` 理解为一个储存线程的队列。当在线程中执行 `Condition#await()` 方法时，线程就会被移入相应的队列，当执行指定 `Condition` 的 `signal` 或者 `signalAll` 方法时就会唤醒指定队列中的线程。

关于 `ReentrantLock` 相关知识请查看 [ReentrantLock详解](# ReentrantLock)


## 阻塞队列

上方生产者与消费者的实例代码中，将一个普通的 `LinkedList` 包装为一个类，这个类中分别存在存入和取出的方法，在 `JDK` 中有着相同功能的实现，被称为阻塞队列 `blocking queue` 可以使用这些定义好的阻塞队列解决在多线程下集合及链表存取带来的安全性问题。

在 `JDK` 中的 `JUC (java.util.concurrent)` 包下有着将各种阻塞队列的实现，包括 `ArrayBlockingQueue` 、`LinkedBlockingQueue` 、`PriorityBlockQueue` 。

### 实现一个简单的阻塞队列

通过继承 `LinkedList` 实现与原生 `LinkedBlockingQueue` 相似的功能。

```java
public class MoChenBlockingQueue<T> extends LinkedList<T> implements BasicLogger {

    private long MAX_SIZE = 10;

    public MoChenBlockingQueue(){}

    public MoChenBlockingQueue(long capacity){
        if (capacity <= 0){
            throw new IllegalArgumentException("不可小于等于0");
        }
        this.MAX_SIZE = capacity;
    }

    public synchronized void put(T resource) throws InterruptedException {
        while (super.size() >= MAX_SIZE){
            LOGGER.info("队列已满");
            this.wait();
        }
        LOGGER.info("插入内容，插入前大小：{}", super.size());
        super.push(resource);
        this.notifyAll();
    }

    public synchronized T take() throws InterruptedException {
        while (super.size() <= 0){
            LOGGER.info("队列已空");
            this.wait();
        }
        LOGGER.info("取出内容，取出前大小：{}", super.size());
        T pop = super.pop();
        this.notifyAll();
        return pop;
    }


}
```

### 原生阻塞队列

#### 队列与阻塞队列继承链对比

**`ArrayList` 与 `ArrayBlockingQueue` 继承链对比** 

![43、List阻塞队列对比(7)](photo/43、List阻塞队列对比(7).png) 

`ArrayList` 和 `ArrayBlockingQueue` 同样最终实现了 `Collection` 接口，阻塞队列最终的实现代码都在 `ArrayBlockingQueue` 中，其父类 `AbstarctQueue` 中只有一些简单的逻辑定义及异常抛出。

**`LinkedList` 与 `LinkedBlockingQueue` 继承链对比** 

![44、链表阻塞队列对比(7)](photo/44、链表阻塞队列对比(7).png) 

同上面的 `ArrayBlockingQueue` 一样，其具体的阻塞实现也是在 `LinkedBlockingQueue` 类中。

#### LinkedBlockingQueue 源码解析

**构造方法** 

```java
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}

public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
    last = head = new Node<E>(null);
}
```

无参构造方法调用了第二个有参构造，传入的 `capacity` 是此链表的最大值，即没有指定最大值时默认使用 `Integer` 的最大值，即为 `0x7fffffff;` $2^{31}-1$ 。

1. 检查传入的数字，不可小于1抛出非法参数错误。
2. 将参数赋值到类属性 `capacity` 中，即为链表最大值。
3. 将属性 `last` 和 `head` 赋值为单向链表。

单项链表（`LinkedBlockingQueue` 静态内部类）：

```java
static class Node<E> {
    E item;
    Node<E> next;
    Node(E x) { item = x; }
}
```

- `item` 当前节点值。
- `next` 下一个节点。

传入集合的构造方法（用于将集合转为链表阻塞队列）：

```java
public LinkedBlockingQueue(Collection<? extends E> c) {
    this(Integer.MAX_VALUE);
    final ReentrantLock putLock = this.putLock;
    putLock.lock(); // Never contended, but necessary for visibility
    try {
        int n = 0;
        for (E e : c) {
            if (e == null)
                throw new NullPointerException();
            if (n == capacity)
                throw new IllegalStateException("Queue full");
            enqueue(new Node<E>(e));
            ++n;
        }
        count.set(n);
    } finally {
        putLock.unlock();
    }
}
```

1. 首先使用第二个带参构造实例化对象。
2. 使用 `ReentrantLock` 。
3. 遍历传入的集合。
   1. 为空抛出空指针异常。
   2. 传入集合等于链表最大值时抛出异常状态 `Queue full` 队列已满。
4. 将遍历的集合元素使用 `enqueue` 方法依次送入链表中。
5. 使用 `count.set()` 方法设置链表现有元素数量。
6. `unlock` 解锁。

**`offer` 方法** 

```java
public boolean offer(E e) {
    if (e == null) throw new NullPointerException(); // 1
    final AtomicInteger count = this.count;
    if (count.get() == capacity)
        return false;	
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        if (count.get() < capacity) {
            enqueue(node);
            c = count.getAndIncrement(); // 将count递增加一返回原值
            if (c + 1 < capacity)
                notFull.signal();
        }
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
    return c >= 0;
}

private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}
```

1. 检查参数是否为空。
2. 检查当前链表元素大小是否为最大。
3. 启动锁。
4. 再次判断当前获取锁后链表元素大小。
5. `enqueue` 存入元素。
6. `count` 添加技术。
7. 如果链表放入元素后的大小依旧小于链表容量，唤醒 `putLock` （存元素锁）的 `Condition` 中任意线程。
8. 释放锁 `unlock` 。
9. 若当前添加完成元素后链表大小为1（`c` 初始化值为 `-1` ，而 `getAndIncrement` 方法返回值为加一前的值，即为链表添加这个元素前的大小，当判断 `c == 0` 为 `true` 时则当前添加的元素为这个链表唯一的元素。），唤醒 `tackLock` （取元素锁）对应的 `Condition` 中的任意线程。

**`put` 方法** 



可以看到在 `put` 方法中使用了 `ReentrantLock` 来保证多线程数据的原子性。

在 `LinkedLockingQueue` 中声明了两个锁和对应的两个线程队列，分别用于存取数据 `put/take` 

## 线程池

