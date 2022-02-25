## 概述 Overview

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


### 多线程基础

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

##### 解析

###### run

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

###### start

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

###### sleep

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

###### yield

`yield` 方法是一个 `native` 方法，作用和 `sleep` 方法相同都是空出 `CPU` 执行时间用于执行其他线程，同样， `yield` 方法也不会释放锁。唯一与 `sleep` 方法不同的是 `yield` 方法无法控制空出的时间，并且 `yield` 方法只能让拥有同样优先级的线程有获取 `CPU` 执行时间的机会。

> 注意：此方法不会将线程进入阻塞状态，只会将线程重新置为可运行状态 `RUNNABLE` 。

###### join

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

###### interrupt

此方法会设置线程的中断标志位，如果线程在 `sleep` 、`wait` 、`join` 处于阻塞状态时，线程会定时检查中断标志位若发现中断则会抛出 `InterruptedException` 异常，并在异常抛出后将中断标志位清除。

```java
Thread thread = new Thread(() -> {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
thread.start();
Thread.sleep(1000);
thread.interrupt(); // java.lang.InterruptedException: sleep interrupted
```

需要注意的是，中断方法只是创建了一个标志，用于建议线程应该中断，而真正的异常抛出应该由线程自己来处理。很明显 `sleep` 方法已经对中断标志做了处理，抛出了线程中断异常。

##### 使用

直接创建 `Thread` 匿名内部类。

```java
Thread thread = new Thread() {
    @Override
    public void run() {
        System.out.println("new thread with rewrite run method");
    }
};
thread.start();
```

此方法使用匿名内部类来继承 `Thread` 并重写其中的 `run` 方法，子线程执行的代码就是 `run` 方法中的代码。所以重写 `run` 方法后，使用 `start` 方法启动线程可以实现创建子线程。

#### Runnable 类

##### 概述

`Runnable` 类本质没有任何操作，只是用来结合 `Thread` 类实现函数式接口。

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

##### 使用

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

使用匿名内部类实例化 `Runnable` 接口，并将实例化的对象引用传入 `Thread` 对象中，最后调用 `start` 方法启动线程。使用 `Runnable` 接口将子线程需要执行的代码间接由 `Thread::run()` 方法的 `target.run();` 执行，这种方式可以实现 `lambda` 简写，降低代码量。

#### Callable 类

`Callable` 类和 `Runnable` 类一样是一个接口类，不同之处在于 `Callable` 的 `call` 方法拥有返回值，而 `Runnable` 的 `run` 方法没有返回值。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

因为 `Callable` 存在返回值，所以无法像 `Runnable` 那样传入 `Thread` 类来使用，需要使用到另一个类 `FutureTask` 。

#### Future 类

```java
public interface Future<V> {
	// 尝试取消此任务执行
    boolean cancel(boolean mayInterruptIfRunning);
	// 任务是否在完成前被取消
    boolean isCancelled();
	// 任务是否已经完成，此完成包括了正常中止、异常和取消。
    boolean isDone();
	// 等待任务完成并获取返回值
    V get() throws InterruptedException, ExecutionException;
	// 在指定的时间内等待任务完成并获取返回值，超时则取消等待
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

`Future` 类也是一个接口类，其中定义了一系列操作线程及判断线程状态的方法。

#### FutureTask 类

##### 概述

`FutureTask` 类实现了 `RunnableFuture` 接口类，而 `RunnableFuture` 则同时继承了 `Runnable` 和 `Future` 类。

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

由于 `Thread` 加 `Runnable` 的组合只能让子线程执行代码，但是没有办法获取到执行的结果。 `Callable` 接口中有一个定义了返回值的方法，而 `Callable` 正是和 `Future` `FutureTask` 配合，实现获取返回值的操作。

相关方法：

![image-20220120101305193](photo\36、FutureTask方法列表(7).png) 

##### 解析源码

###### 构造方法

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

###### 包装接口

从下图可知，`FutureTask` 实现了 `RunnableFuture` 而 `RunnableFuture` 接口类继承了 `Runnable` 和 `Future` ，因此 `FutureTask` 本质上还是 `Runnable` ，所以可以直接传入 `Thread` 类中。

![image-20220120112001460](photo\37、FutureTask继承结构图(7).png) 

在 `FutureTask` 中的 `run` 方法最后还是执行的是 `Callable` 的 `call` 方法。

![image-20220120170609195](photo\38、FutureTask包装任务类(7).png) 

###### 获取返回值

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
- `COMPLETING` ：即将完成，完成中，运行中
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
private int awaitDone(boolean timed, long nanos) throws InterruptedException {
    // 取消等待时间
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        // 线程中断
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }
		// 线程已经结束
        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        // 线程还在执行中
        else if (s == COMPLETING)
            Thread.yield(); // 等待
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset, q.next = waiters, q);
        // 定时等待
        else if (timed) {
            // 减去已经用掉的时间
            nanos = deadline - System.nanoTime();
            // 等待时间已经是0了
            if (nanos <= 0L) {
                removeWaiter(q); // 移除等待
                return state; // 返回值
            }
            // 使用定时等待
            LockSupport.parkNanos(this, nanos);
        }
        else
            // 不限时等待
            LockSupport.park(this);
    }
} 
```

