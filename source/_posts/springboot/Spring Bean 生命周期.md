---
title: Spring Bean 生命周期
date: 2022/03/14 13:09:40
math: true
categories:
  - [springboot]
tags:
  - [java]
---
# Spring Bean 生命周期

Spring Bean 生命周期主要有 13 个阶段。

1. Bean 元信息配置阶段
2. Bean 元信息解析阶段
3. Bean 注册到容器中
4. BeanDefinition 合并阶段
5. Bean Class 加载阶段
6. Bean 实例化阶段
   - Bean 实例化前阶段
   - Bean 实例化阶段
7. 合并后的 BeanDefinition 处理
8. 属性赋值阶段
   - Bean 实例化后阶段
   - Bean 属性赋值前阶段
   - Bean 属性赋值阶段
9. Bean 初始化阶段
   - Bean Aware 接口回调阶段
   - Bean 初始化前阶段
   - Bean 初始化阶段
   - Bean 初始化后阶段
10. 所有单例 bean 初始化完成后阶段
11. Bean 的使用阶段
12. Bean 销毁前阶段
13. Bean 销毁阶段

可以直接简化 5 个阶段

1. Bean 的实例化
2. *Bean 属性赋值
3. Bean 的初始化
4. Bean 的使用
5. Bean 的销毁

## Spring Bean 整个执行流程

1. Spring 启动，查找并加载需要被 Spring 管理的 Bean，对 Bean 进行实例化。
2. 对 Bean 进行属性注入。
3. 如果 Bean 实现了 BeanNameAware 接口，则 Spring 调用 Bean 的 setBeanName() 方法传入当前 Bean 的 id 值。
4. 如果 Bean 实现了 BeanFactoryAware 接口，则 Spring 调用 setBeanFactory() 方法传入当前工厂实例的引用。
5. 如果 Bean 实现了 ApplicationContextAware 接口，则 Spring 调用 setApplicationContext() 方法传入当前 ApplicationContext 实例的引用。
6. 如果 Bean 实现了 BeanPostProcessor 接口，则 Spring 调用该接口的预初始化方法 `postProcessBeforeInitialzation()` 对 Bean 进行加工操作，此处非常重要，Spring 的 AOP 就是利用它实现的。
7. 如果 Bean 实现了 InitializingBean 接口，则 Spring 将调用 afterPropertiesSet() 方法。
8. 如果在配置文件中通过 init-method 属性指定了初始化方法，则调用该初始化方法。
9. 如果 BeanPostProcessor 和 Bean 关联，则 Spring 将调用该接口的初始化方法 postProcessAfterInitialization()。此时，Bean 已经可以被应用系统使用了。
10. 如果在 `<bean>` 中指定了该 Bean 的作用域为 singleton，则将该 Bean 放入 Spring IoC 的缓存池中，触发 Spring 对该 Bean 的生命周期管理；如果在 `<bean>` 中指定了该 Bean 的作用域为 prototype，则将该 Bean 交给调用者，调用者管理该 Bean 的生命周期，Spring 不再管理该 Bean。
11. 如果 Bean 实现了 DisposableBean 接口，则 Spring 会调用 destory() 方法销毁 Bean；如果在配置文件中通过 destory-method 属性指定了 Bean 的销毁方法，则 Spring 将调用该方法对 Bean 进行销毁。

## Bean 元信息配置阶段

> 这个阶段主要是 Bean 消息的定义阶段

### Bean 信息定义 4 中方式

1. API 的定义方式
2. XML 文件定义方式
3. properties 文件定义方式
4. 注解的定义方式

## API 的方式

> 其他所有的方式最终都将会采用这种方式来定义 Bean 配置信息。

**Spring 容器启动的过程中，会将 Bean 解析成 Spring 内部的 BeanDefinition 结构**。

### BeanDefinition: bean 定义信息接口

表示 bean 定义信息的接口，里面定义了一些获取 bean 定义配置信息的各种方法。

