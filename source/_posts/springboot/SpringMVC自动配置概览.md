---
title: SpringMVC自动配置概览
date: 2021/09/11 20:00
math: true
categories:
  - [springboot]
tags:
  - [java]
---

# SpringMVC自动配置详情

:::info

自动配置详细信息「org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration」

:::

配置类只有一个有参的构造函数主要将一些配置参数传递进来

> 比如下方的构造函数主要传递了

1. ResourceProperties: 绑定 spring.resources 配置
2. WebMvcProperties: 绑定 spring.web 配置
3. WebMvcProperties: 绑定spring.mvc 配置
4. 消息转换提供者, 资源处理注册定制提供者
5. dispatcherServletPath: 匹配请求路径
6. servletRegistrations: servlet 注册

```java
public WebMvcAutoConfigurationAdapter(
    org.springframework.boot.autoconfigure.web.ResourceProperties resourceProperties,
    WebProperties webProperties, WebMvcProperties mvcProperties, ListableBeanFactory beanFactory,
    ObjectProvider<HttpMessageConverters> messageConvertersProvider,
    ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
    ObjectProvider<DispatcherServletPath> dispatcherServletPath,
    ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
    this.resourceProperties = resourceProperties.hasBeenCustomized() ? resourceProperties
        : webProperties.getResources();
    this.mvcProperties = mvcProperties;
    this.beanFactory = beanFactory;
    this.messageConvertersProvider = messageConvertersProvider;
    this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
    this.dispatcherServletPath = dispatcherServletPath;
    this.servletRegistrations = servletRegistrations;
    this.mvcProperties.checkConfiguration();
}
```

## 对静态资源的默认配置

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // 如果配置了 spring.web.resources.add-mappings = false 将不会进行静态的资源配置
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    // webjars 资源的配置
    addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
    // getStaticLocations() 默认值 "classpath:/META-INF/resources/","classpath:/resources/", "classpath:/static/", "classpath:/public/"
    addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
        registration.addResourceLocations(this.resourceProperties.getStaticLocations());
        if (this.servletContext != null) {
            // 配置 servlet 上下文对象
            ServletContextResource resource = new ServletContextResource(this.servletContext, SERVLET_LOCATION);
            registration.addResourceLocations(resource);
        }
    });
}
```

## REST 风格的请求

- REST风格

  | 操作     | 以前        | 现在         |
  | -------- | ----------- | ------------ |
  | 获取用户 | /getUser    | GET /user    |
  | 删除用户 | /deleteUser | DELETE /user |
  | 修改用户 | /editUser   | PUT /user    |
| 保存用户 | /saveUser   | POST /user   |

:::info

如果需要使用表单使用 PUT或者DELETE请求需要开启 `spring.mvc.hiddenmethod.filter` 配置项默认关闭

开启后需要在表单使用隐藏域`<input type="hide" name="_method" value="PUT/DELETE/PATCH"/>`

:::

:::waring

注意: 表单提高必选是 **POST** 方法

:::

```java HiddenHttpMethodFilter核心方法
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
    throws ServletException, IOException {

    HttpServletRequest requestToUse = request;

    if ("POST".equals(request.getMethod()) && request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) == null) {
        String paramValue = request.getParameter(this.methodParam);
        if (StringUtils.hasLength(paramValue)) {
            String method = paramValue.toUpperCase(Locale.ENGLISH);
            if (ALLOWED_METHODS.contains(method)) {
                requestToUse = new HttpMethodRequestWrapper(request, method);
            }
        }
    }

    filterChain.doFilter(requestToUse, response);
}
```
1. 先判断了表单是否是 POST 请求并且含有 _method 表单
2. 进行判断是否是「PUT」「DELETE」「PATCH」方法提高
3. 然后利用「HttpMethodRequestWrapper」 进行对请求进行保证
4. 将包装好的请求传递给下一个拦截器
```java HttpMethodRequestWrapper
private static class HttpMethodRequestWrapper extends HttpServletRequestWrapper {

	private final String method;

	public HttpMethodRequestWrapper(HttpServletRequest request, String method) {
		super(request);
		this.method = method;
	}

	@Override
	public String getMethod() {
		return this.method;
	}
}
```

> 这个「HttpMethodRequestWrapper」类可以进行参照，以后需要修改请求头的可以通过继承「HttpServletRequestWrapper」进行对现有的请求进行包装

## 请求映射分析

 1. 先找到 `org.springframework.web.servlet.DispatcherServlet` 这个类负责将请求分发，所有的请求都会通过这个类

![image-20211227205953694](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1640616275507image-20211227205953694.png)

从观察类图可以发现 DispatcherServlet 实质也是一个 HttpServlet。在它的类中有 FrameworkServlet 中含有 各种请求的处理方法。

 ![image-20211227210123506](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1640616283558image-20211227210123506.png)

观察其源码都调用了 `processRequest(HttpServletRequest request, HttpServletResponse response)` 

```java
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    processRequest(request, response);
}
@Override
protected final void doPost(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    processRequest(request, response);
}
```

查看 processRequest 方法(部分) 有一个 `doService(request, response)` 核心方法而这个方法是由它的子类 DispatcherServlet 实现的。

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
    	//.... 一些前置处理
		try {
            // 核心方法
			doService(request, response);
		}
		catch (ServletException | IOException ex) {}
		catch (Throwable ex) {}
		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}
```

