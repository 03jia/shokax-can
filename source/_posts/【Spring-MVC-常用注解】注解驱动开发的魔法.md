---
title: 【Spring MVC 常用注解】注解驱动开发的魔法
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2026-01-11 13:42:00
updated:
description:
cover:
---
> 在 Spring MVC 中，注解可以说是开发者的“魔法棒”，通过简单的注解配置，开发者能够实现**请求处理、参数绑定、响应返回**等复杂功能，真正做到**“少写代码多干活”**。
>
> ------
>
> 我们接下来就来一起看看 Spring MVC 中常用的注解，它们的功能是什么，又该如何使用。如果你对这些注解已经有所了解，那不妨通过本文再来巩固一下，说不定还能学到一些新玩法！

------

## 1. @Controller：声明控制器的身份

`@Controller` 是 Spring MVC 中的核心注解，用于标识一个类为控制器组件。控制器是 MVC 中的“C”，负责**接收请求、调用业务逻辑并返回视图（可以是一个页面，可以是JSON数据）**。

**示例**：

```java
@Controller
public class UserController {
    @RequestMapping("/hello")
    public String sayHello() {
        return "helloView"; // 返回视图名
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**工作原理**：

- 被 `@Controller` 标注的类会被 Spring 容器扫描并注册为一个 Bean。
- 配合其他注解处理请求映射和参数绑定。

> **小提示**：`@Controller` 是 `@Component` 的派生注解，因此它也会被**自动扫描到 Spring 容器**中。

------

## 2. @RequestMapping：请求路径的导航标志

`@RequestMapping` 用于**定义请求路径与控制器方法的映射关系**，是 Spring MVC 中最常见的注解之一。

**功能特点**：

- 可以作用**在类上**，指定控制器的**基础路径**。
- 可以作用**在方法上**，进一步指定**具体路径**。

**示例**：

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/profile")
    public String getUserProfile() {
        return "profileView"; // 返回视图名
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> 请求路径 `/user/profile` 会被映射到 `getUserProfile` 方法。

**扩展功能**：

- 可以指定请求方法（如 GET、POST）：

  ```java
  @RequestMapping(value = "/update", method = RequestMethod.POST)
  public String updateUser() {
      return "updateSuccess";
  }
  ```

  ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- 还可以用简化注解，如 `@GetMapping`、`@PostMapping` 等。

------

## 3. @RequestParam：请求参数绑定到方法参数

`@RequestParam` 用于将请求中的参数映射到方法参数。适用于 URL 查询参数（如 `?name=John`）或表单提交的数据。

**示例**：

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/greet")
    public String greetUser(@RequestParam("name") String userName, Model model) {
        model.addAttribute("greeting", "Hello, " + userName + "!");
        return "greetView";
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> 请求路径 `/user/greet?name=John` 会将参数 `name` 的值映射到 `userName`。

**可选参数**：

- 设置默认值：

  ```java
  @RequestParam(value = "age", defaultValue = "18") int userAge
  ```

  ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- 标记为可选：

  ```java
  @RequestParam(value = "nickname", required = false) String nickname
  ```

  ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## 4. @PathVariable：URL 中的路径变量映射

`@PathVariable` 用于将 URL 中的动态路径部分绑定到方法参数。它非常适合 **REST 风格的接口**。

**示例**：

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/{id}")
    public String getUserById(@PathVariable("id") int userId, Model model) {
        model.addAttribute("userId", userId);
        return "userDetailView";
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

请求路径 `/user/123` 会将 `123` 绑定到 `userId` 参数。

> **注意**：路径变量名称和方法参数名称一致时，`@PathVariable` 的 `value` 属性可以省略。

------

## 5. @ModelAttribute：对象绑定与预处理神器

`@ModelAttribute` 用于将请求参数**自动绑定到 Java 对象**，同时也可以用于在请求处理之前**预填充数据**。

**自动绑定**：

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/register")
    public String registerUser(@ModelAttribute User user) {
        // User 对象会自动绑定请求参数
        return "registerSuccess";
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

假设请求参数为 `name=John&age=25`，Spring MVC 会将这些参数填充到 `User` 对象中。

**预填充数据**： 当作用在方法上时，可以在 Controller 方法执行前为模型添加数据：

```java
@ModelAttribute
public void addDefaultAttributes(Model model) {
    model.addAttribute("appName", "Spring MVC Demo");
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## 6. @RequestBody：绑定请求体数据到方法参数

`@RequestBody` 用于将 **HTTP 请求体的数据**直接绑定到方法参数上，特别适用于**处理 JSON 格式的请求体**。

**功能：**

- 将请求体数据反序列化为 Java 对象。
- 适用于 JSON、XML 等多种数据格式。
- 常用于 RESTful API，简化请求体数据的处理。

**示例：**

假设用户通过 POST 请求提交以下 JSON 数据：

```java
{
  "name": "Alice",
  "age": 25
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

控制器代码：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @PostMapping("/register")
    public String registerUser(@RequestBody User user) {
        return "User registered: " + user.getName();
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

> **说明**：
>
> - JSON 数据中的 `name` 和 `age` 会自动绑定到 `User` 对象的对应字段。
> - 返回结果将是字符串：`User registered: Alice`。

------

**注意事项**

- 请求头必须包含 `Content-Type: application/json`。
- 如果请求体为空或格式不正确，会抛出异常（如 `HttpMessageNotReadableException`），建议配合**全局异常处理器**使用。

------

## 7. @ResponseBody：直接返回数据

`@ResponseBody` 是一个强大的注解，用于将方法的**返回值直接作为 HTTP 响应体**，而不是视图名。它非常适合用来**返回 JSON 或纯文本数据**。

**示例**：

```java
@Controller
@RequestMapping("/api")
public class ApiController {

    @RequestMapping("/hello")
    @ResponseBody
    public String sayHello() {
        return "Hello, JSON!";
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

请求 `/api/hello` 会直接返回**字符串** `Hello, JSON!`。

> **扩展**：在现代项目中，`@ResponseBody` 更常与 `@RestController` 一起使用。

------

## 8. @RestController：简化你的 REST API

`@RestController` 是 `@Controller` 和 `@ResponseBody` 的组合注解，专门用于构建 REST API。标记为 `@RestController` 的类中的所有方法**默认返回 JSON**，而不是视图。

**示例**：

```java
@RestController
@RequestMapping("/api")
public class ApiController {

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable int id) {
        return new User(id, "John");
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

请求 `/api/user/123` 会返回一个 **JSON 对象**：

```java
{
  "id": 123,
  "name": "John"
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## 总结：注解让开发更简单

Spring MVC 提供了丰富的注解，极大简化了 Web 开发的复杂度：

1. **`@Controller` 和 `@RestController`**：定义控制器。
2. **`@RequestMapping`**：映射请求路径。
3. **`@RequestParam` 和 `@PathVariable`**：处理请求参数和路径变量。
4. **`@ModelAttribute`**：对象绑定和数据预填充。
5. **`@RequestBody`**：请求体为JSON对象的获取
6. **`@ResponseBody`**：直接返回数据。

> *这些注解让开发者能够专注于业务逻辑，而不用担心底层的实现细节。如果你在项目中用到这些注解，有什么有趣的用法或者疑问，欢迎留言讨论！ 😊*