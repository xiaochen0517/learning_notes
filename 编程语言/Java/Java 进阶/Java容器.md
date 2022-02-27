前置知识：

- 并发相关**概念** 
- Java 基础
- Java 8 Lambda

## 容器基础

### 概述

Java 中包含了很多的基础类型例如 `int` 、`long` 、`double`、`float` 等多种基础数据类型，还有很多引用数据类型，如：`String`、`Integer`、`Long` 等等。这些都是单独的数据，如果需要使用多个单独数据进行打包时，就会用到数组。但是数组在使用过程中也会面临诸多问题，如大小不可变、根据值查找元素较慢、增加删除效率差并且无封装方法，操作都需要用户实现。所以就需要一个新的容器，一个可以有着多种实现，并可以用来解决多种问题的容器。于是 Java 就诞生了以 `Collection` 集合与 `Map` 键值映射表为主的两类数据容器。

#### 基础特性

Java 中常用的储存方式是数组和容器，而其又有一下特点。

- 数组
  - 长度固定
  - 可储存基本类型，也可储存引用类型（对象）
- 容器
  - 长度可变
  - 只能储存引用数据类型，基础类型需要转换为对应包装类（ int->Integer ）

#### 分类

在 Java 中容器有两大类，分别是 `Collection` 和 `Map` ，而 `Collection` 又分为 `Set` 、`List` 和 `Queue` 。

![image-20220225093927476](photo/72、容器结构01.png)  

### 接口分析

#### 函数式接口

在容器中，很多地方都使用了函数式接口（ Functional Interface ），所以在开始之前，需要了解一下最基础的函数式接口类。

首先是 `Consumer` 接口类，`Consumer` 接口是从 Java8 开始引入的 `java.util.function` 包的一部分，用于用 Java 实现函数式编程。一般来说函数式接口的类上会使用 `@FunctionalInterface` 注解在标记，用于检测其中的方法定义是否合规。他的作用就如同名字一样其用于消费数据，最主要的是 `accept` 方法并没有返回值。

```java
@FunctionalInterface
public interface Consumer<T> {
    
    void accept(T t);
    
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

其中还有另一个方法 `andThen` ，当一个方法的参数和返回值全都是 `Consumer` 类型时，就可以实现依次执行。

**使用示例** 

```java
@Test
public void test1(){
    testConsumer(
        e -> {
            System.out.println(e.toUpperCase());
        },
        e -> {
            System.out.println(e.toLowerCase());
        },
        "Hello");
}

