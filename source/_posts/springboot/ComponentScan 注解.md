---
title: ComponentScan 注解
date: 2022/03/14 09:38:03
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# ComponentScan 注解

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
5. 指定包名的方式配置扫描范围存在隐患，包名被重命名之后，会导致扫描实现，所以一般我们在需 要扫描的包中可以创建一个标记的接口或者类，作为 `basePackageClasses` 的值，通过这个来控制包的扫描范围
6. @CompontScan 注解会被 `ConfigurationClassPostProcessor` 类递归处理，最终得到所有需要注册的类。