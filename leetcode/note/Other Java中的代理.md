---
title: Java中的代理
date: 2023-11-20 10:38:31
tags: [Java, 八股文]
---

# Java 中的代理

Apple 类实现了 Fruit 接口，我们对 Apple 类进行代理。无论是静态代理还是动态代理，代理的本质都是我们不需要修改被代理类源码、不需要重新编译被代理类，而只需要修改代理类源码即可完成被代理类功能的修改与增强。

```java
interface Fruit {
    public void eat();
}

class Apple implements Fruit {

    @Override
    public void eat() {
        System.out.println("eat apple");
    }
}
```

## 静态代理

静态代理需要自己编写代理类代码，经过编译生成代理类字节码。缺点是每当我们想要去代理某个类的时候都要手动创建一个代理类，还有就是当被代理的接口和类修改的时候，代理类的代码也需要手动修改。

```java
public class Solution {
    public static void main(String[] args) {
        Fruit fruit = new Proxy(new Apple());
        fruit.eat();
    }
}

class Proxy implements Fruit {
    private Apple apple;

    public Proxy(Apple apple) {
        this.apple = apple;
    }

    @Override
    public void eat() {
        before();
        apple.eat();
        after();
    }

    private void before() {
        System.out.println("eat before");
    }

    private void after() {
        System.out.println("eat after");
    }
}
```

## jdk 动态代理

代理类是基于反射信息自动生成的，并且 jdk 动态代理的工作过程也是依靠反射完成的。具体来讲是自动生成一个实现代理接口的匿名类，这个类依靠反射实现了接口方法，完成了方法的增强。jdk 动态代理生成类的速度很快，但是运行时因为是基于反射，调用后续的类操作会很慢，并且 jdk 动态代理只能针对接口实现类进行代理增强。

Proxy.newProxyInstance 的前两个参数指定了生成哪个接口的代理类，第三个参数决定了是对哪个接口对象进行代理以及每个接口方法分别进行怎样的增强。

```java
public class Solution {
    public static void main(String[] args) {
        FruitHandler fruitHandler = new FruitHandler(new Apple());
        Fruit fruit = (Fruit) Proxy.newProxyInstance(Fruit.class.getClassLoader(), new Class[]{Fruit.class}, fruitHandler);
        fruit.eat();
    }
}

class FruitHandler implements InvocationHandler {
    private Object target;

    public FruitHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object result = method.invoke(this.target, args);
        after();
        return result;
    }

    private void before() {
        System.out.println("eat before");
    }

    private void after() {
        System.out.println("eat after");
    }
}
```

我们在虚拟机参数中加入`-Djdk.proxy.ProxyGenerator.saveGeneratedFiles=true`，可以保存生成的代理类字节码文件，反编译可以看到如下结果。从中可以清晰的看到利用反射实现代理的过程，也可以看出是每一个接口对应生成一个代理类。

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

import java.lang.invoke.MethodHandles;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

final class $Proxy0 extends Proxy implements Fruit {
    private static final Method m0;
    private static final Method m1;
    private static final Method m2;
    private static final Method m3;

    public $Proxy0(InvocationHandler var1) {
        super(var1);
    }

    public final int hashCode() {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final boolean equals(Object var1) {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void eat() {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("Fruit").getMethod("eat");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }

    private static MethodHandles.Lookup proxyClassLookup(MethodHandles.Lookup var0) throws IllegalAccessException {
        if (var0.lookupClass() == Proxy.class && var0.hasFullPrivilegeAccess()) {
            return MethodHandles.lookup();
        } else {
            throw new IllegalAccessException(var0.toString());
        }
    }
}
```

## cglib 动态代理

CGLib 是针对类实现代理，对指定的类生成一个子类，并覆盖其中的方法。这种通过继承类的实现方式，不能代理 final 修饰的类。CGLib 代理使用字节码处理框架 ASM，通过修改被代理类的字节码直接生成代理子类的字节码，其代理执行过程不依靠反射。所以 CGLib 动态代理创建效率较低，执行效率高。CGLib 使用继承机制，具体来说，被代理类和代理类是继承关系，所以代理类对象是可以赋给被代理类引用的，如果被代理类有接口，那么代理类对象也可以赋给接口引用。

## 参考资料

https://juejin.cn/post/6844904098580398088