public <T> void testConsumer(Consumer<T> c, Consumer<T> c1, T t){
    c.andThen(c1).accept(t);
}
```

定义一个 `testConsumer` 方法，在其中使用 `andThen` 方法将两个 `Consumer` 对象串联，最后查看执行结果。

```java
HELLO
hello
```

可以看到程序先执行了第一个 `c` 中的 `accept` 方法后执行了第二个 `c1` 中的 `accept` 方法。

#### 迭代器

`Iterable` 定义了迭代器模式，用于遍历容器中的元素。

```java
public interface Iterable<T> {
    // 获取迭代器
    Iterator<T> iterator();
    // 顺序迭代元素，并对元素执行指定的操作
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
}
```

**迭代器模式** - **提供一种方法顺序访问一个聚合对象中各个元素，而又无须暴露该对象的内部表示**。

#### 集合接口

本质上 `Collection` 是对 `Iterable` 的补充和拓展。

```java
public interface Collection<E> extends Iterable<E> {
    // 元素数量
    int size();
    // 是否为空
    boolean isEmpty();
    // 是否包含指定的元素
    boolean contains(Object o);
    // 获取迭代器
    Iterator<E> iterator();
    // 返回包含此集合中所有元素的数组
    Object[] toArray();
    // 返回所有元素的数组，可以用泛型指定类型
    <T> T[] toArray(T[] a);
    // 添加元素
    boolean add(E e);
    // 移除元素
    boolean remove(Object o);
    // 是否包含指定集合的所有元素
    boolean containsAll(Collection<?> c);
    // 将集合多个元素添加到此集合
    boolean addAll(Collection<? extends E> c);
    // 移除多个元素
    boolean removeAll(Collection<?> c);
    // 移除此集合中没有指定集合中的元素
    boolean retainAll(Collection<?> c);
    // 从此集合中删除所有元素
    void clear();
    // 比较对象是否与此集合相同
    boolean equals(Object o);
    // 返回此集合的hash值
    int hashCode();
}
```

`Collection` 接口类还有一个抽象的实现类 `AbstractCollection` 。

```java
public abstract class AbstractCollection<E> implements Collection<E> {
    public boolean contains(Object o) {
        Iterator<E> it = iterator(); // 迭代器
        if (o==null) { // 查找的元素为空
            // 查找集合中为null的元素
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else { // 查找元素不为空
            // 查找集合中匹配的元素
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        // 没有找到，返回false
        return false;
    }
    
    public Object[] toArray() {
        // 创建一个大小为当前集合元素数量的数组
        Object[] r = new Object[size()];
        // 迭代器
        Iterator<E> it = iterator();
       	// 将集合中的元素通过迭代器依次送入数组中
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext())
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        // 最后在判断一下元素是否都遍历完成
        return it.hasNext() ? finishToArray(r, it) : r;
    }
    
    public <T> T[] toArray(T[] a) {
        // 元素数量
        int size = size();
        // 如果传入的数组大小大于等于当前元素数量，直接使用传入的数据进行赋值
        // 如果小于当前元素数量，需要创建一个新的数组，此处用了反射等效于 T[] x = {length};
        T[] r = a.length >= size ? a : (T[])java.lang.reflect.Array.newInstance(a.getClass().getComponentType(), size);
        // 迭代器
        Iterator<E> it = iterator();
        // 依次放入元素
        for (int i = 0; i < r.length; i++) {
            // 没有下一个元素
            if (! it.hasNext()) {
                // a等于r表示当前下标集合中已经没有值，只能填空
                if (a == r) {
                    r[i] = null;
                // a大小小于i，说明r是新建的数组，r比元素数量大
                } else if (a.length < i) {
                    // 新建的数组大小大于元素的大小，截取定长的数组并返回
                    return Arrays.copyOf(r, i);
                } else {
                    // a不等于r且a大小不小于i，
                    // 说明r是新建的数组，a也等于或大于元素数
                    // 将r的值复制一份给a
                    System.arraycopy(r, 0, a, 0, i);
                    // 如果a大小比元素数大，则当前数组元素置空
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                // 返回a，当前a肯定是有值的
                return a;
            }
            // 将集合中的元素赋值到数组中
            r[i] = (T)it.next();
        }
        // 当前集合还有元素，继续遍历，没有则返回r
        return it.hasNext() ? finishToArray(r, it) : r;
    }
    
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        int i = r.length;
        while (it.hasNext()) {
            int cap = r.length;
            if (i == cap) {
                int newCap = cap + (cap >> 1) + 1;
                // overflow-conscious code
                if (newCap - MAX_ARRAY_SIZE > 0)
                    newCap = hugeCapacity(cap + 1);
                r = Arrays.copyOf(r, newCap);
            }
            r[i++] = (T)it.next();
        }
        // trim if overallocated
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }
    
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError
                ("Required array size too large");
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
    
    public boolean containsAll(Collection<?> c) {
        // 使用 contains 方法循环
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }
    
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
    
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }
    
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }
    
    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }
    
    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }
}
```



#### 排序接口

##### Comparable

若某个类实现了 `Comparable` 接口则表示此类是可以被进行比较的，`Comparator` 接口类则可以根据此方法的结果对元素进行排序。

```java
public interface Comparable<T> {
    // 将此对象与指定的对象进行比较
    public int compareTo(T o);
}
```

接下来看一下 `String` 类，已知其是可以进行比较的，查看一下它是如何实现的。

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
```

可以看到 `String` 类实现了 `Comparable` 接口，下面是其 `compareTo` 方法的实现。

```java
public int compareTo(String anotherString) {
    // 字符串1的长度
    int len1 = value.length;
    // 字符串2的长度
    int len2 = anotherString.value.length;
    // 得到其中最小的长度
    int lim = Math.min(len1, len2);
    // 字符串1的char数组
    char v1[] = value;
    // 字符串2的char数组
    char v2[] = anotherString.value;
    // 计数值
    int k = 0;
    // 当计数小于最小值时执行循环
    while (k < lim) {
        // 字符串1指定下标char
        char c1 = v1[k];
        // 字符串2指定下标char
        char c2 = v2[k];
        // 如果两个char不相等，直接返回
        if (c1 != c2) {
            // ASCII相减
            return c1 - c2;
        }
        k++;
    }
    // 最后判断一下两个字符串长度，为0则完全相同，不为0则大小不一
    return len1 - len2;
}
```

