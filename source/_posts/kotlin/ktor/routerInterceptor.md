---
title: ktor 自定义验证拦截器
date: 2021/09/11 13:57
math: true
categories:
  - [ktor]
tags:
  - [kotlin]
---

# ktor 自定义验证拦截器

:::info

在使用官方的用户身份校验不满足需求的情况下,模仿官方的写法自定义验证拦截器

:::

## 先来看看官方的「authenticate」源码

```kotlin
public class AuthenticationRouteSelector(public val names: List<String?>) : RouteSelector() {
    override fun evaluate(context: RoutingResolveContext, segmentIndex: Int): RouteSelectorEvaluation {
        return RouteSelectorEvaluation(true, RouteSelectorEvaluation.qualityTransparent)
    }

    override fun toString(): String = "(authenticate ${names.joinToString { it ?: "\"default\"" }})"
}
public fun Route.authenticate(
    vararg configurations: String? = arrayOf<String?>(null),
    optional: Boolean = false,
    build: Route.() -> Unit
): Route {
    require(configurations.isNotEmpty()) { "At least one configuration name or null for default need to be provided" }
    val configurationNames = configurations.distinct()
    val authenticatedRoute = createChild(AuthenticationRouteSelector(configurationNames))

    application.feature(Authentication).interceptPipeline(authenticatedRoute, configurationNames, optional = optional)
    authenticatedRoute.build()
    return authenticatedRoute
}
```

1. 可以发现官方直接注册一个`Authentication`模式并拦截请求管道实现校验。

```kotlin
class TokenRouteSelector(val role: String) : RouteSelector() {
    override fun evaluate(context: RoutingResolveContext, segmentIndex: Int): RouteSelectorEvaluation {
        return RouteSelectorEvaluation.Constant
    }
    override fun toString(): String = "(tokenCheck ${role})"
}
fun Route.tokenCheck(role: String = "user", build: Route.() -> Unit): Route {
     val route = createChild(TokenRouteSelector(role))
     route.intercept(ApplicationCallPipeline.Features) {
         val token = call.request.header("Authorization")
         token?.let {
             // 对 token 进行校验
         }?: let {
             // 没有传递token
         }
     }
    route.build()
    return route;
}
```

1. role 参数代表当前请求所需要的权限
2. 另一个参数是函数类型参数，用来实现DSL语法。这个函数基于Route类的拓展函数，拥有Route的上下文。
3. 随后我们通过`createChild()`获取一个子Route类。
4. 其中`TokenRouteSelector`是一个继承自`RouteSelector`类的自定义类。

