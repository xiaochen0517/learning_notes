## 概述 Overview



## 实践使用

### JS Lambda

JS 中的基础单位通常是函数，虽然在 ES5 与 ES6 中引入了类的概念，但本质上 JS 依旧还是使用函数作为“单位”。JS 与 Java 一个很大的区别就是，JS 可以在函数之间传递函数也就是将函数作为参数进行传递，并可以在函数中的任意位置执行由参数传入的函数。

通常，接收传入函数参数的函数被称为执行器（ Actuator ），而被当做参数传入的函数则被称为目标方法（ Target ）。下面是 JS 中使用 Lambda 表达式例子。

```js
function Hand(func){
    func();
}

function Hammer(){
	console.log("use hammer");
}

Hand(Hammer);

Hand(function BigHammer(){
	console.log("this is big hammer");
});

Hand(()=>{
	console.log("this is my code");
});
```

> 在很多编程语言中，如 `js` 中函数（方法）是可以作为参数进行传递的，因此会出现在方法中执行参数中传入的方法。传入的方法可以使用一早就已经定义好的方法，也可以在执行方法时在传入参数的位置临时定义一个方法，这种方法可能会使代码阅读性下降和代码过于冗长。因为在方法传参时直接定义方法，这个方法又很多部分都没有意义，即可以省略不写，`lambda` 表达式就是为了解决这一问题出现的，它的本质就是一个语法糖。

### Java Lambda

在 `js` 中我们可以直接传递函数，但是在 `java` 中参数的传递全部都是值传递，无法传递函数。因此我们需要一种退而求其次的方法来实现类似于方法的传递，首先在 `java` 中我们是可以传递对象引用的，那我们传递一个对象到另一个对象的方法中，再让接收对象的方法去执行我们传递的对象中的方法，这时就可以实现类似函数传递相似的功能。

再次进一步进行思考，在 `js` 中需要传递的函数是需要在传递时直接去定义才可以使用 `lambda` 表达式，但是在 `java` 中需要传递的对象必须是已经定义好的对象，需要将指定的对象 `new` 出来然后再进行传递。如何解决这个问题，这时就需要用到接口，在代码中使用匿名类 `new` 接口的方法是这样的：

```java
Hammer hammer = new Hammer(){
    @Override
    public void hammerNail() {}
};
```

可以在 `new` 的过程中编写方法中需要执行的代码，此时我们直接将这个 `new` 好的对象进行传递就完美解决了这个问题。

下面是整体代码：

执行类

```java
public class Person {

    private Hammer myRunnable;

    public Person(Hammer myRunnable){
        this.myRunnable = myRunnable;
    }

    public Person hand(){
        if (myRunnable != null){
            myRunnable.hammerNail();
        }
        return this;
    }
}
```

接口类

```java
public interface Hammer {

    void hammerNail();

}
```

测试类

```java
public static void main(String[] args) {
    Person hammerNail = new Person(() -> {
        System.out.println("锤钉子");
    }).hand();
}
```

> 与 `js` 中不同的是，在 `java` 中 `lambda` 表达式使用的是 `->` 而 `js` 使用的是 `=>` ，其余的用法与 `js` 完全相同。

**参数缩写** 

`lambda` 表达式在参数和执行代码不同的情况下也可以进行相应的缩写

无参数

```java
new Person(() -> {});
```

一个参数

```java
new Person((param) -> {});
```

```java
new Person(param -> {});
```

多个参数

```java
new Person((param1, param2, ...) -> {});
```

当执行的代码为一行

```java
new Person(() -> System.out.println("锤钉子")); // 无返回值
new Person(() -> 1>2); // 有返回值，不用写 return 直接返回
```

