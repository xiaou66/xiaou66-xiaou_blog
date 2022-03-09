---
title: FactoryBean 和 BeanFactory
date: 2022/03/09 09:20:20
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# FactoryBean 和 BeanFactory

> FactoryBean 和 BeanFactory 是两个功能完全不一样的接口。

## FactoryBean

> 我们通过 FactoryBean 可以让 Spring 容器通过这个接口的实现来获取我们需要创建的 bean 对象

### 接口方法

接口中主要有三个方法

```java
public interface FactoryBean<T> {
    
    // 容器中获取对象的时候会调用这个方法去生成 bean 对象
    @Nullable
    T getObject() throws Exception;
    
    // 指定要创建 bean 类型
    @Nullable
    Class<?> getObjectType();
    
    // 是否是单例对象
    default boolean isSingleton() {
        return true;
    }
}
```

### 接口使用

```java
@Component
public class DemoFactory implements FactoryBean<Users> {
    @Override
    public Users getObject() throws Exception {
        return new Users(1, "xiaou");
    }

    @Override
    public Class<?> getObjectType() {
        return Users.class;
    }
}
```

```java
@Test
public void test2() {
    Users user1 = applicationContext.getBean(Users.class);
    Users user2 = applicationContext.getBean(Users.class);
    // user1 == user2: true
    System.out.println("user1 == user2: " + (user1 == user2));
    // user1: Users(id=1, username=xiaou)
    System.out.println("user1: " + user1);
    // user2: Users(id=1, username=xiaou)
    System.out.println("user2: " + user2);
}
```

> 从上面的测试点来看，在 `FactoryBean` 中的 `getObject()` 在默认的情况下只调用一次。

如果想获取工厂本身的实例的话可以在 bean 名称前面加 `&` 字符

```java
@Test
public void factoryBeanTest2() {
    // org.example.factory.DemoFactory@65f2f9b0
    DemoFactory bean = (DemoFactory) applicationContext.getBean("&demoFactory");
    System.out.println(bean);
}
```

## BeanFactory

> BeanFactory 会在 bean 的生命周期的各个阶段中对 bean 进行管理，并且 spring 将这些阶段通过各种接口暴露给我们，让我们可以对 bean 进行处理，我们只要让 bean 实现对应的接口，那么 spring 就会在 bean 的生命周期调用我们实现的接口来处理该 bean。

BeanFactory 的实现类，需要实现 BeanDefinitionRegistry 接口，因为通过这个接口可以将这些 bean 注册到 beanFactory 中。

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
      implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {...}
```

## 总结

`BeanFactory`: 负责生产和管理 Bean 的一个工厂接口，提供一个 Spring Ioc 容器规范 。

`FactoryBean`:  一种 Bean 创建的一种方式，对 Bean 的一种扩展。对于复杂的 Bean 对象初始化创建使用其可封装对象的创建细节。