使用最小的长度对字符串进行遍历，这里使用最小值是为了防止下标越界，逐个比对字符，如果有不同则直接返回两个字符的 `ASCII` 码的差值。如果遍历完成后全部相等，则需要去判断两个字符串大小是否相等，不相等则表示两个字符串不一样，返回长度的差值。

由于 `String` 无法像 `int` 值一样去判断大小，所以大小是根据 `ASCII` 编码大小和字符串长度判断的，总结下来就是为 0 则相等，不为 0 则不等。

接下来，我们使用一段代码验证我们的分析结论是否正确。

```java
String a = "我的世界真好玩";
String b = "我的世界";
System.out.println(a.compareTo(b));
String c = "我的键盘";
String d = "我的笔记本";
System.out.println(c.compareTo(d));
char e = '键';
char f = '笔';
System.out.println(e-f);
```

结果

```java
3
6682
6682
```

可以看到，第一个返回的是两个字符串长度差值，而第二个返回的是 `键` 和 `笔` 两个字的 `ASCII` 码的差值。

##### Comparator

`Comparator` 是比较接口，如果某个没有继承 `Comparable` 类需要排序时，但是其本身并不支持比较，此时就需要实现一个比较器来进行排序，这个比较器只需要实现 `Comparator` 接口即可。

```java
public interface Comparator<T> {
    // 比较大小1大于2返回正数，1小于2返回负数，相等返回0
    int compare(T o1, T o2);
    // 是否相等
    boolean equals(Object obj);
}
```

在 Java 8 之前，`Comparator` 接口中只有两个方法，分别是区分大小的 `compare` 方法和区分是否相等 `equals` 方法。剩下的所有方法都是使用 `default` 修饰在 Java 8 中新增的方法，用于实现流式编程。

#### 克隆接口

如果在 `Java` 中一个类需要实现克隆功能即复制，必须实现 `Cloneable` 接口，否则其在调用 `clone` 方法时会报错。

```java
public interface Cloneable {}
```

在 `Cloneable` 接口中并没有内容，具体的 `clone` 方法在 `Object` 类中。

```java
protected native Object clone() throws CloneNotSupportedException;
```

`clone` 方法是一个本地方法，在类未实现 `Cloneable` 接口时调用 `clone` 方法会抛出 `CloneNotSupportedException` 异常。

#### fail-fast

`fail-fast` 是一种 `Java` 容器的错误检测机制，当多个线程改变容器的结构时，就可能会触发 `fail-fast` 机制。

定义两个线程，一个从线程中遍历元素，另一个在改变容器结构。也就是在删除容器中的某些内容，令容器中元素组成变化。

```java
// static global variable
private static List<Integer> list = new ArrayList<>();
// main method code
for (int i = 0; i < 100; i++) {
    list.add(i);
}
new Thread(()->{
    Iterator<Integer> iterator = list.iterator();
    while (iterator.hasNext()){
        Integer next = iterator.next();
        LOGGER.info("读取元素 -- {}", next);
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            LOGGER.error("线程中断", e.fillInStackTrace());
        }
    }
}).start();
new Thread(()->{
    for (int i = 1; i < list.size(); i++) {
        if (i%2 == 0){
            LOGGER.info("移除元素 -- {}", i);
            list.remove(i);
        }
    }
}).start();
```

执行结果

```java
// more info ....
2022-02-25 15:21:45.683 [Thread-1] INFO  java.lang.Thread - 移除元素 -- 64
2022-02-25 15:21:45.683 [Thread-1] INFO  java.lang.Thread - 移除元素 -- 66
Exception in thread "Thread-0" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at com.mochen.advance.container.FailFastDemo.lambda$main$0(FailFastDemo.java:21)
```

可以看到此操作触发了 `fail-fast` 机制，抛出了 `ConcurrentModificationException` 异常。解决方法也很简单，在修改和读取中的代码中加入 `synchronized` 关键字或者使用锁同步，也可以使用 `juc` 包下的并发容器。后续会在 `ArrayList` 的分析中详解。

## List

### 概述

