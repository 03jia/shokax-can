---
title: 【Spring MVC 核心概念】揭秘概念和整体架构
keywords: []
categories:
    - [后端,框架,Spring MVC]
tags:
    - Spring MVC
    - MVC
date: 2024-12-26 23:04:58
updated:
description: "概述 Spring MVC 的整体架构与核心概念，便于理解请求从入口到响应的完整流程。"
cover: "https://s2.loli.net/2026/01/12/iLGB42y5qduVvAO.jpg"
---
> 你有没有想过，当你在浏览器地址栏敲下一个 URL，按下回车，后台到底发生了什么？如果你用的是 Spring MVC，那这一切其实被精妙地拆解成了 **三大块：Model、View 和 Controller**，共同完成了这次的任务，也就是大名鼎鼎的 **MVC架构**
>
> ------
>
> *这篇我们先大致来讲解一下 Spring MVC，在后续的博客中会展开来讲解*

------

## **1. Spring MVC 的定位**

Spring MVC 是 Spring Framework 的重要模块之一，专注于 **构建基于 MVC 模式的 Web 应用程序**。它的主要任务是：**把用户的请求和后台的数据逻辑连接起来，然后再把结果展示出来**。简单来说，它是 Web 应用中的“总指挥”，负责**调度各类组件**一起工作。

------

## **2. 什么是 MVC 模式？**

MVC 代表了三种不同的角色：**Model、View 和 Controller**。别看它们名字听起来挺高大上，其实背后的分工和我们日常生活很像。

**想象一个餐厅：**

- **Model 是后厨**：它负责制作菜品（处理数据和业务逻辑），比如获取用户信息、保存订单等。
- **View 是服务员**：它负责端上菜单和菜品（展示数据），让顾客能够愉快地看到结果。
- **Controller 是前台收银员**：它负责接收顾客的点单请求（用户的输入），然后把需求传达给后厨（Model），再通知服务员（View）上菜。

对应到 Web 应用中：

- **Model**：你的数据和业务逻辑（比如**数据库**查询）。
- **View**：网页（HTML 页面、JSON 数据等）。
- **Controller**：负责中间协调，连接用户请求和数据处理。

> **打个比方**：用户点了一碗牛肉面，Controller 会去后厨要一份“牛肉面”（业务逻辑），后厨做好后交给服务员，最后服务员把面端到顾客面前。

------

## **3. Spring MVC 的整体架构**

整个系统以一个“指挥官”组件为中心，这个组件叫做 **DispatcherServlet**。它是 Spring MVC 的**“前端控制器”**，负责协调各方组件共同完成一次用户请求的处理。

**工作流程分解：**

1. **用户发起请求**：浏览器发出请求后，DispatcherServlet 首先接收这个请求。
2. **寻找合适的处理器（Handler）**：DispatcherServlet 根据请求路径，通过 **HandlerMapping** 找到对应的控制器（**Controller**）。
3. **执行控制器方法**：找到控制器后，DispatcherServlet 会交给 **HandlerAdapter** 来执行对应的方法。
4. **返回数据**：Controller 方法处理完逻辑后，通常会返回一个模型和视图对象（ModelAndView）。
5. **解析视图**：**ViewResolver** 根据返回的视图名，找到对应的页面模板（比如 JSP 或 **Thymeleaf**）。
6. **生成响应**：渲染视图后，将最终的 HTML 或 **JSON 数据**返回给浏览器。

> 这就是一个标准的请求流程：从用户发出请求，到浏览器接收响应，Spring MVC 精心设计了每一步的分工。

------

## **4. 你需要认识的核心组件**

这里有几个 Spring MVC 架构中不可或缺的角色：

1. **DispatcherServlet（前端控制器）**
    它是整个流程的入口和出口，负责把请求转发给合适的组件处理，并将结果返回给客户端。
2. **HandlerMapping（处理器映射器）**
    通过映射规则（如路径匹配、注解等）找到对应的 Controller 方法。
3. **HandlerAdapter（处理器适配器）**
    负责执行 Controller 方法，支持多种处理器类型（如注解方式或接口方式）。
4. **ViewResolver（视图解析器）**
    根据 Controller 返回的视图名，找到实际的页面模板或数据生成逻辑。

------

## **5. 一个请求示例**

为了更直观，咱们用一个实际场景走一遍流程：假设用户想通过 `/user/1` 查询用户 ID 为 1 的信息。

1. **用户请求**：浏览器发送 `GET /user/1` 请求。

2. **前端控制器接收请求**：DispatcherServlet 收到请求，开始寻找匹配的 Controller 方法。

3. **找到处理器**：**HandlerMapping** 找到 `/user/{id}` 对应的 `UserController` 的方法。

4. **执行逻辑**：**HandlerAdapter** 执行方法，方法可能调用服务层或数据库，获取用户信息。

   ```java
   @Controller
   @RequestMapping("/user")
   public class UserController {
       @GetMapping("/{id}")
       public String getUser(@PathVariable int id, Model model) {
           model.addAttribute("user", userService.findById(id));
           return "userView"; // 返回视图名
       }
   }
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

5. **视图解析器渲染结果**：**ViewResolver** 将 `userView` 解析为 JSP 或 Thymeleaf 模板，并填充 `Model` 数据。

6. **响应返回**：最终生成的 HTML 页面发送回浏览器，展示用户信息。

------

## **6. Spring MVC 的重要性**

你可能会问：现在有了 Spring Boot，前后端分离还流行，为什么还要学 Spring MVC？

**答案很简单**：

- Spring MVC 是 Web 开发的经典架构，许多现有项目仍在使用。
- 它是理解 Spring Boot 的基础，很多概念（如注解、组件分层）都源自 Spring MVC。
- 如果需要开发传统的**服务端渲染**应用，Spring MVC 是首选。

------

## **总结**

> Spring MVC 让我们能够轻松构建**基于 MVC 模式的 Web 应用**。它的核心就是通过 **DispatcherServlet** 将请求和处理逻辑解耦，各组件分工明确、高度可定制。如果把 Web 应用比作一部机器，Spring MVC 就是这部机器的“精密引擎”，每一个零件都有其独特作用。