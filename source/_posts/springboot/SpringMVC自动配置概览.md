---
title: SpringBoot自动配置概览
date: 2021/09/11 20:00
math: true
categories:
  - [springboot]
tags:
  - [java]
---

# SpringBoot 自动配置详情

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

![image-20211227205953694](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1640616275507image-20211227205953694.png)

从观察类图可以发现 DispatcherServlet 实质也是一个 HttpServlet。在它的类中有 FrameworkServlet 中含有 各种请求的处理方法。

 ![image-20211227210123506](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1640616283558image-20211227210123506.png)

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

 ![image-20211227215500548](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1640616287252image-20211227215500548.png)

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

 ![image-20211227215824150](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1640616291083image-20211227215824150.png)

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

![image-20211227223533358](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1640616295785image-20211227223533358.png)

## 请求参数映射解析

### 普通参数与基本注解

@PathVariable、@RequestHeader、@ModelAttribute、@RequestParam、@MatrixVariable、@CookieValue、@RequestBody

### Servlet API

WebRequest、ServletRequest、MultipartRequest、 HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、Locale、TimeZone、ZoneId

### 复杂参数

**Map**、**Model（map、model里面的数据会被放在request的请求域  request.setAttribute）、**Errors/BindingResult、**RedirectAttributes（ 重定向携带数据）**、**ServletResponse（response）**、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder

### 在 DispatcherServlet 类中的 doDispatch 方法

```java
// 确定当前请求的处理程序适配器
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1640868221671image-20211229153857567.png)

0. 支持方法上标注@RequestMapping 
1. 函数式 RouterFunctionMapping 的

### 使用刚刚获得的处理适配器进行调用方法

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

在 `ServletInvocableHandlerMethod`  `invokeAndHandle` 方法中的 

```java
// 获取请求参数
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
```

![](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1640868199071image-20211229155023045.png)

获取参数的实际方法

```java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,Object... providedArgs) throws Exception {
    
    MethodParameter[] parameters = getMethodParameters();
    if (ObjectUtils.isEmpty(parameters)) {
        return EMPTY_ARGS;
    }
	// 初始化获取参数后存放的容器
    Object[] args = new Object[parameters.length];
    for (int i = 0; i < parameters.length; i++) {
        MethodParameter parameter = parameters[i];
        // 设置方法参数名称发现器，缺省会是 DefaultParameterNameDiscoverer，
        // 方法参数名称发现器用于发现该参数的名称，该名称用于从请求上下文中分析该参数的参数值
        parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
        //  首先尝试  从 providedArgs 中获取该参数的参数值，
        // Spring MVC 默认情况下 providedArgs 总是为空，所以这里关于应用 providedArgs 的
        // 逻辑默认总是空转，不产生实际效果
        args[i] = findProvidedArgument(parameter, providedArgs);
        if (args[i] != null) {
            continue;
        }
        // this.resolvers 是调用者 RequestMappingHandlerAdapter 设置给当前对象的
        // 多个参数解析器的组合对象 HandlerMethodArgumentResolverComposite,
        // 检测当前参数是否被 this.resolvers 支持，如果不支持则抛出异常
        // 具体执行看下面获取参数解析器代码块
        // 注意:这里只是查找是否有支持这个参数的参数解析器而不会获取参数解析器，
        // 但是执行过这个方法会将找到的参数解析器缓起来
        if (!this.resolvers.supportsParameter(parameter)) {
            throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
        }
        try {
            // 正常情况下这里会使用上面找到的参数解析器的缓存来获取参数解析器
            // 获取后将使用参数解析器解析参数将解析完成的参数返回如果解析识别则抛出异常
            args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
        }
        catch (Exception ex) {
            // Leave stack trace for later, exception may actually be resolved and handled...
            if (logger.isDebugEnabled()) {
                String exMsg = ex.getMessage();
                if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
                    logger.debug(formatArgumentError(parameter, exMsg));
                }
            }
            throw ex;
        }
    }
    // 返回解析完成的参数列表
    return args;
}
```

**获取参数解析器**

参数解析器接口是 `org.springframework.web.method.support.HandlerMethodArgumentResolver`

主要有两个方法

1. `boolean supportsParameter(MethodParameter parameter)`;  是否支持解析
2. `Object resolveArgument(....) throws Exception`; 解析的方法

```java 获取参数解析器
private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
    HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
    if (result == null) {
        for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
            if (resolver.supportsParameter(parameter)) {
                result = resolver;
                // 找到后放入缓存
                this.argumentResolverCache.put(parameter, result);
                break;
            }
        }
    }
    return result;
}
```

### 自定义对象参数封装

自定义对象

```java
@Data
public class User {
    private String username;
    private int age;
    private Car car;
}
@Data
public class Car {
    private String name;
}
```

> 请求地址 `user?username=xiaou&age=1&car.name=qq`

```json
{
    "username": "xiaou",
    "age": 1,
    "car": {
        "name": "qq"
    }
}
```

使用  `ServletModelAttributeMethodProcessor` 参数解析器

## 返回值处理

还是在 `org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod` 这个类的 `invokeAndHandle` 方法
```java
this.returnValueHandlers.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
```
### 所有的返回值解析器
 ![image-20220108105800946](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1644745912620image-20220108105800946.png)

### 查找返回值处理器

```java HandlerMethodReturnValueHandlerComposite.java
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                              ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
    // 根据返回类型查找返回值解析器
    HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
    if (handler == null) {
        // 没有找到就抛出异常
        throw ....
    }
    // 使用找到的返回值解析器解析返回值
    handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
}
// 查找过程
@Nullable
private HandlerMethodReturnValueHandler selectHandler(@Nullable Object value, MethodParameter returnType) {
    boolean isAsyncValue = isAsyncReturnValue(value, returnType);
    for (HandlerMethodReturnValueHandler handler : this.returnValueHandlers) {
        if (isAsyncValue && !(handler instanceof AsyncHandlerMethodReturnValueHandler)) {
            continue;
        }
        if (handler.supportsReturnType(returnType)) {
            return handler;
        }
    }
    return null;
}
```

###  返回值处理器接口

```java HandlerMethodReturnValueHandler
public interface HandlerMethodReturnValueHandler {

