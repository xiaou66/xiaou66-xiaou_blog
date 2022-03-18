---
title: Spring Bean 循环依赖
date: 2022/03/18 09:06:07
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# Spring Bean 循环依赖

## 什么是循环依赖

bean 之间相互依赖，形成了一个闭环。

A 依赖于 B、B 依赖于 C、C 依赖于 A。

**图示例**

 ![image-20220318091134489](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1647595130747image-20220318091134489.png)

**代码示例**

```java
public class A{
    B b;
}
public class B{
    C c;
}
public class C{
    A a;
}
```

## 循环依赖怎么检测

检测循环依赖比较简单，使用一个列表来记录正在创建中的 Bean，Bean 创建之前，先去记录中看一下 自己是否已经在列表中了，如果在，说明存在循环依赖，如果不在，则将其加入到这个列表，Bean  完毕之后，将其再从这个列表中移除。

在 Spring 创建单例 Bean 的时候，会调用下面的方法

```java
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}
```

`singletonsCurrentlyInCreation` 用来记录目前正在创建中的 Bean 名称列表，如果 `!this.singletonsCurrentlyInCreation.add(beanName)` 返回的 false 那么说明了当前 BeanName 已经在当前列表中了，所以到抛出异常 `BeanCurrentlyInCreationException`。这个异常的构造器是

```java
public BeanCurrentlyInCreationException(String beanName) {
    super(beanName,
          "Requested bean is currently in creation: Is there an unresolvable circular reference?");
}
```

上面是单例 bean 检测循环依赖处理的源码，再来看看非单例 bean 如何处理循环依赖。

以 prototype 情况为例，源码位于
`org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean` 方法中。

```java
// 检查正在创建的 bean 列表中是否存在 beanName，如果存在,说明存在循环依赖, 抛出循环依赖的异常
if (isPrototypeCurrentlyInCreation(beanName)) {
    throw new BeanCurrentlyInCreationException(beanName);
}
if (mbd.isPrototype()) {.
    Object prototypeInstance = null;
    try {
        // 将 beanName 放入正在创建的列表中
        beforePrototypeCreation(beanName);
        prototypeInstance = createBean(beanName, mbd, args);
    }
    finally {
        // 将 beanName 从正在创建的列表中移除
        afterPrototypeCreation(beanName);
    }
}
```

## Spring 中解决循环依赖方案

在 Spring 创建 Bean 主要的几个步骤

1. 实例化 Bean，即调用构造器创建 Bean 实例。
2. 注入 Bean，比如通过 Set 方式、@Autowired 注解的方式注入依赖 Bean。
3. Bean 的初始化，比如调用 init 的方法。

从上述的 Bean 的创建步骤中可以看出可能出现循环依赖步骤是 **注入 Bean** 和在 **构造器注入**。

### 构造器的依赖注入实例

```java
@Component
public class ServiceA {
    private ServiceB serviceB;
    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
@Component
public class ServiceB {
    private ServiceA serviceA;
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}
```

> 实例化 ServiceA 的时候，需要有 serviceB，而实例化 ServiceB 的时候需要有 serviceA，**构造器循环依赖是无法解决的**

### Set 注入方式

```java
@Component
public class ServiceA {
    private ServiceB serviceB;
    @Autowired
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
@Component
public class ServiceB {
    private ServiceA serviceA;
    @Autowired
    public void setServiceA(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}
```

由于单例 bean 在 spring 容器中只存在一个，所以 spring 容器中肯定是有一个缓存来存放所有已创建好的
单例 bean；获取单例 bean 之前，可以先去缓存中找，找到了直接返回，找不到的情况下再去创建，创
建完毕之后再将其丢到缓存中，可以使用一个 map 来存储单例 bean。

```java
Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
```

下面来看一下 Spring 中 set 方法创建上面 2 个 Bean 的过程

```java
1.spring 轮询准备创建 2 个 bean：serviceA 和 serviceB
2.spring 容器发现 singletonObjects 中没有 serviceA
3.调用 serviceA 的构造器创建 serviceA 实例
4.serviceA 准备注入依赖的对象，发现需要通过 setServiceB 注入 serviceB
5.serviceA 向 spring 容器查找 serviceB
6.spring 容器发现 singletonObjects 中没有 serviceB
7.调用 serviceB 的构造器创建 serviceB 实例
8.serviceB 准备注入依赖的对象，发现需要通过 setServiceA 注入 serviceA
9.serviceB 向 spring 容器查找 serviceA
10.此时又进入步骤 2 了
```

