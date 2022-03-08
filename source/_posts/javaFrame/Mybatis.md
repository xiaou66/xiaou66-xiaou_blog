---
title: Mybaits
date: 2021/12/26 15:49:16
math: true
categories:
  - [javaFrame]
tags:
  - [java]
---
# MyBaits 架构

[官方文档](https://mybatis.org/mybatis-3/zh/getting-started.html)

## MyBatis 架构

Mybatis的功能架构分为三层：

![image-20211210210241172](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646712808315image-20211210210241172.png)

### 接口层

首先接口层是我们打交道最多的。核心对象是 SqlSession，它是上层应用和 MyBatis 打交道的桥梁，SqlSession 上定义了非常多的对数据库的操作方法。接口层在接收到调用请求的时候，会调用核心处理层的相应模块来完成具体的数据库操作

### 核心处理层

接下来是核心处理层。既然叫核心处理层，也就是跟数据库操作相关的动作都是在这一层完成的。

核心处理层主要做了这几件事：

1. 把接口中传入的参数解析并且映射成 JDBC 类型；
2. 解析 xml 文件中的 SQL 语句，包括插入参数，和动态 SQL 的生成；
3. 执行 SQL 语句；
4. 处理结果集，并映射成 Java 对象。

插件也属于核心层，这是由它的工作方式和拦截的对象决定的。

### 基础支持层

最后一个就是基础支持层。基础支持层主要是一些抽取出来的通用的功能（实现复用），用来支持核心处理层的功能。比如数据源、缓存、日志、xml 解析、反射、IO、事务等等这些功能。

## 核心成员类

1. **Configuration**：MyBatis 所有的配置信息都保存在 Configuration 对象之中，配置文件中的大部分配置都会存储到该类中
2. **SqlSession**：作为 MyBatis 工作的主要顶层 API，表示和数据库 交互时的会话，完成必要数据库增删改查功能
3. **Executor**：MyBatis 执行器，是 MyBatis 调度的核心，负责 SQL 语句的生成和查询缓存的维护
4. **StatementHandler**：封装了 JDBC Statement 操作，负责对 JDBC statement 的操作，如设置参数等
5. **ParameterHandler**：负责对用户传递的参数转换成 JDBC Statement 所对应的数据类型
6. **ResultSetHandler**：负责将 JDBC 返回的 ResultSet 结果集对象转换成 List 类型的集合
7. **TypeHandler**：负责 java 数据类型和 jdbc 数据类型 (也可以说是 数据表列类型) 之间的映射和转换
8. **MappedStatement**：MappedStatement 维护一条节点的封装
9. **SqlSource**：负责根据用户传递的 parameterObject，动态地生 成 SQL 语句，将信息封装到 BoundSql 对象中，并返回
10. **BoundSql**：表示动态生成的 SQL 语句以及相应的参数信息

![Mybatis-层次结构](D:\code\xiaou_blog\source\_posts\designMode\image\Mybatis-层次结构.png)

## SqlSessionFactory 构建过程
1. SqlSessionFactoryBuilder 75 行

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
        // 更新当前环境初始化 XMLConfigBuilder 
        // typeAliasRegistry,typeHandlerRegistry 如果有配置页在这里解析对应类 BaseBuilder:39L
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        // 构造 Configuration 对象
        return build(parser.parse());
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
        ErrorContext.instance().reset();
        try {
            inputStream.close();
        } catch (IOException e) {
            // Intentionally ignore. Prefer previous error.
        }
    }
}
public SqlSessionFactory build(Configuration config) {
    // 更新解析出现的 Configuration 配置 DefaultSqlSessionFactory 对象
    return new DefaultSqlSessionFactory(config);
}
```

2. parser.parse() 方法

```java
   public Configuration parse() {
       if (parsed) {
           throw new BuilderException("Each XMLConfigBuilder can only be used once.");
       }
       parsed = true;
       // 从 xml 根结点 configuration 开始解析
       parseConfiguration(parser.evalNode("/configuration"));
       return configuration;
   }
   
   private void parseConfiguration(XNode root) {
       try {
           // properties 为了定义变量所以是最优先解析
           // 在 xml 中 properties 也是需要放在所有标签的最前面
           propertiesElement(root.evalNode("properties"));
           Properties settings = settingsAsProperties(root.evalNode("settings"));
           // 是获取Vitual File System 的自定义实现类
           // 要读取本地文件，或者 FTP 远程文件的时候，就可以用到自定义的 VFS 类
           loadCustomVfs(settings);
           // 获取日志的实现类
           // name 固定值为 logImpl
           // Class<? extends Log> logImpl = resolveClass(props.getProperty("logImpl"));
           loadCustomLogImpl(settings);
           typeAliasesElement(root.evalNode("typeAliases"));
           pluginElement(root.evalNode("plugins"));
           objectFactoryElement(root.evalNode("objectFactory"));
           objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
           reflectorFactoryElement(root.evalNode("reflectorFactory"));
           // 将所有的设置设置到 configuration 对象上
           // 配置具体查看 https://mybatis.org/mybatis-3/zh/configuration.html#settings
           settingsElement(settings);
           // read it after objectFactory and objectWrapperFactory issue #631
           environmentsElement(root.evalNode("environments"));
           // 会根据当前配置环境读取数据源
           //String databaseId = databaseIdProvider.
           //                      getDatabaseId(environment.getDataSource());
           databaseIdProviderElement(root.evalNode("databaseIdProvider"));
           typeHandlerElement(root.evalNode("typeHandlers"));
           // mapper
           mapperElement(root.evalNode("mappers"));
       } catch (Exception e) {
          throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
       }
   }
