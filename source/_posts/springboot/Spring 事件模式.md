---
title: Spring 事件模式
date: 2022/03/17 13:01:28
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# Spring 事件模式

## 事件模式概念

**事件源**：事件的触发者，比如上面的注册器就是事件源。 

**事件**：描述发生了什么事情的对象，比如 xxx注册成功的事件。

**事件监听器**：监听到事件发生的时候，做一些处理。

## Spring 中实现事件模式

| Spring 事件类                                                | 作用                 |
| ------------------------------------------------------------ | -------------------- |
| `org.springframework.context.ApplicationEvent`               | 示事件对象的父类     |
| `org.springframework.context.ApplicationListener`            | 事件监听接口         |
| `org.springframework.context.event.ApplicationEventMulticaster` | 事件广播器           |
| `org.springframework.context.event.SimpleApplicationEventMulticaster` | 事件广播器的简单实现 |

### 硬编码的方式使用 Spring 事件 3 步骤

#### 定义事件

自定义事件，需要继承 `ApplicationEvent` 类。

#### 定义监视器

自定义事件监听器，需要实现 `ApplicationListener` 接口，这个接口有个方法 `onApplicationEvent` 需要实现，用来处理感兴趣的事件。

```java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> 
    extends EventListener {
	void onApplicationEvent(E event);
}
```

#### 创建事件广播器

创建事件广播器 `ApplicationEventMulticaster` ，这是个接口，你可以自己实现这个接口，也可以直 接使用系统给我们提供的 `SimpleApplicationEventMulticaster` ，如下：

创建事件广播器 `ApplicationEventMulticaster` ，这是个接口，你可以自己实现这个接口，也可以直 接使用系统给我们提供的 `SimpleApplicationEventMulticaster` ，如下：

```java
ApplicationEventMulticaster applicationEventMulticaster = 
    new SimpleApplicationEventMulticaster();
```

#### 向广播器中注册事件监听器

将事件监听器注册到广播器 `ApplicationEventMulticaster` 中，如：

```java
ApplicationEventMulticaster applicationEventMulticaster = 
    new SimpleApplicationEventMulticaster();
applicationEventMulticaster.addApplicationListener(
    new SendEmailOnOrderCreateListener());
```

#### 通过广播器发布事件

广播事件，调用 `ApplicationEventMulticaster#multicastEvent` 方法 广播事件，此时广播器中对这 个事件感兴趣的监听器会处理这个事件。

```java
applicationEventMulticaster.multicastEvent(
    new OrderCreateEvent(applicationEventMulticaster, 1L))
```

### 案例

#### 事件类

```java
public class OrderCreateEvent extends ApplicationEvent {
    @Setter
    @Getter
    private Long orderId;
    public OrderCreateEvent(Object source, Long orderId) {
        super(source);
        this.orderId = orderId;
    }
}
```

#### 监听器

```java
public class SendEmailOnOrderCreateListener
        implements ApplicationListener<OrderCreateEvent> {
    @Override
    public void onApplicationEvent(OrderCreateEvent event) {
        System.out.println(
            String.format("订单【%d】创建成功，给下单人发送邮件通知!",
                event.getOrderId()));
    }
}
```

#### 测试

```java
@Test
void test1() {
    ApplicationEventMulticaster applicationEventMulticaster =
        new SimpleApplicationEventMulticaster();
    applicationEventMulticaster.addApplicationListener(
        new SendEmailOnOrderCreateListener());
    applicationEventMulticaster.multicastEvent(
        new OrderCreateEvent(applicationEventMulticaster, 1L));
}
```

#### 运行输出

```
订单【1】创建成功，给下单人发送邮件通知!
```

### ApplicationContext 容器中事件的支持

使用以 `ApplicationContext` 结尾的类作为 Spring 的容器来启动应用，下面 2 个是 比较常见的

1. `AnnotationConfigApplicationContext`
2. `ClassPathXmlApplicationContext`

这两个类都继承了 `AbstractApplicationContext`

- `AbstractApplicationContext` 实现了 `ApplicationEventPublisher` 接口。
- `AbstractApplicationContext` 内部有个 `ApplicationEventMulticaster` 类型的字段

说明了 `AbstractApplicationContext` 内部已经集成了事件广播器 `ApplicationEventMulticaster` ，说明 `AbstractApplicationContext` 内部是具体事件相关功能的，这些功能是通过其内部的 `ApplicationEventMulticaster` 来实现的，也就是说将事件的功能委托给了内部的 `ApplicationEventMulticaster` 来实现。

### ApplicationEventPublisher 接口

上面类图中多了一个新的接口 `ApplicationEventPublisher` 

