# NIO 概述

## 1、介绍

### 1.1、 `NIO` 与 `IO` 

| IO                       | NIO                           |
| ------------------------ | ----------------------------- |
| 面向流 `Stream Oriented` | 面向缓冲区 `Buffer Oriented`  |
| 阻塞 `IO` `Blocking IO`  | 非阻塞 `IO` `Non Blocking IO` |
| 无                       | 选择器 `Selectors`            |

### 1.2、通道和缓冲区

`NIO` 系统的核心在于通道 `Channel` 和缓冲区 `Buffer` 。通道标识打开到 `IO` 设备，如：文件、套接字等的连接。若需要使用 `NIO` 系统，需要获取用于连接 `IO` 设备的通道以及用于容纳数据的缓冲区，然后操作缓冲区对数据进行处理。

## 2、功能介绍

### 2.1、缓冲区 `Buffer` 

一个用于特定基本数据类型的容器，由 `java.nio` 包定义所有缓冲区都是 `Buffer` 抽象类的子类。

`Buffer` 主要用于与 `NIO` 通道进行交互，数据从通道读入缓冲区，再从缓冲区写入通道中。

`Buffer` 本质上是一个数组，用来储存不同类型的数据。不同的数据类型有不同的缓冲区， `bolean` 类型除外。

```java
ByteBuffer
CharBuffer
ShortBuffer
IntBuffer
LongBuffer
FloatBuffer
DoubleBuffer
```

> 使用 `allocate()` 方法可以获取缓冲区。

#### 2.1.1、 `Buffer` 属性

- `capacity` ：容量，表示缓冲区中最大储存数据的容量。
- `limit` ：界限，表示缓冲区可以操作数据的大小。
- `position` 位置，表示缓冲区中正在操作数据的位置

> 其中， `position <= limit <= capacity` 

 #### 2.1.2、基础方法

创建缓冲区：

```java
ByteBuffer bb = ByteBuffer.allocate(1024);
```

写入数据：

```java
String str = "abcdefg";
bb.put(str.getBytes());
```

切换到读取模式：

```java
bb.flip(); // 切换模式
byte[] bs = new byte[5];
bb.get(bs);
System.out.println(new String(bs));
```

清空缓冲区：

```java
bb.clear();
```

> 在缓冲区执行某些操作时，其实并不是修改其中的值，而是修改 `Buffer` 的属性指针的值，从而实现读取、写入和清空等操作。

#### 2.1.3、标记和恢复标记

缓冲区可以使用 `mark()` 方法标记当前 `Buffer` 指针的值，并可以使用 `reset()` 方法复原。

示例：

```java
bb.flip();

bb.mark(); // 标记
byte[] bs = new byte[5];
bb.get(bs);
System.out.println(new String(bs));

bb.reset(); // 恢复到标记处
System.out.println(bb.position());
```

#### 2.1.4、直接缓冲区和非直接缓冲区

非直接缓冲区：通过 `allocate()` 方法分配缓冲区，将缓冲区建立在 JVM 的内存中
直接缓冲区：通过 `allocateDirect()` 方法分配直接缓冲区，将缓冲区建立在物理内存中。

1. 若字节缓冲区为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在此缓冲区上执行本机 `I/O` 操作。
2. 直接字节缓冲区可以通过调用此类的 `allocateDirect()` 方法创建。此方法返回的缓冲区进行分配和取消分配所需成本通常高于非直接缓冲区 。直接缓冲区的内容可以驻留在常规的垃圾回收堆之外，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机 `I/O` 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们。
3. 直接字节缓冲区还可以过 通过 `FileChannel` 的 `map()` 方法 将文件区域直接映射到内存中来创建 。该方法返回 `MappedByteBuffer` 。 `Java` 平台的实现有助于通过 `JNI` 从本机代码创建直接字节缓冲区。如果以上这些缓冲区中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常。
4. 字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 `isDirect()` 方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理。

示例：

```java
ByteBuffer bb = ByteBuffer.allocateDirect(1024);
System.out.println(bb.isDirect());
```

### 2.2、通道 `Channel` 

#### 2.2.1、概述

通道 `Channel` ：由 `java.nio.channel` 包定义，通道表示 `IO` 源与目标打开的连接。通道和流类似，但通道不可以直接访问数据，通道只能与另一个通道和缓存进行交互。

**通道的主要实现类：** 

- `FileChannel` 
- `SocketChannel` 
- `ServerSocketChannel` 
- `DatagramChannel` 

**获取通道：** 

- `getChannel()` 方法
  - 本地 `IO` 
    - `FileInputStream or FileOutputStream` 
    - `RandomAccessFile` 
  - 网络 `IO` 
    - `Socket` 
    - `ServerSocket` 
    - `DatagramSocket` 
- 通道的静态方法 `open()` 
- `Files` 工具类 `newByteChannel()` 方法