```

3. 详细看 mapperElement 方法核心代码

```java
// https://mybatis.org/mybatis-3/zh/configuration.html#mappers
if ("package".equals(child.getName())) {
    String mapperPackage = child.getStringAttribute("name");
    configuration.addMappers(mapperPackage);
} else {
    String resource = child.getStringAttribute("resource");
    String url = child.getStringAttribute("url");
    String mapperClass = child.getStringAttribute("class");
    if (resource != null && url == null && mapperClass == null) {
        ErrorContext.instance().resource(resource);
        try(InputStream inputStream = Resources.getResourceAsStream(resource)) {
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            // 解析
            mapperParser.parse();
        }
    } else if (resource == null && url != null && mapperClass == null) {
        ErrorContext.instance().resource(url);
        try(InputStream inputStream = Resources.getUrlAsStream(url)){
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
        }
    } else if (resource == null && url == null && mapperClass != null) {
        Class<?> mapperInterface = Resources.classForName(mapperClass);
        configuration.addMapper(mapperInterface);
    } else {
        throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
    }
}
```

4. 解析什么

```java
public void parse() {
    // 是否加载过了
    if (!configuration.isResourceLoaded(resource)) {
        // 获取 mapper xml 配置信息
        // https://mybatis.org/mybatis-3/zh/sqlmap-xml.html
        configurationElement(parser.evalNode("/mapper"));
        // 标志已经加载过
        configuration.addLoadedResource(resource);
        // 绑定命名空间和对应接口
        bindMapperForNamespace();
    }
    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
}
```

```java bindMapperForNamespace()
private void bindMapperForNamespace() {
    String namespace = builderAssistant.getCurrentNamespace();
    if (namespace != null) {
        Class<?> boundType = null;
        try {
            // 这里通过命名空间加载接口
            // 这里说明了如果有对应接口那么命名空间填全类名
            // Mybatis 会给我们将 *mapper.xml 和接口进行关联
            boundType = Resources.classForName(namespace);
        } catch (ClassNotFoundException e) {
            // ignore, bound type is not required
        }
        if (boundType != null && !configuration.hasMapper(boundType)) {
            // Spring may not know the real resource name so we set a flag
            // to prevent loading again this resource from the mapper interface
            // look at MapperAnnotationBuilder#loadXmlResource
            configuration.addLoadedResource("namespace:" + namespace);
            configuration.addMapper(boundType);
        }
    }
}
```

## 开启会话源码 sessionFactory.openSession()

1. DefaultSqlSessionFactory: 91L

```java
// 执行类型, 事物等级, 是否自动提交
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
        // 获取当前的环境
        final Environment environment = configuration.getEnvironment();
        final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        // 创建执行器, 事物和类型
        final Executor executor = configuration.newExecutor(tx, execType);
        return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
        closeTransaction(tx); // may have fetched a connection so lets call close()
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

