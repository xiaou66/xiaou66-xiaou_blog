---
title: Spring Value 注解动态刷新
date: 2022/03/17 09:14:48
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# Spring Value 注解动态刷新

## @Value 的用法

### @Value 使用步骤

1. 将 @PropertySource 放在类上面，如下

```java
@Component
@PropertySource({"classpath:db.properties"})
public class DbConfig {}
```

2. 使用 @Value 注解引用配置文件的值

通过 @Value 引用上面配置文件中的值：

**语法**

```java
@Value("${配置文件中的key:默认值}")
@Value("${配置文件中的key}")
```

**例子**

```java
// 如果 password 不存在，将 123 作为值
@Value("${password:123}")
// 如果 password 不存在，值为 ${password}
@Value("${password}")
```

## @Value 数据来源

通常情况下我们 @Value 的数据来源于配置文件，不过，还可以用其他方式，比如我们可以将配置文件的
内容放在数据库，这样修改起来更容易一些。

我们需要先了解一下 @Value 中数据来源于 Spring 的什么地方。

Spring 中有个类

```java
org.springframework.core.env.PropertySource
```

> 可以将其理解为一个配置源，里面包含了 key->value 的配置信息，可以通过这个类中提供的方法获 取 key 对应的 value 信息。

内部有个方法：

```java
// 通过 name 获取对应的配置信息
@Nullable
public abstract Object getProperty(String name);
```

还有个比较重要的接口 `org.springframework.core.env.Environment`。

`org.springframework.core.env.ConfigurableEnvironment`

用来表示环境配置信息，这个接口有几个方法比较重要。



```java
// 用来解析 ${...} 的, @Value 注解最后就是调用这个方法来解析
String resolvePlaceholders(String text);
// 返回 MutablePropertySources 对象。
MutablePropertySources getPropertySources();
```

**MutablePropertySources** 

```java
public class MutablePropertySources implements PropertySources {
    private final List<PropertySource<?>> propertySourceList = 
        new CopyOnWriteArrayList<>();
}
```

内部包含一个 propertySourceList 列表。 Spring 容器中会有一个 Environment 对象，最后会调用这个对象的resolvePlaceholders 方法解析 @Value。

### 解析 @Value 过程 

1. 将 @Value 注解的 value 参数值作为 Environment.resolvePlaceholders 方法参数进行解析
2. Environment 内部会访问 MutablePropertySources 来解析
3. MutablePropertySources 内部有多个 PropertySource，此时会遍历 PropertySource 列表，调用PropertySource.getProperty 方法来解析 key 对应的值

> 通过上面过程，如果我们想改变 @Value 数据的来源，只需要将配置信息包装为 PropertySource 对象， 丢到 Environment 中的 MutablePropertySources 内部就可以了。

## 自定义 Bean 作用域 

