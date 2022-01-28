# Java 并发编程

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

`Runnable` 类：

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

`Callable` 类：

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

`Thread` 类：

```java
public class Thread implements Runnable {
    // code
}
```

### 源码解析

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

#### Callable 类

`Callable` 类和 `Runnable` 类一样是一个接口类，不同之处在于 `Callable` 的 `call` 方法拥有返回值，而 `Runnable` 的 `run` 方法没有返回值。

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

## 锁

### 线程安全



### AQS

`AQS` 全称 `AbstractQueuedSynchronizer` （抽象队列同步器），`AQS` 定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于 `AQS` 例如 `ReentrantLock` 等。

#### 概述

**`CLH` 队列锁** 

`CLH` 锁是有由 `Craig` ,  `Landin` ,  `Hagersten` 这三个人发明的锁，取了三个人名字的首字母，所以叫 `CLH Lock` 。

`CLH` 队列锁也是一种基于**双向链表**的可扩展、高性能、公平的自旋锁，申请线程仅仅在本地变量上自旋，它不断轮询前驱的状态，假设发现前驱释放了锁就结束自旋。



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



##### releaseShared方法



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



###### unparkSuccessor方法



```java
private void unparkSuccessor(Node node) {

    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```



### ReentrantLock


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

