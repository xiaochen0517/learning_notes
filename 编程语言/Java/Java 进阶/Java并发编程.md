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

#### Java 中的线程

我们在编写 `Java` 程序时首先要编写一个 `main` 方法，而这个方法会在当前运行的 `Java` 进程的主线程中执行，可以使用以下的代码获取到当前执行的线程名称。

```java
public static void main(String[] args) {
    System.out.println(Thread.currentThread().getName()); // 输出：main
}
```

当程序需要进行一些需要耗费大量时间进行的操作时（例如网络请求、 `I/O` 操作等），当这些操作依旧直接在主线程中执行时就会使主线程停止运行，直到操作完成程序才会继续运行，于是就需要多线程来执行这些操作防止主线程阻塞，将耗时操作放到子线程中执行。

### 多线程基础类介绍

在Java中最重要的三个和线程相关的类是 `Thread` `Runnable` `Callable` ，其中 `Thread` 是真正创建和运行线程的类而其余两个只是接口，用于向 `Thread` 类中传入对象引用用于执行方法的。以下源代码中可以看到 `Runnable` 和 `Callable` 类并没有实质性的方法，真正创建线程和执行的是 `Thread` 类。Thread 类是线程类而 `Runnable` 和 `Callable` 是任务类，任务类只是将需要执行的任务进行封装后传入线程类来真正执行。

![image-20220120091319719](photo\33、Runnable源码(7).png) 

![image-20220120091409143](photo\34、Callable源码(7).png) 

![image-20220120091519767](photo\35、Thread源码(7).png) 

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

`Thread` 状态

在 `java` 中 `Thread` 类有以下几种状态。

- `NEW` ：初始状态
- `RUNNABLE` ：运行状态
- `BLOCKED` ：阻塞状态
- `WAITING` ：等待状态
- `TIMED_WAITING` ：有超时的等待状态
- `TERMINATED` ：结束状态



#### Callable 类

`Callable` 类和 `Runnable` 类一样是一个接口类，不同之处在于 `Callable` 的 `call` 方法拥有返回值，而 `Runnable` 的 `run` 方法没有返回值。

## Java 中的多线程

### 使用 Thread 类创建线程

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

### 使用 Runnable 传入 Thread 

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

#### 线程状态

- `NEW` ：初始状态

```java
Thread thread1 = new Thread(() -> {
    // nothing...
});
LOGGER.info("线程状态：{}", thread1.getState()); // NEW
```

当 `Thread` 对象不为空时，线程当前的状态为初始化状态。

- `RUNNABLE` ：可运行状态

```java
Thread thread1 = new Thread(() -> {
    // nothing...
});
thread1.start();
LOGGER.info("线程状态：{}", thread1.getState()); // RUNNABLE
```

当执行 `start()` 方法时线程为可运行状态。

- `BLOCKED` ：阻塞状态

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

- `WAITING` ：等待状态

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

- `TIMED_WAITING` ：有超时的等待状态

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

- `TERMINATED` ：结束状态

```java
Thread thread5 = new Thread(() -> {});
thread5.start();
Thread.sleep(500L);
LOGGER.info("线程状态：{}", thread5.getState()); // TERMINATED
```

正常执行完成的线程进入结束状态。

#### 状态转换图

![image-20220121154348812](photo\42、Thread生命周期转换图(7).png)

### 使用线程池和 Callable

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

![image-20220120171941083](photo\39、FutureTask状态(7).png) 

- `NEW` ：新建
- `COMPLETING` ：即将完成
- `NORMAL` ：完成
- `EXECEPTIONAL` ：抛异常
- `CANCELLED` ：任务取消
- `INTERRRUPTING` ：任务即将被打断
- `INTERRUPTED` ：任务被打断

`get` 方法源码

![image-20220120172607784](photo/40、FutureTask#get()源码(7).png) 

`get` 方法会阻塞线程等待返回，阻塞部分的核心代码就是 `awaitDone` 方法。

![image-20220121091414977](photo/41、FutureTask#awaitDone()源码(7).png) 

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



## 阻塞队列



## 线程池



## 锁

