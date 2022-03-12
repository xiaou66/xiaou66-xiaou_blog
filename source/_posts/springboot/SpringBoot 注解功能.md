---
title: SpringBoot 注解功能
date: 2021/12/26 14:53:02
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# SpringBoot 注解功能

## 扫描注解

### @ComponentScan

:::info

@ComponentScan 扫描某些包及其子包中所有的类，然后将满足一定条件的类作为 bean 注册到
Spring 容器容器中。

:::

#### 注解定义

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    // 指定需要扫描的包
    @AliasFor("basePackages")
    String[] value() default {};
    
    // 指定需要扫描的包
    @AliasFor("value")
    String[] basePackages() default {};
    
    // 指定一些类, Spring 容器会扫描这些类所在的包及其子包中的类
    Class<?>[] basePackageClasses() default {};
    
    // 自定义 Bean 名称生成器
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;
    
    // 
    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;
    
    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;
    
    // 需要扫描包中的那些资源, 默认是：**/*.class，
    // 即会扫描指定包中所有的 class 文件
    String resourcePattern() default "**/*.class";
    
    // 对扫描的类是否启用默认过滤器，默认为 true
    boolean useDefaultFilters() default true;

    // 过滤器: 用来配置被扫描出来的那些类会被作为组件注册到容器中
    ComponentScan.Filter[] includeFilters() default {};
    
    // 过滤器, 和 includeFilters 作用刚好相反 
    // 用来对扫描的类进行排除的, 被排除的类不会被注册到容器中
    ComponentScan.Filter[] excludeFilters() default {};
    
    // 是否延迟初始化被注册的 Bean
    boolean lazyInit() default false;

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    public @interface Filter {
        FilterType type() default FilterType.ANNOTATION;

        @AliasFor("classes")
        Class<?>[] value() default {};

        @AliasFor("value")
        Class<?>[] classes() default {};

        String[] pattern() default {};
    }
}
```

#### 工作过程

1. Spring 会扫描指定的包, 并且会递归下面的子包, 获得到一批类的数组.
2. 然后这些类会经过上面的各种过滤器, 最后剩下的类会被注册到容器中.

所以使用这个注解需要考虑 2 个问题:

1. 需要扫描哪一些包, 可以通过 `value`、`backPackages`、`basePackageClasses` 这三个参数来控制.
2. 需要使用过滤器吗, 如果需要可以通过 `useDefaultFilters`、`includeFilters`、`excludeFilters` 这三个参数来控制.

这 2 个问题搞清楚了，就可以确定哪些类会被注册到容器中。

默认情况下，任何参数都不设置的情况下，此时，会将 @ComponentScan 修饰的类所在的包作为扫描
包；**默认情况下 `useDefaultFilters` 为 true，这个为 true 的时候，Spring 容器内部会使用默认过滤器**，
规则是：凡是类上有 `@Repository`、`@Service`、`@Controller`、`@Component` 这几个注解中的任何一
个的，那么这个类就会被作为 Bean注册到 Spring 容器中，所以默认情况下，只需在类上加上这几个注解
中的任何一个，这些类就会自动交给 Spring 容器来管理了。

#### includeFilters 的使用

**Filter 定义**

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({})
public @interface Filter {
    FilterType type() default FilterType.ANNOTATION;

    @AliasFor("classes")
    Class<?>[] value() default {};

    @AliasFor("value")
    Class<?>[] classes() default {};

    String[] pattern() default {};
}
```

**FilterType 主要有**

1. ANNOTATION：通过注解的方式来筛选候选者，即判断候选者是否有指定的注解
   - 通过 classes 参数可以指定一些注解，用来判断被扫描的类上是否有 classes 参数指定的注解
2. ASSIGNABLE_TYPE：通过指定的类型来筛选候选者，即判断候选者是否是指定的类型
   - 通过 classes 参数可以指定一些类型，用来判断被扫描的类是否是 classes 参数指定的类型
3. ASPECTJ：ASPECTJ 表达式方式，即判断候选者是否匹配 ASPECTJ 表达式
4. REGEX：正则表达式方式，即判断候选者的完整名称是否和正则表达式匹配
5. CUSTOM：用户自定义过滤器来筛选候选者，对候选者的筛选交给用户自己来判断
   - 表示这个过滤器是用户自定义的，classes 参数就是用来指定用户自定义的过滤器，自定义的过滤器需要实现 `org.springframework.core.type.filter.TypeFilter` 接口

