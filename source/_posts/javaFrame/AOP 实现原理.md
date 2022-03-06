---
title: AOP 实现原理
date: 2022/02/21 19:55:35
math: true
categories:
  - [javaFrame]
tags:
  - [java]
---
# AOP 实现原理

## 什么是 AOP?

全称为 Aspect Oriented Programming: 面向切面编程 . 通过预编译方式和运行期动态代理的方式实现功能的一种技术 。

利用 AOP 可以对业务逻辑的各个部分进行隔离 , 从而使得业务逻辑各部分之间的耦合度降低 , 提高程序的可重用性 , 并且提高开发效率 。

## AOP 作用

AOP可以做到在程序的运行期间, 不修改业务代码的情况下对方法进行功能的增强。

## AOP 优势

1. AOP可以减少重复的代码
2. AOP可以在很大程度上提高开发效率
3. AOP编写出来的代码, 可以很方便的进行维护

## AOP 的实现原理

AOP 的底层是通过 spring 提供的动态代理技术实现的 . 在程序的运行期间 , spring 动态生成代理对象 , 代理对象的方法在执行时就可以进行增强功能的介入 , 从而完成目标对象方法的功能增强 。

### 基于 JDK 的动态代理

#### 目标接口

```java
public interface ITarget {
    void save();
}
```

#### 目标方法

```java
public class Target implements ITarget{
    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
```

#### 增强方法

```java
public class Advice {
    public void before() {
        System.out.println("前置增强");
    }
    public void after() {
        System.out.println("后置增强");
    }
}
```

#### 代理

```java
public class JDKProxy {
    public static void main(String[] args) {
        Target target = new Target();
        Advice advice = new Advice();
        ITarget proxy = (ITarget) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                (proxy1, method, args1) -> {
            advice.before();
            final Object invoke = method.invoke(target, args1);
            advice.after();
            return invoke;
        });
        proxy.save();
    }
}
```

### 基于cglib的动态代理