	/**
	 * 该处理器是否支持这个类型
	 * @param returnType the method return type to check
	 * @return {@code true} if this handler supports the supplied return type;
	 * {@code false} otherwise
	 */
	boolean supportsReturnType(MethodParameter returnType);

	/**
	 * 先模型中设置值或者视图
	 * {@link ModelAndViewContainer#setRequestHandled} flag to {@code true}
	 * to indicate the response has been handled directly.
	 * @param returnValue the value returned from the handler method
	 * @param returnType the type of the return value. This type must have
	 * previously been passed to {@link #supportsReturnType} which must
	 * have returned {@code true}.
	 * @param mavContainer 当前请求的 ModelAndViewContainer
	 * @param webRequest 当前的请求的 Request
	 * @throws Exception if the return value handling results in an error
	 */
	void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
			ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception;
}
```

### 处理返回值并且写到返回流中

```java writeWithMessageConverters 方法
protected <T> void writeWithMessageConverters(@Nullable T value, MethodParameter returnType,
                                              ServletServerHttpRequest inputMessage, ServletServerHttpResponse outputMessage)
    throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {
    
    Object body;
    Class<?> valueType;
    Type targetType;
    // 获取返回值的类型
    if (value instanceof CharSequence) {
        body = value.toString();
        valueType = String.class;
        targetType = String.class;
    }
    else {
        body = value;
        valueType = getReturnValueType(body, returnType);
        targetType = GenericTypeResolver.resolveType(getGenericType(returnType), returnType.getContainingClass());
    }
    // 是否是流类型的返回值
    if (isResourceType(value, returnType)) {
        outputMessage.getHeaders().set(HttpHeaders.ACCEPT_RANGES, "bytes");
        if (value != null && inputMessage.getHeaders().getFirst(HttpHeaders.RANGE) != null &&
            outputMessage.getServletResponse().getStatus() == 200) {
            Resource resource = (Resource) value;
            try {
              List<HttpRange> httpRanges = inputMessage.getHeaders().getRange();
              outputMessage.getServletResponse().setStatus(HttpStatus.PARTIAL_CONTENT.value());
              body = HttpRange.toResourceRegions(httpRanges, resource);
              valueType = body.getClass();
              targetType = RESOURCE_REGION_LIST_TYPE;
            }
            catch (IllegalArgumentException ex) {
                // 异常处理
                ...
            }
        }
    }
    // 响应媒体类型
    MediaType selectedMediaType = null;
    MediaType contentType = outputMessage.getHeaders().getContentType();
    // 判断是否已经设置了媒体类型
    boolean isContentTypePreset = contentType != null && contentType.isConcrete();
    if (isContentTypePreset) {
        if (logger.isDebugEnabled()) {
            logger.debug("Found 'Content-Type:" + contentType + "' in response");
        }
        // 直接将设置的媒体类型设置上去
        selectedMediaType = contentType;
    }
    else {
        HttpServletRequest request = inputMessage.getServletRequest();
        List<MediaType> acceptableTypes;
        try {
            // 在请求中默认获取 accept 字段并且解析
            // 这里会调用内容协商管理器
            acceptableTypes = getAcceptableMediaTypes(request);
        }
        catch (HttpMediaTypeNotAcceptableException ex) {
            int series = outputMessage.getServletResponse().getStatus() / 100;
            if (body == null || series == 4 || series == 5) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Ignoring error response content (if any). " + ex);
                }
                return;
            }
            throw ex;
        }
        // 获取服务器可以提供的媒体类型
        List<MediaType> producibleTypes = getProducibleMediaTypes(request, valueType, targetType);
        if (body != null && producibleTypes.isEmpty()) {
            // 未找到
            throw new HttpMessageNotWritableException(
                "No converter found for return value of type: " + valueType);
        }
        // 可以使用的媒体类型
        List<MediaType> mediaTypesToUse = new ArrayList<>();
        // 通过循环找到游览器和服务器都支持的媒体类型(交集)
        for (MediaType requestedType : acceptableTypes) {
            for (MediaType producibleType : producibleTypes) {
                if (requestedType.isCompatibleWith(producibleType)) {
                    mediaTypesToUse.add(getMostSpecificMediaType(requestedType, producibleType));
                }
            }
        }
        if (mediaTypesToUse.isEmpty()) {
            // 没有交集
            if (body != null) {
                throw new HttpMediaTypeNotAcceptableException(producibleTypes);
            }
            if (logger.isDebugEnabled()) {
                logger.debug("No match for " + acceptableTypes + ", supported: " + producibleTypes);
            }
            return;
        }
        // 根据游览器设置的权重对可以使用的媒体类型的集合进行排序
        MediaType.sortBySpecificityAndQuality(mediaTypesToUse);
        
        for (MediaType mediaType : mediaTypesToUse) {
            // 判断是否是具体的媒体类型不带
            if (mediaType.isConcrete()) {
                selectedMediaType = mediaType;
                break;
            }
            else if (mediaType.isPresentIn(ALL_APPLICATION_MEDIA_TYPES)) {
                selectedMediaType = MediaType.APPLICATION_OCTET_STREAM;
                break;
            }
        }

        if (logger.isDebugEnabled()) {
            logger.debug("Using '" + selectedMediaType + "', given " +
                         acceptableTypes + " and supported " + producibleTypes);
        }
    }
    // 使用消息转换器转换数据并且输出
    if (selectedMediaType != null) {
        selectedMediaType = selectedMediaType.removeQualityValue();
        for (HttpMessageConverter<?> converter : this.messageConverters) {
            GenericHttpMessageConverter genericConverter = (converter instanceof GenericHttpMessageConverter ?
                                                            (GenericHttpMessageConverter<?>) converter : null);
            if (genericConverter != null ?
                ((GenericHttpMessageConverter) converter).canWrite(targetType, valueType, selectedMediaType) :
                converter.canWrite(valueType, selectedMediaType)) {
                body = getAdvice().beforeBodyWrite(body, returnType, selectedMediaType,
                                                   (Class<? extends HttpMessageConverter<?>>) converter.getClass(),
                                                   inputMessage, outputMessage);
                if (body != null) {
                    Object theBody = body;
                    LogFormatUtils.traceDebug(logger, traceOn ->
                                              "Writing [" + LogFormatUtils.formatValue(theBody, !traceOn) + "]");
                    addContentDispositionHeader(inputMessage, outputMessage);
                    if (genericConverter != null) {
                        genericConverter.write(body, targetType, selectedMediaType, outputMessage);
                    }
                    else {
                        ((HttpMessageConverter) converter).write(body, selectedMediaType, outputMessage);
                    }
                }
                else {
                    if (logger.isDebugEnabled()) {
                        logger.debug("Nothing to write: null body");
                    }
                }
                return;
            }
        }
    }

    if (body != null) {
        Set<MediaType> producibleMediaTypes =
            (Set<MediaType>) inputMessage.getServletRequest()
            .getAttribute(HandlerMapping.PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE);

        if (isContentTypePreset || !CollectionUtils.isEmpty(producibleMediaTypes)) {
            throw new HttpMessageNotWritableException(
                "No converter for [" + valueType + "] with preset Content-Type '" + contentType + "'");
        }
        throw new HttpMediaTypeNotAcceptableException(getSupportedMediaTypes(body.getClass()));
    }
}
```

#### Spring 支持的返回值

1. ModelAndView
2. Model
3. View
4. ResponseEntity 
5. ResponseBodyEmitter
6. HttpEntity
7. HttpHeaders
8. Callable
9. DeferredResult
10. ListenableFuture
11. CompletionStage
12. WebAsyncTask
13. 有 @ModelAttribute 且为对象类型的
14. @ResponseBody 注解 ---> RequestResponseBodyMethodProcessor

#### 总结

1. 返回值处理器判断是否支持这种类型的返回值 `supportsReturnType`
2. 返回值处理器可调用 `handleReturnValue` 进行处理
3. RequestResponseBodyMethodProcessor 可以处理返回值标了@ResponseBody 注解的
   - 内容协商 ( 根据游览器默认会以请求头 **accept** 告诉服务器自己可以处理什么样的格式信息 )
   - 服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据
   - Spring MVC 会按照顺序遍历所有容器底层的 `HttpMessageConverter` 看谁可以处理
     - 得到 `MappingJackson2CborHttpMessageConverter` `object` 转 `JSON`
     - 利用 `MappingJackson2CborHttpMessageConverter` 转换为 JSON 字符串再写到流中

#### 游览器 appect

![image-20220113204806200](D:\code\xiaou_blog\source\_posts\designMode\image\image-20220113204806200.png)

### HttpMessageConverter

 ![image-20220113205448939](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1644745972141image-20220113205448939.png)

#### **默认消息转换器**

 ![image-20220108140902237](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1644745975199image-20220108140902237.png)

0. 只支持Byte类型的
1. String
2. String
3. Resource
4. ResourceRegion
5. DOMSource.class \SAXSource.class \ StAXSource.class\StreamSource.class \ Source.class
6. MultiValueMap
7. true
8. false
9. 支持注解方式xml处理的

> 最终 MappingJackson2HttpMessageConverter  把对象转为 JSON（利用底层的 jackson 的 objectMapper 转换的）

### 内容协商

#### 配置

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

```yaml
spring:
    contentnegotiation:
      favor-parameter: true  # 开启请求参数内容协商模式