#### 实验

1. 包含指定类型的类

```java
public interface IController { }
public class Controller1 implements IController{ }
public class Controller2 implements IController{ }
```

```java
@ComponentScan(
    useDefaultFilters = false,
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = IController.class)
    }
)
public class ScanBean1 {}
```

```java 测试类
@Test void ScanBeanTest1() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ScanBean1.class);
    for (String beanName : context.getBeanDefinitionNames()) {
        System.out.println(beanName + "->" + context.getBean(beanName));
    }
}
```

运行输出

 ```
 controller1->com.example.springdemo.controller.Controller1@350a94ce
 controller2->com.example.springdemo.controller.Controller2@7e00ed0f
 ```

#### 自定义 Filter

自定义 Filter 的步骤为: 

1. 设置 @Filter 中 type 的类型为：FilterType.CUSTOM
2. 自定义过滤器类, 需要实现接口 `org.springframework.core.type.filter.TypeFilter`
3. 设置 @Filter 中的 classses 为自定义的过滤器类型

TypeFilter 接口的定义：

```java TypeFilter.class
@FunctionalInterface
public interface TypeFilter {
    boolean match(MetadataReader metadataReader,
                  MetadataReaderFactory metadataReaderFactory) throws IOException;
}
```

**MetadataReader 接口**

```java MetadataReader.class
public interface MetadataReader {
    /**
     * 返回类文件的资源引用
     */
    Resource getResource();
    /**
     * 返回一个 ClassMetadata 对象，可以通过这个读想获取类的一些元数据信息，如类的 class 对象、
     * 是否是接口、是否有注解、是否是抽象类、父类名称、接口名称、内部包含的之类列表等等，可以去看一下源
     * 码
     */
    ClassMetadata getClassMetadata();
    /**
     * 获取类上所有的注解信息
     */
    AnnotationMetadata getAnnotationMetadata();
}
```

**MetadataReaderFactory 接口**

 类元数据读取器工厂，可以通过这个类获取任意一个类的 MetadataReader 对象。

```java
public interface MetadataReaderFactory {
    /**
	 * 返回指定资源的 MetadataReader 对象
	 */
    MetadataReader getMetadataReader(String className) throws IOException;
    /**
	 * 返回指定资源的 MetadataReader 对象
	 */
    MetadataReader getMetadataReader(Resource resource) throws IOException;
}
```

了解完定义 Filter 接下来动手实践了.

```java 自定义 Filter
public class MyFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader metadataReader,
                         MetadataReaderFactory metadataReaderFactory) throws IOException {
        Class<?> curClass = null;
        try {
            curClass = Class.forName(
                metadataReader.getClassMetadata().getClassName()
            );
        }catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return curClass != null && IController.class.isAssignableFrom(curClass);
    }
}
```

```java 测试方法
@Test
void ScanBeanTest2() {
    AnnotationConfigApplicationContext context =
        new AnnotationConfigApplicationContext(ScanBean2.class);
    for (String beanName : context.getBeanDefinitionNames()) {
        System.out.println(beanName + "->" + context.getBean(beanName));
    }
}
```

**输出**

```
controller1->com.example.springdemo.controller.Controller1@1b11171f
controller2->com.example.springdemo.controller.Controller2@1151e434
```

#### 注解重复使用

第一种写法

```java
@ComponentScan(basePackageClasses = ScanClass.class)
@ComponentScan(
    useDefaultFilters = false, //不启用默认过滤器
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes
                              = IService.class)
    })
public class ScanBean3 {}
```

第二种写法

```java
@ComponentScans({
    @ComponentScan(basePackageClasses = ScanClass.class),
    @ComponentScan(
        useDefaultFilters = false, // 不启用默认过滤器
        includeFilters = {
            @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,
                                  classes = IService.class)
        })})
public class ScanBean4 {}
```

#### 源码部分

