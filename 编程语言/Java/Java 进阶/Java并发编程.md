# Java 并发编程

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

