---
title: Java通配符详解
date: 2023-11-19 21:21:33
tags: [Java, 八股文]
---

# Java 通配符详解

## 类范型通配符

类范型可以应用到属性、方法形参、方法局部变量、方法返回类型

### 第一种情况

类范型类型可以是任意类型，不指定类型默认 Object 类型

```java
class A<T> {
    public T t;

    public void set(T t) {
        T localVar = t;
        this.t = localVar;
    }

    public T get() {
        return t;
    }
}
```

### 第二种情况

类范型类型只允许是 B 类及其子类，不指定类型默认 B 类型

```java
class A<T extends B> {
    public T t;

    public void set(T t) {
        T localVar = t;
        this.t = localVar;
    }

    public T get() {
        return t;
    }
}
```

## 方法范型通配符

方法范型可以由形参或形参范型确定

### 第一种情况

方法范型可以是任意类型，由形参确定

```java
class A {
    public <T> T test(T t) {
        T tmp = t;
        return tmp;
    }
}
```

### 第二种情况

方法范型只能是 B 类型及其子类型，由形参确定

```java
class A {
    public <T extends B> T test(T t) {
        T tmp = t;
        return tmp;
    }
}
```

### 第三种情况

方法范型可以是任意类型，由形参范型确定

```java
class A {
    public <T> T test(List<T> t) {
        List<T> tmp = t;
        return tmp.get(0);
    }
}
```

### 第四种情况

方法范型只能是 B 类型及其子类型，由形参范型确定

```java
class A {
    public <T extends B> T test(List<T> t) {
        List<T> tmp = t;
        return tmp.get(0);
    }
}
```

## 引用范型通配符

引用范型通配符指的是在引用处指定范型范围，所谓引用处可以是属性、静态属性、局部变量、方法形参，下面以局部变量为例。

```java
class A {
    private static class Some<T> {
        public T t;

        public void set(T t) {
            T tmp = t;
            this.t = tmp;
        }

        public T get() {
            return t;
        }
    }

    private static class B extends A {
    }

    private static class C extends B {
    }
}
```

### 第一种情况

这种情况引用直接确定了范型类型，其中`Some some3 = new Some()`默认是 Object 类型。

```java
Some<B> some1 = new Some<>();
Some<Object> some2 = new Some<>();
Some some3 = new Some();
```

### 第二种情况

这种情况引用允许任意类型，这里我们传入 B 范型的 Some 对象，set 方法失效，get 方法返回 Object

原理：<?>代表某种确定类型，但是不知道具体是什么类型。因此，set 方法没有办法传入任何对象，而 get 方法只能返回 Object 类型

```java
Some<?> some4 = new Some<B>();
Object object = some4.get();
```

### 第三种情况

这种情况引用允许 B 类型及其子类型，这里我们传入 B 范型的 Some 对象和 C 范型的 Some 对象，set 方法失效，get 方法返回 B 类型

原理：<? extends B>代表某种确定类型，我们只知道这种类型是 B 类型及其子类型，但是不知道具体是什么类型。因此，set 方法没有办法传入任何对象，而由于我们能够确定 get 方法返回的一定是 B 类型及其子类型，所以可以确定为 B（可能存在向上转型）。

```java
Some<? extends B> some5 = new Some<B>();
Some<? extends B> some6 = new Some<C>();
B b1 = some5.get();
B b2 = some6.get();
```

### 第四种情况

这种情况引用允许 B 类型及其父类型，这里我们传入 B 范型的 Some 对象和 A 范型的 Some 对象，set 方法可以传入 B 类型及其子类型，get 方法返回 Object

原理：<? super B>代表某种确定类型，我们只知道这种类型是 B 类型及其父类型，但是不知道具体是什么类型。因此，set 方法可以传入 B 类型及其子类型（可能存在向上转型），而 get 方法只能返回 Object 类型

```java
Some<? super B> some7 = new Some<B>();
Some<? super B> some8 = new Some<A>();
some7.set(new B());
some7.set(new C());
some8.set(new B());
some8.set(new C());
Object object1 = some7.get();
Object object2 = some8.get();
```

## 参考资料

https://segmentfault.com/a/1190000005337789