核心方法 `org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions`

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
    String[] candidateNames = registry.getBeanDefinitionNames();

    for (String beanName : candidateNames) {
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        if (beanDef.getAttribute(
            ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            // debug 日志
        }
        // checkConfigurationClassCandidate()
        // 判断一个是否是一个配置类,并为BeanDefinition设置属性为lite或者full。
        // 如果加了 @Configuration，那么对应的 BeanDefinition 为 full;
        // 如果加了 @Bean,@Component,@ComponentScan,@Import,@ImportResource 这些注解，则为 lite。
        else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    // 如果没有找到 @Configuration 类，立即返回
    if (configCandidates.isEmpty()) {
        return;
    }

    // 按照之前确定的 @Order 值排序 (如果适用的话)
    configCandidates.sort((bd1, bd2) -> {
        int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
        int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
        return Integer.compare(i1, i2);
    });

    // 检测通过外围应用程序上下文提供的任何定制 bean 名称生成策略
    SingletonBeanRegistry sbr = null;
    if (registry instanceof SingletonBeanRegistry) {
        sbr = (SingletonBeanRegistry) registry;
        if (!this.localBeanNameGeneratorSet) {
            // beanName 的生成器，因为后面会扫描出所有加入到 spring 容器中 calss 类，然后把这些 class
            // 解析成 BeanDefinition 类,
            // 此时需要利用 BeanNameGenerator 为这些 BeanDefinition 生成 beanName
            BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
                AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
            if (generator != null) {
                this.componentScanBeanNameGenerator = generator;
                this.importBeanNameGenerator = generator;
            }
        }
    }

    if (this.environment == null) {
        this.environment = new StandardEnvironment();
    }

    // 解析所有加了 @Configuration 注解的类
    ConfigurationClassParser parser = new ConfigurationClassParser(
        this.metadataReaderFactory, this.problemReporter, this.environment,
        this.resourceLoader, this.componentScanBeanNameGenerator, registry);

    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
    do {
        // 解析配置类，在此处会解析配置类上的注解 
        // (ComponentScan 扫描出的类, @Import 注册的类，以及 @Bean 方法定义的类)
        // 注意：这一步只会将加了 @Configuration 注解以及通过 @ComponentScan 注解
        // 扫描的类才会加入到 BeanDefinitionMap 中
        // 通过其他注解 (例如 @Import、@Bean) 的方式，在 parse() 方法这一步并不会将其
        // 解析为 BeanDefinition 放入到 BeanDefinitionMap 中，而是先解析成 ConfigurationClass 类
        // 真正放入到 map 中是在下面的 this.reader.loadBeanDefinitions() 方法中实现的
        StartupStep processConfig = this.applicationStartup.start("spring.context.config-classes.parse");
        parser.parse(candidates);
        parser.validate();

        Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
        configClasses.removeAll(alreadyParsed);

        // Read the model and create bean definitions based on its content
        if (this.reader == null) {
            this.reader = new ConfigurationClassBeanDefinitionReader(
                registry, this.sourceExtractor, this.resourceLoader, this.environment,
                this.importBeanNameGenerator, parser.getImportRegistry());
        }
        // 将上一步 parser 解析出的 ConfigurationClass 类加载成 BeanDefinition
        // 实际上经过上一步的 parse() 后，解析出来的 bean 已经放入到 BeanDefinition 中了，
        // 但是由于这些 bean 可能会引入新的 bean，
        // 例如实现了 ImportBeanDefinitionRegistrar 
        // 或者 ImportSelector 接口的 bean，或者 bean 中存在被 @Bean 注解的方法
        this.reader.loadBeanDefinitions(configClasses);
        alreadyParsed.addAll(configClasses);
        processConfig.tag("classCount", () -> String.valueOf(configClasses.size())).end();

        candidates.clear();
        // 这里判断 registry.getBeanDefinitionCount() > candidateNames.length 的目的是为了
        // 知道 reader.loadBeanDefinitions(configClasses)
        // 这一步有没有向 BeanDefinitionMap 中添加新的 BeanDefinition
		// 实际上就是看配置类 (例如 AppConfig 类会向 BeanDefinitionMap 中添加 bean)
		// 如果有，registry.getBeanDefinitionCount() 就会大于 candidateNames.length
		// 这样就需要再次遍历新加入的 BeanDefinition，
        // 并判断这些 bean 是否已经被解析过了，如果未解析，需要重新进行解析
		// 这里的 AppConfig 类向容器中添加的 bean，实际上在 parser.parse() 这一步已经全部被解析了
        if (registry.getBeanDefinitionCount() > candidateNames.length) {
            String[] newCandidateNames = registry.getBeanDefinitionNames();
            Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
            Set<String> alreadyParsedClasses = new HashSet<>();
            for (ConfigurationClass configurationClass : alreadyParsed) {
                alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
            }
            for (String candidateName : newCandidateNames) {
                if (!oldCandidateNames.contains(candidateName)) {
                    BeanDefinition bd = registry.getBeanDefinition(candidateName);
                    if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                        !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                        candidates.add(new BeanDefinitionHolder(bd, candidateName));
                    }
                }
            }
            candidateNames = newCandidateNames;
        }
    }
    while (!candidates.isEmpty());

    // 将 ImportRegistry 注册为 bean，以支持 ImportAware @Configuration 类
    if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
        sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
    }

    if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
        // 清除外部提供的MetadataReaderFactory的缓存;这是一个没有操作的
        // 共享缓存，因为它将被ApplicationContext清除
        ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
    }
}
```

#### 总结

1. @ComponentScan 用于批量注册  Bean，Spring 会按照这个注解的配置，递归扫描指定包中的所 有类，将满足条件的类批量注册到 spring 容器中
2. 可以通过`value`、`basePackages`、`basePackageClasses` 这几个参数来配置包的扫描范围 
3. 可以使用 `useDefaultFilters`、`includeFilters`、`excludeFilters` 这三个参数来配置过滤器,被过滤器处理之后剩下的类会被注册到容器中.
4. 指定包名的方式配置扫描范围存在隐患，包名被重命名之后，会导致扫描实现，所以一般我们在需
   要扫描的包中可以创建一个标记的接口或者类，作为 `basePackageClasses` 的值，通过这个来控制
   包的扫描范围
5.  指定包名的方式配置扫描范围存在隐患，包名被重命名之后，会导致扫描实现，所以一般我们在需 要扫描的包中可以创建一个标记的接口或者类，作为 `basePackageClasses` 的值，通过这个来控制包的扫描范围
6. @CompontScan 注解会被 `ConfigurationClassPostProcessor` 类递归处理，最终得到所有需要注册的类。

## 配置注解

### @Configuration

:::info

标识是配置类 = 配置文件, 配置类也是一个组件 默认单实例

:::

#### 参数

proxyBeanMethods 默认true

1. Full(proxyBeanMethods = true) :proxyBeanMethods 参数设置为 true 时即为：Full 全模式。 该模式下注入容器中的同一个组件无论被取出多少次都是同一个 bean 实例，即单实例对象，在该模式下 SpringBoot 每次启动都会判断检查容器中是否存在该组件。
2. Lite(proxyBeanMethods = false) :proxyBeanMethods 参数设置为 false 时即为：Lite 轻量级模式。该模式下注入容器中的同一个组件无论被取出多少次都是不同的 bean 实例，即多实例对象，在该模式下 SpringBoot 每次启动会跳过检查容器中是否存在该组件。

**什么时候用 Full 全模式，什么时候用 Lite 轻量级模式？**

当在你的同一个 Configuration 配置类中，注入到容器中的 bean 实例之间有依赖关系时，建议使用 Full 全模式；当在你的同一个 Configuration 配置类中，注入到容器中的 bean 实例之间没有依赖关系时，建议使用 Lite 轻量级模式，以提高 SpringBoot 的启动速度和性能。

### @Bean

简单的例子

```java
// 给容器总添加组件, 使用当前的方法名做为组件名称
// 如果需要改变则可以使用 @Bean("user") 进行改变
@Bean
public User user01() {
    return new User("xiaou", 18);
}
```

对应的 xml 配置

```xml
<beans>
    <bean id="user01" class="com.lean.springboot.bean.User"/>