`List` 是容器中的一个大类，代表一个有序的队列，其包含了许多不同实现和数据结构的容器，包括 `ArrayList` 、`LinkedList` 等实现。

### List

`List` 是一个接口，它继承于 `Collection` 的接口，其中除了 `Collection` 定义的方法外还实现或定义了一些 `List` 独有的方法。

```java
public interface List<E> extends Collection<E> {
    // ...
```

#### 修改内容

函数式编程，修改元素的内容。

```java
default void replaceAll(UnaryOperator<E> operator) {
    Objects.requireNonNull(operator); // 检查是否为空
    // 获取迭代器
    final ListIterator<E> li = this.listIterator();
    // 遍历当前集合元素
    while (li.hasNext()) {
        // 对当前元素进行指定操作
        li.set(operator.apply(li.next()));
    }
}
```

获取到集合的迭代器，遍历所有元素并进行修改，此处使用了函数式编程，下面是 `UnaryOperator` 接口类的代码。

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```

可以看到，接口中并没有定义 `apply` 方法，此方法是在其父类 `Function` 接口类中定义的。

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    // ...
}
```

> 容器在很多的数据处理方法中都使用了函数式编程，这也是 JDK 8 最为重要的新特性之一。

#### 排序方法

```java
@SuppressWarnings({"unchecked", "rawtypes"})
default void sort(Comparator<? super E> c) {
    // 将所有元素转为数组
    Object[] a = this.toArray();
    // 使用Arrays工具类排序
    Arrays.sort(a, (Comparator) c);
    // 获取迭代器
    ListIterator<E> i = this.listIterator();
    // 遍历数组中的元素，将数组中的元素覆盖到容器中
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```

在这个方法中使用到了比较器接口，核心的排序方法是由 `Arrays` 工具类实现的，会在后面的工具类部分中详细 分析。最后，获取到容器的迭代器，将排序好的数组逐一覆盖到容器中。

#### 常用方法

```java
// 获取指定下标元素并返回
E get(int index);
// 设置指定下标元素为指定对象，返回之前下标位置的元素
E set(int index, E element);
// 将元素插入到指定下标，如果指定下标存在元素则将其和后面的元素统一后移一位
void add(int index, E element);
// 删除自动下标的元素，将后面的元素（如果存在）向前移动一位
E remove(int index);
// 返回指定元素的在容器中第一次匹配的元素下标，如果不存在返回-1
int indexOf(Object o);
// 返回指定元素在容器中最右一个匹配的元素下标，如果不存在返回-1
int lastIndexOf(Object o);
// 返回容器的迭代器
ListIterator<E> listIterator();
// 返回从指定下标开始的迭代器
ListIterator<E> listIterator(int index);
// 返回指定下标范围的容器，[fromIndex, toIndex)
List<E> subList(int fromIndex, int toIndex);
```

下面是获取可拆分迭代器的方法，是用了 `Spliterators` 工具类之后会在工具类部分分析。

```java
@Override
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, Spliterator.ORDERED);
}
```



### AbstractList



```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
    // ...
```



## 流式编程

### 概述

流式编程（ Stream ）是 Java8 的新功能，是对集合等容器对象功能的增强，可以进行各种非常高效的聚合操作（ aggregate operation ）和大批量数据操作（ bulk data operation ）。其与 Lambda 表达式相互结合，极大的提高了容器编程的效率。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join 并行方式来拆分任务和加速处理过程。Stream 流式编程还提供了 Stream API ，使用其无需编写并发代码即可使用高并发特性快速解决问题。



### 使用



### 解析



#### 拆分器

在上面的迭代器接口中还有另一个方法没有介绍，因为其是在 Java 8 的流式编程中新增的内容，所以统一在这里介绍。以下是这个方法的代码。

```java
// 查看流式编程相关内容
default Spliterator<T> spliterator() {
    return Spliterators.spliteratorUnknownSize(iterator(), 0);
}
```

`Spliterator`（ splitable iterator 可分割迭代器）接口是Java为了**并行**遍历数据源中的元素而设计的迭代器，这个接口与Java提供的顺序遍历迭代器 `Iterator` 相比，一个是顺序遍历，一个是并行遍历。

`Iterator` 是 Java 早期产物，但是由于 CPU 的多核化趋势导致顺序遍历已经无法满足性能需求，于是， `Spliterator` 可分割迭代器应运而生。`Spliterator` 的主要原理就是将一个大任务递归拆分成小任务，将各个小任务分配到不同的核中进行处理，最后将结果合并。==Java7的Fork/Join(分支/合并)框架== 

