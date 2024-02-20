---
title: Java自限定泛型详解
date: 2023-12-11 20:06:41
tags: [Java, 八股文]
---

# Java 自限定泛型详解

## 先介绍下最基本的泛型

类泛型的应用范围是属性、方法中的形参、方法中的返回类型、方法中的局部变量。非应用范围是静态属性、静态方法中的形参、静态方法中的返回类型、静态方法中的局部变量。原因在于泛型只和对象有关，而静态属性和静态方法只和类有关，和对象无关。

```java
// T可以是任意类型，因此对于T类型的对象，我们只能调Object类的属性和方法
class A <T> {}

// T只能是B类型及其子类型，因此对于T类型的对象，我们可以调B类访问修饰符范围内的所有成员
class A <T extends B> {}

// T只能是B类型及其父类型，因此对于T类型的对象，我们只能调Object类的属性和方法
class A <T super B> {}
```

## 普通泛型类定义一个自限定子类

在下面这段代码中，BasicHolder 只是一个普通的泛型类，里面定义了关于泛型 T 的一些操作，接下来定义了一个 A 类，我们发现 A 类继承 BasicHolder 父类的时候给的泛型就是 A 类自身，这也意味着 A 类继承了其父类关于 A 类的所有操作。这种子类称之为自限定子类，自限定子类的意义在于限定了父类关于泛型的操作必须是针对子类同类的，简单来说就是自限定子类只能和自己同类型对象交互。

```java
class BasicHolder<T> {
    private T t;
    public void set(T t) {
        this.t=t;
    }
    public T get() {
        return t;
    }
    public void print() {
        T t = null;
        System.out.println(t);
    }
}

class A extends BasicHolder<A> {
    public static void main(String[] args) {
        A a = new A();
        a.set(new A());
        a.get();
        a.print();
    }
}
```

## 当我们把自限定子类用作泛型定义

在下面的代码中，我们定义 SelfBounded 类的泛型是`<T extends SelfBounded<T>>`，`<T extends SelfBounded<T>>`的意思是 SelfBounded 类的子类且子类在继承 SelfBounded 父类时指定的父类泛型只能是子类自己，简单来说就是`<T extends SelfBounded<T>>`要求 SelfBounded 类的泛型只能是 SelfBounded 类的自限定子类。那么当我们初始化 SelfBounded 类的对象的时候，需要给的泛型是`SelfBounded 类的自限定子类`，对应`new SelfBounded<A>();`。当我们定义子类继承 SelfBounded 父类的时候，需要给的泛型是`SelfBounded 类的自限定子类`，这里又分为两种情况，第一种情况子类本身就是 SelfBounded 类的自限定子类，对应`class A extends SelfBounded<A> {}`，第二种情况是子类本身不是 SelfBounded 类的自限定子类，对应`class B extends SelfBounded<A> {}`。

```java
class SelfBounded <T extends SelfBounded<T>> {}

class A extends SelfBounded<A> {
    public static void main(String[] args) {
        new SelfBounded<A>();
    }
}

class B extends SelfBounded<A> {}
```

## 自限定也可用于泛型方法

在下面的代码中，test1 静态方法的方法泛型要求是 SelfBounded 类的自限定子类，test2 静态方法的方法泛型要求是 NoBounded 类的自限定子类。

```java
class SelfBounded<T extends SelfBounded<T>> {}

class NoBounded<T> {
    public static <U extends SelfBounded<U>> U test1(U u) {
        return u;
    }

    public static <V extends NoBounded<V>> V test2(V v) {
        return v;
    }
}
```

## 除了自限定子类还有自限定接口

自限定接口和自限定子类的作用基本一致，像这里 Integer 类实现了 Integer 自身泛型的 Comparable 接口，相当于 Integer 要实现 Comparable 接口中关于操作 Integer 自身的方法，也即之前讲到的限定只能和自己同类型对象交互。

```java
public interface Comparable<T> {
    public int compareTo(T o);
}

public final class Integer extends Number implements Comparable<Integer> {}
```

## JDK 源码里自限定的应用 Enum

java 中使用 enum 关键字来创建枚举类，实际创建出来的枚举类都继承了 java.lang.Enum。也正因为这样，所以 enum 不能再继承别的类了。其实 enum 就是 java 的一个语法糖，编译器在背后帮我们继承了 java.lang.Enum。

以下是 jdk 中 Enum 的定义：

```java
public abstract class Enum<E extends Enum<E>> implements Comparable<E>, Serializable {}
```

比如我们定义了一个枚举类：

```java
public enum WeekDay {}
```

其反编译回来是这样的：

```java
public final class WeekDay extends java.lang.Enum<WeekDay> {}
```

可以看到确实所有枚举类都是 Enum 类的自限定子类，这么做有什么好处呢？好处是限定只有相同枚举类的对象才可以交互，也就是 WeekDay 的实例就只能和 WeekDay 的实例交互。

## 参考资料

https://blog.csdn.net/anlian523/article/details/102511783
