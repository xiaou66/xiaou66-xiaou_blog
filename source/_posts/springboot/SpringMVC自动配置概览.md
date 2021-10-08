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

如果需要使用表单使用 PUT或者DELETE请求需要开启「spring.mvc.hiddenmethod.filter」配置项默认关闭

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



