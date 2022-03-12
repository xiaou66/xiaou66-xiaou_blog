---
title: @Import 注解
date: 2022/03/12 17:43:09
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# @Import 注解

:::info

@Import 注解可以用来批量导入需要注册的各种类是, 如普通的类/配置的类 完成对普通类和配置类中所有 bean 的注册。

:::

如果配置类在标准的 SpringBoot 包结构下 (SpringBootApplication 启动类包的根目录下)。是不需要 @Import 导入配置类的，SpringBoot 自动帮做了。上面的情况一般用于 @Configuration 配置类不在标准的 SpringBoot 包结构下面。所以一般在自定义 starter 的时候用到。

## @Import 源码

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

    /**
	 * {@link Configuration @Configuration}, {@link ImportSelector},
	 * {@link ImportBeanDefinitionRegistrar}, or regular component classes to import.
	 */
    Class<?>[] value();

}
```

> 1. @Import 可以使用在任何类型上，通常情况下，类和注解上用的比较多。
> 2. value：一个 Class 数组，设置需要导入的类，可以是 @Configuration 标注的列，可以是
>    ImportSelector 接口或者 ImportBeanDefinitionRegistrar 接口类型的，或者需要导入的普通组件
>    类。

## @Import 常见五种用法

1. value 为普通的类
2. value 为 @Configuration 标注的类
3. value 为 @ComponentScan 标注的类
4. value 为 ImportBeanDefinitionRegistrar 接口类型
5. value 为 ImportSelector 接口类型
6. value 为 DeferredImportSelector 接口类型

## 实验1

### 普通和配置类情况

```java 普通类
@Component
public class ImportBean { }
```

```java 配置类
@Configuration
public class ImportConfigBean {
    @Bean
    public String module2() {
        return "我是配置类";
    }
}
```

```java
importBean1->com.example.springdemo.ImportBean1@196a42c3
com.example.springdemo.config.ImportConfigBean->com.example.springdemo.config.ImportConfigBean$$EnhancerBySpringCGLIB$$5ffb1c43@4c60d6e
module2->我是配置类
com.example.springdemo.config.ImportBean->com.example.springdemo.config.ImportBean@15043a2f
```

**结果分析**

1. ImportBean1 成功注入到容器中。bean 名称为完整的类名

### 导入 @CompontentScan 标注的类

```java
public interface IController { }
public class Controller1 implements IController{ }
public class Controller2 implements IController{ }
@ComponentScan(
    useDefaultFilters = false,
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = IController.class)
    }
)
public class ScanBean1 {}

@Import(ScanBean1.class)
public class ImportBean2 {}
```

```java 测试类
@Test 
void importTest2() {
    AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(ImportBean1.class);
    for (String beanName : context.getBeanDefinitionNames()) {
        System.out.println(beanName + "->" + context.getBean(beanName));
    }
}
```

**输出**

```
importBean2->com.example.springdemo.ImportBean2@428640fa
controller1->com.example.springdemo.controller.Controller1@d9345cd
controller2->com.example.springdemo.controller.Controller2@2d710f1a
com.example.springdemo.ScanBean1->com.example.springdemo.ScanBean1@29215f06
```

## ImportBeanDefinitionRegistrar 接口

:::info

这个接口提供了通过 Spring 容器 API 的方式直接向容器中注册 bean。

:::

这个完整的名称

```
org.springframework.context.annotation.ImportBeanDefinitionRegistrar
```

该接口的源码

```java
public interface ImportBeanDefinitionRegistrar {
   default void registerBeanDefinitions(
       AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry,
         BeanNameGenerator importBeanNameGenerator) {
      registerBeanDefinitions(importingClassMetadata, registry);
   }

