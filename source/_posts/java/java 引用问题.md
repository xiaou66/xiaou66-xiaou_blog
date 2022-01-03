---
title: java 引用问题
date: 2022/01/02 14:24:16
math: true
categories:
  - [java]
tags:
  - [java]
---
# java 引用问题

:::info

在 java 里，除了基础的数据类型以外，其他的都为引用类型，而 java 根据生命周期的长短又把引用类型分为强引用、软引用、弱引用和虚引用。正常的情况下我们平时基本上只适用到了强引用的类型，而其他的引用类型也就在面试中或者阅读源码的时候才能看到

:::

## 强引用

像 new 了一个对象就是强引用 `Object obj = new Object（）`

当内存空间不足时，Java 虚拟机宁愿抛出 `OutOfMemoryError` 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。如果强引用对象不使用时，需要弱化从而使 `GC` 能够回收

```java
strongReference = null;
```

显式地设置 `strongReference` 对象为 `null`，或让其超出对象的生命周期范围，则 `gc` 认为该对象不存在引用，这时就可以回收这个对象。具体什么时候收集这要取决于 `GC` 算法。

在一个方法的内部有一个强引用，这个引用保存在 Java 栈中，而真正的引用内容 (Object) 保存在 Java 堆中。 当这个方法运行完成后，就会退出方法栈，则引用对象的引用数为 0，这个对象会被回收。

```java
public void test() {
    Object strongReference = new Object();
    // 省略其他操作
}
```



## 软引用

:::info

生命周期会比强引用短一些，是通过 `SoftReference` 类实现的，**当内存有足够的空间，那么垃圾回收器就不会回收它**；因为当 JVM 认为内存空间出现不足的时候，就会尝试回收软引用指定的对象，就是说在 JVM 会在抛出 `OutOfMemoryError` 这个异常之前，会清理软引用对象

:::

> 软引用可用来实现内存敏感的高速缓存。

```java
String str = new String("hello world");
SoftReference<String> softReference = new SoftReference<String>(str);
```

:::warning 

软引用对象是在 JVM 内存不够的时候才会被回收，我们调用 `System.gc()` 方法只是起通知作用，JVM 什么时候扫描回收对象是 JVM 自己的状态决定的。就算扫描到软引用对象也不一定会回收它，**只有内存不够的时候才会回收**。

:::

当内存不足时，JVM 首先将软引用中的对象引用置为 null，然后通知垃圾回收器进行回收.垃圾收集线程会在虚拟机抛出 `OutOfMemoryError` 之前回收软引用对象，而且虚拟机会尽可能优先回收长时间闲置不用的软引用对象。对那些**刚构建**或**刚使用过**的 " 较新的 " 软对象会被虚拟机尽可能保留，这就是引入引用队列 `ReferenceQueue` 的原因

## 弱引用

:::info

弱引用就是通过 `WeakReference` 类来实现的，它的生命周期比软引用还要短（一个比一个短），在进行垃圾回收的时候，**不管内存的空间够不够都会回收掉这对象**

:::

### 构造器使用

```java
String str = new String("abc");
WeakReference<String> weakReference = new WeakReference<>(str);
str = null;
```

### 继承 WeakReference 实现

```java ThreadLocalMap.css
static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
}
```

:::warning

如果一个对象是偶尔(很少)的使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用 Weak Reference 来记住此对象

:::

### 弱引用转强引用

```java
String str = new String("abc");
WeakReference<String> weakReference = new WeakReference<>(str);
// 弱引用转强引用
String strongReference = weakReference.get();
```
