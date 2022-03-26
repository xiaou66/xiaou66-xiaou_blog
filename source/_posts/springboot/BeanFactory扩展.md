---
title: BeanFactory 扩展
date: 2022/03/22 09:21:12
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# BeanFactory扩展

> Spring 中有 2 个非常重要的接口：BeanFactoryPostProcessor 和 BeanDefinitionRegistryPostProcessor，这 2 个接口。

## Spring 容器中主要的 4 个阶段

1. 阶段 1：Bean 注册阶段，此阶段会完成所有 bean 的注册。
2. 阶段 2：BeanFactory 后置处理阶段。
3. 阶段 3：注册 BeanPostProcessor
4. 阶段 4：Bean 插件阶段，此阶段完成所有单例 Bean 的注册和装载操作。

## 阶段1：Bean 注册阶段

### 概述

spring 中所有 bean 的注册都会在此阶段完成，按照规范，所有 bean 的注册必须在此阶段进行，其他阶段
不要再进行 bean 的注册。

这个阶段 spring 为我们提供 1 个接口：BeanDefinitionRegistryPostProcessor，spring 容器在这个阶段
中会获取容器中所有类型为 `BeanDefinitionRegistryPostProcessor` 的 bean，然后会调用他们的
`postProcessBeanDefinitionRegistry` 方法，源码如下，方法参数类型是
`BeanDefinitionRegistry` ，这个类型大家都比较熟悉，即 bean 定义注册器，内部提供了一些方法可
以用来向容器中注册 bean。

```java
public interface BeanDefinitionRegistryPostProcessor extends
    BeanFactoryPostProcessor {
    void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)
        throws BeansException;
}
```

这个接口还继承了 `BeanFactoryPostProcessor` 接口。

当容器中有多个 `BeanDefinitionRegistryPostProcessor` 的时候，可以通过下面任意一种方式来指定顺序。

1. 实现 `org.springframework.core.PriorityOrdered` 接口
2. 实现 `org.springframework.core.Ordered` 接口

**执行顺序**

```java
PriorityOrdered.getOrder() asc,Ordered.getOrder() asc
```

### BeanDefinitionRegistryPostProcessor 简单使用

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor
        implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)
            throws BeansException {
        AbstractBeanDefinition userNameBdf = BeanDefinitionBuilder.genericBeanDefinition(String.class)
                .addConstructorArgValue("xiaou")
                .getBeanDefinition();
        registry.registerBeanDefinition("username", userNameBdf);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
            throws BeansException {
    }
}
```

```JAVA
@ComponentScan
public class MyMainConfigure {}
```

```java 测试方法
@Test
public void test1() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.register(MyMainConfigure.class);
    context.refresh();
    System.out.println(context.getBean("username"));
}
```

**输出**

```
xiaou
```

### 多个指定顺序

下面我们定义 2 个 `BeanDefinitionRegistryPostProcessor` ，都实现 Ordered 接口，第一个 order 的 值为 2，第二个 order 的值为 1，我们来看一下具体执行的顺序。

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor
    implements BeanDefinitionRegistryPostProcessor, Ordered {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)
        throws BeansException {
        AbstractBeanDefinition userNameBdf = BeanDefinitionBuilder.genericBeanDefinition(String.class)
            .addConstructorArgValue("xiaou")
            .getBeanDefinition();
        registry.registerBeanDefinition("username", userNameBdf);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
        throws BeansException {
    }

    @Override
    public int getOrder() {
        return 2;
    }
}
```

```java
@Component
public class MyBeanDefinitionRegistryPostProcessor2
        implements BeanDefinitionRegistryPostProcessor, Ordered {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)
            throws BeansException {
        AbstractBeanDefinition userNameBdf = BeanDefinitionBuilder.genericBeanDefinition(String.class)
                .addConstructorArgValue("把卡把卡")
                .getBeanDefinition();
        registry.registerBeanDefinition("petName", userNameBdf);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
            throws BeansException {
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

**测试方法**

```java
@Test
public void test2() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.register(MyMainConfigure.class);
    context.refresh();
    context.getBeansOfType(String.class).forEach((beanName, bean) ->
             System.out.println(String.format("%s->%s", beanName, bean)));
}
```

**输出**

```
petName->把卡把卡
username->xiaou
```

## BeanFactory 后置处理阶段

### 描述

在这个阶段 Spring 容器已经完成所有 bean 的注册，这个阶段中可以对 BeanFactory 中的一下信息进行修改，比如改阶段 1 中一些 bean 的定义信息，修改 BeanFactory 的一些配置等等，此阶段 spring 也提供了一个接口来进行扩展： BeanFactoryPostProcessor ，简称 bfpp ，接口中有个方法 `postProcessBeanFactory` ，spring 会获取容器中所有 BeanFactoryPostProcessor 类型的 bean，然后调用他们的 `postProcessBeanFactory` ，来看一下这个接口的源码：

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
        throws BeansException;
}
```

当容器中有多个 `BeanFactoryPostProcessor` 的时候，可以通过下面任意一种方式来指定顺序

1. 实现 `org.springframework.core.PriorityOrdered` 接口
2. 实现 `org.springframework.core.Ordered` 接口

执行顺序

```java
PriorityOrdered.getOrder() asc,Ordered.getOrder() asc
```

### 实验

> 在 BeanFactoryPostProcessor 来修改 bean 中已经注册的 bean 定义的信息，给一个 bean 属性设置一个值。

```java
@Component
public class MyBeanFactoryPostProcessor
    implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
        throws BeansException {
        System.out.println("准备修改类信息");
        BeanDefinition user1 = beanFactory.getBeanDefinition("user1");
        user1.getPropertyValues().add("name", "xiaoY");
    }
}
```

user1 Bean

```java
@Bean
public Users user1() {
    System.out.println("user1 被执行了");
    return new Users()
        .setAge(18)
        .setName("xiaou");
}
```

**测试方法**

```java
@Test
public void test3() {
    AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext();
    context.register(MyMainConfigure.class);
    context.refresh();
    System.out.println(context.getBean("user1"));
}
```

**输出**

```
准备修改类信息
Users(name=xiaoY, age=18, father=null)
```