   default void registerBeanDefinitions(
       AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
   }

}
```

1. **importingClassMetadata**: 
   - AnnotationMetadata 类型的，通过这个可以获取被 @Import 注解标注的类所有注解的信息。
2. registry
   - BeanDefinitionRegistry 类型，是一个接口，内部提供了注册 bean 的各种方法。
3. importBeanNameGenerator
   - BeanNameGenerator 类型，是一个接口，内部有一个方法，用来生成 bean 的名称。

### BeanDefinitionRegistry 接口：bean 定义注册器

bean 定义注册器，提供了 bean 注册的各种方法，来看一下源码：

```java
public interface BeanDefinitionRegistry extends AliasRegistry {
    /**
     * 注册一个新的bean定义
     * beanName：bean的名称
     * beanDefinition：bean定义信息
     */
    void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException;
    /**
     * 通过bean名称移除已注册的bean
     * beanName：bean名称
     */
    void removeBeanDefinition(String beanName) throws
        NoSuchBeanDefinitionException;
    /**
     * 通过名称获取bean的定义信息
     * beanName：bean名称
     */
    BeanDefinition getBeanDefinition(String beanName) throws
        NoSuchBeanDefinitionException;
    /**
     * 查看beanName是否注册过
     */
    boolean containsBeanDefinition(String beanName);
    /**
     * 获取已经定义（注册）的bean名称列表
     */
    String[] getBeanDefinitionNames();
    /**
     * 返回注册器中已注册的bean数量
     */
    int getBeanDefinitionCount();
    /**
     * 确定给定的bean名称或者别名是否已在此注册表中使用
     * beanName：可以是bean名称或者bean的别名
     */
    boolean isBeanNameInUse(String beanName);
}
```

基本上所有 bean 工厂都实现了这个接口，让 bean 工厂拥有 bean 注册的各种能力。

### BeanNameGenerator 接口：bean 名称生成器

bean 名称生成器，这个接口只有一个方法，用来生成 bean 的名称：

```java
public interface BeanNameGenerator {
	String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry);
}
```

Spring 内置了 3 个实现。

#### DefaultBeanNameGenerator

默认 bean 名称生成器，xml 中 bean 未指定名称的时候，默认就会使用这个生成器，默认为：完整的类名 #bean 编号。

#### AnnotationBeanNameGenerator

注解方式的 bean 名称生成器，比如通过 @Component(bean 名称) 的方式指定 bean 名称，如果没有通过 注解方式指定名称，默认会将完整的类名作为 bean 名称。

#### FullyQualifiedAnnotationBeanNameGenerator

将完整的类名作为bean的名称

### BeanDefinition接口：bean定义信息

用来表示 bean 定义信息的接口，我们向容器中注册 bean 之前，会通过 xml 或者其他方式定义 bean 的各种配置信息，bean 的所有配置信息都会被转换为一个 BeanDefinition 对象，然后通过容器中 BeanDefinitionRegistry 接口中的方法，将 BeanDefinition 注册到 Spring 容器中，完成 bean 的注册操作。

这个接口有很多实现类，有兴趣的可以去看看源码，BeanDefinition 的各种用法，以后会通过专题细 说。

## ImportBeanDefinitionRegistrar接口类型

### 步骤

1. 定义 ImportBeanDefinitionRegistrar 接口实现类，在 registerBeanDefinitions 方法中使用 registry 来注册 bean 
2. 使用 @Import 来导入步骤 1 中定义的类
3. 使用步骤 2 中 @Import 标注的类作为 AnnotationConfigApplicationContext 构造参数创建 spring 容器 
4. 使用 AnnotationConfigApplicationContext 操作 bean

### 实验

1.  **ImportBeanDefinitionRegistrar 接口实现类**

```java 
public class MyImportBeanDefinitionRegistrar implements
    ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
                                        BeanDefinitionRegistry registry,
                                        BeanNameGenerator importBeanNameGenerator) {
        final AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder
            .genericBeanDefinition(BaseController.class).getBeanDefinition();
        registry.registerBeanDefinition("baseController", beanDefinition);
        final AbstractBeanDefinition beanDefinition1 = BeanDefinitionBuilder.genericBeanDefinition(Base2Controller.class)
            .addPropertyReference("baseController", "baseController")
            .getBeanDefinition();
        registry.registerBeanDefinition("base2Controller", beanDefinition1);
    }
}
```

2. **使用 @Import 来导入步骤 1 中定义的类**

```java
@Import(value = MyImportBeanDefinitionRegistrar.class)
public class ImportBean1 { }
```

3. 测试方法

```java
@Test
public void test1() {
    AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(ImportBean1.class);
    for (String beanName : context.getBeanDefinitionNames()) {
        System.out.println(String.format("%s->%s", beanName,
                                         context.getBean(beanName)));
    }
}
```

**结果**

```
importBean1->com.lean.springboot.ImportBean1@740fb309
baseController->com.lean.springboot.controller.BaseController@7bd7d6d6
base2Controller->Base2Controller{baseController=com.lean.springboot.controller.BaseController@7bd7d6d6}
```

## ImportSelector 接口类型

### 接口源码

```java
public interface ImportSelector {
    /**
     * 返回需要导入的类名的数组，可以是任何普通类，配置类（@Configuration、@Bean、
     *  @CompontentScan等标注的类）
     * @importingClassMetadata：用来获取被@Import标注的类上面所有的注解信息
     */
    String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```

### 使用步骤

1. 定义 ImportSelector 接口实现类，在 selectImports 返回需要导入的类的名称数组
2. 使用 @Import 来导入步骤 1 中定义的类
3. 使用步骤 2 中 @Import 标注的类作为 AnnotationConfigApplicationContext 构造参数创建 spring 容 器 4. 使用
4. 使用 AnnotationConfigApplicationContext 操作 bean

### 实验

1. 普通类

```java
public class BaseController {}
```

2. 配置类

```java
@Configuration
public class ImportDemoConfig {
    @Bean
    public String name() {
        return "xiaou";
    }
}
```

3. 定义 ImportSelector 接口实现类

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[] {
            BaseController.class.getName(),
            ImportDemoConfig.class.getName(),
        };
    }
}
```

4. @Import 注解

```java
@Import(value = MyImportSelector.class)
public class ImportBean2 { }
```

5. 测试方法

```java
@Test
public void test1() {
    AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(ImportBean2.class);
    for (String beanName : context.getBeanDefinitionNames()) {
        System.out.println(String.format("%s->%s", beanName,
                                         context.getBean(beanName)));
    }

}
```

**结果**

```
importBean2->com.lean.springboot.ImportBean2@2a7ed1f
com.lean.springboot.controller.BaseController->com.lean.springboot.controller.BaseController@3fa247d1
com.lean.springboot.configure.ImportDemoConfig->com.lean.springboot.configure.ImportDemoConfig$$EnhancerBySpringCGLIB$$6acd82fc@2cb2fc20
name->xiaou
```

> 如果不想开启方法耗时统计，只需要将 @EnableMethodCostTime 去掉就可以 了，用起来是不是特别舒服。

## DeferredImportSelector 接口

DeferredImportSelector 是 ImportSelector 的子接口，既然是 ImportSelector 的子接口，所以也可以通 过 @Import 进行导入，这个接口和 ImportSelector 不同地方有两点：

1. 延迟导入
2. 指定导入的类的处理顺序

### 延迟导入

比如 @Import 的 value 包含了多个普通类、多个 @Configuration 标注的配置类、多个 ImportSelector 接 口的实现类，多个 ImportBeanDefinitionRegistrar 接口的实现类，还有 DeferredImportSelector 接口实 现类，此时 spring 处理这些被导入的类的时候，**会将 DeferredImportSelector 类型的放在最后处理， 会先处理其他被导入的类，其他类会按照 value 所在的前后顺序进行处理**。

## 自定义耗时注解

### 需求

凡是类名中包含 service 的，调用他们内部任何方法，我们希望调用之后能够输出这些方法的耗时。

### 分析

1. 创建一个代理类，通过代理来间接访问需要统计耗时的 bean 对象
2. 拦截 bean 的创建，给 bean 实例生成代理生成代理

### 实现

1. service

```java
@Component
public class Service2 {
    public void m2() {
        System.out.println(this.getClass() + "-> m2");
    }
}
```

```java
@Component
public class Service1 {
    public void m1() {
        System.out.println(this.getClass() + "-> m1");
    }
}
```

2. CGLIB 代理类

> 统计耗时的代理类

```java
public class CostTimeProxy implements MethodInterceptor {
    private Object target;

    public CostTimeProxy(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object o, Method method,
                            Object[] objects, MethodProxy methodProxy)
        throws Throwable {
        long starTime = System.nanoTime();
        // 调用被代理对象（即 target）的方法，获取结果
        Object result = method.invoke(target, objects);
        long endTime = System.nanoTime();
        System.out.println(method + "，耗时(纳秒)：" + (endTime - starTime));
        return result;
    }
    /**
     * 创建任意类的代理对象
     *
     * @param target
     * @param <T>
     * @return
     */
    public static <T> T createProxy(T target) {
        CostTimeProxy costTimeProxy = new CostTimeProxy(target);
        Enhancer enhancer = new Enhancer();
        enhancer.setCallback(costTimeProxy);
        enhancer.setSuperclass(target.getClass());
        return (T) enhancer.create();
    }
}
```

3. 拦截 bean 实例的创建，返回代理对象

`BeanPostProcessor` 这个接口是 bean 处理器，内部有 2 个方法，分别在 bean 初始化前后会进行调用，以后讲声明周期的时候 还会细说的，这里你只需要知道 bean 初始化之后会调用 postProcessAfterInitialization 方法就 行，这个方法中我们会给 bean 创建一个代理对象。

```java
public class MethodCostTimeProxyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean,
                        String beanName) throws BeansException {
        if (beanName.startsWith("service")) {
            return CostTimeProxy.createProxy(bean);
        }
        return bean;
    }
}

```

4. 通过@Import 结合ImportSelector的方式来导入刚刚创建拦截 bean 实例的类

```java
public class MethodCostTimeImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[] {
                MethodCostTimeProxyBeanPostProcessor.class.getName()
        };
    }
}
```

5. 定义 `@EnableMethodCostTime`

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(MethodCostTimeImportSelector.class)
public @interface EnableMethodCostTime { }
```

6. 扫描类

```java
@ComponentScan("com.lean.springboot.service")
@EnableMethodCostTime
public class ImportBean3 { }
```

7. 测试类

```java
@Test
public void test2() {
    AnnotationConfigApplicationContext context = new
        AnnotationConfigApplicationContext(ImportBean3.class);
    Service1 service1 = context.getBean(Service1.class);
    Service2 service2 = context.getBean(Service2.class);
    service1.m1();
    service2.m2();
}
```

**测试输出**

```
class com.lean.springboot.service.Service1-> m1
public void com.lean.springboot.service.Service1.m1()，耗时(纳秒)：174000
class com.lean.springboot.service.Service2-> m2
public void com.lean.springboot.service.Service2.m2()，耗时(纳秒)：49200
```