```java
// 顺序处理元素
boolean tryAdvance(Consumer<? super T> action);
// 将当前集合划分一部分创建一个新的拆分迭代器
Spliterator<T> trySplit();
// 剩余多少元素需要遍历
long estimateSize();
// ==待定==
int characteristics();
```

具体的并行操作就是使用 `trySplit` 对数组进行分割，以便于进行并行操作提高效率。

批量遍历元素，本质使用 `tryAdvance` 方法。

```java
default void forEachRemaining(Consumer<? super T> action) {
    do { } while (tryAdvance(action));
}
```

参数类型

```java
public static final int DISTINCT   = 0x00000001;
public static final int SORTED     = 0x00000004;
public static final int ORDERED    = 0x00000010;
public static final int SIZED      = 0x00000040;
public static final int NONNULL    = 0x00000100;
public static final int IMMUTABLE  = 0x00000400;
public static final int CONCURRENT = 0x00001000;
public static final int SUBSIZED   = 0x00004000;
```

结构图

![image-20220226102419124](photo/73、可分割迭代器接口类结构.png) 



#### 集合接口

Java 8 在 `Collection` 接口中添加四个关于流式编程的方法，其都使用 `default` 关键字修饰不需要实现类实现这些方法，也是为了在新增功能时不会大幅度修改继承链中其他类的内容，同时也为了第三方继承相关接口的类不需要修改。

```java
// 删除满足给条件的此集合的所有元素。 
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    // 获取迭代器
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        // 是否匹配
        if (filter.test(each.next())) {
            each.remove();  // 移除元素
            removed = true;
        }
    }
    return removed;
}
// 返回可拆分迭代器
@Override
default Spliterator<E> spliterator() {
    return Spliterators.spliterator(this, 0);
}
// 返回顺序流
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
// 返回并行流
default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true);
}
```



#### 排序接口

`Comparator` 

```java

default Comparator<T> reversed() {
    return Collections.reverseOrder(this);
}

default Comparator<T> thenComparing(Comparator<? super T> other) {
    Objects.requireNonNull(other);
    return (Comparator<T> & Serializable) (c1, c2) -> {
        int res = compare(c1, c2);
        return (res != 0) ? res : other.compare(c1, c2);
    };
}

default <U> Comparator<T> thenComparing(Function<? super T, ? extends U> keyExtractor, Comparator<? super U> keyComparator) {
    return thenComparing(comparing(keyExtractor, keyComparator));
}

default <U extends Comparable<? super U>> Comparator<T> thenComparing(Function<? super T, ? extends U> keyExtractor) {
    return thenComparing(comparing(keyExtractor));
}

default Comparator<T> thenComparingInt(ToIntFunction<? super T> keyExtractor) {
    return thenComparing(comparingInt(keyExtractor));
}

default Comparator<T> thenComparingLong(ToLongFunction<? super T> keyExtractor) {
    return thenComparing(comparingLong(keyExtractor));
}

default Comparator<T> thenComparingDouble(ToDoubleFunction<? super T> keyExtractor) {
    return thenComparing(comparingDouble(keyExtractor));
}

public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
    return Collections.reverseOrder();
}

@SuppressWarnings("unchecked")
public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
    return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
}

public static <T> Comparator<T> nullsFirst(Comparator<? super T> comparator) {
    return new Comparators.NullComparator<>(true, comparator);
}

public static <T> Comparator<T> nullsLast(Comparator<? super T> comparator) {
    return new Comparators.NullComparator<>(false, comparator);
}

public static <T, U> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor, Comparator<? super U> keyComparator) {
    Objects.requireNonNull(keyExtractor);
    Objects.requireNonNull(keyComparator);
    return (Comparator<T> & Serializable)
        (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                          keyExtractor.apply(c2));
}

public static <T, U extends Comparable<? super U>> Comparator<T> comparing(Function<? super T, ? extends U> keyExtractor) {
    Objects.requireNonNull(keyExtractor);
    return (Comparator<T> & Serializable)
        (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
}

public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
    Objects.requireNonNull(keyExtractor);
    return (Comparator<T> & Serializable)
        (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
}
```



## 工具类

### 概述



### 相关工具类

#### Arrays



#### Spliterators

