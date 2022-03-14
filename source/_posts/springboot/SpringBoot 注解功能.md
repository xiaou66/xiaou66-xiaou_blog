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