```java newExecutor()
// Configuration.java:667L
// https://mybatis.org/mybatis-3/zh/java-api.html
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        // 这里会将很多对象进行初始化或者进行配置
        executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
        // 如果 cacheEnabled 为 true，表示开启缓存机制，缓存的实现类为 CachingExecutor，
        // 这里使用了经典的装饰模式，处理了缓存的相关逻辑后，委托给的具体的 Executor 执行。
        // public CachingExecutor(Executor delegate) {
        //  this.delegate = delegate;
        //  delegate.setExecutorWrapper(this);
        //}
        executor = new CachingExecutor(executor);
    }
    // mybaits 插件增强
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```

1. `ExecutorType.SIMPLE`：该类型的执行器没有特别的行为。它为每个语句的执行创建一个新的预处理语句。
2. `ExecutorType.REUSE`：该类型的执行器会复用预处理语句。
3. `ExecutorType.BATCH`：该类型的执行器会批量执行所有更新语句，如果 SELECT 在多个更新中间执行，将在必要时将多条更新语句分隔开来，以方便理解。

SimpleExecutor 初始化

```java
this.transaction = transaction;
this.deferredLoads = new ConcurrentLinkedQueue<>();
this.localCache = new PerpetualCache("LocalCache");
this.localOutputParameterCache = new PerpetualCache("LocalOutputParameterCache");
this.closed = false;
this.configuration = configuration;
this.wrapper = this;
```

### 总结

> ​	**openSession()** 主要做了 创建了一个 **DefaultSqlSession** 对象，同时根据指定的类型创建  **Executor** 并有可能做了 **缓存** 和 **插件的增强**

## 对一次 SQL 执行的源码阅读

###  测试代码

```java
@Test
public void test1() {
    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
    // 这个对象是核心，一个工程数据库相关的操作都是围绕 SqlSessionFactory
    final SqlSessionFactory sessionFactory = builder.build(
        Thread.currentThread().getContextClassLoader()
        .getResourceAsStream("mybatis-config.xml"));
    try (SqlSession sqlSession = sessionFactory.openSession()){
        //
        final List<Object> objects = sqlSession.selectList("mapper.UserMapper.select", 1);
        objects.forEach(System.out::println);
    }
}
```

### 开始查询流程

1. 在 DefaultSqlSession.css 150 行

