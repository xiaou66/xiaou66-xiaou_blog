---
title: Spring Bean 作用域
date: 2022/03/09 10:25:59
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# Spring Bean 作用域

> Spring 提供的默认环境作用域有 `singleton` 和 `prototype` 在 Web 环境下还有 `request`、`session`、`application` 三个作用域。

## singleton 单例

1. 当 scope 的值设置为 singleton 的时候，整个 spring 容器中只会存在一个 bean 实例，通过容器多次查找
   bean 的时候（调用 BeanFactory 的 getBean 方法或者 bean 之间注入依赖的 bean 对象的时候），返回的
   都是同一个 bean 对象。
2. singleton 是 scope 的**默认值**，所以 Spring 容器中默认创建的 bean 对象是单例的。
3. 通常 Spring 容器在启动的时候，会将 scope 为 singleton 的 bean 创建好放在容器中
   - 有个特殊的情况，当 bean 的 lazy 被设置为 true 的时候，表示懒加载，那么使用的时候才会创建

### 测试

```java
@RestController
@Scope(value = "singleton")
public class TestController { ... }
```

```java
@Test void scopeTest1() {
    TestController c1 = applicationContext.getBean(TestController.class);
    TestController c2 = applicationContext.getBean(TestController.class);
    // true
    System.out.println(c1 == c2);
}
```

发现两次获取都是 bean 体现了 bean 的单例情况。

### 注意点

单例 bean 是整个应用共享的，所以需要考虑到线程安全问题。

在 Spring MVC 的时候，Spring MVC 中 controller 默认是单例的，在 controller 中创建了一些变量，那么这些变量实际上就变成**共享**的了，controller 可能会被很多线程同时访问，这些线程并发去修改 controller 中的共享变量，可能会出现数据错乱的问题；所以使用的时候需要特别注意。

## prototype 多例

如果 scope 被设置为 prototype 类型的了，表示这个 bean 是多例的，通过容器每次获取的 bean 都是不同
的实例，每次获取都会重新创建一个 bean 实例对象。

### 测试

```java
@RestController
@Scope(value = "prototype")
public class TestController { ... }
```

```java
@Test void scopeTest1() {
    TestController c1 = applicationContext.getBean(TestController.class);
    TestController c2 = applicationContext.getBean(TestController.class);
    // false
    System.out.println(c1 == c2);
}
```

获取 TestController 实例得到的都是不同的实例，每次获取的时候才会去调用构造方法创建 bean 实例。

### 注意点

多例 bean 每次获取的时候都会重新创建，如果这个 bean 比较复杂，创建时间比较长，会影响系统的性
能，这个地方需要注意。

## request

当一个 bean 的作用域为 request，表示在一次 http 请求中，一个 bean 对应一个实例；对每个 http 请求都 会创建一个 bean 实例，request 结束的时候，这个 bean 也就结束了，request 作用域用在 Spring 容器的 web 环境中，这个以后讲 Spring MVC 的时候会说，spring 中有个 web 容器接口 WebApplicationContext， 这个里面对 request 作用域提供了支持。

## session

这个和 request 类似，也是用在 web 环境中，session 级别共享的 bean，每个会话会对应一个 bean 实
例，不同的 session 对应不同的 bean 实例。

## application

全局 web 应用级别的作用于，也是在 web 环境中使用的，一个 web 应用程序对应一个 bean 实例，通常情
况下和 singleton 效果类似的，不过也有不一样的地方，singleton 是每个 Spring 容器中只有一个 bean 实
例，一般我们的程序只有一个 Spring 容器，但是，一个应用程序中可以创建多个 Spring 容器，不同的容
器中可以存在同名的 bean，但是 sope = aplication 的时候，不管应用中有多少个 Spring 容器，这个应用中
同名的 bean 只有一个。

## 自定义 scope

### 步骤

1. 实现 scope 接口