```

#### 效果

请求地址: `/user?username=xiaou&age=1&car.name=qq&format=json`

```json
{
    "username": "xiaou",
    "age": 1,
    "car": {
        "name": "qq"
    }
}
```

请求地址: `/user?username=xiaou&age=1&car.name=qq&format=xml`

```xml
<User>
    <username>xiaou</username>
    <age>1</age>
    <car>
        <name>qq</name>
    </car>
</User>
```

#### 执行流程

在 ContentNegotiationManager 内容协商管理器中

```java
public List<MediaType> resolveMediaTypes(NativeWebRequest request) throws HttpMediaTypeNotAcceptableException {
    for (ContentNegotiationStrategy strategy : this.strategies) {
        List<MediaType> mediaTypes = strategy.resolveMediaTypes(request);
        if (mediaTypes.equals(MEDIA_TYPE_ALL_LIST)) {
            continue;
        }
        return mediaTypes;
    }
    return MEDIA_TYPE_ALL_LIST;
}
```

 **协商策略**

 ![image-20220113214247788](https://fastly.jsdelivr.net/gh/xiaou66/picture@master/image/1644745980140image-20220113214247788.png)

0. 开启请求参数内容协商策略才会有
1. 请求头协商默认

:::warning

这里可以注意到 **请求参数内容协商策略** 高于 **请求头协商策略**

:::

#### 自定义内容协商

1. 实现 `HttpMessageConverter` 接口

```java
public class CustomHttpMessageConvert implements HttpMessageConverter<User> {
    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        return false;
    }

    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        return clazz.isAssignableFrom(User.class);
    }

    /**
     * 服务器需要统计所有的 messageConverter 都能写出那些内容
     * application/x-xiaou
     * @return
     */
    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return MediaType.parseMediaTypes("application/x-xiaou");
    }

    @Override
    public User read(Class<? extends User> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    @Override
    public void write(User user, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        String data = user.getUsername() + ":" + user.getAge();
        outputMessage.getBody().write(data.getBytes(StandardCharsets.UTF_8));
    }
}
```

2. 实现 `WebMvcConfigurer` 接口中的 

`public void extendMessageConverters(List<HttpMessageConverter<?>> converters)` 方法

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new CustomHttpMessageConvert());
    }
}
```



