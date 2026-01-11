---
title: 【Spring MVC 核心机制】核心组件和工作流程解析
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2024-12-27 23:04:587
updated:
description:
cover:
---

在 Web 应用开发中，处理用户请求的逻辑常常会涉及到**路径匹配、请求分发、视图渲染**等多个环节。Spring MVC 作为一款强大的 Web 框架，将这些复杂的操作高度抽象化，通过组件协作简化了开发者的工作。

无论是处理表单请求、**生成动态页面**，还是**返回 JSON 数据**，Spring MVC 的核心组件都在背后默默完成了一系列任务。

> 这篇文章，我们将深入剖析 Spring MVC 的核心组件，包括 **DispatcherServlet**、**HandlerMapping**、**HandlerAdapter** 等，带你一步步了解**请求从发送到返回的完整工作流程**。相信看完之后，你对 Spring MVC 的底层机制会有一个更清晰的认知！

------

## **1. DispatcherServlet：Spring MVC 的“大总管”**

Spring MVC 的每一次请求之旅都始于 **DispatcherServlet**，它是 Spring MVC 的**前端控制器**，也是**整个流程的“调度中心”**。它的职责是：

- **接收用户请求**：DispatcherServlet 是整个应用的唯一入口，所有**用户请求**都会先到它这里。
- **分发请求**：根据配置和规则，将请求**交给合适的组件**处理。
- **处理返回结果**：将织处理结果组起来，通过视图解析器**生成最终的响应**。

在 Spring MVC 的架构中，`DispatcherServlet` 是整个工作流程的核心，贯穿了请求的每一步。它就像一个“总指挥”，负责调度整个团队完成任务。

------

## **2. HandlerMapping：找到你的处理器**

当请求到达 DispatcherServlet 后，第一步就是 **寻找合适的处理器（Handler）**。这个任务由 **HandlerMapping** 来完成。

**HandlerMapping 的工作原理**：

- 根据请求路径（例如 `/user/123`），匹配相应的**处理器（Controller）**。
- Spring MVC 提供了多种匹配策略（基于**注解、配置文件**等），开发者可以根据需求选择。

> **【类比】**HandlerMapping 就像电话客服系统，当你拨打不同的分机号，系统会将你的请求转接给不同的部门。

------

## **3. HandlerAdapter：适配各种处理器**

找到处理器只是开始，接下来就需要 **执行处理器的方法**。由于不同的处理器（如基于注解的 Controller 或简单的接口实现）有不同的调用方式，Spring MVC 提供了 **HandlerAdapter** 来完成适配工作。

**HandlerAdapter 的职责**：

- 识别**处理器的类型**。
- 调用**处理器的方法**，并将**结果返回给 DispatcherServlet**。

HandlerAdapter 的存在使得 Spring MVC 支持多种处理器模式，同时对开发者透明，不需要关心底层细节。

------

## **4. ModelAndView：承载数据和视图信息**

处理器方法执行后，会返回一个结果，而 Spring MVC 需要一个容器来存储这些结果。这个容器就是 **ModelAndView**。

- **Model**：存放业务数据，通常是 Java 对象或 Map。
- **View**：存放视图信息，通常是视图的名称（例如 `userView`）。

**工作机制**：

1. 处理器方法将结果（数据和视图名）封装到 ModelAndView 中。
2. DispatcherServlet 将 ModelAndView 交给下一个环节：视图解析器。

> **【小提示】**如果返回的是 JSON 数据，ModelAndView 的视图部分会被忽略，直接通过**消息转换器生成 JSON 响应**。

------

## **5. ViewResolver：解析视图，生成响应**

最后一步是将结果展示给用户，而这需要 **视图解析器（ViewResolver）** 的帮助。

**ViewResolver 的作用**：

- 将返回的视图名称（例如 `userView`），解析为实际的视图文件（例如 `userView.jsp` 或 `userView.html`）。
- 渲染视图并填充数据，生成最终的 HTML 或 JSON 响应。

Spring MVC 支持多种视图类型，例如：

- JSP
- Thymeleaf
- FreeMarker
- JSON（通过消息转换器）

**视图解析的灵活性**使得 Spring MVC 能够适配不同的前端技术栈，无论是传统的服务端渲染还是现代化的前后端分离。

------

## **6. 工作流程完整解析**

我们用一个场景来走一遍 Spring MVC 的完整流程：

**场景**：用户访问 `/user/123`，希望查询 ID 为 123 的用户信息。

1. **请求到达 DispatcherServlet**： 浏览器发出 `GET /user/123` 请求，DispatcherServlet 首先接收。

2. **HandlerMapping 匹配处理器**： DispatcherServlet 调用 HandlerMapping，找到匹配的 Controller 方法，例如：

   ```java
   @GetMapping("/user/{id}")
   public String getUser(@PathVariable int id, Model model) {
       model.addAttribute("user", userService.findById(id));
       return "userView";
   }
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

3. **HandlerAdapter 调用方法**： DispatcherServlet 使用 HandlerAdapter 调用 `getUser` 方法，得到返回值 `userView` 和数据模型。

4. **生成 ModelAndView**： `userView` 和 `Model` 数据被封装到 ModelAndView 对象中。

5. **视图解析器解析视图**：当使用 Thymeleaf 作为模板引擎时，视图解析器（**`ViewResolver`**）会根据视图名 `userView`，找到对应的 Thymeleaf 模板文件 `templates/userView.html`，并通过 Thymeleaf 渲染后返回结果给用户。

6. **渲染并返回响应**： 视图渲染完成后，将 HTML 页面返回给用户。

------

## **总结**

***最后我们来通过一张图更形象直观的理解这个流程\***

![img](https://s2.loli.net/2026/01/11/rlqnU8VKXQ1ofE2.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

Spring MVC 的工作流程层次分明，核心组件各司其职，协同完成一次请求处理。简单来说：

1. DispatcherServlet 是**“调度中心”**，统筹全局。
2. HandlerMapping 和 HandlerAdapter 负责找到并执行**处理器**。
3. ModelAndView 是**数据和视图的桥梁**。
4. ViewResolver 则负责将**结果**呈现给用户。

*理解这些组件的工作机制，不仅能帮助你更好地使用 Spring MVC，也能让你在遇到问题时快速定位到具体的环节。如果你有其他疑问或心得，欢迎留言，一起探讨！ 😊*