```java
public interface Scope {
    /**
     * 返回当前作用域中 name 对应 bean 对象 
     * name: 需要检索的 bean 的名称
     * objectFactory：如果 name 对应的 bean 在当前作用域中没有找到，那么可以调用这个
     *                ObjectFactory 来创建这个对象
     */
    Object get(String name, ObjectFactory<?> objectFactory);
    
    // 将 name 对应的 bean 从当前作用域中移除
	@Nullable
	Object remove(String name);
    
    // 用于注册销毁回调，
    // 如果想要销毁相应的对象 ,
    // 则由 Spring 容器注册相应的销毁回调，而由自定义作用域选择是不是要销毁相应的对象
	void registerDestructionCallback(String name, Runnable callback);
    
    // 用于解析相应的上下文数据，比如 request 作用域将返回 request 中的属性。
	@Nullable
	Object resolveContextualObject(String key);
    
    // 作用域的会话标识，比如 session 作用域将是 sessionId
	@Nullable
	String getConversationId();

}
```

2. 将自定义的 scope 注册到容器

需要调用 `org.springframework.beans.factory.config.ConfigurableBeanFactory#registerScope`

```java
/**
* 向容器中注册自定义的 Scope
* scopeName：作用域名称
* scope：作用域对象
**/
void registerScope(String scopeName, Scope scope);
```

3. 使用自定义的作用域

定义 bean 的时候，指定 bean 的 scope 属性为自定义的作用域名称。

### 具体操作

> 同一个线程中同名的 bean 是同一个实例，不同的线程中的 bean 是不同的实例。

1. 实现 Scope 接口

```java
public class ThreadScope implements Scope {
    public static final String THREAD_SCOPE = "thread";
    private ThreadLocal<Map<String, Object>> beanMap = new ThreadLocal<Map<String, Object>>() {
        @Override
        protected Map<String, Object> initialValue() {
            return new HashMap<>();
        }
    };
    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Object bean = beanMap.get().get(name);
        if (Objects.isNull(bean)) {
            bean = objectFactory.getObject();
            beanMap.get().put(name, bean);
        }
        return bean;
    }

    @Override
    public Object remove(String name) {
        return beanMap.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        System.out.println("name:" + name);
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}
```

2. 注册使用

```java
@Configuration
public class ConfigurableBeanFactoryConfig implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 
        throws BeansException {
        beanFactory.registerScope(ThreadScope.THREAD_SCOPE, new ThreadScope());
    }
}
```

3. 使用自定义作用域

```java
@RestController
@Scope(value = ThreadScope.THREAD_SCOPE)
public class TestController { ... }
```

4. 测试

```java
@Test void threadScopeTest1() throws InterruptedException {
    for (int i = 0; i < 2; i++) {
        new Thread(() -> {
            System.out.println(Thread.currentThread() + "," +
                               applicationContext.getBean(TestController.class));
            System.out.println(Thread.currentThread() + "," +
                               applicationContext.getBean(TestController.class));
        }).start();
        TimeUnit.SECONDS.sleep(1);
    }
}
```

5. 结果

```
Thread[Thread-2,5,main],org.example.controller.TestController@2a9a5f0b
Thread[Thread-2,5,main],org.example.controller.TestController@2a9a5f0b
Thread[Thread-3,5,main],org.example.controller.TestController@7ade64c4
Thread[Thread-3,5,main],org.example.controller.TestController@7ade64c4
```

## 总结

1. Spring 容器自带的有 2 种作用域，分别是 singleton 和 prototype；还有 3 种分别是 Spring WEB 容器环 境中才支持的 request、session、application 
2. singleton 是 Spring 容器默认的作用域，一个 Spring 容器中同名的 bean 实例只有一个，多次获取得 到的是同一个 bean；单例的 bean 需要考虑线程安全问题 
3. prototype 是多例的，每次从容器中获取同名的 bean，都会重新创建一个；多例 bean 使用的时候需 要考虑创建 bean 对性能的影响 
4. 一个应用中可以有多个 Spring 容器 
5. 自定义 scope 的 3 个步骤，实现 Scope 接口，将实现类注册到 Spring 容器，使用自定义的 sope