### @Scope  源码

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {

   @AliasFor("scopeName")
   String value() default "";

   @AliasFor("value")
   String scopeName() default "";
    // *** 重要 ***
   ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```

这个参数的值是个 ScopedProxyMode 类型的枚举，值有下面 4 中

```java
public enum ScopedProxyMode {
    // 默认值通常等于 NO，除非在组件扫描指令级别配置了不同的默认值。
    DEFAULT,
    // 不要创建作用域代理。当与非单例作用域的实例一起使用时，这种代理模式通常不太有用，
    // 如果要将其作为依赖项使用，则应该倾向于使用 INTERFACES 或 TARGET CLASS 代理模式。
    NO,
    // 创建一个 JDK 动态代理，实现由目标对象的类公开的所有接口
    INTERFACES,
    // 创建一个基于类的代理 (使用 CGLIB)。
    TARGET_CLASS;
}
```

### 自定义一个 bean 作用域的注解

```JAVA
@Getter
@Setter
@AllArgsConstructor
@MyScope
@Component
public class UsersScope {
    private String name;

    public UsersScope() {
        System.out.println("------------创建 User 对象" + this);
        this.name = UUID.randomUUID().toString();
    }
}
```

```java
public class BeanMyScope implements Scope {
    public static final String SCOPE_MY = "my";


    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        System.out.println("BeanMyScope >>>>> get:" + name);
        return objectFactory.getObject();
    }

    @Override
    public Object remove(String name) {
        return null;
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {

    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return null;
    }
}
```

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Scope(BeanMyScope.SCOPE_MY)
public @interface MyScope {
    // 通过上面的案例可以看出，当自定义的 Scope 中 proxyMode=ScopedProxyMode.TARGET_CLASS 的时候，会给     // 这个 bean 创建一个代理对象，调用代理对象的任何方法，都会调用这个自定义的作用域实现
    // (上面的 BeanMyScope) 中 get 方法来重新来获取这个 bean 对象。
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;
}
```

```java 测试类
@Test
void test1() {
    AnnotationConfigApplicationContext  context = new AnnotationConfigApplicationContext();
    context.getBeanFactory().registerScope(BeanMyScope.SCOPE_MY, new BeanMyScope());
    context.register(MainConfig.class);
    context.refresh();
    UsersScope usersScope = context.getBean(UsersScope.class);
    System.out.println("usersScope: " + usersScope.getClass());
    for (int i = 1; i <= 3; i++) {
        System.out.println(
            String.format("********\n第%d次开始调用getUsername", i));
        System.out.println(usersScope.getName());
        System.out.println(
            String.format("第%d次调用getUsername结束\n********\n",i));

    }
}
```

**输出**

```
usersScope: class com.example.springdemo.scope.UsersScope$$EnhancerBySpringCGLIB$$6b6d3629
********
第1次开始调用getUsername
BeanMyScope >>>>> get:scopedTarget.usersScope
------------创建 User 对象com.example.springdemo.scope.UsersScope@700fb871
cd215da0-c8c5-4c41-9c43-7a266912d52f
第1次调用getUsername结束
********

********
第2次开始调用getUsername
BeanMyScope >>>>> get:scopedTarget.usersScope
------------创建 User 对象com.example.springdemo.scope.UsersScope@2bdd8394
6f35d6a5-f32f-448d-baaf-4c98185342a5
第2次调用getUsername结束
********

********
第3次开始调用getUsername
BeanMyScope >>>>> get:scopedTarget.usersScope
------------创建 User 对象com.example.springdemo.scope.UsersScope@5f9edf14
796d2a6c-1b4e-417b-baf8-fe6b4269acb5
第3次调用getUsername结束
********
```

从输出的前2行可以看出： 

1. 调用 context.getBean(User.class) 从容器中获取 bean 的时候，此时并没有调用 User 的构造函数去 创建 User 对象。
2. 第二行输出的类型可以看出，getBean 返回的 user 对象是一个 cglib 代理对象。
2. 后面的日志输出可以看出，每次调用 user.getUsername 方法的时候，内部自动调用了 BeanMyScope#get 方法和 User 的构造函数。

> 通过上面的案例可以看出，当自定义的 Scope 中 `proxyMode = ScopedProxyMode.TARGET_CLASS` 的时候，会给这个 bean 创建一个代理对象，调用代理对象的任何方法，都会调用这个自定义的作用域实现类（上面的 BeanMyScope）中 get 方法来重新来获取这个 bean 对象。

## 动态刷新 @Value 具体实现

1. 先实现一个 RefreshScode

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Scope(BeanRefreshScope.SCOPE_REFRESH)
public @interface RefreshScope {
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;
}
```

2. 自定义 RefreshScode 对应的解析类

```java
public class BeanRefreshScope implements Scope {
    public static final String SCOPE_REFRESH = "refresh";
    private static final BeanRefreshScope INSTANCE = new BeanRefreshScope();
    private ConcurrentHashMap<String, Object> beanMap = new ConcurrentHashMap<>();

    private BeanRefreshScope() {}

    public static BeanRefreshScope getInstance() {
        return INSTANCE;
    }
    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        /**
         *  等价与
         *  Object bean = beanMap.get(name);
         *  if (bean == null) {
         *      bean = objectFactory.getObject();
         *      beanMap.put(name, bean);
         *   }
         *   return bean;
         */
        return INSTANCE.beanMap
            .computeIfAbsent(name, (key) -> objectFactory.getObject());
    }
    public void clean() {
        INSTANCE.beanMap.clear();
    }
    @Override
    public Object remove(String name) {
        return INSTANCE.beanMap.remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {

    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return null;
    }
}
```

3. 来个邮件配置类，使用 @Value 注解注入配置，这个 bean 作用域为自定义的 @RefreshScope

```java
@Component
@RefreshScope
@Getter
@Setter
@ToString
public class MailConfig {
    @Value("${mail.username}")
    private String username;
}
```

4. 再来个普通的 bean，内部会注入 MailConfig

```java
@Component
public class MailService {
    @Autowired
    private MailConfig mailConfig;

    @Override
    public String toString() {
        return "MailService{" +
                "mainConfig=" + mailConfig +
                '}';
    }
}
```

5. 来个类，模拟从 db 中获取邮件配置信息

```java
public class DbUtil {
    public static Map<String, Object> getMailInfoFromDb() {
        Map<String, Object> result = new HashMap<>();
        result.put("mail.username", UUID.randomUUID().toString());
        return result;
    }
}
```

6. 来个 Spring 配置类，扫描加载上面的组件

```java
@ComponentScan
@Configuration
public class MainConfig {}
```

**工具类**

```java
public class RefreshConfigUtil {
    /**
     * 模拟改变数据库中配置信息
     */
    public static void updateDbConfig(AbstractApplicationContext context) {
        // 更新 content 中 mailPropertySource 配置信息
        refreshMailPropertySource(context);
        // 清空 BeanRefreshScope 中所有 bean 的缓存
        BeanRefreshScope.getInstance().clean();
    }
    public static void refreshMailPropertySource(AbstractApplicationContext context) {
        Map<String, Object> mailInfoFormDb = DbUtil.getMailInfoFromDb();
        // 将其丢在 MapPropertySource 中
        // (MapPropertySource 类是 spring 提供的一个类 , PropertySource 的子类)
        MapPropertySource mapPropertySource = new MapPropertySource("mail", mailInfoFormDb);
        context.getEnvironment().getPropertySources().addFirst(mapPropertySource);
    }
}
```

**测试类**

```java
@Test
@SneakyThrows
void test3() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.getBeanFactory()
        .registerScope(BeanRefreshScope.SCOPE_REFRESH, BeanRefreshScope.getInstance());
    context.register(MainConfig.class);
    RefreshConfigUtil.refreshMailPropertySource(context);
    context.refresh();
    MailService mailService = context.getBean(MailService.class);
    System.out.println("配置未更新的情况下");
    for (int i = 0; i < 3; i++) {
        System.out.println(mailService);
        TimeUnit.MICROSECONDS.sleep(200);
    }
    System.out.println("模拟 3 次跟更新配置的效果");
    for (int i = 0; i < 3; i++) {
        RefreshConfigUtil.updateDbConfig(context);
        System.out.println(mailService);
        TimeUnit.MICROSECONDS.sleep(200);
    }
}
```

**输出**

```
配置未更新的情况下
MailService{mainConfig=MailConfig(username=6ec67cf7-fb4d-4712-94b9-97306ef207ff)}
MailService{mainConfig=MailConfig(username=6ec67cf7-fb4d-4712-94b9-97306ef207ff)}
MailService{mainConfig=MailConfig(username=6ec67cf7-fb4d-4712-94b9-97306ef207ff)}
模拟 3 次跟更新配置的效果
MailService{mainConfig=MailConfig(username=92306969-f4fb-4f95-8a0e-a8f55023a1ef)}
MailService{mainConfig=MailConfig(username=0d155231-4a6e-499e-a493-012a1f136b0e)}
MailService{mainConfig=MailConfig(username=93125586-507f-4735-91f8-c0d1a598eee1)}
```

## 总结

动态 @Value 实现的关键是 @Scope 中 proxyMode 参数，值为 ScopedProxyMode.DEFAULT，会生成一 个代理，通过这个代理来实现 @Value 动态刷新的效果，这个地方是关键。 

有兴趣的可以去看一下 SpringBoot 中的 @RefreshScope 注解源码，和上面自定义的 @RefreshScope 类似，实现原理类似的。