#### 定义 

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    /**
     * 设置此 bean 的父 bean 名称 (对应 xml 中 bean 元素的 parent 属性)
     */
    void setParentName(@Nullable String parentName);
    /**
     * 返回此 bean 定义时指定的父 bean 的名称
     */
    @Nullable
    String getParentName();
    /**
     * 指定此 bean 定义的 bean 类名 (对应 xml 中 bean 元素的 class 属性)
     */
    void setBeanClassName(@Nullable String beanClassName);
    /**
     * 返回此 bean 定义的当前 bean 类名
     * 注意，如果子定义重写/继承其父类的类名，则这不一定是运行时使用的实际类名。此外，这可能只
     * 是调用工厂方法的类，或者在调用方法的工厂 bean 引用的情况下，它甚至可能是空的。因此，不要认为这是运
     * 行时的最终 bean 类型，而只将其用于单个 bean 定义级别的解析目的。
     */
    @Nullable
    String getBeanClassName();
    /**
     * 设置此 bean 的生命周期，如：singleton、prototype (对应 xml 中 bean 元素的 scope 属性)
     */
    void setScope(@Nullable String scope);
    /**
     * 返回此 bean 的生命周期，如：singleton、prototype
     */
    @Nullable
    String getScope();
    /**
     * 设置是否应延迟初始化此 bean（对应 xml 中 bean 元素的 lazy 属性）
     */
    void setLazyInit(boolean lazyInit);
    /**
     * 返回是否应延迟初始化此 bean，只对单例 bean 有效
    */
    boolean isLazyInit();
    /**
     * 设置此 bean 依赖于初始化的 bean 的名称 ,bean 工厂将保证 dependsOn 指定的 bean 会在当前 bean 初
     * 始化之前先初始化好
     */
    void setDependsOn(@Nullable String... dependsOn);
    /**
     * 返回此 bean 所依赖的 bean 名称
     */
    @Nullable
    String[] getDependsOn();
    /**
     * 设置此 bean 是否作为其他 bean 自动注入时的候选者
     * autowireCandidate
     */
    void setAutowireCandidate(boolean autowireCandidate);
    /**
     * 返回此 bean 是否作为其他 bean 自动注入时的候选者
     */
    boolean isAutowireCandidate();
    /**
     * 设置此 bean 是否为自动注入的主要候选者
     * primary：是否为主要候选者
     */
    void setPrimary(boolean primary);
    /**
     * 返回此 bean 是否作为自动注入的主要候选者
     */
    boolean isPrimary();
    /**
     * 指定要使用的工厂 bean (如果有指定)。这是要对其调用指定工厂方法的 bean 的名称。
     * factoryBeanName：工厂 bean 名称
     */
    void setFactoryBeanName(@Nullable String factoryBeanName);
    /**
     * 返回工厂 bean 名称 (如果有) (对应 xml 中 bean 元素的 factory-bean 属性)
     */
    @Nullable
    String getFactoryBeanName();
    /**
     * 指定工厂方法（如果有）。此方法将使用构造函数参数调用，如果未指定任何参数，则不使用任何参
     * 数调用。该方法将在指定的工厂 bean（如果有的话）上调用，或者作为本地 bean 类上的静态方法调用。
     * factoryMethodName：工厂方法名称
     */
    void setFactoryMethodName(@Nullable String factoryMethodName);
    /**
     * 返回工厂方法名称 (对应 xml 中 bean 的 factory-method 属性)
     */
    @Nullable
    String getFactoryMethodName();
    /**
     * 返回此 bean 的构造函数参数值
     */
    ConstructorArgumentValues getConstructorArgumentValues();
    /**
     * 是否有构造器参数值设置信息（对应xml中bean元素的<constructor-arg />子元素）
    */
    default boolean hasConstructorArgumentValues() {
        return !getConstructorArgumentValues().isEmpty();
    }
    /**
     * 获取 bean 定义是配置的属性值设置信息
     */
    MutablePropertyValues getPropertyValues();
    /**
     * 这个 bean 定义中是否有属性设置信息（对应 xml 中 bean 元素的 <property /> 子元素）
     */
    default boolean hasPropertyValues() {
        return !getPropertyValues().isEmpty();
    }
    /**
     * 设置 bean 初始化方法名称
     */
    void setInitMethodName(@Nullable String initMethodName);
    /**
    * bean 初始化方法名称
    */
    @Nullable
    String getInitMethodName();
    /**
     * 设置 bean 销毁方法的名称
     */
    void setDestroyMethodName(@Nullable String destroyMethodName);
    /**
     * bean 销毁的方法名称
     */
    @Nullable
    String getDestroyMethodName();
    /**
     * 设置 bean 的 role 信息
     */
    void setRole(int role);
    /**
    * bean 定义的 role 信息
    */
    int getRole();
    /**
     * 设置 bean 描述信息
     */
    void setDescription(@Nullable String description);
    /**
     *bean 描述信息
     */
    @Nullable
    String getDescription();
    /**
     * bean类型解析器
     */
    ResolvableType getResolvableType();
    /**
	 * 是否是单例的bean
	 */
    boolean isSingleton();
    /**
	 * 是否是多列的bean
	 */
    boolean isPrototype();
    /**
	 * 对应xml中bean元素的abstract属性，用来指定是否是抽象的
	 */
    boolean isAbstract();
    /**
	 * 返回此bean定义来自的资源的描述 (以便在出现错误时显示上下文)
	 */
    @Nullable
    String getResourceDescription();
    @Nullable
    BeanDefinition getOriginatingBeanDefinition();
}
```

BeanDefinition接口上面还继承了2个接口：

- AttributeAccessor: 属性访问接口
- BeanMetadataElement: 配置源对象的 bean 元数据元素实现的接口
  

**AttributeAccessor 接口**

```java
public interface AttributeAccessor {
    /**
	 * 设置属性 -> 值
	 */
    void setAttribute(String name, @Nullable Object value);
    /**
	 * 获取某个属性对应的值
	 */
    @Nullable
    Object getAttribute(String name);
    /**
	 * 移除某个属性
	 */
    @Nullable
    Object removeAttribute(String name);
    /**
	 * 是否包含某个属性
	 */
    boolean hasAttribute(String name);
    /**
	 * 返回所有的属性名称
	 */
	 String[] attributeNames();
}
```

> 这个接口相当于 key -> value 数据结构的一种操作， BeanDefinition 实现这个接口内部也是使用 LinkedHashMap 来实现这个接口中的所有方法，通常我们通过这些方法保存 BeanDefinition 定义过程中产生的一下附加信息。

**BeanMetadataElement 接口**

```java
public interface BeanMetadataElement {