关于阻塞部分，方法中使用了 `LockSupport` 详见 [LockSupport类](# LockSupport 类) ，这是一个阻塞线程工具类，提供了多种方法用来阻塞及唤醒线程。


##### 使用 FutureTask

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

在传入 `Callable` 和 `Runnable` 时可以对不同参数的构造方法使用 `lambda` 缩写。

```java
// Callable
FutureTask<String> futureTask = new FutureTask<>(() -> {});
// Runnable
FutureTask<String> futureTask1 = new FutureTask<>(() -> {}, "result");
```

#### LockSupport 类

##### 概述

`LockSupport` 类中的所有方法都是静态方法，且都是调用了 `Unsafe` 类的本地方法，这些阻塞操作等都是由系统执行的。


```java
// 暂停当前线程
public static void park(Object blocker); 
// 暂停当前线程，不过有超时时间的限制
public static void parkNanos(Object blocker, long nanos); 
// 暂停当前线程，直到某个时间
public static void parkUntil(Object blocker, long deadline); 
// 无期限暂停当前线程
public static void park(); 
// 暂停当前线程，不过有超时时间的限制
public static void parkNanos(long nanos); 
// 暂停当前线程，直到某个时间
public static void parkUntil(long deadline); 
// 恢复指定线程
public static void unpark(Thread thread);
```

> `park` 等阻塞方法会被中断所唤醒，但是并不会抛出异常和清除线程中断标志位，因此需要在线程被唤醒后判断是否中断过，以及设计如何去处理中断。

##### 使用

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

可以看到使用 `LockSupport` 实现了线程的阻塞和唤醒，`LockSupport` 和使用 `notify` 等方法的不同之处在于，前者可以准确的唤醒某个线程，而 `notify` 是随机唤醒一个线程， `notifyAll` 则是全部唤醒。由于这个特点，`AQS` 锁的底层最终实现阻塞唤醒功能的就是 `LockSupport` 的 `park` 和 `unpark` 方法。

### 线程状态

#### 状态转换图

![image-20220128173943816](photo\42、Thread生命周期转换图(7).png) 

主要方法介绍 [Thread类主要方法](# Thread 类) 

#### 代码实现

##### NEW

初始状态

```java
Thread thread1 = new Thread(() -> {
    // nothing...
});
LOGGER.info("线程状态：{}", thread1.getState()); // NEW
```

当 `Thread` 对象不为空时，线程当前的状态为初始化状态。

##### RUNNABLE

可运行状态

```java
Thread thread1 = new Thread(() -> {
    // nothing...
});
thread1.start();
LOGGER.info("线程状态：{}", thread1.getState()); // RUNNABLE
```

当执行 `start()` 方法时线程为可运行状态。

##### BLOCKED

阻塞状态

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

##### WAITING

等待状态

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

##### TIMED_WAITING

有超时的等待状态

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

##### TERMINATED

结束状态

```java
Thread thread5 = new Thread(() -> {});
thread5.start();
Thread.sleep(500L);
LOGGER.info("线程状态：{}", thread5.getState()); // TERMINATED
```

正常执行完成的线程进入结束状态。


### 线程间通信

#### 概述

此处的线程通信并不是指线程之间互相传入数据，而是线程之间的相互阻塞与唤醒等操作。在多线程访问一个共享的资源时，如果有两种做不同类型操作的线程。如生产与消费，一类线程是负责在生产 `put` 资源，而另一类线程则在 `take` 拿取资源。那么在生产线程生产资源后，需要通知消费线程拿取资源，同样消费线程也需要在没有资源时通知生产线程生产资源。

#### 实现

##### 轮询实现

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

##### 等待唤醒

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

##### 使用 Condition

使用 `ReentrantLock` 的 `Condition` 对两类线程分别进行阻塞与唤醒操作。

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

`condition` 使用 `ReentrantLock` 对象的 `newCondition` 方法创建，可以将 `condition` 理解为一个储存线程的队列。当在线程中执行 `Condition::await()` 方法时，线程就会被移入相应的队列，当执行指定 `Condition` 的 `signal` 或者 `signalAll` 方法时就会唤醒指定队列中的线程。

关于 `Condition` 相关知识请查看 [Condition详解](# Condition) 

关于 `ReentrantLock` 相关知识请查看 [ReentrantLock详解](# ReentrantLock) 

## ThreadLocal

### 概述

在单线程状态下，变量分为局部变量和全局变量，单线程状态下可以随意使用。但是在多线程状态下，局部变量是没有问题的，但使用全局变量就会引发[线程安全](# 线程安全)问题。所以 `JDK` 在 `java.lang` 包中提供了 `ThreadLocal` 用来解决这个问题，`ThreadLocal` 是一种介于全局变量和局部变量之间的变量。 `ThreadLocal` 可以为每个线程单独储存一个相同类的不同的实例，即每个线程设置和储存的对象都可以互不干扰，在需要的时候随时获取大大减少了方法之间显式的参数传递。

![image-20220217093638572](photo/63、ThreadLocal结构图(7).png) 

### 使用

`ThreadLocal` 只需要在指定的线程中使用 `set` 方法和 `get` 方法即可设置和获取其中的对象，在线程执行结束时需要使用 `remove` 去移除 `ThreadLocal` 中的对象。

```java
public class ThreadLocalTestClass implements BasicLogger {

    private static ThreadLocal<String> stl = new ThreadLocal<>();

    public static void main(String[] args) {

        for (int i = 0; i < 100; i++) {
            int finalI = i;
            new Thread(() -> {
                try {
                    stl.set("Test -- " + finalI);
                    test1();
                    test2();
                } finally {
                    stl.remove();
                }
            }).start();
        }
    }

    public static void test1() {
        String str = stl.get();
        LOGGER.info("test1 method output content === {}", str);
    }

    public static void test2() {
        String str = stl.get();
        LOGGER.info("test2 method output content === {}", str);
    }
}
```

结果

```
test1 method output content === Test -- 2
test1 method output content === Test -- 1
test2 method output content === Test -- 1
test1 method output content === Test -- 0
test2 method output content === Test -- 0
test2 method output content === Test -- 2
```

可以看到在同一线程执行的 `test1` 和 `test2` 方法输出的内容不同，不同线程获取到的 `String` 对象不是同一个。

### 解析

首先 `ThreadLocal` 是一个泛型类，保证可以接受任何类型的对象。

因为一个线程内可以存在多个 `ThreadLocal` 对象，所以其实是 `ThreadLocal` 内部维护了一个 `Map` ，这个 `Map` 不是直接使用的 `HashMap` ，而是 `ThreadLocal` 实现的一个叫做 `ThreadLocalMap` 的静态内部类。而我们使用的 `get()`、`set()` 方法其实都是调用了这个 `ThreadLocalMap` 类对应的 `get()`、`set()` 方法。例如下面的 `set` 方法：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

调用 `ThreadLocal` 的 `set` 方法时，首先获取到了当前线程，然后获取当前线程维护的 `ThreadLocalMap` 对象，最后在`ThreadLocalMap` 实例中添加上。如果 `ThreadLocalMap` 实例不存在则初始化并赋初始值。

这里看到 `set` 方法的第一个参数是 `this` ，`this`即指的是当前的 `ThreadLocal` 对象，会看上看的代码就是指的 `mLocal` 这个对象。而在 `ThreadLocalMap` 的 `set` 方法中会根据当前 `ThreadLocal` 对象实例，做一些操作和判断，最终实现赋值操作。

所以说，最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 可以理解为只是一个中间工具，传递了变量值。

#### 内存泄露问题

实际上 `ThreadLocalMap` 中使用的 `key` 为 `ThreadLocal` 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

所以如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 `ThreadLocalMap` 中使用这个 `ThreadLocal` 的 `key` 也会被清理掉。但是，`value` 是强引用，不会被清理，这样一来就会出现 `key` 为 `null` 的 `value`。

`ThreadLocalMap` 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 `key` 为 `null` 的记录。如果说会出现内存泄漏，那只有在出现了 `key` 为 `null` 的记录后，没有手动调用 `remove()` 方法，并且之后也不再调用 `get()`、`set()`、`remove()` 方法的情况下。

> 在使用完成 `ThreadLocal` 之后，一定要记得调用 `remove` 方法，以防内存泄露。

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

在 `JDK1.5` 之前的 `synchronized` 默认为重量级锁，在之后的版本优化之后，`synchronized` 锁可以根据情况不断升级，直至成为重量级锁。详见 [无锁 偏向锁 轻量级锁 重量级锁](# 无锁 偏向锁 轻量级锁 重量级锁) 

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

#### 概述

**CLH 队列锁** 

`CLH` 锁是有由 `Craig` ,  `Landin` ,  `Hagersten` 这三个人发明的锁，取了三个人名字的首字母，所以叫 `CLH` 锁。

`CLH` 队列锁也是一种基于**双向链表**的可扩展、高性能、公平的自旋锁，申请线程仅仅在本地变量上自旋，它不断轮询前驱的状态，发现前驱释放了锁就结束自旋。而 `AQS` 则是 `CLH` 的变体，在 `CLH` 的基础上进行了优化，并将自旋锁修改为了阻塞锁。

**AQS 类** 

`AQS` 全称 `AbstractQueuedSynchronizer` （抽象队列同步器），`AQS` 定义了一套多线程访问共享资源的同步器框架，许多同步类实现都基于 `AQS` 的理念，例如 `ReentrantLock` 等。

`AQS` 类只是一个框架，展示了队列锁的设计方法。使用 `volatile int state` 来表示共享资源，使用 `volatile` 来保证 `state` 对所有线程的可见性。多线程通过争抢此资源（及使用 `CAS` 对此值进行自增，`CAS` 成功执行即为获取锁），无法获取资源则进入同步队列中阻塞等待唤醒。

#### 源码详解

##### Node 类

###### 主要变量

在 `AQS` 类中由 `Node` 静态内部类实现队列，其使用 `prev` 和 `next` 变量分别指向了当前节点的上一个和下一个节点。

```java
volatile int waitStatus; // 当前节点状态
volatile Node prev; // 上一个node
volatile Node next; // 下一个node
volatile Thread thread; // 当前节点储存的线程，实例化Node时传入，使用后置空。
Node nextWaiter; // 条件队列内容
```

`node` 类中有五个重要的变量，其中 `prev` 和 `next` 相连构成了一个双向链表。在接下来的描述中，我会将 `prev` 称为前驱而 `next` 则称为后继，方便理解。

![image-20220125095028713](photo/45、CLH队列Node图示1(7).png)

###### 节点状态

接下来看 `waitStatus` ，此变量表示当前 `Node` 节点的状态，在 `node` 类中有以下定义。

```java
static final int CANCELLED =  1;
static final int SIGNAL    = -1;
static final int CONDITION = -2;
static final int PROPAGATE = -3;
```

- `CANCELLED 1` ：表示当前结点已取消调度。	
- `SIGNAL -1` ：表示后继结点在等待当前结点唤醒。
- `CONDITION -2` ：表示结点等待在条件队列中等待。
- `PROPAGATE -3` ：共享模式下，前继结点可能会唤醒后继及其之后的结点。
- `0` ：新结点入队时的默认状态。

> 接下来的源码中，有很多地方直接判断状态是否大于0，即表示判断节点是否已经失效。

`CONDITION` 状态只有在用到条件队列时才会使用，具体内容在 `Condition` 中再进行讨论。

###### 独占与共享锁

在 `AQS` 中定义了两种不同类型的锁，分别是独占锁与共享锁。前者同时只能有一个线程获取此锁，后者则可以有多个线程同时获取。具体的独占锁与共享锁将会在 `ReentrantReadWriteLock` 中进行介绍。在创建节点是我们需要标记这个节点是什么类型，下面是节点类型的源码。

```java
static final Node SHARED = new Node(); // 共享模式
static final Node EXCLUSIVE = null; // 独占模式
```

`node` 节点分别使用了两个 `node` 对象用来表示共享或独占，共享使用新建的 `node` 对象，而独占模式则直接为 `null` 。下面是 `node` 类的构造方法。

```java
Node() {} // 使用其初始化head或者生成共享锁标记
Node(Thread thread, Node mode) { // addWaiter方法使用
    this.nextWaiter = mode;
    this.thread = thread;
}
```

可以看到在新建 `node` 时需要传入 `node` 中需要储存的线程以及 `node` 的类型，此 `mode` 类型直接被储存在 `nextWaiter` 变量中。注意此处的 `nextWaiter` 变量，在新建同步队列 `node` 时会标记为是否共享，而在条件队列中则会是另一个用处，具体作用在 `Condition` 中再进行介绍。

`Node` 类中还有一个 `isShared` 方法，用于判断当前节点是否为共享模式。

```java
final boolean isShared() {
    return nextWaiter == SHARED;
}
```

##### head 和 tail

接下来分析 `AQS` 类中的 `head` 和 `tail` 变量，这两个变量位于 `AQS` 类中 `head` 指向 `node` 链表的第一个，而 `tail` 变量指向链表的最后一个。

```java
private transient volatile Node head; // 指向线程链表的第一个node
private transient volatile Node tail; // 指向线程链表的最后一个node
```

由此可知，在 `AQS` 中的链表是这样一个结构。

![image-20220125104832047](photo/46、CLH队列Node图示2(7).png)

##### 独占锁

###### 获取资源

首先先看独占锁的获取资源的方法。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && // 直接尝试获取资源
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt(); // 中断当前线程
}
```

首先调用 `tryAcquire` 方法用于尝试获取资源，在返回获取失败 `false` 之后使用 `addWaiter` 方法将当前线程节点加入到队列中，然后使用 `acquireQueued` 方法进行自旋阻塞。

首先我们先看一下 `tryAcquire` 尝试获取资源方法的代码。


```java
// 此方法用户尝试以独占模式获取，成功为true
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

使用此方法直接去获取资源，成功返回 `true` 不成功返回 `false` 。在概述中已经说明 `AQS` 只是一个框架，所以此处没有真正的实现，需要在自定义同步器中去实现。

###### 线程节点入队

如果在执行上一个方法当前线程没有获取到资源，那就需要执行 `addWaiter` 方法将节点入队，以下是此方法的源码。


```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode); // 包装为node
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

首先用传入的类型和获取到的当前线程对象创建 `node` ，再判断尾节点是否为空。当尾结点为空则表示当前同步队列中没有节点，因此需要调用 `enq` 方法去初始化队列并将节点放入队列。若尾结点不为空则表示当前同步队列中已经存在节点，接下来就需要三步将大象放冰箱中了。

第一步，将当前节点的前驱设置为当前队列的尾节点。第二步是一个 `CAS` 操作将 `tail` 指向的节点修改为当前节点。如果上一步成功，接下来就会将之前的尾结点的后继节点修改为当前节点。至此，当前的节点就被放置在了队列的尾部。

由于将节点放置到队列尾部的这个操作并不是原子操作，它一共需要三步，那么在多线程并发的情况下会发生一个特殊的现象——尾分叉。其具体原因在分析完下面的 `enq` 方法后再进行介绍。

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

从 `addWaiter` 方法中进入此方法的原因有两种，一是队列中没有节点，二是节点在加入队尾时 `CAS` 操作失败。无论是那种情况这个节点是必须要入队的，所以此处使用了 `for` 循环自旋，每次都获取新的 `tail` 尾结点的值。

此时尾结点的情况分两种，一种是为空，即队列中没有节点。那么现在就需要初始化队列，首先使用 `CAS` 操作将 `head` 的值由 `null` 修改为新建的空白 `node` 。如果 `CAS` 成功则将 `tail` 尾结点也指向这个空白 `node` ，若不成功则说明 `head` 已经被修改了值。不管无论成功与否，队列中必须最少需要有一个空白的节点，才可以进行下一步的入队操作。第二种情况就是队列中已经存在节点可以进行入队操作，此处的入队方法与 `addWaiter` 方法中的入队流程相同，就不再赘述。

###### 尾分叉

由于方法是多线程执行，那么在执行入队操作的三步代码中就会发生以下情况。

第一步因为是非同步操作，此时可能同时有多个拿着同一个 `tail` 节点的线程，然后将自己节点的前驱修改为当前的 `tail` 。

![](photo/65、AQS链表尾分叉01(7).png) 

此时，这些线程进入 `CAS` 操作，而 `CAS` 最后只会有一个线程成功执行，此时链表会变成下面所示。

![AQS链表-尾分叉2](photo/66、AQS链表尾分叉02(7).png) 

当一个节点成功切换 `tail` 后会将之前的 `tail` 节点的后继进行修改，而其他的线程会 `CAS` 失败后重新获取新的 `tail` 并尝试入队。但在其修改原 `tail` 后继之前如果我们正在遍历链表（如上图），此时从前（`head`）到后查找后继节点的方式是无法遍历到最后一个新加入的节点的，因为此时原 `tail` 节点也就是倒数第二个节点后继是为 `null` 的。而从后（`tail`）到前查找前驱的方式却可以遍历到所有的节点，这也解释了为什么在之后的很多遍历代码是从后向前遍历的，而不是从前向后。

###### 阻塞与唤醒

至此，当前线程已成功入队，此时则需要将线程在合适的位置和状态下阻塞，等待中断或其他线程的唤醒。


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

首先方法中定义了两个变量作为标记，分别是是否获取资源失败 `failed` 和是否存在中断 `interrupted` 。之后在代码中使用了自旋，只要当前节点的前驱节点是头结点 `head` ，则会去尝试获取资源并在获取到资源之后返回。若当前节点的前驱不为头节点或获取资源失败时，会使用 `shouldParkAfterFailedAcquire` 判断是否需要阻塞，若需要阻塞则会使用 `parkAndCheckInterrupt` 对线程进行阻塞，等待中断或被唤醒。

在继续分析接下来的代码之前，我们先分析一下 `shouldParkAfterFailedAcquire` 方法做了那些操作。

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus; // 获取当前节点上一级节点的状态
    if (ws == Node.SIGNAL) // 当上一级节点的状态，标记为需要唤醒当前节点时直接返回true
        return true;
    if (ws > 0) { // 上一级节点已经为取消状态
        // 接下来的操作是将从后到前所有已取消的节点删除
        do {
            // 将取消状态的节点从链表中删除
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0); // 循环删除直到遇到状态正常的节点为止
        // 将从后向前最前一个状态正常节点的后继修改为当前节点
        pred.next = node; 
    } else { // 前驱节点为正常状态
        // 将前驱节点状态修改为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

如果前驱节点的状态不为 `SIGNAL` ，则会将前驱节点状态修改为 `SIGNAL` 。如果前驱节点状态为已取消，则会从当前节点开始从后向前将节点所有已取消状态的节点删除，直到前驱节点的状态不为取消为止。

这个方法只有在当前节点的前驱节点状态为 `SIGNAL` 时才会返回 `true` ，否则就会删除已取消的节点或者修改前驱节点状态为 `SIGNAL` 状态，并返回 `false` 。因为在 `acquireQueued` 方法中调用这个方法是在自旋中调用，所以当返回 `false` 之后会在无法去获取资源时再次进入此方法，直到当前节点的前驱节点状态为 `SIGNAL` 并返回 `true` ，`acquireQueued` 中的判断会在此之后调用 `parkAndCheckInterrupt` 方法阻塞线程。

简单来说，此方法返回 `true` 就表示队列已经处理完成，线程可以进入阻塞，反之则需要继续自旋不可以阻塞。

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this); // 将线程进入等待状态
    return Thread.interrupted(); // 如果被唤醒，检查是否为中断
}
```

`park` 和 `sleep` 方法的异同之处在于其都会被中断所唤醒，而 `sleep` 会直接抛出中断错误，而 `park` 则只会被唤醒。此方法会在线程被唤醒之后返回线程是否在等待过程中中断过。

> 需要注意的是， `Thread.interrupted()` 会清除当前线程的中断标记位，若中断的线程在第一调用时返回 `true` 在第二次调用时则会返回 `false` 。

继续返回 `acquireQueued` 方法的剩余部分。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) { 
                setHead(node);
                p.next = null;
                failed = false;
                return interrupted;
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

线程被唤醒的情况有以下几种：

1、当前节点是 `head` 的后继节点，是拥有资源的线程在释放锁时唤醒的。

此时因为没有中断 `parkAndCheckInterrupt` 返回为 `false` ，直接重新执行循环。因为当前节点是 `head` 的后继，即 `p == head` 为 `true` ，此时线程会进入 `tryAcquire` 尝试获取线程。

2、当前线程被中断唤醒

此时 `parkAndCheckInterrupt` 返回值为 `true` ，会将中断标记变量 `interrupted` 设置为 `true` 。接着会重新执行循环，若不满足获取资源的条件线程依旧会在合适的位置继续阻塞。

说明线程被中断唤醒之后 `AQS` 只是做了一个标记，而会在获取资源后返回标记进行处理。

3、线程被其他的线程唤醒，且当前线程不是 `head` 的后继节点

当然，线程也不是只有可能被上面的两种方式唤醒，也有可能被其他不是 `head` 节点的线程所唤醒。此时当没有中断且前驱也不为 `head` 时，线程依旧会在合适的位置继续阻塞。

在线程获取到资源之后，首先设置 `head` 为当前节点，并将原 `head` 节点从队列中删除。然后将失败标志变量 `failed` 设置为 `false` 代表成功，最后返回是否存在中断。

在最后，线程要么是被 `head` 的线程唤醒后获取到了资源，要么就是在自旋时非正常退出（例如在执行 `tryAcquire` 时抛出错误）。无论是这两种那个情况都要在最后执行判断，检查线程是否正常的获取到了资源，如果失败标记变量 `failed` 为 `true` 时，则表示 `for` 循环是非正常退出的，且当前线程没有获取到资源。

最后会在 `for` 循环非正常退出时执行 `cancelAcquire` 方法。

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

这个方法看起来内容很多但是只做了两件事，将当前的 `node` 从链表中删除，如果当前 `node` 是 `head` 的后继节点时，还需要使用 `unparkSuccessor` 方法去唤醒当前节点的后继节点。

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
        // 从后向前遍历最接近当前node的状态正常的node
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0) // 如果遍历节点状态不为取消
                s = t;
    }
    if (s != null)
        // 若传入node的下一级node不为空且状态正常，或者传入node之后第一个不为空且状态正常的node
        // 则将这个node中的线程唤醒
        LockSupport.unpark(s.thread);
}
```

如果在 `acquireQueued` 中非正常退出 `for` 及当前线程没有获取到资源，`head` 节点的线程也在释放锁唤醒当前线程后停止执行，此时就需要当前线程将当前 `node` 从同步队列中删除并唤醒后面的节点，否则就会因为非正常退出造成死锁。

###### 中断处理

继续回到 `acquire` 方法中，此处的 `acquireQueued` 方法会返回线程是否被中断，若为 `true` 时，其会直接进入`selfInterrupt` 方法中，对线程进行自我中断。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && // 直接尝试获取资源
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt(); // 中断当前线程
}
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

在 `acquire` 方法中整个争锁以及入队的流程中，是不会响应中断，只会把中断的状态保存下来。

当然在 `AQS` 中也有响应中断的方法，即抛出线程中断异常的方法。

```java
public final void acquireInterruptibly(int arg) throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```

可以看到在执行 `tryAcquire` 之前，程序就判断当前线程是否存在中断，且调用的等待阻塞方法也与 `acquire` 方法不同。

```java
private void doAcquireInterruptibly(int arg) throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

可以看到 `doAcquireInterruptibly` 方法最大的不同之处在于，线程被中断唤醒之后只要检测到中断就会直接抛出 `InterruptedException` 异常，最后执行 `finally` 中的 `cancelAcquire` 方法。

###### 总结 acquire 方法

1. 使用 `tryAcquire` 尝试获取资源，若获取到直接返回。
2. 使用 `addWaiter` 方法将当前线程包装为节点加入等待队列中，并标记为独占模式。
3. 使用 `acquireQueued` 方法使线程进入等待状态，并在唤醒后尝试获取资源，获取到资源后返回线程是否存在中断。
4. 若存在中断，则使用 `selfInterrupt` 进行自我中断。

流程图：

![image-20220125172503874](photo/47、acquire方法流程图(7).png)

###### 释放独占锁

此方法是独占模式下线程释放共享资源的方法。

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

`tryRelease` 方法同样与 `tryAcquire` 方法相同，在 `AQS` 类中并没有具体的实现。其作用是释放共享资源，若全部释放完毕（`state` 为0）则返回 `true` ，反之则返回 `false` 。

`unparkSuccessor` 方法的作用已经在上面分析过，用于唤醒传入节点之后的第一个不为空且状态不为取消的节点。至此当前线程也就是拥有资源的线程，完成了资源的释放以及对后继节点的唤醒。

##### 共享锁

###### 获取资源

`acquireShared` 方法和独享模式的 `acquire` 方法流程基本相同，唯一不同的是当前资源可以被多个线程获取。这也是独享模式和共享模式最大个区别，独享模式同时只有一个线程获取到资源并执行同步代码，而共享模式可能会同时有多个线程获取到资源。

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0) // 获取资源
        doAcquireShared(arg); // 添加到等待队列
}
```

第一步依旧是尝试获取资源，不过与独占模式不同的是 `tryAcquireShared` 方法返回的是一个 `int` 值，同样需要自定义同步器去实现。第二步则是线程获取资源失败后将线程加入等待队列。

`tryAcquireShared` 的返回值为负数表示获取资源失败，0表示获取成功但是没有多余的资源，正数表示获取成功并且有剩余的资源其他线程还可以获取资源。

###### 入队阻塞

此方法用于等待被唤醒后获取资源。

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

此方法和独占锁调用的 `acquireQueued` 方法很相似，并且在其中也使用了 `addWaiter` 方法来讲节点添加到队列中。唯一不同之处在于，线程被唤醒获取到资源之后（返回值大于等于0）会调用 `setHeadAndPropagate` 方法，此方法的作用是设置 `head` 为当前节点，并在还有剩余资源时唤醒后继节点。

下面是 `setHeadAndPropagate` 的源码。

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // 获取head
    setHead(node); // 将head指向当前节点，且将thread和prev置空
    // 还有剩余资源 或 head不为空 或 head状态不为新建和取消
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next; // 获取到当前节点的下一级节点
        // 下一级节点为空或节点为共享模式
        if (s == null || s.isShared())
            doReleaseShared(); // 继续唤醒下一级节点
    }
}
```