### Spring 中的解决方案

**三级缓存**

```java
// 第一级缓存：单例bean的缓存
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
// 第二级缓存：早期暴露的bean的缓存
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
// 第三级缓存：单例bean工厂的缓存
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

1. 创建 ServiceA 时候会调用下面的方法
   - `org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean`

```java
Object sharedInstance = getSingleton(beanName);
// 查看缓存中是否已经存在这个 Bean 了
if (sharedInstance != null && args == null) {
    beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
}else {
    // 缓存中不存在，准备创建这个 Bean
    if (mbd.isSingleton()) {
        // 进入单例 Bean 创建过程
        sharedInstance = getSingleton(beanName, () -> {
            try {
                return createBean(beanName, mbd, args);
            }
            catch (BeansException ex) {
                // 显式地从单例缓存中移除实例:
                // 它可能已经被创建过程热切地放在那里，以允许循环引用解析。
                // 删除所有接收到临时引用的 bean。
                destroySingleton(beanName);
                throw ex;
            }
        });
        beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
    }
}

```

接下来详细看一下 Spring 如何判断 Bean 是否在缓存中。

```java
//allowEarlyReference:
// 是否允许从三级缓存 singletonFactories 中通过 getObject 拿到 bean
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 快速检查没有完全单例锁的现有实例
    // 一级缓存中查找
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        // 从二级缓存中查找
        singletonObject = this.earlySingletonObjects.get(beanName);
        // 二级缓存没有找到并开启三级缓存开始向三级缓存中查找
        if (singletonObject == null && allowEarlyReference) {
            synchronized (this.singletonObjects) {
                // 在完整的单例锁中一致地创建早期引用
                // 再重新查询一级缓存和二级缓存
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    // 重新查找没有找到进入三级缓存
                    if (singletonObject == null) {
                        // 三级缓存返回的是一个工厂，通过工厂来获取创建 bean
                        ObjectFactory<?> singletonFactory = 					 this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            // 三级缓存返回的是一个工厂，通过工厂来获取创建 bean
                            singletonObject = singletonFactory.getObject();
                            // 将创建好的 bean 丢到二级缓存中
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            // 从三级缓存移除
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }
    return singletonObject;
}
```

刚开始，3 个缓存中肯定是找不到的，会返回 null，接着会执行下面代码准备创建 serviceA。

```java
if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, () -> { 
        try {
            return createBean(beanName, mbd, args);
        }
        catch (BeansException ex) {
            destroySingleton(beanName);
            throw ex;
        }
    });
}
```

接下来看一下 `getSingleton` 方法代码比较多。

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            // 单例 Bean 创建之前调用, 将其加入正在创建的列表中。
            // 上面有提到过，主要用来检查循环依赖使用
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            try {
                // 调用工厂创建 Bean
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }finally {
                // 单例 Bean 创建之前调用, 主要是将其从正在创建的列表中移除。
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                // 将创建完成的单例 Bean 加入到缓存中
                addSingleton(beanName, singletonObject);
            }
        }
    }
    return singletonObject;
}
```

`return createBean(beanName, mbd, args);` 最终会调用这个方法

```java
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
```

其内部主要代码如下:

