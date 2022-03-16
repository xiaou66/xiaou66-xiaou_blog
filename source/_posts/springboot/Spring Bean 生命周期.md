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

### 合并阶段是做什么？

可能我们定义 bean 的时候有父子 bean 关系，此时子 BeanDefinition 中的信息是不完整的。

比如设置属性的时候配置在父 BeanDefinition 中，此时子 BeanDefinition 中是没有这些信息的，需要将子 bean BeanDefinition 和父 bean 的 BeanDefinition 进行合并，得到最终的一个 RootBeanDefinition ，合并之后得到的 RootBeanDefinition 包含 bean 定义的所有信息，包含了从父 bean 中继继承过来的所有信息，后续 bean 的所有创建工作就是依靠合并之后 BeanDefinition 来进行的。

合并 BeanDefinition 会使用下面这个方法：

- `org.springframework.beans.factory.support.AbstractBeanFactory#getMergedBeanDefinition`



## Bean Class 加载阶段

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
2. Bean 实例化后操作

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
// Bean 实例化前操作
@Nullable
default Object postProcessBeforeInitialization(Object bean,
                        String beanName) throws BeansException {
    return bean;
}
```

```java
// Bean 实例化后操作
@Nullable
default Object postProcessAfterInitialization(Object bean,
                        String beanName) throws BeansException {
    return bean;
}
```

BeanPostProcessor 又有很多的子接口来细分不同的功能，从下面的代码片段中可以看出

代码片段出自 `org.springframework.beans.factory.support.AbstractBeanFactory#getBeanPostProcessorCache`

```java 
for (BeanPostProcessor bp : this.beanPostProcessors) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        bpCache.instantiationAware.add((InstantiationAwareBeanPostProcessor) bp);
        if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
            bpCache.smartInstantiationAware.add((SmartInstantiationAwareBeanPostProcessor) bp);
        }
    }
    if (bp instanceof DestructionAwareBeanPostProcessor) {
        bpCache.destructionAware.add((DestructionAwareBeanPostProcessor) bp);
    }
    if (bp instanceof MergedBeanDefinitionPostProcessor) {
        bpCache.mergedDefinition.add((MergedBeanDefinitionPostProcessor) bp);
    }
}
```

### InstantiationAwareBeanPostProcessor

> 通过返回一个代理对象的方式，达到改变目标类类型的目的。在不想改变现有类的逻辑而又想借助现有类实现其他功能，就可以使用这种方式。

**方法**

```java
// 在工厂将给定的属性值应用到给定的 bean 之前，对它们进行后处理，而不需要任何属性描述符。
@Nullable
default PropertyValues postProcessProperties(PropertyValues pvs,
                         Object bean, String beanName) throws BeansException {
    return null;
}
```

```java
// 在 Bean 实例化前调用该方法，返回值可以为代理后的 Bean，以此代替 Bean 默认的实例化过程。
// 返回值不为 null 时，后续只会调用 BeanPostProcessor 的 postProcessAfterInitialization 方法，
// 而不会调用别的后续后置处理方法
@Nullable
default Object postProcessBeforeInstantiation(Class<?> beanClass,
                              String beanName) throws BeansException {
    return null;
}
```

```java
// 当 Bean 通过构造器或者工厂方法被实例化后，当属性还未被赋值前，该方法会被调用，一般用于自定义属性赋值。
// 方法返回值为布尔类型，返回 true 时，表示 Bean 属性需要被赋值；返回 false 表示跳过 Bean 属性赋值，
// 并且 InstantiationAwareBeanPostProcessor 的 postProcessProperties 方法不会被调用
default boolean postProcessAfterInstantiation(Object bean, String beanName) 
    throws BeansException {
    return true;
}
```

### SmartInstantiationAwareBeanPostProcessor

```java
// 预测 Bean 的类型，返回第一个预测成功的 Class 类型，如果不能预测返回 null
default Class<?> predictBeanType(Class<?> beanClass,
                      String beanName) throws BeansException {
    return null;
}
```

```java
// 选择合适的构造器，比如目标对象有多个构造器，在这里可以进行一些定制化，选择合适的构造器
// beanClass 参数表示目标实例的类型，beanName 是目标实例在 Spring 容器中的 name
// 返回值是个构造器数组，如果返回 null，会执行下一个 
// PostProcessor 的 determineCandidateConstructors 方法；否则选取该 PostProcessor 选择的构造器
default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass,
                              String beanName) throws BeansException {
    return null;
}
```