```java DefaultSqlSession.css-150L
private <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler) {
    try {
        // 获取 MappedStatement
        // MappedStatement 包含了当前指定方法在配置类中的清空
        MappedStatement ms = configuration.getMappedStatement(statement);
        // 调用下一层查询
        return executor.query(ms, wrapCollection(parameter), rowBounds, handler);
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

 ![image-20211210224606007](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646712819317image-20211210224606007.png)

 ![image-20211210224632416](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646712824314image-20211210224632416.png)

2. 在 query 方法 BaseExecutor 在 151 行

```java
try {
  //进入查询栈 防止递归查询重复处理缓存
  queryStack++;
  // 判断是否有一级缓存
  // ResultHandler 和 ResultSetHandler 的区别
  list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
  if (list != null) {
    handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
  } else {
    // 去数据库查询数据
    list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
  }
} finally {
  // 退出查询栈
  queryStack--;
}
if (queryStack == 0) {
  for (DeferredLoad deferredLoad : deferredLoads) {
    deferredLoad.load();
  }
  // issue #601
  deferredLoads.clear();
  if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
    // issue #482
    clearLocalCache();
  }
}
```

3. 在 queryFromDatabase 方法 BaseExecutor 在 321 行

```java BaseExecutor.java
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    // 先在一级缓存把 key 占用
    localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
        // 查询
        list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
        // 删除占用的 key
        localCache.removeObject(key);
    }
    // 将 key 和查询结果放入一级缓存
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
        localOutputParameterCache.putObject(key, parameter);
    }
    return list;
}
```

4. 在doQuery 方法 SimpleExecutor 在 60 行

```java
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
        // 获取配置项
        Configuration configuration = ms.getConfiguration();
        // 获取处理程序
        StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
        // 处理
        stmt = prepareStatement(handler, ms.getStatementLog());
        // 执行SQL并处理结果集
        return handler.query(stmt, resultHandler);
    } finally {
        closeStatement(stmt);
    }
}
```

5. 在 newStatementHandler 方法

```java
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    // 更新当前statementType 类型获取
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    // 使用拦截器进行包装
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
}
```

```java StatementType.java
public enum StatementType {
  STATEMENT, PREPARED, CALLABLE
}
```

> STATEMENT: 直接操作 sql，不进行预编译，获取数据：$—Statement
>
> PREPARED: 预处理，参数，进行预编译，获取数据：#—–PreparedStatement:默认
>
> CALLABLE: 执行存储过程————CallableStatement

6. doQuery 方法 中调用prepareStatement  方法

```java
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    // 获取数据库连接
    Connection connection = getConnection(statementLog);
    // 封装 Statement 对象
    stmt = handler.prepare(connection, transaction.getTimeout());
    // 确定参数
    handler.parameterize(stmt);
    return stmt;
}
```

7. 参数实际方法 DefaultParameterHandler.java 中的 66 行

```java
public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
        for (int i = 0; i < parameterMappings.size(); i++) {
            ParameterMapping parameterMapping = parameterMappings.get(i);
            if (parameterMapping.getMode() != ParameterMode.OUT) {
                Object value;
                String propertyName = parameterMapping.getProperty();
                if (boundSql.hasAdditionalParameter(propertyName)) { 
                    value = boundSql.getAdditionalParameter(propertyName);
                } else if (parameterObject == null) {
                    value = null;
                } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                    value = parameterObject;
                } else {
                    MetaObject metaObject = configuration.newMetaObject(parameterObject);
                    value = metaObject.getValue(propertyName);
                }
                TypeHandler typeHandler = parameterMapping.getTypeHandler();
                JdbcType jdbcType = parameterMapping.getJdbcType();
                if (value == null && jdbcType == null) {
                    jdbcType = configuration.getJdbcTypeForNull();
                }
                try {
                    typeHandler.setParameter(ps, i + 1, value, jdbcType);
                } catch (TypeException | SQLException e) {
                    throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
                }
            }
        }
    }
}
```

这个方法执行后方法的 SQL 中的参数已经确定下来了然后就返回到第「4」步执行SQL并封装结果集返回

![image-20211211122049621](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646712830313image-20211211122049621.png)

# Mybaits 缓存

## 一级缓存

在应用运行过程中，我们有可能在一次数据库会话中，执行多次查询条件完全相同的 SQL，MyBatis 提供了一级缓存的方案优化这部分场景，如果是相同的 SQL 语句，会优先命中一级缓存，避免直接对数据库进行查询，提高性能。

![image-20220308121515113](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646715734799image-20220308121515113.png)

每个 SqlSession 中持有了 Executor，每个 Executor 中有一个 LocalCache。当用户发起查询时，MyBatis 根据当前执行的语句生成 MappedStatement，在 Local Cache 进行查询，如果缓存命中的话，直接返回结果给用户，如果缓存没有命中的话，查询数据库，结果写入 Local Cache，最后返回结果给用户。

![image-20220308134057361](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646718080830image-20220308134057361.png)

### 配置

我们来看看如何使用 MyBatis 一级缓存。开发者只需在 MyBatis 的配置文件中，添加如下语句，就可以使用一级缓存。共有两个选项，SESSION 或者 STATEMENT，默认是 `SESSION` 级别，即在一个 MyBatis 会话中执行的所有语句，都会共享这一个缓存。一种是 `STATEMENT` 级别，可以理解为缓存只对当前执行的这一个 Statement 有效。

```java
@Test
public void sessionTest() {
    SqlSession sqlSession = factory.openSession(false);
    UsersMapper mapper = sqlSession.getMapper(UsersMapper.class);
    System.out.println(mapper.getAllUser());
    System.out.println(mapper.getAllUser());
    sqlSession.close();
}
```

```yaml
mybatis:
    local-cache-scope: session
