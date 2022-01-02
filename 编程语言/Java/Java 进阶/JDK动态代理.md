# `JDK` 动态代理

## 什么是代理

`Java` 中代理的实现一般分为三种： `JDK` 静态代理、 `JDK` 动态代理以及 `CGLIB` 动态代理。在 `Spring` 的 `AOP` 实现中，主要应用了 `JDK` 动态代理以及 `CGLIB` 动态代理。

静态代理基础内容

![image-20211231093341292](photo/27、静态代理结构图(4).png) 

代理主要用作在目标类的指定方法前后增加操作，主要为了解决代码重用以及过耦合的问题。

## 静态代理

静态主要表现在静态代理的代理类需要手动编写，不可以在程序运行过程中动态生成。代理类是固定在代码中的，如果需要代理的类和方法较多，代理类也会不可避免的会变得臃肿。且在这种情况下如果整体代码需要统一修改需求，静态代理并无法有效节约时间，代码重用及耦合问题都没有很好的解决。

需要使用代理最常见的需求就是日志打印和权限过滤等功能，现在需要在目标方法前后增加日志输出，以下是静态代理的代码示例。

### 静态代理示例

- 接口类

```java
public interface Human {
    void eat();
}
```

- 目标类

```java
public class MoChen implements Human {
    @Override
    public void eat() {
        System.out.println("eat function run");
    }
}
```

- 代理类

```java
public class HumanProxy implements Human {

    private MoChen moChen;

    public HumanProxy(MoChen moChen){
        this.moChen = moChen;
    }

    @Override
    public void eat() {
        System.out.println("start");
        moChen.eat();
        System.out.println("end");
    }
}
```

- 测试类

```java
public static void main(String[] args) {
    HumanProxy humanProxy = new HumanProxy(new MoChen());
    humanProxy.eat();
}
```

可以看到，目标类方法的执行操作是由代理类执行，静态代理就是在默认的直接执行的基础上在目标类外又增加了一层功能，如果目标类或方法有多个或者需要修改需求，程序会十分臃肿目标类加代理类相当于工作量翻倍，且可维护性很低代码重用率不佳。

## 动态代理

动态代理，顾名思义是在程序运行的过程中去创建代理类实现代理操作，动态创建类和对象首先想到的就是反射，通过反射如何创建代理类呢？

这就要用到 `Proxy` 类的 `getProxyClass` 类，下面直接给出示例代码，最后在分析源码看一下动态代理是如何实现的。

### 动态代理示例

![image-20211231102215131](photo/28、动态代理getProxyClass方法(4).png) 

可以看到 `api` 文档上的解释，此方法会返回一个代理类的 `Class` 对象。

```java
Class<?> proxyClass = Proxy.getProxyClass(Human.class.getClassLoader(), Human.class);
Constructor<?>[] constructors = proxyClass.getConstructors();
for (Constructor<?> constructor : constructors) {
    System.out.println(constructor.getName());
    Class<?>[] parameterTypes = constructor.getParameterTypes();
    for (Class<?> parameterType : parameterTypes) {
        System.out.println(parameterType.getName());
    }
    System.out.println("--------------------------");
}
```

获取到代理类后我们需要将代理类实例化，获取代理类的构造方法并打印查看。

```
com.sun.proxy.$Proxy4
java.lang.reflect.InvocationHandler
--------------------------
```

可以看到，在代理类中只有一个需要 `InvocationHandler` 对象的构造方法。而 `InvocationHandler` 是一个接口类，首先需要实例化接口类再使用代理对象的构造方法实例化代理类。

```java
InvocationHandler handler = new InvocationHandler() {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return null;
    }
};
```

在调用代理类对象的方法时，首先就会执行 `InvocationHandler` 的 `invoke` 方法，并将此方法的返回值作为执行方法的返回值。

参数说明

- `proxy` 代理类
- `method` 当前需要执行的方法
- `args` 当前执行方法的参数

```java
try {
    // 实例化目标类
    MoChen target = new MoChen();
    // 获取到代理类
    Class<?> proxyClass = Proxy.getProxyClass(Human.class.getClassLoader(), Human.class);
    // 获取构造方法
    Constructor<?> constructor = proxyClass.getConstructor(InvocationHandler.class);
    // 实现调用处理器
    InvocationHandler handler = new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // 前置
            System.out.println("start");
            // 执行方法
            Object result = method.invoke(target, args);
            // 后置
            System.out.println("end");
            // 返回方法执行返回值
            return result;
        }
    };
    // 实例化代理类并强转为接口类
    Human human = (Human) constructor.newInstance(handler);
    // 执行方法
    human.eat();
} catch (Exception e) {
    e.printStackTrace();
}
```

这就是动态代理的基本流程，此时查看运行结果。

```
start
eat function run
end
```

### 源码解析

![image-20211231104317518](photo/29、动态代理newProxyInstance方法(4).png) 

可以看到在 `Proxy` 类中还有一个方法，会返回已经实例化的代理类的实例。

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException {
    Objects.requireNonNull(h);
    final Class<?>[] intfs = interfaces.clone();
    
    // 安全检查
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }
    // 获取代理类
    Class<?> cl = getProxyClass0(loader, intfs);
    try {
        // 安全检查
        if (sm != null) {
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }
	   // 获取构造函数
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        // 关闭权限检查
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
        // 使用 InvocationHandler 实例化代理类
        return cons.newInstance(new Object[]{h});
    } catch ...	
}
```

可以看到 `newProxyInstance` 源码中获取实例化代理类的流程与刚才的示例并无区别，获取代理类的 `getProxyClass0` 与 `getProxyClass` 中使用的方法相同。

![image-20211231105005914](photo/30、动态代理getProxy方法源码(4).png) 

- `getProxyClass` 源码

```java
private static Class<?> getProxyClass0(ClassLoader loader, Class<?>... interfaces) {
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }

    // If the proxy class defined by the given loader implementing
    // the given interfaces exists, this will simply return the cached copy;
    // otherwise, it will create the proxy class via the ProxyClassFactory
    return proxyClassCache.get(loader, interfaces);
}
```



根据获取到的代理类获取构造方法 `constructorParams` 参数就是 `InvocationHandler` 。

```java
private static final Class<?>[] constructorParams = { InvocationHandler.class };
```