```java
BeanWrapper instanceWrapper = null;
if (instanceWrapper == null) {
    // 通过反射调用构造器实例化 ServiceA
    instanceWrapper = createBeanInstance(beanName, mbd, args);
}
// 变量bean: 表示刚刚同构造器创建好的 Bean 实例
Object bean = instanceWrapper.getWrappedInstance();
// 判断是否需要暴露早期的 bean，条件为
// (是否是单例 bean && 当前容器允许循环依赖 && bean 名称存在于正在创建的 bean 名称清单中)
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                  isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
    // 若 earlySingletonExposure 为 true，通过下面代码将早期的 bean 暴露出去
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

**什么是早期 bean ?**

> 刚刚实例化好的 bean 就是早期的 bean，此时 bean 还未进行属性填充，初始化等操作

通过 `addSingletonFactory` 用于将早期的 bean 暴露出去，主要是将其丢到第 3 级缓存中，代码如下：

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?>
                                   singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        // 第 1 级缓存中不存在 bean
        if (!this.singletonObjects.containsKey(beanName)) {
            // 将其加入第 3 级缓存中
            this.singletonFactories.put(beanName, singletonFactory);
            // 后面的 2 行代码不用关注
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

上面的方法执行之后，serviceA 就被丢到第 3 级的缓存中了。

后续的过程 serviceA 开始注入依赖的对象，发现需要注入 serviceB，会从容器中获取 serviceB，而 serviceB 的获取又会走上面同样的过程实例化 serviceB，然后将 serviceB 提前暴露出去，然后 serviceB 开 始注入依赖的对象，serviceB 发现自己需要注入 serviceA，此时去容器中找 serviceA，找 serviceA 会先去 缓存中找，会执行 getSingleton("serviceA",true) ，此时会走下面代码：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 1.先从一级缓存中找
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            // 2.从二级缓存中找
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                // 3.二级缓存中没找到 && allowEarlyReference为true的情况下,从三级缓存中
                找
                    ObjectFactory<?> singletonFactory =
                    this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    // 三级缓存返回的是一个工厂，通过工厂来获取创建bean
                    singletonObject = singletonFactory.getObject();
                    // 将创建好的bean丢到二级缓存中
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    // 从三级缓存移除
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

上面的方法走完之后，serviceA 会被放入二级缓存 earlySingletonObjects 中，会将 serviceA 返回，此 时 serviceB 中的 serviceA 注入成功，serviceB 继续完成创建，然后将自己返回给 serviceA，此时 serviceA 通过 set 方法将 serviceB 注入。 serviceA 创建完毕之后，会调用 addSingleton 方法将其加入到缓存中，这块代码如下：

```java
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        // 将 bean 放入第 1 级缓存中
        this.singletonObjects.put(beanName, singletonObject);
        // 将其从第 3 级缓存中移除
        this.singletonFactories.remove(beanName);
        // 将其从第 2 级缓存中移除
        this.earlySingletonObjects.remove(beanName);
    }
}
```

```java
1. 从容器中获取 serviceA
2. 容器尝试从 3 个缓存中找 serviceA，找不到
3. 准备创建 serviceA
4. 调用 serviceA 的构造器创建 serviceA，得到 serviceA 实例，此时 serviceA 还未填充属性，未进行其他
任何初始化的操作
5. 将早期的 serviceA 暴露出去：即将其丢到第 3 级缓存 singletonFactories 中
6. serviceA 准备填充属性，发现需要注入 serviceB，然后向容器获取 serviceB
7. 容器尝试从 3 个缓存中找 serviceB，找不到
8. 准备创建 serviceB
9. 调用 serviceB 的构造器创建 serviceB，得到 serviceB 实例，此时 serviceB 还未填充属性，未进行其他
任何初始化的操作
10. 将早期的 serviceB 暴露出去：即将其丢到第 3 级缓存 singletonFactories 中
11.serviceB 准备填充属性，发现需要注入 serviceA，然后向容器获取 serviceA
12. 容器尝试从 3 个缓存中找 serviceA，发现此时 serviceA 位于第 3 级缓存中，经过处理之后，serviceA 会
从第 3 级缓存中移除，然后会存到第 2 级缓存中，然后将其返回给 serviceB，此时 serviceA 通过 serviceB 中
的 setServiceA 方法被注入到 serviceB 中
13.serviceB 继续执行后续的一些操作，最后完成创建工作，然后会调用 addSingleton 方法，将自己丢到
第 1 级缓存中，并将自己从第 2 和第 3 级缓存中移除
14.serviceB 将自己返回给 serviceA
15.serviceA 通过 setServiceB 方法将 serviceB 注入进去
16.serviceB 继续执行后续的一些操作，最后完成创建工作 , 然后会调用 addSingleton 方法，将自己丢到第
1 级缓存中，并将自己从第 2 和第 3 级缓存中移除