```

> 可以发现一级缓存在当前的 session 中有效，两条查询语句实际就执行了一条

![image-20220308135429324](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646719253335image-20220308135429324.png)

在同一个 session 中 如果对数据发生了修改的操作，一级缓存会失效

```java
@Test
public void sessionTest2() {
    SqlSession sqlSession = factory.openSession(false);
    UsersMapper mapper = sqlSession.getMapper(UsersMapper.class);
    System.out.println(mapper.getAllUser());
    mapper.insertUser(new Users(0, UUID.randomUUID().toString().replace("-", "")));
    System.out.println(mapper.getAllUser());
    sqlSession.close();
}
```

![image-20220308140616527](D:\otherCode\xiaou66-xiaou_blog\source\posts_images\image-20220308140616527.png)

开启两个`SqlSession`，在`sqlSession1`中查询数据，使一级缓存生效，在`sqlSession2`中更新数据库，验证一级缓存只在数据库会话内部共享。

```java
@Test
public void sessionTest3() {
    SqlSession session1 = factory.openSession(false);
    UsersMapper usersMapper1 = session1.getMapper(UsersMapper.class);
    SqlSession session2 = factory.openSession(false);
    UsersMapper usersMapper2 = session2.getMapper(UsersMapper.class);
    System.out.println(usersMapper1.getAllUser());
    usersMapper2.insertUser(new Users(0, "xiaoz"));
    session2.commit();
    session2.close();
    System.out.println(usersMapper1.getAllUser());
    session1.close();
}
```

在 session1 做两次查询在这两次查询的中间 session2  插入一行数据而 session1 的二次读取并没有触发真实的查询而是走了一级缓存，所以出现了脏数据。

![image-20220308141534541](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646720175317image-20220308141534541.png)

### 在对数据进行操作后缓存失效

在 `BaseExecutor` 中 `update` 方法中可以看到在委托子类执行 `doUpdate` 方法之前会执行 `clearLocalCache()` 会清理本地缓存。

```java
@Override
public int update(MappedStatement ms, Object parameter) throws SQLException {
  ErrorContext.instance()
      .resource(ms.getResource())
      .activity("executing an update")
      .object(ms.getId());
  if (closed) {
    throw new ExecutorException("Executor was closed.");
  }
  clearLocalCache();
  return doUpdate(ms, parameter);
}
```

insert 和 delete 方法都是统一走 update 方法

```java
@Override
public int delete(String statement, Object parameter) {
    return update(statement, parameter);
}
@Override
public int insert(String statement, Object parameter) {
    return update(statement, parameter);
}
```

### 缓存的 key 是怎么生成的

将 `MappedStatement` 的 Id、SQL 的 offset、SQL 的 limit、SQL 本身以及 SQL 中的参数传入了 CacheKey 这个类，最终构成 CacheKey

```java
CacheKey cacheKey = new CacheKey();
cacheKey.update(ms.getId());
cacheKey.update(rowBounds.getOffset());
cacheKey.update(rowBounds.getLimit());
cacheKey.update(boundSql.getSql());
if (configuration.getEnvironment() != null) {
    // issue #176
    cacheKey.update(configuration.getEnvironment().getId());
}
```

### 总结

1. MyBatis 一级缓存的生命周期和 SqlSession 一致。
2. MyBatis 一级缓存内部设计简单，只是一个没有容量限定的 HashMap，在缓存的功能性上有所欠缺。
3. MyBatis 的一级缓存最大范围是 SqlSession 内部，有多个 SqlSession 或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为 Statement。

## 二级缓存

在上文中提到的一级缓存中，其最大的共享范围就是一个 SqlSession 内部，如果多个 SqlSession 之间需要共享缓存，则需要使用到二级缓存。开启二级缓存后，会使用 `CachingExecutor` 装饰 Executor，进入一级缓存的查询流程前，先在 `CachingExecutor` 进行二级缓存的查询，具体的工作流程如下所示。

![image-20220308145354322](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646722448575image-20220308145354322.png)

> 二级缓存开启后，同一个 namespace 下的所有操作语句，都影响着同一个 Cache，即二级缓存被多个 SqlSession 共享，是一个全局的变量。
>
> 当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

### 配置

1. 开启二级缓存

```xml
<setting name="cacheEnabled" value="true"/>
```

或

```yml
mybatis:
  configuration:
    cache-enabled: true
