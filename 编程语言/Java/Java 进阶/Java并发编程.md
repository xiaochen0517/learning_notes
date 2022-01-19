# Java 并发编程

## 概述

### 并发与并行

#### 并发是什么

并发是指当有多个线程在操作时,如果系统同时之能执行一个线程，它只能把CPU运行时间划分成若干个时间段,再将时间 段分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状。这种方式称之为并发(Concurrent)。

#### 并行是什么

并行是当系统可同时执行的线程大于等于需要执行的线程数时，这时所有的线程都可以在同一时间同时执行，而这时程序的执行就被称为并行(parallel)。

### 进程和线程

#### 进程是什么

进程可以认为是一个执行单元，每个进程都有独立的内存空间和资源，进程和进程之间相互独立。一个进程无法去访问另一个进程的数据。进程间的资源如内存和CPU时间等资源都是由操作系统来分配。

#### 线程是什么

进程可以拥有多个线程，进程下的线程可以访问进程的资源及线程之间的数据。每个线程都有独自的缓存，如果线程读取了某个数据，那么它将会将这些数据存放到自己的缓存中以供使用。因此多线程程序最重要的一个目标就是要保证数据在多线程下的安全性。

> 一个 JAVA 程序默认以一个进程的形式运行，在一个进程中使用不同的多个线程一起完成并行运行算或实现异步操作。

#### 进程与线程的关系

一个进程包含一个或以上的线程，若只包含一个线程则这个线程被称为主线程。CPU的调度单位（也就是CPU真正执行的单位是线程），当CPU在执行多个线程时会通过时间片轮转来调度每个线程（如下图），当线程的数量小于CPU核数时才会“并行”执行。

![image-20220119162307176](photo\32、多线程运行逻辑(7).png)

> 线程解决了多任务同时执行的问题（例如边打游戏边听歌），当然线程也是一种受限的系统资源，系统不可能无限制的创建线程。当线程在创建和销毁时都会有相应的开销，为了解决线程频繁的创建和销毁带来的性能问题通常会采用线程池技术，即在线程池总会缓存一定数量的线程，通过这种方法来避免线程的频繁创建和销毁带来的性能问题。

#### Java 中的线程

我们在编写Java程序时首先要编写一个main方法，而这个方法会在当前运行的Java进程的主线程中执行，可以使用以下的代码获取到当前执行的线程名称。

```java
public static void main(String[] args) {
    System.out.println(Thread.currentThread().getName()); // 输出：main
}
```

当程序需要进行一些需要耗费大量时间进行的操作时（例如网络请求、I/O操作等），当这些操作依旧直接在主线程中执行时就会使主线程停止运行，直到操作完成程序才会继续运行，于是就需要多线程来执行这些操作防止主线程阻塞，将耗时操作放到子线程中执行。

### JUC 基础类介绍

#### Thread



#### Runnable 和 Callable



## 基础使用

### 直接使用 Thread 类

```java
Thread thread = new Thread() {
    @Override
    public void run() {
        System.out.println("new thread with rewrite run method");
    }
};
thread.start();
```

### 使用 Runnable

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

### 使用线程池

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

## FutureTask

### FutureTask 是什么



## 线程间通信与阻塞队列



## 线程池



## 锁