```

## 循环依赖无法解决的情况

> 只有单例的 bean 会通过三级缓存提前暴露来解决循环依赖的问题，而非单例的 bean，每次从容器中获取都是一个新的对象，都会重新创建，所以非单例的 bean 是没有缓存的，不会将其放到三级缓存中。

那就会有下面几种情况需要注意。

是以 2 个 bean 相互依赖为例：serviceA 和 serviceB

### 情况 1

**条件**

- serviceA: 多例
- serviceB: 多例

**结果**

此时不管是任何方式都是无法解决循环依赖的问题，最终都会报错，因为每次去获取依赖的 bean 都会重新创建。

### 情况 2

**条件**

- serviceA：单例 
- serviceB：多例

**结果**

若使用构造器的方式相互注入，是无法完成注入操作的，会报错。

## 为什么需要用 3级缓存

> **如果只使用 2 级缓存，直接将刚实例化好的 bean 暴露给二级缓存出是否可以解决问题呢？**

### 原因

**这样做是可以解决：早期暴露给其他依赖者的 bean 和最终暴露的 bean 不一致的问题。**

若将刚刚实例化好的 bean 直接丢到二级缓存中暴露出去，如果后期这个 bean 对象被更改了，比如可能在上面加了一些拦截器，将其包装为一个代理了，那么暴露出去的 bean 和最终的这个 bean 就不一样的，将自己暴露出去的时候是一个原始对象，而自己最终却是一个代理对象，最终会导致被暴露出去的和最终的 bean 不是同一个 bean 的，将产生意向不到的效果，而三级缓存就可以发现这个问题，会报错。

### 实验

```java Service1
@Component
public class Service1 {
    public void m1() {
        System.out.println("Service1 m1()");
    }
    @Autowired
    private Service2 service2;
}
```

```java Service2
@Component
public class Service2 {
    public void m1() {
        System.out.println("Service2 m1()");
    }
    @Autowired
    private Service1 service1;
}
```

在 service1 上面加个拦截器，要求在调用 service1 的任何方法之前需要先输出一行日志。

```java Service1 拦截器
@Component
public class MethodBeforeInterceptor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean,
                                 String beanName) throws BeansException {
        if ("service1".equals(beanName)) {
            // 代理创建工厂，需传入被代理的目标对象
            ProxyFactory proxyFactory = new ProxyFactory(bean);
            // 添加一个方法前置通知，会在方法执行之前调用通知中的 before 方法
            proxyFactory.addAdvice((MethodBeforeAdvice) (method, args, target) -> {
                System.out.println("你好, service1");
            });
            // 返回代理对象
            return proxyFactory.getProxy();
        }
        return bean;

    }
}
```

**配置类**

```java
@ComponentScan
public class MainConfig { }
```

**测试方法**

```java
public class BeanCircleTestTest {
    @Test
    void test() {
        AnnotationConfigApplicationContext context =
            new AnnotationConfigApplicationContext();
        context.register(MainConfig.class);
        context.refresh();
    }
}
```

**输出**

```java
org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'service1': Bean with name 'service1' has been injected into other beans [service2] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.
```

```java
if (earlySingletonExposure) {
    // 调用 getSingleton(beanName, false) 方法，这个方法用来从 3 个级别的缓存中获取 bean，但是注
    // 意了，这个地方第二个参数是 false，此时只会尝试从第 1 级和第 2 级缓存中获取 bean，如果能够获取
    // 到，说明了什么？说明了第 2 级缓存中已经有这个 bean 了，而什么情况下第 2 级缓存中会有 bean？说明
    // 这个 bean 从第 3 级缓存中已经被别人获取过，然后从第 3 级缓存移到了第 2 级缓存中，说明这个早期的
    // bean 被别人通过 getSingleton(beanName, true) 获取过
    Object earlySingletonReference = getSingleton(beanName, false);
    if (earlySingletonReference != null) {
        // 这个地方用来判断早期暴露的 bean 和最终 spring 容器对这个 bean 走完创建过程之后是否还是同一
        // 个 bean，上面我们的 service1 被代理了，所以这个地方会返回 false，此时会走到
        if (exposedObject == bean) {
            exposedObject = earlySingletonReference;
        }
        // allowRawInjectionDespiteWrapping 这个参数用来控制是否允许循环依赖的情况下，
        // 早期暴露给被人使用的 bean 在后期是否可以被包装
        // 通俗点理解就是：是否允许早期给别人使用的 bean 和最终 bean 不一致的情况，
        // 这个值默认是 false，表示不允许，也就是说你暴露给别人的 bean 和你最终的 bean
        // 需要是一致的，你给别人的是 1，你后面不能将其修改成 2 了啊，不一样了，你给我用个 3。
        else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
            String[] dependentBeans = getDependentBeans(beanName);
            Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
            for (String dependentBean : dependentBeans) {
                if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                    actualDependentBeans.add(dependentBean);
                }
            }
            if (!actualDependentBeans.isEmpty()) {
                // 
                throw new BeanCurrentlyInCreationException(beanName,"Bean with name...“);
            }
        }
    }
}
```

而上面代码注入到 service2 中的 service1 是早期的 service1，而最终 spring 容器中的 service1 变成一个代理对象了，早期的和最终的不一致了，而 `allowRawInjectionDespiteWrapping` 又是 false，所以报异常了。

那么如何解决这个问题：

很简单，将 `allowRawInjectionDespiteWrapping` 设置为 true 就可以了，下面改一下代码如下：

```java
@Test
void test() {
    AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext();
    context.addBeanFactoryPostProcessor(beanFactory -> {
        if (beanFactory instanceof DefaultListableBeanFactory) {
            ((DefaultListableBeanFactory) beanFactor.setAllowRawInjectionDespiteWrapping(true);
        }
    });
    context.register(MainConfig.class);
    context.refresh();
}
```

上面代码中将 allowRawInjectionDespiteWrapping 设置为true了，是通过一个 BeanFactoryPostProcessor 来实现的。

现在看一下 Service1 和 Service2 中的 Bean。

```java
你好, service1
com.example.springdemo.beancircule.Service1@387a8303
com.example.springdemo.beancircule.Service2@28cda624
你好, service1
com.example.springdemo.beancircule.Service2@28cda624
com.example.springdemo.beancircule.Service1@387a8303
Service1 m1()
```

可以发现 Service2 是同一个，而 Service1 并不是同一个，最后一行调用 `service2.getService1().m1();` 输出的可以发现在 Service2 中的 Service1 并没有触发拦截。

继续看暴露早期 bean 的源码，注意了下面是重点：

```java
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

注意有个 `getEarlyBeanReference` 方法，来看一下这个方法是干什么的，源码如下：

```java
protected Object getEarlyBeanReference(String beanName,
                                       RootBeanDefinition mbd,Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp =
                    (SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject,beanName);
            }
        }
    }
    return exposedObject;
}
```

从 3 级缓存中获取 bean 的时候，会调用上面这个方法来获取 bean，这个方法内部会看一下容器中是否有 SmartInstantiationAwareBeanPostProcessor 这种处理器，然后会依次调用这种处理器中的 getEarlyBeanReference 方法，那么思路来了，我们可以自定义一个 SmartInstantiationAwareBeanPostProcessor ，然后在其 getEarlyBeanReference 中来创建代理 不就可以了，聪明，我们来试试，将 MethodBeforeInterceptor 代码改成下面这样：

```java
@Component
public class MethodBeforeInterceptor2
        implements SmartInstantiationAwareBeanPostProcessor {
    @Override
    public Object getEarlyBeanReference(Object bean,
                        String beanName) throws BeansException {
        if ("service1".equals(beanName)) {
            ProxyFactory proxyFactory = new ProxyFactory(bean);
            proxyFactory.addAdvice((MethodBeforeAdvice) (method, args, target) -> {
                System.out.println("你好,service1");
            });
            return proxyFactory.getProxy();
        }
        return bean;
    }
}
```

**测试方法**

```java
@Test
void test2() {
    AnnotationConfigApplicationContext context = 
        new AnnotationConfigApplicationContext(MainConfig.class);
    Service1 service1 = context.getBean(Service1.class);
    Service2 service2 = context.getBean(Service2.class);
    service1.m1();
    service2.m1();
    System.out.println(service1);
    System.out.println(service2.getService1());
    System.out.println(service2.getService1() == service1);
}
```

**输出**

```
你好,service1
Service1 m1()
Service2 m1()
你好,service1
com.example.springdemo.beancircule.Service1@38145825
你好,service1
com.example.springdemo.beancircule.Service1@38145825
true
```

## 单例 bean 解决了循环依赖，还存在什么问题？

循环依赖的情况下，由于注入的是早期的 bean，此时早期的 bean 中还未被填充属性，初始化等各种操
作，也就是说此时 bean 并没有被完全初始化完毕，此时若直接拿去使用，可能存在有问题的风险。