```java
// 获得提前暴露的 bean 引用。主要用于解决循环引用的问题
// 只有单例对象才会调用此方法
default Object getEarlyBeanReference(Object bean,
                          String beanName) throws BeansException {
    return bean;
}
```

### DestructionAwareBeanPostProcessor

```java
// 该方法是 bean 在 Spring 在容器中被销毁之前调用
void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException;
```

```java
// 确定给定的 bean 实例是否需要这个后处理程序销毁。
default boolean requiresDestruction(Object bean) {
    return true;
}
```

### MergedBeanDefinitionPostProcessor

```java
// 为指定的 bean 对给定的合并 bean 定义进行后处理。
void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition,
                             Class<?> beanType, String beanName);
```

```java
// 指定名称的 bean 定义已被重置的通知, 
// 并且该后处理程序应清除受影响 bean 的任何元数据。
default void resetBeanDefinition(String beanName) {}
```

### 实验

> 自定义一个注解，当构造器被这个注解标注的时候，让 Spring 自动选择使用这个构造器创建对象。

因为涉及到构造器所以使用的是 `SmartInstantiationAwareBeanPostProcessor` 接口。

```java 注解
@Target(ElementType.CONSTRUCTOR)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAutowired { }
```

```java
public class MySmartInstantiationAwareBeanPostProcessor
        implements SmartInstantiationAwareBeanPostProcessor {
    @Override
    public Constructor<?>[] determineCandidateConstructors(Class<?> beanClass,
                                    String beanName) throws BeansException {
        System.out.println(beanClass);
        System.out.println("调用 MySmartInstantiationAwareBeanPostProcessor" +
                ".determineCandidateConstructors 方法");
        Constructor<?>[] declaredConstructors = beanClass.getDeclaredConstructors();
        Constructor<?>[] constructors = Arrays.stream(declaredConstructors)
                .filter(constructor -> constructor.isAnnotationPresent(MyAutowired.class))
                .toArray(Constructor[]::new);
        return constructors.length != 0 ? constructors : null;
    }
}
```

```java 测试方法
@Test
public void test1() {
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        // 创建一个 SmartInstantiationAwareBeanPostProcessor, 将其添加到容器中
      factory.addBeanPostProcessor(new
              MySmartInstantiationAwareBeanPostProcessor());
      factory.registerBeanDefinition("name",
              BeanDefinitionBuilder.
                        genericBeanDefinition(String.class).
                        addConstructorArgValue("xiaou").
                        getBeanDefinition());
      factory.registerBeanDefinition("age",
              BeanDefinitionBuilder.
                        genericBeanDefinition(Integer.class).
                        addConstructorArgValue(30).
                        getBeanDefinition());
      factory.registerBeanDefinition("person",
              BeanDefinitionBuilder.
                        genericBeanDefinition(Person.class).
                        getBeanDefinition());
      Person person = factory.getBean("person", Person.class);
      System.out.println(person);
}
```

**输出**

```
class com.example.springdemo.bean.Person
调用 MySmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors 方法
class java.lang.String
调用 MySmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors 方法
调用了 Person(String name)
Person(name=xiaou, age=null)
```

## 合并后的 BeanDefinition 处理

postProcessMergedBeanDefinition 有 2 个实现类。

1. 在 postProcessMergedBeanDefinition 方法中对 @Autowired、@Value 标注的方法、字段进行缓存
   org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
2. 在 postProcessMergedBeanDefinition 方法中对 @Resource 标注的字段、@Resource 标注的方
   法、 @PostConstruct 标注的字段、 @PreDestroy 标注的方法进行缓存

## Bean 属性设置阶段

属性设置阶段分为 3 个子阶段

1. 实例化后阶段
2. Bean 属性赋值前处理
3. Bean 属性赋值

### 实例化后阶段

```java
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        InstantiationAwareBeanPostProcessor ibp =
            (InstantiationAwareBeanPostProcessor) bp;
        if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(),
                                               beanName)) {
            return;
        }
    }
}
```

> postProcessAfterInstantiation 方法返回 false 的时候，后续的 Bean 属性赋值前处理、Bean 属性赋值都会被跳过了。

方法签名

```java
default boolean postProcessAfterInstantiation(Object bean,
                String beanName) throws BeansException {
    return true;
}
```

