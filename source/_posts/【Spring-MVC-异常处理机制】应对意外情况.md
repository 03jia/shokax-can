---
title: 【Spring MVC 异常处理机制】应对意外情况
keywords: []
categories:
    - [后端,框架,Spring MVC]
tags:
    - Spring MVC
date: 2024-12-31 19:00:00
updated:
description: "讲解 Spring MVC 中局部与全局异常处理策略，以及如何使用自定义异常规范化错误响应。"
cover: "https://s2.loli.net/2026/01/11/GkWjn2b7QFuVqiB.jpg"
---
> 在 Web 应用中，**异常**是不可避免的。用户的**输入不合法**，服务的某部分出错，或者**数据库**连接失败，这些情况都可能**触发异常**。那么问题来了：如何优雅地捕获并处理这些异常，让用户体验不至于因为一时的错误而受损？
>
> ------
>
> Spring MVC 提供了灵活而强大的**异常处理机制**，包括**局部**异常处理和**全局**异常处理，帮助我们轻松管理这些“意外情况”。接下来，我们就来详细聊聊如何用 Spring MVC 的异常处理机制，做一个从容应对错误的开发者！

------

## **1. 异常处理的两种方式**

### **1.1 局部异常处理：Controller 内部的“急救”机制**

如果某些异常仅与特定的控制器相关，我们可以直接在该 Controller 中捕获并处理这些异常。

**`@ExceptionHandler` 注解**用于指定一个方法处理某种特定的异常。这种方式的好处是逻辑清晰，控制器自己的异常自己处理。

**示例：局部异常处理**

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @GetMapping("/{id}")
    public String getUser(@PathVariable int id) {
        if (id <= 0) {
            throw new IllegalArgumentException("Invalid user ID");
        }
        return "userView";
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public String handleIllegalArgumentException(IllegalArgumentException e, Model model) {
        model.addAttribute("errorMessage", e.getMessage());
        return "error";
    }
}
```

> **运行效果**：
>
> - 当用户访问 `/user/-1` 时，抛出的 `IllegalArgumentException` 会被 `handleIllegalArgumentException` 方法捕获。
> - 返回错误页面 `error`，并在页面上显示错误信息 `Invalid user ID`。

------

### **1.2 全局异常处理：让异常不再“局限于一隅”**

对于跨多个 Controller 的通用异常，例如**权限校验失败、参数格式不合法**等，局部处理显然不够灵活。此时，我们可以使用**全局异常处理机制**。

**关键注解：`@ControllerAdvice`**

- `@ControllerAdvice` 是 Spring MVC 提供的**全局异常处理**工具。
- 它会扫描所有标注了 `@Controller` 的类，捕获其中抛出的异常。

**示例：全局异常处理**

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public String handleIllegalArgumentException(IllegalArgumentException e, Model model) {
        model.addAttribute("errorMessage", e.getMessage());
        return "error";
    }

    @ExceptionHandler(Exception.class)
    public String handleGenericException(Exception e, Model model) {
        model.addAttribute("errorMessage", "An unexpected error occurred: " + e.getMessage());
        return "error";
    }
}
```

> **运行效果**：
>
> - 不管哪个 Controller 抛出 `IllegalArgumentException`，都会由 `handleIllegalArgumentException` 方法处理。
> - 如果捕获不到具体的异常类型，就可以指定为 **Exception 异常大类**，捕获到之后进入**通用异常处理逻辑**。

------

## **2. 实践中的常见用法**

### **2.1 返回 JSON 格式的错误信息**

在**前后端分离的项目**中，返回错误页面显然不是最佳选择。通过 **`@ResponseBody` 或** `**@RestControllerAdvice**（@ControllerAdvice` 和 `@ResponseBody` 的组合注解`）`，可以**直接返回 JSON 格式**的错误信息。可以返回一个对象或者Map集合来自动处理成 **JSON 格式**返回

**示例：全局 JSON 异常处理**

```java
@RestControllerAdvice
public class GlobalRestExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public Map<String, Object> handleIllegalArgumentException(IllegalArgumentException e) {
        Map<String, Object> response = new HashMap<>();
        response.put("error", true);
        response.put("message", e.getMessage());
        return response;
    }

    @ExceptionHandler(Exception.class)
    public Map<String, Object> handleGenericException(Exception e) {
        Map<String, Object> response = new HashMap<>();
        response.put("error", true);
        response.put("message", "An unexpected error occurred");
        return response;
    }
}
```

**运行效果**：

例如请求 `/user/-1`，则会返回：

```java
{
    "error": true,
    "message": "Invalid user ID"
}
```

------

### **2.2 自定义异常与枚举**

将异常信息和状态码绑定到自定义异常中，让错误响应更加规范化。

**定义枚举：错误类型与状态码**

```java
public enum ErrorCode {
    INVALID_USER_ID(400, "Invalid user ID"),
    SERVER_ERROR(500, "Internal Server Error");

    private final int code;
    private final String message;

    ErrorCode(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}
```

**自定义异常**

```java
public class CustomException extends RuntimeException {
    private final ErrorCode errorCode;

    public CustomException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    public int getCode() {
        return errorCode.getCode();
    }
}
```



**统一异常处理**

```java
@RestControllerAdvice
public class GlobalCustomExceptionHandler {

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<Map<String, Object>> handleCustomException(CustomException e) {
        Map<String, Object> response = new HashMap<>();
        response.put("error", true);
        response.put("code", e.getCode());
        response.put("message", e.getMessage());
        return new ResponseEntity<>(response, HttpStatus.valueOf(e.getCode()));
    }
}
```

> 【我们来对其中的一些代码片段做一些解释】
>
> ------
>
> **返回值** `ResponseEntity<Map<String, Object>>`**说明**：
>
> - `ResponseEntity` 是 Spring 提供的**响应封装类**，用于**设置 HTTP 状态码和响应体**。其中**`HttpStatus.valueOf(e.getCode())是`**根据**异常中的状态码**设置的 **HTTP 响应的状态码**。
> - `Map<String, Object>` 是**响应体的数据结构**，存储自定义的错误信息。
>
> 
>
> ------
>
> ***\*注意区别\**：HTTP 响应状态码和 JSON 响应体中的状态码是 \**两种不同的内容\****

**运行效果**：

- 在一些逻辑判断之后，手动抛出自定义异常： 

  ```java
  throw new CustomException(ErrorCode.INVALID_USER_ID);
  ```

- 响应结果： 

  ```java
  {
      "error": true,
      "code": 400,
      "message": "Invalid user ID"
  }
  ```


------

## **3. 局部与全局处理的对比**

| 特性         | 局部异常处理               | 全局异常处理                       |
| ------------ | -------------------------- | ---------------------------------- |
| **适用范围** | 单个 Controller 中         | 所有 Controller                    |
| **注解**     | `@ExceptionHandler`        | `@ControllerAdvice`                |
| **优点**     | 定制化处理，逻辑清晰       | 统一处理异常，代码更简洁           |
| **缺点**     | 只能处理局部异常，扩展性差 | 可能需要额外判断 Controller 作用域 |

------

## **4. 总结**

1. **局部异常处理**：使用 `@ExceptionHandler` 捕获特定 Controller 的异常，适合个性化逻辑需求。
2. **全局异常处理**：通过 `@ControllerAdvice` 实现全局异常管理，特别适用于通用异常类型。
3. **高级用法**：结合 `@RestControllerAdvice` **返回 JSON 错误信息**，自定义异常类与状态码，让接口更规范。

> 在实际项目中，局部与全局处理可以**结合使用**。局部处理特殊逻辑，全局处理通用错误，真正做到优雅应对各种“意外”。