    /**
	 * 返回这个元数据元素的配置源
	 * (may be {@code null}).
	 */
    @Nullable
    default Object getSource() {
        return null;
    }

}
```

> BeanDefinition 继承这个接口，getSource 返回 BeanDefinition 定义的来源，比如我们通过 xml 定 BeanDefinition 的，此时 getSource 就表示定义 bean 的 xml 资源；若我们通过 api 的方式定义 BeanDefinition，我们可以将 source 设置为定义 BeanDefinition 时所在的类，出错时，可以根据这个来源方便排错。

1. RootBeanDefinition 类：表示根 bean 定义信息
   - 通常 bean 中没有父 bean 的就使用这种表示
2. ChildBeanDefinition 类：表示子 bean 定义信息
   - 如果需要指定父 bean 的，可以使用 ChildBeanDefinition 来定义子 bean 的配置信息，里面有个
     parentName 属性，用来指定父 bean 的名称。
3. GenericBeanDefinition 类：通用的 bean 定义信息
   - 既可以表示没有父 bean 的 bean 配置信息，也可以表示有父 bean 的子 bean 配置信息，这个类里面也有 parentName 属性，用来指定父 bean 的名称。
4. ConfigurationClassBeanDefinition 类：表示通过配置类中 @Bean 方法定义 bean 信息
   - 可以通过配置类中使用 @Bean 来标注一些方法，通过这些方法来定义 bean，这些方法配置的 bean 信息最后会转换为 ConfigurationClassBeanDefinition 类型的对象。
5. AnnotatedBeanDefinition 接口：表示通过注解的方式定义的 bean 信息

​			里面有个方法

```java
AnnotationMetadata getMetadata();
```

用来获取定义这个 bean 的类上的所有注解信息。

### 实验

> BeanDefinitionBuilder：构建BeanDefinition的工具类
>
> Spring 中为了方便操作 BeanDefinition，提供了一个类： BeanDefinitionBuilder ，内部提供了很多
> 静态方法，通过这些方法可以非常方便的组装 BeanDefinition 对象。

#### 普通类

```java
public class Users {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Users{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

```java 测试类
public class BuildBeanTest {
    @Test void buildBeanTest1() {
        BeanDefinitionBuilder builder = BeanDefinitionBuilder
            .rootBeanDefinition(Users.class.getName());
        AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
        System.out.println(beanDefinition);
    }
}
```

```
Root bean: class [com.example.springdemo.bean.Users]; 
scope=; abstract=false; lazyInit=null; 
autowireMode=0; dependencyCheck=0; autowireCandidate=true; 
primary=false; factoryBeanName=null; 
factoryMethodName=null; initMethodName=null; destroyMethodName=null
```



#### 组装一个有属性的 Bean

```java 测试类
@Test 
void buildBeanTest2() {
    BeanDefinitionBuilder builder =
        BeanDefinitionBuilder.rootBeanDefinition(Users.class.getName());
    // 给 user 中 name 赋值
    builder.addPropertyValue("name", "xiaou");
    AbstractBeanDefinition userDefinition = builder.getBeanDefinition();
    System.out.println(userDefinition);
    // 创建一个 Spring 容器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    // 将 userDefinition 这个 bean 配置信息注册到 Spring 容器中, bean 名称指定为 user
    factory.registerBeanDefinition("user", userDefinition);
    // 从 Spring 容器获取 User 名称这个 Bean 然后进行输出
    Users user = factory.getBean("user", Users.class);
    System.out.println(user);
}
```

**输出**

```
Root bean: class [com.example.springdemo.bean.Users]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
Users{name='xiaou'}
```

### 组合依赖 Bean

```java 实体对象
// Users Bean
@ToString
@Data
public class Users {
    private String name;
    private Pet pet;
}
// Pet Bean
@ToString
@Data
public class Pet {
    private String name;
}
```

```java 测试方法
@Test
void buildBeanTest3() {
    BeanDefinition petDefinition = BeanDefinitionBuilder
        .rootBeanDefinition(Pet.class.getName())
        .addPropertyValue("name", "xiaoy")
        .getBeanDefinition();
    BeanDefinition usersDefinition = BeanDefinitionBuilder
        .rootBeanDefinition(Users.class.getName())
        .addPropertyValue("name", "xiaou")
        .addPropertyReference("pet", "pet")
        .getBeanDefinition();
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.registerBeanDefinition("pet", petDefinition);
    factory.registerBeanDefinition("user", usersDefinition);
    System.out.println(factory.getBean("pet"));
    System.out.println(factory.getBean("user"));
}
```

**输出**

```
Pet(name=xiaoy)
Users(name=xiaou, pet=Pet(name=xiaoy))
```

### 设置（Map、Set、List）属性

### 阶段小结

bean 注册者只识别 BeanDefinition 对象，不管什么方式最后都会将这些 bean 定义的信息转换为 BeanDefinition 对象，然后注册到 Spring 容器中。

## Bean 元信息解析阶段

> Bean 元信息的解析就是将各种方式定义的 bean 配置信息解析为 BeanDefinition 对象。

### Bean 元信息的解析主要有 3 种方式

1. xml 文件定义 bean 的解析
   - 使用 XmlBeanDefinitionReader 这个类解析
2. properties 文件定义 bean 的解析
   - 使用 PropertiesBeanDefinitionReader 这个类解析
3. 注解方式定义 bean 的解析
   - 使用 PropertiesBeanDefinitionReader 这个类解析

## Spring Bean注册阶段

> Bean 注册阶段需要用到一个非常重要的接口：**`BeanDefinitionRegistry`**

## BeanDefinitionRegistry 接口

```java
public interface BeanDefinitionRegistry extends AliasRegistry {
    /**
	 * 注册一个新的 bean 定义
	 * beanName：bean 的名称
	 * beanDefinition：bean 定义信息
	 */
    void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException;
    /**
	 * 通过 bean 名称移除已注册的 bean
	 * beanName：bean 名称
	 */
    void removeBeanDefinition(String beanName) throws
        NoSuchBeanDefinitionException;
    /**
	 * 查看 beanName 是否注册过
	 * beanName：bean 名称
	 */
    BeanDefinition getBeanDefinition(String beanName) throws
        NoSuchBeanDefinitionException;
    /**
	 * 查看 beanName 是否注册过
	 */
    boolean containsBeanDefinition(String beanName);
    /**
	 * 获取已经定义 (注册) 的 bean 名称列表
	 */
    String[] getBeanDefinitionNames();
    /**
	 * 返回注册器中已注册的 bean 数量
	 */
    int getBeanDefinitionCount();
    /**
	 * 确定给定的 bean 名称或者别名是否已在此注册表中使用
	 * beanName：可以是 bean 名称或者 bean 的别名
	 */
    boolean isBeanNameInUse(String beanName);
}
```

### AliasRegistry 别名注册器

```java
public interface AliasRegistry {
    /**
	 * 给 name 指定别名 alias
	 */
    void registerAlias(String name, String alias);
    /**
	 * 从此注册表中删除指定的别名
	 */
    void removeAlias(String alias);
    /**
	 * 判断 name 是否作为别名已经被使用了
	 */
    boolean isAlias(String name);
    /**
	 * 返回 name 对应的所有别名
	 */
    String[] getAliases(String name);
}
```

### BeanDefinitionRegistry 唯一实现： DefaultListableBeanFactory

Spring 中 BeanDefinitionRegistry 接口有一个 **唯一** 的实现类：

```java
org.springframework.beans.factory.support.DefaultListableBeanFactory
```

### 实验

```java
@Test
public void test() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    GenericBeanDefinition nameBdf = new GenericBeanDefinition();
    nameBdf.setBeanClass(String.class);
    nameBdf.getConstructorArgumentValues()
        .addIndexedArgumentValue(0, "xiaou");
    // 将 bean 注册到容器中
    factory.registerBeanDefinition("name", nameBdf);
    // 通过名称获取 BeanDefinition
    System.out.println("BeanDefinition: " + factory.getBean("name"));
    // 通过名称判断是否注册过 BeanDefinition
    System.out.println("是否注册过 Bean: " + factory.containsBeanDefinition("name"));
    // 获取所有注册的名称
    System.out.println("所有注册的名称: " + Arrays.asList(factory.getBeanDefinitionNames()));
    // 获取已经注册 BeanDefinition 数量
    System.out.println("已经注册 BeanDefinition 数量: " + factory.getBeanDefinitionCount());
    // 判断 name bean 的名称是否被使用
    System.out.println("name 的名称是否被使用: " + factory.isBeanNameInUse("name"));

    // 注册别名
    factory.registerAlias("name", "alias_name_1");
    factory.registerAlias("name", "alias_name_2");

    // 判断 alias_name_1 别名是否使用
    System.out.println("判断 alias_name_1 别名是否使用: " + factory.isAlias("alias_name_1"));
    // 获取对应名称所有别名
    System.out.println("获取对应名称所有别名: " + Arrays.asList(factory.getAliases("name")));

    // 获取 bean
    System.out.println(factory.getBean("name"));
}
```

**结果**

```
BeanDefinition: xiaou
是否注册过 Bean: true
所有注册的名称: [name]
已经注册 BeanDefinition 数量: 1
name 的名称是否被使用: true
判断 alias_name_1 别名是否使用: true
获取对应名称所有别名: [alias_name_1, alias_name_2]
xiaou
```

getBean 核心方法可以看

```
org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
```

[源码解析链接](https://juejin.cn/post/6844903949753909255)

## BeanDefinition合并阶段

## Bean Class加载阶段

**这个阶段就是将bean的class名称转换为Class类型的对象。**

BeanDefinition中有个Object类型的字段：beanClass

```java
private volatile Object beanClass;
```

用来表示bean的class对象，通常这个字段的值有2种类型，

1. bean 对应的 Class 类型的对象
   - 不需要解析
2. bean 对应的 Class 的完整类名
   - 即这个字段是 bean 的类名的 时候，就需要通过类加载器将其转换为一个 Class 对象。

阶段 4 中合并产生的 RootBeanDefinition 中的 beanClass 进行解析，将 bean 的类名转换为 Class 对象 ，然后赋值给 beanClass 字段。

源码位置:

```java
org.springframework.beans.factory.support.AbstractBeanFactory#resolveBeanClass
```

## Bean实例化阶段

有两个小阶段

1. Bean 实例化前操作
2. Bean 实例化操作

在 `DefaultListableBeanFactory` 中有一个在这个阶段特别重要的字段

```java
private final List<BeanPostProcessor> beanPostProcessors = new CopyOnWriteArrayList<>();
```

这个字段是一个 `BeanPostProcessor` 类型的集合。

`BeanPostProcessor` 是一个接口，还有很多子接口，这些接口中提供了很多方法，Spring 在 Bean 生命
周期的不同阶段，会调用上面这个列表中的 BeanPostProcessor 中的一些方法，来对生命周期进行扩
展。

接口的就有两个方法其方法签名为: 

```java
@Nullable
default Object postProcessBeforeInitialization(Object bean,
                        String beanName) throws BeansException {
    return bean;
}
```

```java
@Nullable
default Object postProcessAfterInitialization(Object bean,
                        String beanName) throws BeansException {
    return bean;
}
```