在 DispatcherServlet  类中找到了 `doService(request, response)`  方法实现并且找到了方法中核心调用方法`doDispatch(HttpServletRequest request, HttpServletResponse response)` 这个方法是实际的处理 url 和 处理方法的映射。是通过 HandlerMappings 获取。

获取 handler 的方法

```java
@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}
```

 ![image-20211227215500548](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1640616287252image-20211227215500548.png)

RequestMappingHandlerMapping 这个类处理 RequestMapping.class 这个注解这个也可以在它的方法 `isHandler(Class<?> beanType)` 方法中可以看出

> 其他的 HandlerMapping 将在本笔记最后补充

```java
@Override
protected boolean isHandler(Class<?> beanType) {
    return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
            AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}
```

 这个是本次扫描到 url 对象处理方法

 ![image-20211227215824150](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1640616291083image-20211227215824150.png)

对上面的「getHandler」方法逐步查找 找到了 `lookupHandlerMethod(String lookupPath, HttpServletRequest request)` 方法这个方法是查找当前请求的最佳匹配处理程序。

```java
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
		List<Match> matches = new ArrayList<>();
    	// 先查找所有 lookupPath开头 的方法
		List<T> directPathMatches = this.mappingRegistry.getMappingsByDirectPath(lookupPath);
		if (directPathMatches != null) {
            // 通过 request 判断最符合的 HandlerMethod 加入到 matches 中
			addMatchingMappings(directPathMatches, matches, request);
		}
		if (matches.isEmpty()) {
			addMatchingMappings(this.mappingRegistry.getRegistrations().keySet(), matches, request);
		}
		if (!matches.isEmpty()) {
            // 如果同时找到很多，认为第一个是最佳匹配
			Match bestMatch = matches.get(0);
			if (matches.size() > 1) {
				Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
				matches.sort(comparator);
				bestMatch = matches.get(0);
				if (logger.isTraceEnabled()) {
					logger.trace(matches.size() + " matching mappings: " + matches);
				}
				if (CorsUtils.isPreFlightRequest(request)) {
					for (Match match : matches) {
						if (match.hasCorsConfig()) {
							return PREFLIGHT_AMBIGUOUS_MATCH;
						}
					}
				}
				else {
					Match secondBestMatch = matches.get(1);
					if (comparator.compare(bestMatch, secondBestMatch) == 0) {
						Method m1 = bestMatch.getHandlerMethod().getMethod();
						Method m2 = secondBestMatch.getHandlerMethod().getMethod();
						String uri = request.getRequestURI();
						throw new IllegalStateException(
								"Ambiguous handler methods mapped for '" + uri + "': {" + m1 + ", " + m2 + "}");
					}
				}
			}
			request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.getHandlerMethod());
			handleMatch(bestMatch.mapping, lookupPath, request);
			return bestMatch.getHandlerMethod();
		}
		else {
			return handleNoMatch(this.mappingRegistry.getRegistrations().keySet(), lookupPath, request);
		}
	}
```

### 补充

#### BeanNameUrlHandlerMapping

```xml
<bean name="/welcome" class="coderead.springframework.mvc.WelcomeController" />
<bean name="/hello" class="coderead.springframework.mvc.HelloGuestController" />
<bean name="/hi" class="coderead.springframework.mvc.HelloLuBanHttpRequestHandler" />
<bean name="/login" class="coderead.springframework.mvc.LoginHttpServlet" />
```

#### RouterFunctionMapping

```java
@Configuration
public class MyRoutes {
    @Bean
    RouterFunction<ServerResponse> home() {
        return route(GET("/"), request -> ok().body(fromObject("Home page")));
    }
    @Bean
    RouterFunction<ServerResponse> about() {
        return route(GET("/about"), request -> ok().body(fromObject("About page")));
    }
}
```

#### SimpleUrlHandlerMapping

```xml
<beans>

    <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="mappings">
            <props>
                <prop key="/welcome.htm">welcomeController</prop>
                <prop key="/*/welcome.htm">welcomeController</prop>
                <prop key="/helloGuest.htm">helloGuestController</prop>
            </props>
        </property>
    </bean>

    <bean id="welcomeController" 
          class="com.mkyong.common.controller.WelcomeController" />

    <bean id="helloGuestController" 
          class="com.mkyong.common.controller.HelloGuestController" />

</beans>
```

#### WelcomePageHandler

> 默认的欢迎页面处理的 handler

![image-20211227223533358](https://cdn.jsdelivr.net/gh/xiaou66/picture@master/image/1640616295785image-20211227223533358.png)
