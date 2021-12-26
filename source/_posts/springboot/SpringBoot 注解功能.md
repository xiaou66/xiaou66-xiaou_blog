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

### @Import

:::info

@Import 注解用来帮助我们把一些需要定义为 Bean 的类导入到 IOC 容器里面。

:::

如果配置类在标准的 SpringBoot 包结构下 (SpringBootApplication 启动类包的根目录下)。是不需要 @Import 导入配置类的，SpringBoot 自动帮做了。上面的情况一般用于 @Configuration 配置类不在标准的 SpringBoot 包结构下面。所以一般在自定义 starter 的时候用到。

#### 简单使用

```java
// 导入 给容器中自动创建出组件(空参构造函数)
// 默认组件的名字是全类名
@Import({User.class})
@Configuration
public class MyConfig {}
```

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