```

2. 声明 namespace 使用二级缓存
   - 在 mapper 配置文件中使用 `<cache/>` 标签
   - 在接口中使用 `@CacheNamespace` 注解

| `@CacheNamespace`    | `<cache/>`    | 描述                                                         |
| -------------------- | ------------- | ------------------------------------------------------------ |
| implementation       | type          | cache使用的类型，默认是 `PerpetualCache`，这在一级缓存中提到过。 |
| eviction             | eviction      | 定义回收的策略，常见的有 FIFO，LRU。默认 LRU                 |
| flushInterval        | flushInterval | 配置一定时间自动刷新缓存，单位是毫秒 默认 0                  |
| size                 | size          | 最多缓存对象的个数。                                         |
| readWrite            | readOnly      | 是否只读，若配置可读写，则需要对应的实体类能够序列化         |
| blocking             | blocking      | 若缓存中找不到对应的 key，是否会一直阻塞，直到有对应的数据进入缓存。 默认false |
| `@CacheNamespaceRef` | cache-ref     | 代表引用别的命名空间的 Cache 配置，两个命名空间的操作使用的是同一个 Cache。 |

implementation 内置 cache 类

- `SynchronizedCache`：同步 Cache，实现比较简单，直接使用 synchronized 修饰方法。

- `LoggingCache`：日志功能，装饰类，用于记录缓存的命中率，如果开启了 DEBUG 模式，则会输出命中率日志。

  `SerializedCache`：序列化功能，将值序列化后存到缓存中。该功能用于缓存返回一份实例的 Copy，用于保存线程安全。

- `LruCache`：采用了 Lru 算法的 Cache 实现，移除最近最少使用的 Key/Value。
- `PerpetualCache`： 作为为最基础的缓存类，底层实现比较简单，直接使用了 HashMap。

### 实验

测试二级缓存效果，不提交事务，`sqlSession1`查询完数据后，`sqlSession2`相同的查询是否会从缓存中获取数据。

```java
@Test
public void cache2Test1() {
    SqlSession session1 = factory.openSession(false);
    UsersMapper usersMapper1 = session1.getMapper(UsersMapper.class);
    SqlSession session2 = factory.openSession(false);
    UsersMapper usersMapper2 = session2.getMapper(UsersMapper.class);
    System.out.println(usersMapper1.getUserById(1));
    System.out.println(usersMapper2.getUserById(1));
    session1.close();
    session2.close();
}
```

我们可以看到，当`sqlsession`没有调用`commit()`方法时，二级缓存并没有起到作用。

![image-20220308153309726](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646725523314image-20220308153309726.png)

测试二级缓存效果，当提交事务时，`sqlSession1`查询完数据后，`sqlSession2`相同的查询是否会从缓存中获取数据。

```java
@Test
public void cache2Test2() {
    SqlSession session1 = factory.openSession(false);
    UsersMapper usersMapper1 = session1.getMapper(UsersMapper.class);
    System.out.println(usersMapper1.getUserById(1));
    session1.commit();
    session1.close();

    SqlSession session2 = factory.openSession(false);
    UsersMapper usersMapper2 = session2.getMapper(UsersMapper.class);
    System.out.println(usersMapper2.getUserById(1));
    session2.close();
}
```

![image-20220308154014323](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646725522318image-20220308154014323.png)

测试`update`操作是否会刷新该`namespace`下的二级缓存。

![image-20220308154506959](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1646725519256image-20220308154506959.png)

我们可以看到，在 sqlSession3 更新数据库，并提交事务后，sqlsession2 的 StudentMapper namespace 下的查询走了数据库，没有走 Cache。

要是多表查询可能处出现缓存不刷新的情况，这时时候就要使用 `@CacheNamespaceRef` 或  `cache-ref`

### 其他

1. 如果要不使用二级缓存
   - 接口上使用 `@Options(useCache = false)`
   - mapper 配置文件使用属性 `useCache = "false"`

### 总结

1. MyBatis 的二级缓存相对于一级缓存来说，实现了 SqlSession 之间缓存数据的共享，同时粒度更加的细，能够到 namespace 级别，通过 Cache 接口实现类不同的组合，对 Cache 的可控性也更强。
2. MyBatis 在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
3. 在分布式环境下，由于默认的 MyBatis Cache 实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将 MyBatis 的 Cache 接口实现，有一定的开发成本，直接使用 Redis、Memcached 等分布式缓存可能成本更低，安全性也更高。
