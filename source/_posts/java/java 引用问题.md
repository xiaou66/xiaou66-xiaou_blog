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

1. 强引用（StrongReference）：最传统的 “引用” 的定义，是指在程序代码之中普遍存在的引用赋值，即类似 “`object obj=new Object()`” 这种引用关系。无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。宁可报 OOM，也不会 GC 强引用
2. 软引用（SoftReference）：在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存溢出异常。
3. 弱引用（WeakReference）：被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。
4. 虚引用（PhantomReference）：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

 ![image-20220208210334547](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1644325456146image-20220208210334547.png)

## 强引用

### 介绍

1. 在 Java 程序中，最常见的引用类型是强引用（普通系统 99% 以上都是强引用），也就是我们最常见的普通对象引用，**也是默认的引用类型**。
2. 当在 Java 语言中使用 new 操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。
3. **只要强引用的对象是可触及的，垃圾收集器就永远不会回收掉被引用的对象** 只要强引用的对象是可达的，JVM 宁可报 OOM，也不会回收强引用。
4. 对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为 null，就是可以当做垃圾被收集了，当然具体回收时机还是要看垃圾收集策略。
5. 相对的，软引用、弱引用和虚引用的对象是软可触及、弱可触及和虚可触及的，在一定条件下，都是可以被回收的。所以，**强引用是造成 Java 内存泄漏的主要原因之一**。

显式地设置 `strongReference` 对象为 `null`，或让其超出对象的生命周期范围，则 `gc` 认为该对象不存在引用，这时就可以回收这个对象。具体什么时候收集这要取决于 `GC` 算法。

```java
strongReference = null;
```

在一个方法的内部有一个强引用，这个引用保存在 Java 栈中，而真正的引用内容 (Object) 保存在 Java 堆中。 当这个方法运行完成后，就会退出方法栈，则引用对象的引用数为 0，这个对象会被回收。

```java
public void test() {
    Object strongReference = new Object();
    // 省略其他操作
}
```

## 软引用

:::info

**软引用（Soft Reference）：内存不足即回收**

:::

### 介绍

1. 软引用是用来描述一些还有用，但非必需的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。注意，这里的第一次回收是不可达的对象
2. 软引用通常用来实现内存敏感的缓存。比如：高速缓存就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。
3. 垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列（Reference Queue）。
4. 类似弱引用，只不过Java虚拟机会尽量让软引用的存活时间长一些，迫不得已才清理。
5. 一句话概括：当内存足够时，不会回收软引用可达的对象。内存不够时，会回收软引用的可达对象

```java
Object obj = new Object();// 声明强引用
SoftReference<Object> sf = new SoftReference<>(obj);
obj = null; // 销毁强引用
```

## 弱引用

:::info

弱引用就是通过 `WeakReference` 类来实现的，它的生命周期比软引用还要短（一个比一个短），在进行垃圾回收的时候，**不管内存的空间够不够都会回收掉这对象**

一句话总结: **弱引用（Weak Reference）发现即回收**

:::

### 简介

1. 弱引用也是用来描述那些非必需对象，**只被弱引用关联的对象只能生存到下一次垃圾收集发生为止。在系统 GC 时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象**。
2. 但是，由于垃圾回收器的线程通常优先级很低，因此，并不一定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间。
3. 弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。
4. 软引用、弱引用都非常适合来保存那些可有可无的缓存数据。如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。



### 构造器使用

```java
Object obj = new Object();
WeakReference<Object> sf = new WeakReference<>(obj);
obj = null; // 销毁强引用
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

## 虚引用

:::info

**虚引用（Phantom Reference）：对象回收跟踪**

:::

1. 也称为 “幽灵引用” 或者 “幻影引用”，是所有引用类型中最弱的一个
2. 一个对象是否有虚引用的存在，完全不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回收。
3. 它不能单独使用，也无法通过虚引用来获取被引用的对象。当试图通过虚引用的 get() 方法取得对象时，总是 null 。**即通过虚引用无法获取到我们的数据**
4. **为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程。比如：能在这个对象被收集器回收时收到一个系统通知**
5. 虚引用必须和引用队列一起使用。虚引用在创建时必须提供一个引用队列作为参数。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况。
6. 由于虚引用可以跟踪对象的回收时间，因此，也可以将一些资源释放操作放置在虚引用中执行和记录。

```java
// 声明强引用
Object obj = new Object();
// 声明引用队列
ReferenceQueue phantomQueue = new ReferenceQueue();
// 声明虚引用（还需要传入引用队列）
PhantomReference<Object> sf = new PhantomReference<>(obj, phantomQueue);
obj = null; 
```

**追踪对象是否被销毁**

```java
public class PhantomReferenceTest {
    public static PhantomReferenceTest obj;// 当前类对象的声明
    static ReferenceQueue<PhantomReferenceTest> phantomQueue = null;// 引用队列

    public static class CheckRefQueue extends Thread {
        @Override
        public void run() {
            while (true) {
                if (phantomQueue != null) {
                    PhantomReference<PhantomReferenceTest> objt = null;
                    objt = (PhantomReference<PhantomReferenceTest>) phantomQueue.poll();
                    if (objt != null) {
                        System.out.println(
                            "追踪垃圾回收过程：PhantomReferenceTest 实例被 GC 了 "
                        );
                    }
                }
            }
        }
    }

    @Override
    // finalize() 方法只能被调用一次！
    protected void finalize() throws Throwable { 
        super.finalize();
        System.out.println(" 调用当前类的 finalize() 方法 ");
        obj = this;
    }

    public static void main(String[] args) {
        Thread t = new CheckRefQueue();
        // 设置为守护线程：当程序中没有非守护线程时，守护线程也就执行结束。
        t.setDaemon(true);
        t.start();

        phantomQueue = new ReferenceQueue<PhantomReferenceTest>();
        obj = new PhantomReferenceTest();
        // 构造了 PhantomReferenceTest 对象的虚引用，并指定了引用队列
        PhantomReference<PhantomReferenceTest> phantomRef = new PhantomReference<PhantomReferenceTest>(obj, phantomQueue);

        try {
            // 不可获取虚引用中的对象
            System.out.println(phantomRef.get());
            System.out.println(" 第 1 次 gc");
            // 将强引用去除
            obj = null;
            // 第一次进行 GC, 由于对象可复活，GC 无法回收该对象
            System.gc();
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 可用 ");
            }
            System.out.println(" 第 2 次 gc");
            obj = null;
            // 一旦将 obj 对象回收，就会将此虚引用存放到引用队列中。
            System.gc(); 
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 可用 ");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

1、第一次尝试获取虚引用的值，发现无法获取的，这是因为虚引用是无法直接获取对象的值，然后进行第一次 GC，因为会调用 finalize 方法，将对象复活了，所以对象没有被回收

2、但是调用第二次 GC 操作的时候，因为 finalize 方法只能执行一次，所以就触发了 GC 操作，将对象回收了，同时将会触发第二个操作就是将待回收的对象存入到引用队列中。

**输出结果：**

```
null
 第 1 次 gc
 调用当前类的 finalize() 方法 
obj 可用 
 第 2 次 gc
null
追踪垃圾回收过程：PhantomReferenceTest 实例被 GC 了 
obj 是 null
```