#### 实验

阻止 user1 被赋值

```java 测试类
@Test
void test() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.addBeanPostProcessor(new InstantiationAwareBeanPostProcessor() {
        @Override
        public boolean postProcessAfterInstantiation(Object bean,
                                      String beanName) throws BeansException {
            return !"user1".equals(beanName);
        }
    });
    factory.registerBeanDefinition("user1", BeanDefinitionBuilder.
                                   genericBeanDefinition(Users.class).
                                   addPropertyValue("name", "xiaou").
                                   getBeanDefinition());
    factory.registerBeanDefinition("user2", BeanDefinitionBuilder.
                                   genericBeanDefinition(Users.class).
                                   addPropertyValue("name", "xiaoy").
                                   getBeanDefinition());
    for (String beanName : factory.getBeanDefinitionNames()) {
        System.out.println(String.format("%s->%s", beanName,
                                         factory.getBean(beanName)));
    }
}
```

**输出**

```
user1->Users(name=null)
user2->Users(name=xiaoy)
```

### Bean 属性赋值前处理

> 这个阶段会调用 InstantiationAwareBeanPostProcessor 接口的 postProcessProperties 方法

```java
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        InstantiationAwareBeanPostProcessor ibp =
            (InstantiationAwareBeanPostProcessor) bp;
        PropertyValues pvsToUse = ibp.postProcessProperties(pvs,
                                         bw.getWrappedInstance(), beanName);
        if (pvsToUse == null) {
            if (filteredPds == null) {
                filteredPds = filterPropertyDescriptorsForDependencyCheck(bw,
                                         mbd.allowCaching);
            }
            pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds,
                                         bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                return;
            }
        }
        pvs = pvsToUse;
    }
}
```

> 从上面可以看出，如果 InstantiationAwareBeanPostProcessor 中的 postProcessProperties 和 postProcessPropertyValues 都返回空的时候，表示这个 bean 不 需要设置属性，直接返回了，
> 直接进入下一个阶段。

这个方法 2 个比较重要的实现类

1. AutowiredAnnotationBeanPostProcessor 在这个方法中对 @Autowired、@Value 标注的字段、方法注入值。
2. CommonAnnotationBeanPostProcessor 在这个方法中对 @Resource 标注的字段和方法注入值。

### Bean 属性赋值阶段

这个过程比较简单了，循环处理 PropertyValues 中的属性值信息，通过反射调用 set 方法将属性的值设 置到 bean 实例中。 

PropertyValues 中的值是通过 bean.xml 中 property 元素配置的，或者调用 MutablePropertyValues 中 add 方法设置的值。

## Bean初始化阶段

这个阶段分成 5 个小阶段:

1. Bean Aware 接口回调
2. Bean 初始化前操作
3. Bean 初始化操作
4. Bean 初始化后操作
5. Bean 初始化完成操作

