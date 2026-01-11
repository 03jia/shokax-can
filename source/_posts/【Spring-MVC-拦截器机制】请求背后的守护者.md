---
title: 【Spring MVC 拦截器机制】请求背后的守护者
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2026-01-11 13:43:27
updated:
description:
cover:
---
> 在 Web 应用中，每一个请求从用户发出到服务器响应，都需要经过一条完整的“处理链”。有时候，我们需要在这条链中加一些“守护者”，为请求执行**权限校验、日志记录，甚至是性能监控**。而在 Spring MVC 中，**拦截器（Interceptor）** 就扮演了这样的角色。

------

## **1. 什么是拦截器？**

Spring MVC 的**拦截器**（`HandlerInterceptor`）是一种机制，允许开发者**在请求到达控制器之前、之后**，或者在视图渲染完成之后，执行一些额外的逻辑。

**拦截器的生命周期**：

1. **请求前**（`preHandle`）：在 Controller 方法执行之前调用。
2. **请求后**（`postHandle`）：在 Controller 方法执行之后、视图渲染之前调用。
3. **完成后**（`afterCompletion`）：在请求完成并且视图渲染之后调用（通常用于**资源清理**，如释放一些连接资源、线程资源等）。

------

## **2. 为什么使用拦截器？**

拦截器的设计让我们可以在**处理请求的不同阶段**执行自定义逻辑，常见用途包括：

- **权限校验**：拦截**未授权的请求**，防止非法访问。
- **日志记录**：记录请求信息，例如请求的 URL、用户 ID 等。
- **请求预处理**：在进入 Controller 之前**统一处理参数或编码**（可以设置请求头，明确指定请求的内容格式为 **JSON**）。
- **性能监控**：统计每个请求的**处理耗时**。

------

## **3. 拦截器的实现步骤**

### **3.1 创建拦截器类**

要实现一个拦截器，只需**实现 `HandlerInterceptor` 接口**。该接口定义了三个方法：

- `preHandle`：请求前逻辑。
- `postHandle`：请求后逻辑。
- `afterCompletion`：请求完成后逻辑。

**示例：统计请求处理耗时**

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class RequestTimeInterceptor implements HandlerInterceptor {

    private static final String START_TIME = "startTime";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        long startTime = System.currentTimeMillis();
        request.setAttribute(START_TIME, startTime);
        System.out.println("Request started at: " + startTime);
        return true; // 返回 true 表示继续执行后续处理
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        long startTime = (long) request.getAttribute(START_TIME);
        long endTime = System.currentTimeMillis();
        System.out.println("Request handled in: " + (endTime - startTime) + "ms");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("Request completed");
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### **3.2 注册拦截器**

只是配置好一个拦截器类还无法实现功能，因为此时它只是一个普通的Bean，还需要通过配置将拦截器**注册到 Spring MVC 的拦截链**（`HandlerExecutionChain`）中。这个注册过程是由 `WebMvcConfigurer` 接口的 `addInterceptors` 方法完成的。通过 `addPathPatterns` 和 `excludePathPatterns` 指定拦截路径，避免全局生效。

**示例：注册拦截器**

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final RequestTimeInterceptor requestTimeInterceptor;

    public WebConfig(RequestTimeInterceptor requestTimeInterceptor) {
        this.requestTimeInterceptor = requestTimeInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(requestTimeInterceptor)
                .addPathPatterns("/**") // 拦截所有路径
                .excludePathPatterns("/static/**", "/login"); // 排除某些路径
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## **4. 拦截器的常见应用场景**

### **4.1 权限校验**

拦截未登录用户的请求，或者根据用户角色限制某些接口访问。

**示例：权限校验拦截器**

```java
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("Authorization");
        if (token == null || !validateToken(token)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false; // 拦截请求，直接返回
        }
        return true;
    }

    private boolean validateToken(String token) {
        // 模拟 token 验证逻辑
        return "valid-token".equals(token);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### **4.2 日志记录**

记录请求路径、参数、处理时间等信息，方便问题排查。

**示例：日志拦截器**

实际开发中用日志框架来记录日志，我们为了方便演示就用 System.out.println() 来替代

```java
@Component
public class LogInterceptor implements HandlerInterceptor {

    @Override
    public void preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        System.out.println("Request URL: " + request.getRequestURL());
        System.out.println("Request Params: " + request.getParameterMap());
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### **4.3 请求预处理**

为每个请求添加通用的参数或调整编码。

**示例：参数预处理**

```java
@Component
public class RequestPreProcessorInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // 设置请求头 Content-Type 为 JSON
        response.setHeader("Content-Type", "application/json");
        request.setAttribute("globalAttribute", "commonValue");
        return true;    //放行，执行后续的业务
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## **5. 拦截器与过滤器的区别**

虽然拦截器和过滤器的功能有相似之处，但它们的定位和作用范围不同：

| 特性            | 拦截器（Interceptor）          | 过滤器（Filter）                 |
| --------------- | ------------------------------ | -------------------------------- |
| 执行范围        | Spring MVC 层，拦截 Controller | Servlet 层，拦截整个请求处理过程 |
| 实现接口        | `HandlerInterceptor`           | `jakarta.servlet.Filter`         |
| 使用场景        | 请求预处理、权限校验等业务逻辑 | 安全认证、编码转换等通用逻辑     |
| 依赖 Spring MVC | 是                             | 否                               |

------

## **6. 总结**

> Spring MVC 的拦截器机制是开发 Web 应用的利器，通过简单的接口实现和配置，就可以在请求的各个阶段插入自定义逻辑。无论是 **权限校验**、**日志记录**，还是 **请求预处理**，拦截器都能让你的代码更加模块化和可维护。