首先先将传入的节点设置为头结点，然后在前驱节点条件满足的情况下，唤醒当前节点的下一个节点。

###### 释放资源

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

###### 唤醒后继

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
                    continue; // loop to recheck cases
                unparkSuccessor(h); // 替换成功，唤醒队列中head后面的线程
            }
            // head状态为新建，且将head状态从0替换到PROPAGATE时失败，continue重启循环
            else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue; // loop on failed CAS
        }
        // 若head未变化则中断循环
        if (h == head) // loop if head changed
            break;
    }
}
```

`head` 不为空且 `head` 不为 `tail` ，即表示同步队列中至少有两个节点。

获取 `head` 节点状态，`head` 状态为 `SIGNAL` 表示需要唤醒后面的节点，如果替换失败说明当前有其他线程在此之前已经替换了 `head` 状态，两种可能，要么有线程释放资源唤醒 `head` 后继，要么获取资源的线程在唤醒后继。如果替换成功，则说明可以唤醒后面的节点。

如果队列中有大于一个节点且在 `else if` 判断时发现 `head` 状态已经为0了，说明再此之前已经有线程将 `head` 状态修改为0，尝试将 `head` 状态修改为 `PROPAGATE` 。成功替换那么无论接下来 `head` 是否被替换成新的 `head` ，都会在执行 `setHeadAndPropagate` 时唤醒下一个节点。不成功就继续重启循环，获取到新的 `head` 再去唤醒下一级节点。

在执行到 `h==head` 时并没有新节点被唤醒并获取资源替换 `head` 则结束循环。

> 对于此处我的理解是，多唤醒节点无所谓，反正获取不到资源会继续 `park` ，但是如果因为没有唤醒导致问题，那就亏大了。

###### PROPAGATE

`PROPAGATE` 的引入是为了解决一个 `BUG` 的，首先先来看一下没有 `PROPAGATE` 时的各方法的代码。

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0) // deleted
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

释放资源的变化主要是不会再进行判断 `head` 的状态是否正常了，因为这个判断在共享锁的情况下很有可能导致无法唤醒后继节点。

```java
private void setHeadAndPropagate(Node node, int propagate) {
    setHead(node);
    if (propagate > 0 && node.waitStatus != 0) { // changed
        Node s = node.next;
        if (s == null || s.isShared())
            unparkSuccessor(node); // changed
    }
}
```

在没有 `PROPAGATE` 时共享锁和独占锁调用的方法还是相对来说比较统一的。

```java
private void unparkSuccessor(Node node) {
    compareAndSetWaitStatus(node, Node.SIGNAL, 0); // changed
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

唤醒节点的方法增加了一个判断如果 `node` 状态小于0才会设置为0，主要原因是 `doReleaseShared` 已经对状态做了修改。

当前有4个线程，其中线程1和2已经获取了共享锁正在执行，而3和4正在队列中等待。

![AQS PROPAGATE用处](photo/67、AQS-PROPAGATE问题01(7).png) 

此时线程1释放了锁，使用 `unparkSuccessor` 方法将 `head` 状态修改为0。然后唤醒了线程3，由于资源只足够线程3获取，因此线程3在使用 `tryAcquireShared` 获取资源之后返回了0。

![AQS PROPAGATE用处 1](photo/68、AQS-PROPAGATE问题02(7).png) 

紧接着此时线程2也释放了资源，此时线程2在执行 `releaseShared` 就会发现，`head` 节点状态已经是0了，所以就不会执行 `unparkSuccessor` 进行下一步的唤醒。

此时线程3执行到了 `setHeadAndPropagate` ，但是发现 `tryAcquireShared` 方法返回的值是0，所以也不会执行 `unparkSuccessor` 方法。

至此，队列中的线程4会因为 `BUG` 无法被唤醒。

参考：[AbstractQueuedSynchronizer源码解读 ](https://www.cnblogs.com/micrari/p/6937995.html) 

##### 总结

![AQS架构图解](photo/61、AQS架构图解(7).png) 

通过上面的分析可知节点状态的切换逻辑。

![AQS Node 状态切换2](photo/69、AQS单节点状态切换(7).png) 

队列中节点状态切换。

![AQS Node 状态切换](photo/70、AQS队列节点状态切换(7).png) 

#### Condition

##### 概述

在上面生产者消费者模式中使用了多种方法实现，由于 `notify` 和 `wait` 来实现阻塞和唤醒，但是 `notify` 和 `notifyAll` 并没有办法去指定一种类型的线程进行唤醒。就当系统中有若干线程，而分为两种类型分别是生产者和消费者，生产者需要唤醒的是消费者，而消费者需要唤醒的生产者。此时我们就需要将线程分类，并精确唤醒某个分类中的线程。

在上面的介绍中，我们已经使用 `Condition` 实现了这个精确唤醒某个分类的操作，下面就来分析一下源码。

##### 源码解析

###### 接口类


```java
public interface Condition {
    void await() throws InterruptedException;
    void awaitUninterruptibly();
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    boolean awaitUntil(Date deadline) throws InterruptedException;
    void signal();
    void signalAll();
}
```

`AQS` 中的 `ConditionObject` 实现的就是这个接口类，而这个接口类定义了很多线程唤醒和阻塞的方法。

###### 链表结构

查看 `ConditionObject` 中的成员变量。

```java
private transient Node firstWaiter; // 头结点
private transient Node lastWaiter; // 尾结点
```

用于指向条件队列的头结点和尾节点。

在 `Node` 类中有一个 `nextWaiter` 变量，在上面分析时用来标记独占锁与共享锁，在 `Condition` 中又有了一个新的作用——单向链表。

![AQS 条件队列](photo/71、AQS条件队列结构(7).png) 

可以看到在条件队列中，`prev` 和 `next` 都是 `null` 使用到的有 `nextWaiter` 和 `thread` 等。同步队列用来让线程争锁，而条件队列用来给线程分类，所以条件队列中的线程最终还是要到同步队列中去争锁的。

###### 入队阻塞

首先我们先查看入队阻塞方法。

```java
public final void await() throws InterruptedException {
    // 检查线程是否被中断
    if (Thread.interrupted())
        throw new InterruptedException();
    // 将当前节点加入链表中
    Node node = addConditionWaiter();
    // 释放当前线程获取的资源
    int savedState = fullyRelease(node);
    int interruptMode = 0; // 是否中断
    // 当前线程是否在同步队列中
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this); // 阻塞
        // 检查中断
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // 入同步队列争锁，此方法就是独占锁使用的阻塞争锁方法
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    // 当前节点后还有节点
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters(); // 清理无用节点
    // 是否存在中断
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode); // 处理中断
}
```

首先检测是否存在中断，直接抛出中断异常。进入此方法表示线程必须阻塞，所以先将线程打包为 `node` 后入队。既然要调用 `await` 方法，那么线程一定是当前持有锁的线程，所以需要释放资源之后才可以阻塞，否则就会变成死锁。当前方法的代码就分析到这里，我们先观察 `addConditionWaiter` 节点入队方法。

```java
private Node addConditionWaiter() {
    // 获取尾端节点
    Node t = lastWaiter;
    // 如果尾端节点不是condition状态，需要清除取消的节点
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters(); // 清除取消的节点
        t = lastWaiter; // 重新获取尾节点
    }
    // 为当前线程创建一个新的节点
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    // 若尾结点为空
    if (t == null)
        // 设置首节点为当前线程节点
        firstWaiter = node;
    else
        // 设置尾结点的下一个节点为当前线程节点
        t.nextWaiter = node;
    // 将尾结点设置为当前节点
    lastWaiter = node;
    // 返回当前线程节点
    return node;
}
```

首先方法需要获取到尾结点，但是当前的尾结点有可能已经进入了同步队列，所以需要判断一下尾结点的状态是否为 `condition` 。清除掉无用的节点之后，获取到最新正常的尾结点开始入队操作，首先创建当前线程的 `node` 状态设置为 `condition` ，在正常情况下条件队列中的节点状态就应该是 `condition` 。最后返回当前线程的节点。

###### 清洗队列

接下来查看清洗队列，也就是删除无用节点的方法。

```java
private void unlinkCancelledWaiters() {
    // 获取首节点
    Node t = firstWaiter;
    Node trail = null;
    // 当前节点不为空
    while (t != null) {
        // 获取当前节点下一个节点
        Node next = t.nextWaiter;
        // 当前节点的状态不为condition，需要删除
        if (t.waitStatus != Node.CONDITION) {
            // 将当前节点与后一个节点断开连接
            t.nextWaiter = null;
            // 如果当前节点的上一个节点为空
            if (trail == null)
                // 那么首节点就是当前节点的下一个节点
                firstWaiter = next;
            else
                // 如果不为空则需要将上一个节点和下一个节点连接
                trail.nextWaiter = next;
            // 如果下一个节点为空，表示链表已经结束
            if (next == null)
                // 将尾结点设置为当前节点的上一个节点
                lastWaiter = trail;
        }
        else
            // 如果当前节点不需要删除，将当前节点设置为下次遍历时t的上一个节点
            trail = t;
        // 设置下一次遍历的主节点为当前节点的下一个节点
        t = next;
    }
}
```

当前链表只有 `nextWaiter` 将整个链表串联起来，因此这是个单项链表只可以从前向后遍历。首先获取到链表的第一个节点从此开始，每次在循环完成后 `t` 就变成了下一个节点，如果 `t` 为空则表示这个链表没有节点或者当前列表已经遍历完成。在每次循环时判断当前节点 `t` 的状态是否为 `condition` ，而 `trail` 则表示当前节点的上一个节点，`next` 则是当前节点的下一个节点。如果不为 `condition` 则表示当前节点的状态是有问题的，则需要删除，一直遍历到最后一个节点结束。

###### 释放资源

接下来分析释放资源方法。

```java
final int fullyRelease(Node node) {
    // 失败默认为true
    boolean failed = true;
    try {
        // 获取到state
        int savedState = getState();
        // 释放资源
        if (release(savedState)) {
            // 释放成功
            failed = false;
            // 返回释放前的state
            return savedState;
        } else {
            // 释放失败直接抛出错误
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed) // 释放失败
            // 将当前节点设置为取消状态
            node.waitStatus = Node.CANCELLED;
    }
}
```

此方法首先获取到 `state` 的值，直接释放所有资源，若成功则返回释放资源的数量，若失败则会将当前节点设置为取消状态，返回的资源数量需要在之后 `await` 阻塞结束争锁时进行恢复。

在 `release` 方法调用的 `tryRelease` 方法中需要去判断当前线程是否是获取资源的线程，若不是就需要抛出错误，此处就会进入 `finally` 方法中取消当前节点，释放操作正常失败也是此流程。

###### 唤醒节点

在分析 `isOnSyncQueue` 方法之前，需要先分析唤醒节点部分代码的操作。

```java
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

`isHeldExclusively` 用于判断当前线程是否为拥有资源的线程，此方法由具体的实现类实现，此处调用 `signal` 一定是已经获取了资源的线程。然后获取首节点若首节点不为空，则表示条件队列中一定有节点，调用 `doSignal` 方法。

```java
private void doSignal(Node first) {
    do {
        // 第一个节点如果为空
        if ( (firstWaiter = first.nextWaiter) == null)
            // 
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```

依次尝试唤醒节点，直到唤醒第一个可以被成功唤醒的节点，或者队列中一个节点也不能唤醒。

```java
final boolean transferForSignal(Node node) {
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```

首先是否 `CAS` 操作修改节点状态，让其他线程在执行时计时发现此节点已经被唤醒入队争锁，若修改失败则返回 `flase` 继续尝试唤醒下一个节点。接着让节点进入同步队列，等待并争锁。入队完成后此时需要将 `enq` 方法返回的当前节点入队后的上一个节点的状态修改为 `SIGNAL` ，以便上一个节点获取锁之后在释放锁时可以正常唤醒当前入队的节点。如果上一个节点的状态为已取消或者修改状态失败，直接唤醒这个节点，节点被唤醒之后会被在下一个循环时调用的 `shouldParkAfterFailedAcquire` 直接删除，使同步队列恢复正常。

###### 节点位置

因为当前的队列中的节点会在被中断和 `single` 等方法唤醒后需要进入同步队列，此时因为是并发所以要一段较为复杂的判断来判断同步队列中是否存在指定的线程。

```java
final boolean isOnSyncQueue(Node node) {
    // 节点状态是condition 或者 节点的前驱已经有值
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;

    if (node.next != null) // 已经在同步队列中
        return true;
    return findNodeFromTail(node);
}

private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) { // 从后到前遍历节点直到找到或结束为止
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```

节点状态是 `condition` 说明节点正常，并没有进行入同步队列操作那么节点必定不会再同步队列中。如果节点的前驱为空那么当前节点也并没有在同步队列，因为入队的三步第一步就是将节点的前驱和队列进行连接。

那么如果节点的后继不为空则表示入队的三步操作已经完全完成了，说明节点必定在同步队列中。如果上面都不满足接下来就没有办法通过单个节点的内容来判断其是否在同步队列中了，所以直接对同步队列从后到前进行遍历判断。

`findNodeFromTail` 的作用就是从同步队列中查询是否存在指定节点，有为 `true` ，无为 `false` 。

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
final void lock() {
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

查看非公平锁中的 `lock` 方法。

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

进入 `lock` 就会首先获取一次资源，若无法获取才会执行 `AQS` 的 `acquire` 方法，在其中调用 `tryAcquire` 方法实现功能。非公平锁的 `tryAcquire` 方法调用了 `Sync` 类中的 `nonfairTryAcquire` 方法来实现。

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

###### 解锁操作

`ReentrantLock` 中解锁操作使用 `unlock` 方法，查看 `unlock` 方法。

```java
public void unlock() {
    sync.release(1);
}
```

`unlock` 方法调用了 `Sync` 对象的 `release` 方法，其实也就是调用了 `AQS` 对象的 `release` 方法。

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

在 `release` 方法中调用了 `Sync` 真正实现的 `tryRelease` 方法。

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases; // 减去要释放的次数
    // 若在解锁时，当前线程不是拥有资源的线程，则会抛出错误
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false; // state是否归零，即真正解锁
    if (c == 0) { // state为0，即为解锁
        free = true;
        // 将储存的拥有资源的线程删除
        setExclusiveOwnerThread(null);
    }
    // 设置state的值。
    setState(c);
    return free;
}
```

首先减去需要释放的数量得出需要设置的剩余的值，再检测当前线程是否为拥有资源的线程，最后判断是否解锁并设置 `state` 的值。在设置值时并没有使用 `CAS` 方法，因为此时进入此方法可以执行到 `setState` 方法的只可能是当前拥有资源的线程，因此没有必要使用 `CAS` 方法。

> 注意：在使用 `ReentrantLock` 或者类似的锁时，`unlock` 方法必须在 `finally` 块中释放，如果在代码中出现错误，会直接导致无法释放锁而死锁。

#### ReentrantReadWriteLock

##### 概述

上面介绍的 `ReentrantLock` 是独享锁的一种，而 `ReentrantReadWriteLock` 则是包含共享锁与独享锁。独享锁与共享锁的区别是，独享锁是排它锁一个线程获取资源，其他线程就需要等待。而共享锁的资源可以被多个线程同时持有，且共享锁的线程只能读数据不能修改数据。

共享锁的场景是针对读写操作的，当一个资源读操作比较频繁而写操作较少时，在读数据时应该使用共享锁，多个线程可以同时读取数据。当在写操作时应该使用排它锁，同时只可以有一个线程进行写操作。

在 `ReentrantReadWriteLock` 锁中有两个锁，一个是读锁（共享锁），一个是写锁（排它锁）。

##### 解析

###### 类继承图

![image-20220216171917560](photo/62、ReentrantReadWriteLock继承关系(7).png) 

在 `ReentrantReadWriteLock` 也分为公平锁与非公平锁，再次基础上锁变为两种类型：读锁与写锁。

###### 读写切换

在之前的设计中 `state` 只可以判断是否被线程获取到锁，但是在读写锁的需求下无法去区分到底是读锁或写锁。读写锁需要在一个整型变量及 `int` 变量上来进行读写锁的切换，`int` 型变量的长度是32位，将32位切割成两个 16位，高位表示读，低位表示写即可解决这个问题。

那在获取读或者写锁的状态时，只需要将无用位抛弃即可。

- 获取写状态：将前16擦除即可，使用按位与操作 `X&0x0000FFFF` ，将前16位全部置0。
- 获取读状态：将后16擦除，使用移位操作，将前16位移到后16位，空位补0并忽略符号位 `X>>>16` 。
- 写状态加1：直接加1即可 `X+1` 。
- 读状态加1：读状态加一有两种方法，直接加第16为1的任意进制的数 `X+0x00010000` 或 `X+65536` ；第二种方法使用1向前位移16位后与值相加 `X+(1<<16)` ；这两种方法结果是相同的。

```java
static final int SHARED_SHIFT   = 16; // 位移的大小
static final int SHARED_UNIT    = (1 << SHARED_SHIFT); // 读操作加的值
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1; // 分成16位之后最大的值
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1; // 获取写操作与运算的值，1位移16位后减一则变成，前16位0，后16位为1
```

如果 `X&0x0000FFFF` 大于0，则表示写锁被获取；若 `X>>>16` 大于0，则表示读锁被获取。

###### 读写锁对比

```java
public static class WriteLock implements Lock, java.io.Serializable {
    private final Sync sync;
    // ....
}

public static class ReadLock implements Lock, java.io.Serializable {
    private final Sync sync;
    // ....
}
```

写锁和读锁与 `ReentrantLock` 同样，实现了 `Lock` 接口。因为 `ReentrantReadWriteLock` 中有两个不同的锁，所以需要分别去实现，所以 `ReentrantReadWriteLock` 不会实现 `Lock` 接口，而是由具体的读锁和写锁的类来实现。

```java
// write lock
public void lock() {
    sync.acquire(1);
}
// read lock
public void lock() {
    sync.acquireShared(1);
}
```

在写锁的 `lock` 方法中，直接调用了 `AQS` 的 `acquire` 方法，`acquire` 方法又调用 `Sync` 类中的 `tryAcquire` 方法。而读锁中调用的则是 `acquireShared` 方法，用来实现共享锁的功能。

```java
// write lock
protected final boolean tryAcquire(int acquires) {
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取state值
    int c = getState();
    int w = exclusiveCount(c); // c&0x0000ffff 获取低16位内容
    if (c != 0) {
        // state不为0但是低16位为0，则读锁已被获取；或者当前线程不为拥有资源线程；
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        // 低16加自增的值大于16位可容纳的最大值，溢出报错
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // 来到此处的线程必定是低16大于0且当前拥有资源线程为当前线程，可重入设计，直接加值即可
        // 设置state低16位直接相加
        setState(c + acquires);
        return true;
    }
    // 公平锁：此方法检查队列中是否有等待的线程；非公平锁：直接返回false；
    if (writerShouldBlock() ||
        // 没有成功获取到资源
        !compareAndSetState(c, c + acquires))
        以上两个条件有一个成立就会返回，进入下一步进行入队
        return false;
    // 此处排除了所有其他条件，state也修改完毕，直接将拥有资源的线程修改为当前线程即可
    setExclusiveOwnerThread(current);
    return true;
}
```

可以获取写锁的条件有两种：一是没有线程获取读锁和写锁；二是获取写锁的是当前线程。

可以从上方的代码中得知：

1. 首先拿到整个 `sate` 的内容 `c` ，再拿到低16位读锁的计数 `w` 。若 `c` 不为0，则表示有线程获取了读锁或者写锁或两者都有，执行第2步。若 `c` 为0，则没有线程获取到读锁和写锁直接执行第4步。
2. 首先 `c` 不为0，第一个判断有两个条件。第一个，低16位写锁为0，则表示读锁已经被获取了直接返回 `false` 。第二个，当前线程不是拥有锁的线程也直接返回 `false` 。只有当 `w` 写锁计数为0，且当前线程是拥有锁的线程，才会执行下一步，接下来剩下的部分肯定就是写锁可重入设计的实现代码了。
3. 判断低16位重入新增之后的值不会超过16位的最大值，之后直接设置 `state` 的值即可，当前即是拥有锁的线程，不必进行 `CAS` 设置最后返回 `true` 。
4. 来到第4步则表明当前的线程可以去尝试获取写锁了，接下来是公平锁与非公平锁的实现。首先第一个`writerShouldBlock` 方法在公平锁下需要判断队列中是否存在等待的线程，有则为 `true` 没有则为 `false` 。而在非公平锁下直接会返回 `false` ，这就是插队的实现。第二个方法直接使用 `CAS` 方法，去修改 `state` 的值来去获取锁，经过上面的一大段判断此时当然有可能已经被其他的线程获取了锁，所以用取反操作，成功执行下一步，不成功直接返回 `false` 。
5. 来到此处的线程必定是已经成功修改 `state` 的值获取了锁，此时最后一步只需要将拥有资源的线程修改为当前线程最后返回 `true` 即可。

这就是线程尝试获取锁 `tryAcquire` 方法的逻辑，当然，读锁肯定要比写锁相对复杂，以下是线程获取读锁的源码。

```java
// 此对象继承了ThreadLocal并用于储存HoldCounter对象
private transient ThreadLocalHoldCounter readHolds;
// 用于在获取读锁时短暂缓存
private transient HoldCounter cachedHoldCounter;
// 第一个获取到资源的读锁
private transient Thread firstReader = null;
// 第一个获取到资源的读锁重入次数
private transient int firstReaderHoldCount;
```

一些用于在获取读锁时使用的全局变量。

```java
Sync() {
    readHolds = new ThreadLocalHoldCounter();
    setState(getState()); // ensures visibility of readHolds
}

// 在创建ThreadLocal时也创建好HoldCounter
static final class ThreadLocalHoldCounter
    extends ThreadLocal<HoldCounter> {
    public HoldCounter initialValue() {
        return new HoldCounter();
    }
}
```

在实例化 `Sync` 时会将线程的计数器创建好，此处的 `initialValue` 即是返回默认的初始值。

```java
// read lock
protected final int tryAcquireShared(int unused) {
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取state
    int c = getState();
    // 低16位不为0且拥有资源的线程不为当前线程，返回-1
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    // 获取读锁state值，高16位
    int r = sharedCount(c);
    /**
     * 非公平锁：调用apparentlyFirstQueuedIsExclusive方法，队列中head下一项有等待的线程，
     *		且等待的线程不是共享模式，即独占模式的写锁，返回true此读线程需要等待。
     * 公平锁：队列中有等待的线程则当前读线程需要等待，返回true。
     */
    if (!readerShouldBlock() &&
        // 当前的高16位小于最大值
        r < MAX_COUNT &&
        // 获取读锁
        compareAndSetState(c, c + SHARED_UNIT)) {
        // 若在之前的高16位为0，则表示当前线程是第一个读线程
        if (r == 0) {
            // 设置第一个读线程
            firstReader = current;
            // 初始化值
            firstReaderHoldCount = 1;
            // 第一个读线程和为当前线程
        } else if (firstReader == current) {
            // 可重入设计，直接自增
            firstReaderHoldCount++;
        } else { // 不是第一个获取读锁的线程
            // 获取计数器
            HoldCounter rh = cachedHoldCounter;
            // 计数器为空或者计数器的tid不是当前相册好难过tid
            if (rh == null || rh.tid != getThreadId(current))
                // 将计数器的tid修改为当前线程tid
                cachedHoldCounter = rh = readHolds.get();
            // 计数器是当前线程，且计数器为0
            else if (rh.count == 0)
                // 将计数器设置为线程的变量
                readHolds.set(rh);
            // 计数自增
            rh.count++;
        }
        // 读锁获取到了资源
        return 1;
    }
    // 队列中有内容，或大小超出，或获取读锁是失败
    return fullTryAcquireShared(current);
}
```

关于读锁的获取资源操作较写锁相对复杂，读锁可以获取到资源需要几个先决条件，首先不可以有线程已经获取到写锁，且非公平锁等待队列中 `head` 下一个节点不可以是写线程，公平锁则是等待队列中不可以有等待的线程。

读锁的复杂主要来源于可以多个线程同时获取读锁并执行，所以如果在单个读线程下，只需要在 `Sync` 类中新建变量来储存当前获取读锁的线程和其重入次数即可。但是在多个读线程的情况下，一个计数器就没有办法储存多个线程的不同状态，此时要使用到 `ThreadLocal` 来对不同的线程设置不同的计数器。下方代码是计数器对象：

```java
static final class HoldCounter {
    int count = 0;
    // Use id, not reference, to avoid garbage retention
    final long tid = getThreadId(Thread.currentThread());
}
```

其中储存了读线程重复获取读锁的次数，而 `tid` 则是储存线程的 `ThreadID` 。

```java
final int fullTryAcquireShared(Thread current) {
    // 将计数器置空
    HoldCounter rh = null;
    for (;;) {
        // 获取state
        int c = getState();
        // 判断低16位，写锁已被获取
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
        // 与上一个方法相同
        } else if (readerShouldBlock()) {
            // 第一个读锁为当前线程，表示第一个读锁的计数大于0
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                // 计数器为空
                if (rh == null) {
                    // 将缓存的计数器设置为当前计数器
                    rh = cachedHoldCounter;
                    // 计数器为空或者计数器tid不为当前线程
                    if (rh == null || rh.tid != getThreadId(current)) {
                        // 从ThreadLocal中获取到当前线程的计数器
                        rh = readHolds.get();
                        // 若计数为0则移除ThreadLocal中的计数器
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                // 计数器为0返回-1，当前线程加入队列
                if (rh.count == 0)
                    return -1;
            }
        }
        // 获取高16位大小已经是最大值，直接抛错
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // 获取读锁
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            // 若高16位为0
            if (sharedCount(c) == 0) {
                // 第一个读线程为当前线程
                firstReader = current;
                firstReaderHoldCount = 1;
            // 当前线程为第一个读线程
            } else if (firstReader == current) {
                // 可重入设计，第一个读线程计数自增
                firstReaderHoldCount++;
            } else {
                // 计数器为空
                if (rh == null)
                    // 赋值缓存计数器
                    rh = cachedHoldCounter;
                // 计数器为空或者计数器不是当前线程
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                // 计数器自增
                rh.count++;
                // 设置缓存计数器 
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

上面的 `fullTryAcquireShared` 方法中有很多的部分都与 `tryAcquireShared` 方法中的内容重复，因此我绘制了一个流程图来方便理解这两个方法究竟做了那些操作。

![](photo/64、读锁获取资源流程图(7).png) 



图中可以看到方块 `fullTryAcquireShared` 以下延伸出的部分，同样操作的已经标记成一样颜色，可以看到有两个部分是重复的。

首先看 `tryAcquireShared` 方法：

1. 首先拿到当前线程和 `state` ，第一个判断用来判断是否已经有线程获取到了写锁，且写锁不是当前线程返回-1表示没法获取锁。为什么要判断是不是当前线程呢？这就要说到读写锁的另一个重点，写锁降级；
2. 从 `state` 取高16位表示读锁的部分取名 `r` ，接下来的判断一共有三个方法，而 `&&` 符的特点是只有前面为 `true` 时才会执行之后的判断。第一个方法判断是否线程应该被阻塞，非公平锁时 [^1] ，公平锁时[^2] 。返回 `true` 时直接执行 `fullTryAcquireShared` 方法，返回 `false` 紧接着执行下一个判断。
3. 如果 `readerShouldBlock` 返回 `false` 才会执行接下来的方法，用于除错和获取锁。第二个用于判断高16位的值是否溢出，如果没有溢出则表示可以获取资源。第三个方法直接使用 `CAS` 操作修改读锁高16位的值，成功则进行下一步，不成功则直接执行 `fullTryAcquireShared` 方法。
4. 此时，当前的线程已经获取到了读锁，紧接着就是一串 `if else` 判断。第一个 `r==0` ，此处使用的 `r` 还是在进行判断之前从成员变量中获取到的内容，这个 `r` 不会因为上一步的 `CAS` 操作而变化。那这个判断的意思就是，在获取读锁之前 `state` 的高16位为0，即当前的线程是第一个获取读锁的线程，设置 `firstReader` 和 `firstReaderHoldCount` 变量，后跳转到第10步。
5. `if else` 的第二个判断 `firstReader==current` ，这个判断的前提是 `r` 之前已经不为0，若第一个获取读锁的线程就是当前线程时，则表示这是当前线程多次重入锁，直接自增 `firstReaderHoldCount` 的值，跳转到第10步。
6. 最后的一个 `else` 分支，能到达此处的线程必定不是第一个获取读锁的线程，在当前线程获取读锁之前已经有线程也获取了读锁。此处需要表达的就是现在有多个线程都获取了读锁，单单只有一个储存第一个线程的变量和重入的次数已经无法需求，所以此处需要动用 `ThreadLocal` 来为每个线程单独储存计数器。
7. 首先获取到缓存的计数器 `cachedHoldCounter` ，因为不能确定当前缓存中有没有值，所以在接下来的判断中，需要确保 `rh` 是当前的线程的计数器。首先 `rh` 为空或者 `rh` 中储存的 `tid` 不是当前线程的 `tid` ，则表示计数器需要重新从 `ThreadLocal` 中去获取，调用 `ThreadLocalHoldCounter` 对象的 `get` 方法获取到本线程的计数器并赋值给 `rh` 变量和缓存计数器变量，跳转到第9步。
8. 当执行到 `else if` 步骤的线程，当前的缓存计数器及 `rh` 变量一定是当前线程的计数器，然后将计数器设置到 `ThreadLocal` 中。
9. 最后将计数器中的 `count` 值自增。
10. 返回 1，表示获取到了资源。

接下来是 `fullTryAcquireShared` 方法：进入此方法的线程应该满足三个条件，读线程应该阻塞、读线程 `state` 不小于最大值并且 `CAS` 获取锁时失败。

1. 首先第一部分有两种可能会直接返回-1，其实也是在排除没有条件去尝试获取锁的线程。一种是写锁已被获取并且获取写锁的不是当前线程，直接返回-1。另一种情况是当写锁没有被获取，且当前读线程应该阻塞（具体逻辑请看[^1] [^2] 的描述）且线程是第一次尝试获取锁（排除不是重入的情况），返回-1。
2. 读锁的 `state` 大小检测，已满则抛出错误。
3. 如果上面的条件都没有满足，其实现在这个线程有以下几种情况：已经获取写锁的线程、第一个已经获取读锁的线程在重入，不应该阻塞的读线程在第一次获取读锁，或者是已经获取读锁的线程在重入。上面的三种情况只要有一个可以满足且计数后 `state` 不会溢出，则表明线程是绝对可以获取锁的，因此在使用 `CAS` 尝试失败后需要返回第一步重新去判断后获取锁。
4. 剩下的操作与 `tryAcquireShared` 方法的4到10步操作相同。

[^1]: 非公平锁：调用 `apparentlyFirstQueuedIsExclusive` 方法，队列中 `head` 下一项有等待的线程，且等待的线程不是共享模式，即独占模式的写锁，返回 `true` 此读线程需要等待。

[^2]: 公平锁：队列中有等待的线程则当前读线程需要等待，返回 `true` 。

###### 释放资源

首先看写锁释放资源的方法。

```java
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively()) // 判断当前线程是否是拥有资源的线程
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

写锁释放资源的逻辑很简单，首先判断当前线程是否为拥有资源的线程。然后将 `state` 减去要释放的值，判断低16位的写锁在减去释放值之后是否为0，为0则表示当前线程彻底释放了写锁，将拥有资源的线程设置为空后设置 `state` 即可。

下面是读锁释放资源的方法。

```java
protected final boolean tryReleaseShared(int unused) {
    // 获取当前线程对象
    Thread current = Thread.currentThread();
    // 第一个读线程是当前线程
    if (firstReader == current) {
        // 当前读锁只获取了一次，直接释放锁，将firstReader储存的thread对象设置为空
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            // 第一个读锁不止获取了一次，将计数值减一
            firstReaderHoldCount--;
    } else { // 当前线程不是第一个读线程
        // 获取到缓存的计数器
        HoldCounter rh = cachedHoldCounter;
        // 如果当前缓存计数器不是当前线程的计数器，则从ThreadLocal中获取本线程的计数器
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        // 如果线程计数器值小于等于1
        if (count <= 1) {
            // 移除计数器
            readHolds.remove();
            // 计数器值小于等于0
            if (count <= 0)
                // 没有获取到读锁就释放，报错
                throw unmatchedUnlockException();
        }
        // 将计数器减1
        --rh.count;
    }
    // 自旋释放
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

同样释放资源的方法也是读锁较为复杂，首先判断当前线程是否为第一个获取读锁的线程。第一个获取读锁的线程直接减少 `firstReaderHoldCount` 和为0时置空 `firstReader` 即可，若不为第一个获取读锁的线程则需要去获取此线程的计数器进行操作。在计数器操作完成后，还需要修改 `state` 的值，此处使用了 `CAS` 加自旋的方式去修改，因为读锁是多个线程都可以同时获取，如果直接使用 `setState` 方法会引发线程安全问题，所以需要用到 `CAS` 方法。

###### 总结

在分析完源码之后，可以发现当一个线程已经获取到写锁时，再去获取读锁也是可以成功获取的，而这时这个线程释放掉写锁也可以只单独持有读锁，这就是写锁降级。

读锁的逻辑相对复杂，其第一个获取读锁的线程计数器直接使用 `Sync` 类的成员变量储存，而其他之后获取读锁的线程的计数器则使用 `ThreadLocal` 来储存。还使用了 `cachedHoldCounter` 变量，用于储存一个线程中的计数器。

可能在阅读时会有一个疑问，这个 `cachedHoldCounter` 变量有点脱裤子放屁的意思，既然 `ThreadLocal` 中已经储存了每个线程响应的计数器，直接 `get` 拿来用不就行了么？为什么还需要储存到 `cachedHoldCounter` 中？

其实这个处理方式只是为了性能而已，每次从 `ThreadLocal` 中取出值肯定要比直接去成员变量中拿更加耗费资源。`cachedHoldCounter` 就在这中间起到一个缓冲的作用，先从成员变量中拿到缓存的计数器，判断是不是当前线程的计数器再去决定是否使用 `ThreadLocal` 中的计数器。

还有值得注意的一个小细节，就是在第一个获取读锁的线程在变量中直接储存的是 `Thread` 对象，而在计数器 `HoldCounter` 中储存的只是一个 `long` 型的 `tid` 。

可以编写两个对象，一个其中有一个 `Thread` 对象，另一个其中有一个 `long` 型变量。

```java
static class ObjThread {
    protected Thread t;
}

static class ObjLong {
    protected long tid;
}
```

将其分别多次创建对象并计算用时。

```java
public static void main(String[] args) {
    ObjThread[] ots = new ObjThread[10000000];
    long s1 = System.currentTimeMillis();
    for (int i = 0; i < 10000000; i++) {
        ots[i] = new ObjThread();
    }
    long e1 = System.currentTimeMillis();
    System.out.println(e1-s1);
    
    ObjLong[] ols = new ObjLong[10000000];
    long s2 = System.currentTimeMillis();
    for (int i = 0; i < 10000000; i++) {
        ols[i] = new ObjLong();
    }
    long e2 = System.currentTimeMillis();
    System.out.println(e2-s2);
}
```

最终结果分别是 `2143` `1011` ，多次测试的结果依旧是储存 `Thread` 变量的对象要比储存 `long` 型变量的对象创建时间要高出一倍。因此我们也可以得出结论，为什么 `Doug Lea` 在编写计数器的代码时，使用 `long` 型变量来储存 `tid` ，而不是直接使用 `Thread` 对象。

#### 自定义锁

可以使用上面学到的知识，简单的创建一个可重入的非公平锁。

```java
public class MyLock {

    private final Sync sync;

    static class Sync extends AbstractQueuedSynchronizer{

        @Override
        protected boolean tryAcquire(int arg) {
            Thread thread = Thread.currentThread();
            int s = getState();
            if (s == 0){
                if (compareAndSetState(0, arg)){
                    setExclusiveOwnerThread(thread);
                    return true;
                }
            }else if (getExclusiveOwnerThread() == thread){
                int c = getState() + arg;
                if (c < 0){
                    throw new Error("state超过最大值，已溢出");
                }
                setState(c);
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            Thread thread = Thread.currentThread();
            int c = getState() - arg;
            if (getExclusiveOwnerThread() != thread){
                throw new IllegalMonitorStateException("不可用未持有锁的线程释放资源");
            }
            boolean result = false;
            if (c == 0){
                result = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return result;
        }
    }

    public MyLock(){
        sync = new Sync();
    }

    public void lock(){
        sync.acquire(1);
    }

    public void unlock(){
        sync.release(1);
    }

}
```

### 自旋锁

#### 概述

自旋锁的出现是因为在不同锁的阻塞和唤醒操作，是由系统底层切换 `cpu` 状态来实现的，而这种状态的转换是十分耗费时间的。当在锁定时间非常短的操作中（即在锁定代码中只执行了非常少的操作），在多线程不断切换的过程中会出现很大的资源浪费。

自旋锁即为让线程执行一个空循环，让其处于“等待”而非“阻塞”状态，减少 `cpu` 状态的切换从而得到更好的性能。当然，自旋锁本身也有很大的缺陷，一旦当锁定代码操作较多时，由于自旋锁并没有放弃 `cpu` 时间，也会造成处理器资源被浪费。`JVM`  提供了一个参数来配置自旋的次数，若一直无法获取锁，则会依旧将线程挂起。

```
-XX:PreBlockSpin
```

> 自旋锁在 `JDK1.4.2` 中引入，使用 ``-XX:+UseSpinning` 来开启。`JDK 6` 中变为默认开启，并且引入了自适应的自旋锁（适应性自旋锁）。

#### 实现

自旋锁有三种实现方式 `TicketLock` `CLHLock` `MCSLock` 。

##### TicketLock

`TicketLock` 是公平锁，每当线程获取锁时，该线程就会被分配一个排队号。`TicketLock` 内部有一个服务号，每当线程释放锁时，服务号就会递增。排队号与服务号一一对应，排队号与服务号相同的线程可以获取到锁。

```java
import java.util.concurrent.atomic.AtomicInteger;

public class TicketLock {
    //当前服务号
    private AtomicInteger serviceNum = new AtomicInteger();
    //每个线程lock时自增，并设置到线程的ThreadLocal中
    private AtomicInteger ticketNum  = new AtomicInteger();
    //用于保存每个线程的票号
    private static final ThreadLocal<Integer> LOCAL = new ThreadLocal<Integer>();

    //获取TicketLock的流程如下：
    //1、线程进入lock方法，原子地获取当前加1的票号
    //2、将票号设置到本地线程ThreadLocal中
    //3、自旋比较本地线程的票号是否和当前服务号，直至相等为止
    public void lock() {
        int myticket = ticketNum.getAndIncrement();
        LOCAL.set(myticket);
        //如果线程持有的票号和当前的服务号不相等就自旋等待
        while (myticket != serviceNum.get()) {}
    }
    
    public void unlock() {
        int myticket = LOCAL.get();
        //释放锁，将服务号加1
        serviceNum.compareAndSet(myticket, myticket + 1);
    }
}
```

**TicketLock的缺点**：每次都要读写一个serviceNum服务号，并且还要保证读写操作在多个处理器缓存之间进行同步，这样会增加系统压力，影响性能。

##### CLHLock

CLHLock采用链表的形式进行排序，有如下好处：

- 公平，FIFO，先来后到的顺序进入锁
- 没有竞争同一个变量，因为每个线程都是在等待前继释放锁

```java
import java.util.concurrent.atomic.AtomicReferenceFieldUpdater;

public class CLHLock {
    public static class CLHNode {
        private volatile boolean isLocked = true;
    }

    @SuppressWarnings("unused")
    private volatile CLHNode tail;
    //每个线程私有的，用于保存每个线程所对应的CLHNode节点
    private static final ThreadLocal<CLHNode> LOCAL = new ThreadLocal<CLHNode>();
    //用于原子地更新CLHLock中的tail字段
    private static final AtomicReferenceFieldUpdater<CLHLock, CLHNode> UPDATER = 						AtomicReferenceFieldUpdater.newUpdater(CLHLock.class, CLHNode.class, "tail");

    //获取CLHLock锁的过程：
    //1、新建CLHNode节点并设置到当前线程的ThreadLocal中
    //2、原子地更新tail字段并获得前置节点
    //3、自旋判断前置节点是否释放锁
    public void lock() {
        CLHNode node = new CLHNode();
        LOCAL.set(node);
        CLHNode preNode = UPDATER.getAndSet(this, node);
        if (preNode != null) {
            while (preNode.isLocked) {
            }
            //用于GC
            preNode = null;
            LOCAL.set(node);
        }
    }

    public void unlock() {
        CLHNode node = LOCAL.get();
        if (!UPDATER.compareAndSet(this, node, null)) {
            node.isLocked = false;
        }
        //用于GC
        node = null;
    }
}
```

CLHLock是不停的查询前驱变量， 导致不适合在**NUMA** 架构下使用（在这种结构下，每个线程分布在不同的物理内存区域）

非统一内存访问架构 `Non-uniform memory access` ，简称 `NUMA` 是一种为多处理器的计算机设计的内存架构，内存访问时间取决于内存相对于处理器的位置。在 `NUMA` 下，处理器访问它自己的本地内存的速度比非本地内存（内存位于另一个处理器，或者是处理器之间共享的内存）快一些。

非统一内存访问架构的特点是：被共享的内存物理上是分布式的，所有这些内存的集合就是全局地址空间。所以处理器访问这些内存的时间是不一样的，显然访问本地内存的速度要比访问全局共享内存或远程访问外地内存要快些。另外， `NUMA` 中内存可能是分层的：本地内存，群内共享内存，全局共享内存。

##### MCSLock

`MCSLock` 则是对本地变量的节点进行循环。不存在 `CLHlock` 的问题。

```java
import java.util.concurrent.atomic.AtomicReferenceFieldUpdater;

public class MCSLock {
    public static class MCSNode {
        volatile MCSNode next;
        volatile boolean isLocked = true;
    }

    private static final ThreadLocal<MCSNode> NODE = new ThreadLocal<MCSNode>();
    @SuppressWarnings("unused")
    private volatile MCSNode queue;
    private static final AtomicReferenceFieldUpdater<MCSLock, MCSNode> UPDATER = 						AtomicReferenceFieldUpdater.newUpdater(MCSLock.class, MCSNode.class, "queue");

    //获取MCSLock锁的过程：
    //1、新建MCSNode节点并设置到当前线程的ThreadLocal中
    //2、原子地更新queue字段并获得前置节点
    //3、设置前置节点的next为当前节点
    //4、自旋判断当前节点是否释放锁
    public void lock() {
        MCSNode currentNode = new MCSNode();
        NODE.set(currentNode);
        MCSNode preNode = UPDATER.getAndSet(this, currentNode);
        if (preNode != null) {
            preNode.next = currentNode;
            while (currentNode.isLocked) {
            }
        }
    }
    
    public void unlock() {
        MCSNode currentNode = NODE.get();
        if (currentNode.next == null) {
            if (UPDATER.compareAndSet(this, currentNode, null)) {

            } else {
                while (currentNode.next == null) {
                }
            }
        } else {
            currentNode.next.isLocked = false;
            currentNode.next = null;
        }
    }
}
```

`CLHLock` 和 `MSCLock` 

- `CLH` 的队列是隐式的队列，没有真实的后继结点属性。
- `MCS` 的队列是显式的队列，有真实的后继结点属性。
- 线程状态借助节点保存，每个线程都有一份独有的节点，这样就解决了 `TicketLock` 多处理器下的问题。

### 锁的种类

#### 乐观锁与悲观锁

悲观锁认为在自己使用资源时，一定会有其他线程来修改资源。因此悲观锁会在使用资源时独占资源，避免资源被其他的线程修改。在 `java` 中，`synchronized` 关键字和相关的 `Lock` 类都是悲观锁。

乐观锁认为在其使用资源时，不会有其他的线程来修改数据，所以在其使用资源时并不会加锁。只有在更新资源时会检查资源是否被其他的线程所修改，若资源已经被更新乐观锁会根据实现方式报错或者重试。在 `java` 中，`CAS` 就是典型的乐观锁。

#### 自旋锁与适应性自旋锁

自旋锁详见 [自旋锁概述](# 自旋锁) 

适应性自旋锁与普通自旋锁的区别则是自旋的次数不再固定，会由上一次或者同一个锁的自旋时间及锁拥有者的状态类决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

#### 无锁 偏向锁 轻量级锁 重量级锁

==这四种锁涉及到JVM虚拟机内容等待补齐== 

#### 公平锁与非公平锁

详情查看 `ReentrantLock` 部分的介绍：[公平锁与非公平锁](# 公平锁与非公平锁) 

#### 可重入锁

可重入锁有名递归锁，是指在同一线程下已经获取锁时，还可能继续获取锁，不会因为之前已经获取锁而阻塞。在 `java` 中 `ReentrantLock` 、`ReentrantReadWriteLock` 和 `synchronized` 都是可重入锁，其优点是可一定程度避免死锁。



## 阻塞队列

上方生产者与消费者的实例代码中，将一个普通的 `LinkedList` 包装为一个类，这个类中分别存在存入和取出的方法，在 `JDK` 中有着相同功能的实现，被称为阻塞队列 `blocking queue` 可以使用这些定义好的阻塞队列解决在多线程下集合及链表存取带来的安全性问题。

在 `JDK` 中的 `JUC (java.util.concurrent)` 包下有着将各种阻塞队列的实现，包括 `ArrayBlockingQueue` 、`LinkedBlockingQueue` 、`PriorityBlockQueue` 。

### ArrayBlockingQueue

#### 概述

`ArrayBlockingQueue` 翻译过来就是数组阻塞队列，它与 `ArrayList` 功能相同，

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

----

==需要先学习容器相关知识，等待后续补全==

----

## 线程池

### 概述

线程池从字面意思来理解就是一个装有线程的池子，线程池的设计初衷就是为了更加高效的使用多线程。线程池将线程的创建、使用和调度等细节进行屏蔽，开发者可以通过暴露出的接口直接对其操作。

线程池有以下优点：

- 降低资源消耗，通过重复利用已经创建好的线程资源，降低线程创建和销毁的消耗。
- 提高响应速度，当任务到达时，任务可以不需要等待线程创建，可以使用线程池中创建好的线程直接用。
- 提高可管理性，线程不可能无限制的创建，线程池可以对线程进行统一的分配管理、调优和监控。

### 使用

直接使用 `ThreadPoolExecutor` 类实现线程池。

```java
ThreadPoolExecutor tpe = new ThreadPoolExecutor(10, 100, 30, TimeUnit.SECONDS, new SynchronousQueue<>());
tpe.execute(()->{
    System.out.println("thread-1 is running");
});
```

使用 `Executors` 类的 `newSingleThreadExecutor` 方法创建线程池。

```java
ExecutorService es = Executors.newSingleThreadExecutor();
Future<?> future = es.submit(() -> {
    System.out.println("thread-2 is running");
});
Future<String> future1 = es.submit(() -> {
    System.out.println("thread-3 is running");
    return "result";
});
Future<String> future2 = es.submit(() -> {
    System.out.println("thread-3 is running");
}, "result");
System.out.println(future.get()); // null
System.out.println(future1.get()); // result
System.out.println(future2.get()); // result
```



### 分析

#### Executor

线程池的主要结构是 `Executor` 框架，`ThreadPoolExecutor` 等类此基础上进行实现，下面是主要类的继承关系。

![image-20220224203824851](photo/72、线程池继承结构(7).png) 

接下来查看一下 `Executor` 类和 `ExecutorService` 类的源码。

```java
public interface Executor {
    void execute(Runnable command);
}
```

`Executor` 接口类只定义了一个 `execute` 方法，下面是继承了 `Executor` 接口类的 `ExecutorService` 类。

```java
public interface ExecutorService extends Executor {
    
	// 启动一次有序的关闭，之前提交的任务执行，但不接受新任务 这个方法不会等待之前提交的任务执行完毕
    void shutdown();
	// 试图停止所有正在执行的任务，暂停处理正在等待的任务，返回一个等待执行的任务列表
    // 这个方法不会等待正在执行的任务终止
    List<Runnable> shutdownNow();
	// 如果已经被shutdown，返回true
    boolean isShutdown();
	// 如果所有任务都已经被终止，返回true
    boolean isTerminated();
	// 在一个shutdown请求后，阻塞的等待所有任务执行完毕,或者到达超时时间，或者当前线程被中断
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
	// 提交任务返回Future代表这个任务，有返回值
    <T> Future<T> submit(Callable<T> task);
	// 提交任务返回Future代表这个任务，返回第二参数
    <T> Future<T> submit(Runnable task, T result);
	// 提交任务返回Future
    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)

    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

`ExecutorService` 接口类更进一步，定义了许多对线程池的操作，至此主要的接口类部分就分析完成了，接下来开始分析真正的实现部分。

#### AbstractExecutorService

这个抽象类继承了 `ExecutorService` 并实现了接口中的很多方法，首先先从 `submit` 方法开始。

```java
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

`submit` 方法又三个重载方法，内容都大同小异，第一步首先判断传入的任务是否为空。第二步直接创建一个 `RunnableFuture` 对象，最后使用 `execute` 方法进行下一步操作，并将 `RunnableFuture` 对象返回，用于获取返回值。

```java
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
    return new FutureTask<T>(runnable, value);
}
protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}
```

`newTaskFor` 方法同样也是有两个重载方法，分别直接接受 `callable` 和接受 `runnable` 和一个自定义的返回值，在上面对 `FutureTask` 类的介绍中已经介绍了其对这两个接口类的包装，这里就不再赘述。



```java
public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
    throws InterruptedException, ExecutionException {
    try {
        return doInvokeAny(tasks, false, 0);
    } catch (TimeoutException cannotHappen) {
        assert false;
        return null;
    }
}
```



```java
private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks,
                          boolean timed, long nanos)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (tasks == null)
        throw new NullPointerException();
    int ntasks = tasks.size();
    if (ntasks == 0)
        throw new IllegalArgumentException();
    ArrayList<Future<T>> futures = new ArrayList<Future<T>>(ntasks);
    ExecutorCompletionService<T> ecs =
        new ExecutorCompletionService<T>(this);

    try {
        ExecutionException ee = null;
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        Iterator<? extends Callable<T>> it = tasks.iterator();

        futures.add(ecs.submit(it.next()));
        --ntasks;
        int active = 1;

        for (;;) {
            Future<T> f = ecs.poll();
            if (f == null) {
                if (ntasks > 0) {
                    --ntasks;
                    futures.add(ecs.submit(it.next()));
                    ++active;
                }
                else if (active == 0)
                    break;
                else if (timed) {
                    f = ecs.poll(nanos, TimeUnit.NANOSECONDS);
                    if (f == null)
                        throw new TimeoutException();
                    nanos = deadline - System.nanoTime();
                }
                else
                    f = ecs.take();
            }
            if (f != null) {
                --active;
                try {
                    return f.get();
                } catch (ExecutionException eex) {
                    ee = eex;
                } catch (RuntimeException rex) {
                    ee = new ExecutionException(rex);
                }
            }
        }

        if (ee == null)
            ee = new ExecutionException();
        throw ee;

    } finally {
        for (int i = 0, size = futures.size(); i < size; i++)
            futures.get(i).cancel(true);
    }
}
```