### Bean Aware 接口回调

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanNameAware) {
            ((BeanNameAware) bean).setBeanName(beanName);
        }
        if (bean instanceof BeanClassLoaderAware) {
            ClassLoader bcl = getBeanClassLoader();
            if (bcl != null) {
                ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
            }
        }
        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware)
             bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        }
    }
}
```

bean 实例实现了上面的接口，会按照下面的顺序依次进行调用：

1. BeanNameAware：将bean的名称注入进去
2. BeanClassLoaderAware：将BeanClassLoader注入进去
3. BeanFactoryAware：将BeanFactory注入进去

### Bean 初始化前操作

```java
@Override
public Object applyBeanPostProcessorsBeforeInitialization(
    Object existingBean, String beanName) throws BeansException {
    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

会调用 BeanPostProcessor 的 postProcessBeforeInitialization 方法，若返回 null，当前方法将结束。

**通常称 postProcessBeforeInitialization 这个方法为：bean 初始化前操作。**

这个接口有两个实现类

1. org.springframework.context.support.ApplicationContextAwareProcessor
2. org.springframework.context.annotation.CommonAnnotationBeanPostProcessor

#### ApplicationContextAwareProcessor 注入 6 个 Aware 接口对象

如果 bean 实现了下面的接口，在 `ApplicationContextAwareProcessor#postProcessBeforeInitialization` 中会依次调用下面接口中的方法，将 Aware 前缀对应的对象注入到 bean 实例中

1. EnvironmentAware：注入 Environment 对象 
2. EmbeddedValueResolverAware：注入 EmbeddedValueResolver 对象 
3. ResourceLoaderAware：注入 ResourceLoader 对象 
4. ApplicationEventPublisherAware：注入 ApplicationEventPublisher 对象 
5. MessageSourceAware：注入 MessageSource 对象 
6. ApplicationContextAware：注入 ApplicationContext 对象

看出这个类以 ApplicationContext 开头的，说明这个类只能在 ApplicationContext 环境中使用。

#### 实验

```java
public class Bean implements EnvironmentAware, EmbeddedValueResolverAware,
        ResourceLoaderAware, ApplicationEventPublisherAware,
        MessageSourceAware, ApplicationContextAware {
    @PostConstruct
    public void postConstruct1() {
        System.out.println("postConstruct1");
    }
    @PostConstruct
    public void postConstruct2() {
        System.out.println("postConstruct2");
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("setApplicationContext:" + applicationContext);
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        System.out.println("setApplicationEventPublisher:" + applicationEventPublisher);
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        System.out.println("setEmbeddedValueResolver:" + resolver);
    }

    @Override
    public void setEnvironment(Environment environment) {
        System.out.println("setEnvironment:" + environment.getClass());
    }

    @Override
    public void setMessageSource(MessageSource messageSource) {
        System.out.println("setMessageSource:" + messageSource);
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        System.out.println("setResourceLoader:" + resourceLoader);
    }
}
```

```java 测试方法
@Test
void test1() {
    AnnotationConfigApplicationContext context
        = new AnnotationConfigApplicationContext();
    context.register(Bean.class);
    context.refresh();
}
```

**输出**

```
setEnvironment:class org.springframework.core.env.StandardEnvironment
setEmbeddedValueResolver:org.springframework.beans.factory.config.EmbeddedValueResolver@175b9425
setResourceLoader:org.springframework.context.annotation.AnnotationConfigApplicationContext@475e586c, started on Tue Mar 15 13:11:39 CST 2022
setApplicationEventPublisher:org.springframework.context.annotation.AnnotationConfigApplicationContext@475e586c, started on Tue Mar 15 13:11:39 CST 2022
setMessageSource:org.springframework.context.annotation.AnnotationConfigApplicationContext@475e586c, started on Tue Mar 15 13:11:39 CST 2022
setApplicationContext:org.springframework.context.annotation.AnnotationConfigApplicationContext@475e586c, started on Tue Mar 15 13:11:39 CST 2022
postConstruct2
postConstruct1
```

### Bean 初始化阶段

这个阶段的有两个步骤：

1. 调用 InitializingBean 接口的 afterPropertiesSet 方法。
2. 调用定义 bean 的时候指定的初始化方法。

InitializingBean 接口方法

```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```

> 当我们的 bean 实现了这个接口的时候，会在这个阶段被调用。

指定 bean 的初始化方法，3 种方式

1. xml 文件指定初始化方法

```xml
<bean init-method="bean中方法名称"/>
```

2. @Bean 的方式指定初始化方法

```java
@Bean(initMethod = "初始化的方法")
```

3. API 的方式指定初始化方法

 ```java
 this.beanDefinition.setInitMethodName(methodName)
 ```

初始化的方法最终会赋值给下面这个字段

`org.springframework.beans.factory.support.AbstractBeanDefinition#initMethodName`

#### 实验

```java
public class InitBean implements InitializingBean {
    public void init() {
        System.out.println("调用 init() 方法");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("调用 afterPropertiesSet() 方法");
    }
}
```

```java 测试类
public class BeanInitTest {
    @Test
    void test1() {
        AnnotationConfigApplicationContext context
            = new AnnotationConfigApplicationContext();
        context.register(Bean.class);
        context.refresh();
    }
    @Test
    void initMethodTest() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        BeanDefinition initBean = BeanDefinitionBuilder.genericBeanDefinition(InitBean.class)
            .setInitMethodName("init")
            .getBeanDefinition();
        factory.registerBeanDefinition("initBean", initBean);
        System.out.println(factory.getBean("initBean"));
    }
}
```

**输出**

```java
调用 afterPropertiesSet() 方法
调用 init() 方法
com.example.springdemo.life.aware.InitBean@49b0b76
```

### Bean 初始化后阶段

> 调用 BeanPostProcessor 接口的 postProcessAfterInitialization 方法 ，返回 null 的时候，会中断上面的操作。

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean,
                                      String beanName)throws BeansException {
    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

> 通常将 BeanPostProcessor 称为 Bean 初始化后置操作 

#### 实验

```java
@Test
void postProcessAfterInitializationTest() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.addBeanPostProcessor(new BeanPostProcessor() {
        @Override
        public Object postProcessAfterInitialization(Object bean,
                                     String beanName) throws BeansException {
            System.out.println("postProcessAfterInitialization: " + beanName);
            return bean;
        }
    });
    factory.registerBeanDefinition("name",
                                   BeanDefinitionBuilder.
                                   genericBeanDefinition(String.class)
                                   .addConstructorArgValue("xiaou")
                                   .getBeanDefinition());
    factory.registerBeanDefinition("personInformation",
                                   BeanDefinitionBuilder.genericBeanDefinition(String.class)
                                   .addConstructorArgValue("xiaouxiaouxiaou").
                                  .getBeanDefinition());
    for (String beanName : factory.getBeanDefinitionNames()) {
        System.out.println(String.format("%s->%s", beanName,
                                         factory.getBean(beanName)));
    }
}
```

**输出**

```
postProcessAfterInitialization: name
name->xiaou
postProcessAfterInitialization: personInformation
personInformation->xiaouxiaouxiaou
```

## 所有单例 bean 初始化完成后阶段

所有单例 bean 实例化完成之后，Spring 会回调下面这个接口：

```java
public interface SmartInitializingSingleton {
    void afterSingletonsInstantiated();
}
```

确保所有非 lazy 的单例都被实例化，同时考虑到 FactoryBeans。如果需要，通常在工厂设置结束时调用。

`org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons`

### 实验

```java
@Test 
void test3() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        factory.registerBeanDefinition("mySmartInitializingSingleton",
                BeanDefinitionBuilder.genericBeanDefinition(MySmartInitializingSingleton.class)
                        .getBeanDefinition());
        factory.registerBeanDefinition("user",
                BeanDefinitionBuilder.genericBeanDefinition(Users.class)
                        .addPropertyValue("name", "xiaou")
                        .getBeanDefinition());
        System.out.println("准备触发所有单例 bean 初始化");
        factory.preInstantiateSingletons();
 }
```

**输出**

```
准备触发所有单例 bean 初始化
所有的 Bean 初始化完成
```

## Bean 使用阶段

> 调用 getBean 方法得到了 bean 之后

## Bean 销毁阶段

### 触发 bean 销毁的几种方式

1. 调用 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#destroyBean`
2. 调用 `org.springframework.beans.factory.config.ConfigurableBeanFactory#destroySingletons`
3. 调用 ApplicationContext 中的 close 方法

### Bean 销毁阶段会依次执行

1.  轮询 beanPostProcessors 列表，如果是 DestructionAwareBeanPostProcessor 这种类型的，会调用其内部的 postProcessBeforeDestruction 方法。
2. 如果 bean 实现了 `org.springframework.beans.factory.DisposableBean` 接口，会调用这个接口中的 destroy 方法。
3. 调用 bean 自定义的销毁方法

### DestructionAwareBeanPostProcessor 接口

```java
public interface DestructionAwareBeanPostProcessor extends BeanPostProcessor {
    /**
     * bean销毁前调用的方法
     */
    void postProcessBeforeDestruction(Object bean, String beanName) throws
        BeansException;
    /**
     * 用来判断bean是否需要触发postProcessBeforeDestruction方法
     */
    default boolean requiresDestruction(Object bean) {
        return true;
    }
}
```

### 自定义销毁方法有 3 种方式

1. XML 中指定摧毁方法

```xml
<bean destroy-method="bean中方法名称"/>
```

2. @Bean 中指定摧毁方法

```java
@Bean(destroyMethod = "初始化的方法")
```

3. API  方式指定

```java
this.beanDefinition.setDestroyMethodName(methodName);
```

### 实验

#### 自定义 DestructionAwareBeanPostProcessor

```java
public class MyDestructionAwareBeanPostProcessor
        implements DestructionAwareBeanPostProcessor {
    @Override
    public void postProcessBeforeDestruction(Object bean,
                               String beanName) throws BeansException {
        System.out.println("准备销毁 >> " + beanName);
    }
}
```

```java 测试方法
@Test
void destructionAwareBeanPostProcessorTest() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.addBeanPostProcessor(new MyDestructionAwareBeanPostProcessor());
    factory.registerBeanDefinition("person1",
                 BeanDefinitionBuilder.genericBeanDefinition(Person.class)
                                   .getBeanDefinition());
    factory.registerBeanDefinition("person2",
                 BeanDefinitionBuilder.genericBeanDefinition(Person.class)
                                   .getBeanDefinition());
    factory.registerBeanDefinition("person3",
                   BeanDefinitionBuilder.genericBeanDefinition(Person.class)
                                   .getBeanDefinition());
    // 触发所有单例 bean 初始化
    factory.preInstantiateSingletons();
    // 销毁指定的 bean
    factory.destroySingleton("person2");
    // 销毁所有的单列 bean
    factory.destroySingletons();
}
```

**输出**

```
调用了 Person()
调用了 Person()
调用了 Person()
准备销毁 >> person2
准备销毁 >> person3
准备销毁 >> person1
```

#### 触发 @PreDestroy 标注的方法被调用

这个注解是在 `CommonAnnotationBeanPostProcessor#postProcessBeforeDestruction` 中被处理的，所以只需要将这个加入 BeanPostProcessor 列表就可以了。

在 bean 在需要销毁前执行 PreDestroy 的方法上添加 @PreDestroy  注解。

```java
public class Person {
    private String name;
    private Integer age;
    public Person() {
        System.out.println("调用了 Person()");
    }
    @PreDestroy
    public void preDestroy() {
        System.out.println("reDestroy()");
    }
}
```

```java 测试方法
@Test
void destructionAwareBeanPostProcessorTest2() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.addBeanPostProcessor(new CommonAnnotationBeanPostProcessor());
    factory.registerBeanDefinition("person1",
                  BeanDefinitionBuilder.genericBeanDefinition(Person.class)
                             .getBeanDefinition());
    // 销毁所有的单列 bean
    factory.destroySingletons();
}
```

#### 销毁阶段的执行顺序

实际上 ApplicationContext 内部已经将 spring 内部一些常见的必须的 BeannPostProcessor 自动装配到
beanPostProcessors 列表中 ，比如我们熟悉的下面的几个：

1. `org.springframework.context.annotation.CommonAnnotationBeanPostProcessor`
   - 用来处理 @Resource、@PostConstruct、@PreDestroy
2. `org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor`
   - 用来处理 @Autowired、@Value 注解
3. `org.springframework.context.support.ApplicationContextAwareProcessor`
   - 用来回调 Bean 实现的各种 Aware 接口

 所以通过 ApplicationContext 来销毁 bean，会触发 3 中方式的执行

```java
public class Car implements DisposableBean {
    private String name;

    public Car() {
        System.out.println("使用构造方法: Car()");
    }
    @PreDestroy
    public void preDestroy1() {
        System.out.println("执行: preDestroy1()");
    }
    @PreDestroy
    public void preDestroy2() {
        System.out.println("执行: preDestroy2()");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean 接口中的 destroy() 方法");
    }
    public void customDestroyMethod() {
        System.out.println("自定义的摧毁方法: customDestroyMethod() 方法");
    }
}
```

```java  测试类
public class DestructionBeanTest {
    @Bean(destroyMethod = "customDestroyMethod")
    public Car car() {
        return new Car();
    }
    @Test
    public void destroyBeanTest() {
        AnnotationConfigApplicationContext context = 
            new AnnotationConfigApplicationContext();
        
        context.register(DestructionBeanTest.class);
        System.out.println("准备启动容器");
        context.refresh();
        System.out.println("容器启动完毕");
        
        System.out.println("serviceA：" + context.getBean(Car.class));
        
        System.out.println("准备关闭容器");
        context.close();
        System.out.println("容器关闭完毕");
    }
}
```

**输出**

```
准备启动容器
使用构造方法: Car()
容器启动完毕
serviceA：com.example.springdemo.bean.Car@69997e9d
准备关闭容器
执行: preDestroy2()
执行: preDestroy1()
DisposableBean 接口中的 destroy() 方法
自定义的摧毁方法: customDestroyMethod() 方法
容器关闭完毕
```

可以看出销毁方法调用的顺序：

1. @PreDestroy 标注的所有方法
2. DisposableBean 接口中的 Destroy()
3. 自定义的销毁方法

Bean 生命周期完整流程图

![Bean生命周期流程图](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1647397416697Bean生命周期流程图.png)
