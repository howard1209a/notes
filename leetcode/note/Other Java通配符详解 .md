---
title: Java通配符详解
date: 2023-11-19 21:21:33
tags: [Java, 八股文]
---

# Java 通配符详解

Java 中通配符有三种，我们考虑如下代码。可以观察到属性类型是 K 范型，set 方法形参类型是 K 范型，get 方法返回值类型是 K 范型，eqauls 方法没有用到 K 范型。

```java
class A {
}

class B extends A {
}

class C extends B {
}

class T<K> {
    public K k;

    public void set(K k) {
        this.k = k;
    }

    public K get() {
        return this.k;
    }

    @Override
    public boolean equals(Object obj) {
        return super.equals(obj);
    }
}
```

## <? extends T>

观察如下代码，<? extends A>代表某种确定类型，我们只知道这种类型是 A 类型及其子类型，但是不知道具体是什么类型。因此，set 方法没有办法传入任何对象，而 get 方法和 k 属性本身，由于我们能够确定 K 范型是 A 类型及其子类型，所以它们的类型可以确定为 A（可能存在向上转型）。equals 方法和范型无关不受影响。

```java
T<? extends A> t=new T<B>();
t.set(new B()); // 错
t.set(new A()); // 错
A k = t.k; // 对
A a = t.get(); // 对
t.equals(t); // 对
```

## <? super T>

观察如下代码，<? super B>代表某种确定类型，我们只知道这种类型是 B 类型及其父类型，但是不知道具体是什么类型。因此，set 方法可以传入 B 类型及其子类型，而 get 方法和 k 属性只能返回 Object 类型。equals 方法和范型无关不受影响。

```java
T<? super B> t = new T<A>();
t.set(new B()); // 对
t.set(new C()); // 对
Object o1 = t.k; // 对
Object o2 = t.get(); // 对
t.equals(t); // 对
```

## <?>

观察如下代码，<?>代表某种确定类型，但是不知道具体是什么类型。因此，set 方法没有办法传入任何对象，而 get 方法和 k 属性只能返回 Object 类型。equals 方法和范型无关不受影响。

```java
T<?> t = new T<B>();
Object o1 = t.k; // 对
Object o2 = t.get(); // 对
t.set(new A()); // 错
t.set(new B()); // 错
t.set(new C()); // 错
t.equals(t); // 对
```

## 参考资料

https://segmentfault.com/a/1190000005337789