</beans>
```

@Bean 也可以依赖其他任意数量的 bean，如果 User 依赖 Car，我们可以通过方法参数实现这个依赖

```java
// 这里会寻找容器中注册过的 car 类型的对象注入的到这里供这个 Bean 创建所需要
@Bean
public User user01(Car car) {
    return new User("xiaou", 18, car);
}
```

```java
// 指定创建 bean 时候执行的方法的名称
String initMethod() default "";
// 指定销毁 bean 时候方法
String destroyMethod() default "(inferred)";
```

#### @Scope Bean 作用域

1. singleton 全局只有一个实例，即单例模式
2. prototype 每次注入Bean都是一个新的实例
3. request 每次HTTP请求都会产生新的Bean
4. session 每次HTTP请求都会产生新的Bean，该Bean在仅在当前session内有效
5. global session 每次HTTP请求都会产生新的Bean，该Bean在 当前global Session（基于portlet的web应用中）内有效

### @Configuration 和 @Bean 特点

:::info

在 Spring 容器中 @Configuration 和 @Bean 都可以对容器注入 Bean，一般情况下在使用 @Configuration 注解的时候都伴随着 @Bean 注解但是不加 @Configuration 也可以对容器中注入 Bean 那么加和不加又有什么区别呢？🤣

:::

#### 实验

```java 使用 @Configuration
@Configuration
public class ConfigBean {
    @Bean
    public Users user1() {
        return new Users()
            .setAge(18)
            .setName("xiaou");
    }
}
```

```java 不使用 @Configuration
@Component
public class ConfigBean {
    @Bean
    public Users user1() {
        return new Users().setAge(18).setName("xiaou");
    }
}
```

```java 测试类
@Test void configurationTest() {
    System.out.println(context.getBean(ConfigBean.class));
    System.out.println(context.getBean("user1"));
}
```

1. 使用  @Configuration 结果

```
com.example.springdemo.config.ConfigBean$$EnhancerBySpringCGLIB$$e5f9c687@3b9632d1
Users(name=xiaou, age=18)
```

2. 不使用 @Configuration 结果

```
com.example.springdemo.config.ConfigBean@2f508f3c
Users(name=xiaou, age=18)
```

发现在使用 @Configuration 注解的时候所标识的类是 CGLIB 代理的, 而没有标识的则没有被 CGLIB 代理. 那么 CGLIB 代理又启到了什么作用。

```java 使用 @Configuration
@Configuration
public class ConfigBean {
    @Bean
    public Users user1() {
        System.out.println("user1 被执行了");
        return new Users()
                .setAge(18)
                .setName("xiaou");
    }
    @Bean
    public Users user2() {
        Users users = this.user1();
        return new Users()
                .setName("xiaoz")
                .setAge(2)
                .setFather(users);
    }
    @Bean
    public Users user3() {
        Users users = this.user1();
        return new Users()
                .setName("xiaoy")
                .setAge(4)
                .setFather(users);
    }
}
```

```java 不使用 @Configuration
@Component
public class ConfigBean {
    @Bean
    public Users user1() {
        System.out.println("user1 被执行了");
        return new Users()
                .setAge(18)
                .setName("xiaou");
    }
    @Bean
    public Users user2() {
        Users users = this.user1();
        return new Users()
                .setName("xiaoz")
                .setAge(2)
                .setFather(users);
    }
    @Bean
    public Users user3() {
        Users users = this.user1();
        return new Users()
                .setName("xiaoy")
                .setAge(4)
                .setFather(users);
    }
}
```

1. 使用  @Configuration 结果

```
user1 被执行了
user1 ->Users(name=xiaou, age=18, father=null)
user2 ->Users(name=xiaoz, age=2, father=Users(name=xiaou, age=18, father=null))
user3 ->Users(name=xiaoy, age=4, father=Users(name=xiaou, age=18, father=null))
```

2. 不使用 @Configuration

```
user1 被执行了
user1 被执行了
user1 被执行了
user1 ->Users(name=xiaou, age=18, father=null)
user2 ->Users(name=xiaoz, age=2, father=Users(name=xiaou, age=18, father=null))
user3 ->Users(name=xiaoy, age=4, father=Users(name=xiaou, age=18, father=null))
```

使用  @Configuration  : @Bean 修饰的方法都只被调用了一次, 这个很关键,  因为这样就是产生一个实例在 user2 和user3 中使用的都是 user1 的 bean 。

在不使用 @Configuration: user1 被执行三次分别在 @Bean 注入容器时候, user2, user3 使用 user1 的时候。

这是为什么？

被 @Configuration 修饰的类，Spring 容器中会通过 CGLIB 给这个类创建一个代理，代理会拦截所有被
@Bean 修饰的方法，默认情况（bean 为单例）下确保这些方法只被调用一次，从而确保这些 bean 是同
一个 bean，即单例的。

### @PropertySource 和 @ConfigurationProperties

:::info

单独一个 @ConfigurationProperties 注解，表示从默认的全局配置文件中获取值注入而加上 @PropertySource 则可以指定配置文件

:::

```java
@PropertySource(value = {"classpath:user.properties"})
@Component
@ConfigurationProperties(prefix = "person")
public class Person {}
```

### @ImportResource

:::info

用于导入 Spring 的 XML 配置文件，让该配置文件中定义的 bean 对象加载到 Spring 容器中。
该注解必须加载 Spring 的主程序入口上。

:::

```java
@ImportResource(locations = {"classpath:beans.xml"})
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorldApplication.class, args);
    }
}
```

### @Conditional 条件创建Bean

:::info

作用：必须是 @Conditional 指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效

:::

| **@Conditional扩展注解**        | **作用**                                            |
| ------------------------------- | --------------------------------------------------- |
| @ConditionalOnJava              | 系统的 java 版本是否符合要求                        |
| @ConditionalOnBean              | 容器中存在指定 Bean                                 |
| @ConditionalOnMissingBean       | 容器中不存在指定 Bean                               |
| @ConditionalOnExpression        | 满足 SpEL 表达式指定要求                            |
| @ConditionalOnClass             | 系统中有指定的类                                    |
| @ConditionalOnMissingClass      | 系统中没有指定的类                                  |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个 Bean 是首选 Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                      |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                        |
| @ConditionalOnWebApplication    | 当前是 web 环境                                     |
| @ConditionalOnNotWebApplication | 当前不是 web 环境                                   |
| @ConditionalOnJndi              | JNDI 存在指定项                                     |

>自动配置类必须在一定的条件下才能生效，我们可以通过启用 debug=true 属性来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效。

## 请求注解

### @PathVariable

:::info

通过 @PathVariable 可以将 URL 中占位符参数 {xxx} 绑定到处理器类的方法形参中 @PathVariable(“xxx“)

:::

#### 参数

| 参数名   | 说明         |      |
| -------- | ------------ | ---- |
| value    | 路径参数名称 |      |
| required | 是否必须     |      |

#### 演示

```java
@GetMapping("/init/{id}/{uid}")
public Map<String, Object> init(@PathVariable Integer id, @PathVariable("uid") String userId,
                                @PathVariable Map<String, String> map) {
    Map<String, Object> result = new HashMap<>();
    result.put("id", id);
    result.put("userId", userId);
    result.put("map", map);
    return result;
}
```

`@PathVariable Integer id` 必须 id 和上面路径的 id 一致如果不一致则需要使用 `@PathVariable("uid")` 指定。如果要获取全部的 PathVariable 可以使用 `Map<String, String>` 定义形参然后使用 @PathVariable 注解修饰。

**结果** 

```json
{
    "id": 11,
    "userId": "22",
    "map": {
        "uid": "22",
        "id": "11"
    }
}
```

### @RequestParam

:::info

@RequestParam：将请求参数绑定到你控制器的方法参数上（是springmvc中接收普通参数的注解）

:::

#### 参数
| 参数名       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| value        | 路径参数名称默认为参数名称                                   |
| required     | 是否必须                                                     |
| defaultValue | 默认参数值，如果设置了该值，required=true将失效，自动为false,如果没有传该参数，就使用默认值 |

#### 演示

```java
@GetMapping("/params")
public Map<String, Object> params(String name, @RequestParam("uid") Integer userId,
                                  @RequestParam(defaultValue = "false") Boolean status) {
    Map<String, Object> result = new HashMap<>();
    result.put("id", name);
    result.put("userId", userId);
    result.put("status", status);
    return result;
}
```

**结果**

```json
{
    "id": "xiaou",
    "userId": 1,
    "status": false
}
```

> 这个注解也是可以使用 `Map<String, String>` 接收所有的 RequestParam 的参数

### @RequestHeader

:::info

@RequestHeader 用于将 Web 请求头中的数据映射到控制器处理方法的参数中。

:::

#### 参数

| 参数名       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| value        | 路径参数名称默认为参数名称                                   |
| required     | 是否必须                                                     |
| defaultValue | 默认参数值，如果设置了该值，required=true将失效，自动为false,如果没有传该参数，就使用默认值 |

#### 演示

```java
@GetMapping("/requestHeader")
public Map<String, Object> requestHeader(@RequestHeader("User-Agent") String userAgent) {
    Map<String, Object> result = new HashMap<>();
    result.put("userAgent", userAgent);
    return result;
}
```

**结果**

```json
{
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36"
}
```

>这个注解也是可以使用 `Map<String, String>` 接收所有的 RequestParam 的参数

### @CookieValue

:::info

@RequestHeader 用于将 Web 请求头中的 cookie 数据取出

:::

#### 参数

| 参数名       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| value        | 路径参数名称默认为参数名称                                   |
| required     | 是否必须                                                     |
| defaultValue | 默认参数值，如果设置了该值，required=true将失效，自动为false,如果没有传该参数，就使用默认值 |

#### 演示

```java
@GetMapping("/getCookie")
public Map<String, Object> getCookie(@CookieValue("NMTID") String cookieValue,
                                     @CookieValue("NMTID") Cookie cookie) {
    Map<String, Object> result = new HashMap<>();
    result.put("NMTID_info", cookie);
    result.put("NMTID", cookieValue);
    return result;
}
```

**结果**

```json
{
    "NMTID_info": {
        "name": "NMTID",
        "value": "00OQcXWEKqZu6SPVEu2pOFJL8dr6ykAAAF6Rr7CkA",
        "version": 0,
        "comment": null,
        "domain": null,
        "maxAge": -1,
        "path": null,
        "secure": false,
        "httpOnly": false
    },
    "NMTID": "00OQcXWEKqZu6SPVEu2pOFJL8dr6ykAAAF6Rr7CkA"
}
```

### @RequestAttribute

:::info

获取 HTTP 的请求（request）对象属性值，用来传递给控制器的参数。

:::

### @RequestBody

:::info

@RequestBody 主要用来接收前端传递给后端的 json 字符串中的数据的 (请求体中的数据的)

:::
#### 参数

| 参数名       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| value        | 路径参数名称默认为参数名称                                   |
| required     | 是否必须                                                     |
| defaultValue | 默认参数值，如果设置了该值，required=true将失效，自动为false,如果没有传该参数，就使用默认值 |

#### 演示

```java
@PostMapping("requestBody")
public Map<String, String> requestBody(@RequestBody Map<String, String> requestHeaderMap) {
    return requestHeaderMap;
}
```

**结果 **

```json
// 传递 body  Content-Type = "application/json"
{"a":"1","b":"2"}
// 返回
{"a":"1","b":"2"}
```

:::warning 

1. 一个请求，只有一个RequestBody
2. 当同时使用 @RequestParam（）和 @RequestBody 时，@RequestParam（）指定的参数可以是普通元素、
   数组、集合、对象等等 (即:当，@RequestBody 与 @RequestParam() 可以同时使用时，原 SpringMVC 接收
   参数的机制不变，只不过 RequestBody 接收的是请求体里面的数据；而 RequestParam 接收的是 key-value里面的参数，所以它会被切面进行处理从而可以用普通元素、数组、集合、对象等接收)。 即：如果参数时放在请求体中，application/json 传入后台的话，那么后台要用 @RequestBody 才能接收到； 如果不是放在请求体中的话，那么后台接收前台传过来的参数时，要用 @RequestParam 来接收，或
   则形参前 什么也不写也能接收。

:::


### @MatrixVariable

:::info

矩阵变量可以出现在任何路径片段中，每一个矩阵变量都用分号（;）隔开。比如 “/cars;color=red;year=2012”。多个值可以用逗号隔开，比如 “color=red,green,blue”，或者分开写 “color=red;color=green;color=blue”

:::

#### 开启方法

Springboot 默认是无法使用矩阵变量绑定参数的。需要覆盖 WebMvcConfigurer 中的 configurePathMatch 方法。

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

#### 参数

| 参数名       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| value        | 路径参数名称默认为参数名称                                   |
| required     | 是否必须                                                     |
| defaultValue | 默认参数值，如果设置了该值，required=true将失效，自动为false,如果没有传该参数，就使用默认值 |
| pathVar      | 矩阵变量所在的URI路径变量的名称，如果需要消除歧义(例如，在多个路径段中出现同名的矩阵变量)。 |