```java
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        publishEvent((Object) event);
    }
    void publishEvent(Object event);
}
```

Spring 中不是有个 `ApplicationEventMulticaster` 接口么，此处怎么又来了一个发布事件的接口？ 

这个接口的实现类中，比如 `AnnotationConfigApplicationContext` 内部将这2个方法委托给 `ApplicationEventMulticaster#multicastEvent` 进行处理了。 所以调用 `AbstractApplicationContext`中的`publishEvent` 方法，也实现广播事件的效果，不过使用 `AbstractApplicationContext` 也只能通过调用 `publishEvent` 方法来广播事件。

### 获取 ApplicationEventPublisher 对象

如果我们想在普通的 bean 中获取 ApplicationEventPublisher 对象, 需要实现 ApplicationEventPublisherAware 接口

```java
public interface ApplicationEventPublisherAware extends Aware {
    void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher);
}
```

## Spring 使用方式

1. 面相接口的方式
2. 面相 @EventListener 注解的方式

### 面相接口的方式

> 实现用户注册成功后发布事件，然后在监听器中发送邮件的功能。

1. 用户注册事件

```java
public class UserRegisterEvent extends ApplicationEvent {
    @Getter
    private final String username;
    public UserRegisterEvent(Object source, String username) {
        super(source);
        this.username = username;
    }
}
```

2. 发送邮件监听器

```java
@Component
public class SendEmailListener implements ApplicationListener<UserRegisterEvent> {
    @Override
    public void onApplicationEvent(UserRegisterEvent event) {
        System.out.println(String.format("给用户【%s】发送注册成功邮件!",
                event.getUsername()));
    }
}
```

3. 用户注册服务

```java
@Component
public class UserRegisterService implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher applicationEventPublisher;
    public void registerUser(String username) {
        //用户注册(将用户信息入库等操作)
        System.out.println(String.format("用户【%s】注册成功", username));
        this.applicationEventPublisher.publishEvent(new UserRegisterEvent(this, username));
    }
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.applicationEventPublisher = publisher;
    }
}
```

4. 配置类

```java
@ComponentScan
public class EventConfig { }
```

5. 测试方法

```java
@Test
void test2() {
    AnnotationConfigApplicationContext context = new
        AnnotationConfigApplicationContext();
    context.register(EventConfig.class);
    context.refresh();
    // 获取用户注册服务
    UserRegisterService userRegisterService = 
        context.getBean(UserRegisterService.class);
    // 模拟用户注册
    userRegisterService.registerUser("xiaou");
}
```

**结果输出**

```
用户【xiaou】注册成功
给用户【xiaou】发送注册成功邮件!
```

Spring 容器在创建 bean 的过程中，会判断 bean 是否为 ApplicationListener 类型，进而会将其作为监
听器注册到 `AbstractApplicationContext#applicationEventMulticaster` 中，这块的源码在下面
这个方法中

```java
org.springframework.context.support.ApplicationListenerDetector#postProcessAfterInitialization
```

### 面相 @EventListener 注解方式

```java
public class UserRegisterListener {
    @EventListener
    public void sendMail(UserRegisterEvent event) {
        System.out.println(String.format("给用户【%s】发送注册成功邮件!",
                event.getUsername()));
    }
    @EventListener
    public void sendIntegral(UserRegisterEvent event) {
        System.out.println(String.format("给用户【%s】发送积分!",
                event.getUsername()));
    }
}
```

**输出结果**

```
用户【xiaou】注册成功
给用户【xiaou】发送注册成功邮件!
给用户【xiaou】发送积分!
```

spring 中处理 @EventListener 注解源码位于下面的方法中

```
org.springframework.context.event.EventListenerMethodProcessor#afterSingletonsInstantiated
```

`EventListenerMethodProcessor` 实现了 `SmartInitializingSingleton` 接口，`SmartInitializingSingleton` 接口中的 `afterSingletonsInstantiated` 方法会在所有单例的 bean 创建完成之后被 Spring 容器调用。

## 监听器排序功能

> 如果某个事件有多个监听器，默认情况下，监听器执行顺序是无序的，不过我们可以为监听器指定顺序。

### 实现方式

#### 实现 Ordered 接口

实现 `org.springframework.core.Ordered` 接口的 `int getOrder();`  方法，返回值越小，优先级就越高。

#### 实现 PriorityOrdered 接口

`org.springframework.core.PriorityOrdered` 接口继承了方式一中的 Ordered 接口，所以如果你实现 PriorityOrdered 接口，也需要实现 getOrder 方法。

#### 使用 @Order 注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
@Documented
public @interface Order {
    int value() default Ordered.LOWEST_PRECEDENCE;
}

```

## 监听器异步模式

监听器最终是通过 `ApplicationEventMulticaster` 内部的实现来调用的，所以我们关注的重点就是这个类，这个类默认有个实现类 `SimpleApplicationEventMulticaster` ，这个类是支持监听器异步调用的，里面有个字段：

```java
private Executor taskExecutor;
```

高并发比较熟悉的朋友对 Executor 这个接口是比较熟悉的，可以用来异步执行一些任务。 

我们常用的线程池类 `java.util.concurrent.ThreadPoolExecutor` 就实现了 Executor 接口。 再来看一下 `SimpleApplicationEventMulticaster` 中事件监听器的调用，最终会执行下面这个方法。

```java
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable
                           ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType :
                           resolveDefaultEventType(event));
    Executor executor = getTaskExecutor();
    for (ApplicationListener<?> listener : getApplicationListeners(event, type))
    {
        if (executor != null) { // @1
            executor.execute(() -> invokeListener(listener, event));
        }
        else {
            invokeListener(listener, event);
        }
    }
}
```

上面的 invokeListener 方法内部就是调用监听器，从代码 @1 可以看出，如果当前 executor 不为空， 监听器就会被异步调用，所以如果需要异步只需要让 executor 不为空就可以了，但是默认情况下 executor 是空的，此时需要我们来给其设置一个值，下面我们需要看容器中是如何创建广播器的，我 们在那个地方去干预。

通常我们使用的容器是 AbstractApplicationContext 类型的，需要看一下 AbstractApplicationContext 中广播器是怎么初始化的，就是下面这个方法，容器启动的时候会被调 用，用来初始化 AbstractApplicationContext 中的事件广播器 applicationEventMulticaster。

```java
public static final String APPLICATION_EVENT_MULTICASTER_BEAN_NAME =
    "applicationEventMulticaster";

protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME))
    {
        this.applicationEventMulticaster =
            beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME,
                                ApplicationEventMulticaster.class);
    }
    else {
        this.applicationEventMulticaster = new
            SimpleApplicationEventMulticaster(beanFactory);
        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME,
                                      this.applicationEventMulticaster);
    }
}
```

上面逻辑解释一下：判断 Spring 容器中是否有名称为 ApplicationEventMulticaster 的 bean，如果有就将其作为事件广播器，否则创建一个 SimpleApplicationEventMulticaster 作为广播器，并将其注册到 Spring 容器中。 从上面可以得出结论：我们只需要自定义一个类型为 SimpleApplicationEventMulticaster 名称为 ApplicationEventMulticaster 的 bean 就可以了，顺便给 executor 设置一个值，就可以实现监听器异步执行了。

从上面可以得出结论: 我们只需要自定义一个类型为 SimpleApplicationEventMulticaster 名称为applicationEventMulticaster 的 bean 就可以了，顺便给 executor 设置一个值，就可以实现监听器异步执行了。

**实现代码**

```java
@ComponentScan
@Configuration
public class AsyncEventConfig {
    @Bean
    public ApplicationEventMulticaster applicationEventMulticaster() { //@1
        //创建一个事件广播器
        SimpleApplicationEventMulticaster result = new
                SimpleApplicationEventMulticaster();
        //给广播器提供一个线程池，通过这个线程池来调用事件监听器
        Executor executor =
                this.applicationEventMulticasterThreadPool().getObject();
        //设置异步执行器
        result.setTaskExecutor(executor);//@1
        return result;
    }
    @Bean
    public ThreadPoolExecutorFactoryBean applicationEventMulticasterThreadPool() {
        ThreadPoolExecutorFactoryBean factory =
                new ThreadPoolExecutorFactoryBean();
        factory.setThreadNamePrefix("applicationEventMulticasterThreadPool-");
        factory.setCorePoolSize(5);
        return factory;
    }
}
```

**输出结果**

实现了监听器异步执行的效果

```
main-用户【xiaou】注册成功
applicationEventMulticasterThreadPool-2-给用户【xiaou】发送注册成功邮件!
applicationEventMulticasterThreadPool-3-给用户【xiaou】发送积分!
```

## 总结

1.  Spring中事件是使用接口的方式还是使用注解的方式？具体使用哪种方式都可以，不过在公司内部 最好大家都统一使用一种方式 。
2. 异步事件的模式，通常将一些非主要的业务放在监听器中执行，因为监听器中存在失败的风险，所 以使用的时候需要注意。如果只是为了解耦，但是被解耦的次要业务也是必须要成功的，可以使用 消息中间件的方式来解决这些问题。
