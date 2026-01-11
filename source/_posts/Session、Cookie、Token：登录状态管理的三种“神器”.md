---
title: Session、Cookie、Token：登录状态管理的三种“神器”
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2026-01-11 13:45:21
updated:
description:
cover:
---
> 在开发中，用户登录后如何确认“这个人是我”？系统又如何在安全的前提下维持**用户状态**？我们带着这些疑问，聊聊三种常见的登录状态管理方式：**Session**、**Cookie** 和 **Token**。并且我会用生动的例子和比喻描述这些概念。

------

## **一、起点：登录状态管理的痛点**

### **为什么需要它们？**

想象你去一家图书馆借书，每次进门都要填写身份信息才能验证会员资格，是不是很麻烦？如果有个“会员卡”，前台一扫就知道是你，是不是方便多了？

在 Web 开发中，这个“会员卡”就是 **Cookie**、**Session** 或 **Token**。它们解决了这些问题：

- **身份确认：** 系统如何知道你是谁？
- **状态保持：** 登录后如何让你访问其他页面时不需要重新登录？
- **安全保障：** 如何防止身份被伪造？

------

## **二、Cookie：客户端的“身份证”**

### **什么是 Cookie？**

Cookie 是一段由**服务端生成**的小数据，**存储在客户端**（浏览器）中。每次你访问网站时，**浏览器会自动携带**这段数据发送给服务端。

> **比喻：**
>  就像你随身携带的一张“身份证”。每次进入图书馆，出示身份证就能证明你的身份。

### **Cookie 的常见场景**

1. **记住登录状态：** 比如“记住我”功能。
2. **用户偏好设置：** 保存语言、主题等个性化选项。
3. **广告追踪：** 用于记录用户行为。

### **Cookie 的特点**

- **自动传递：** 浏览器负责发送，无需手动操作。
- **容量有限：** 单个 Cookie 大小最多 4KB。
- **容易被篡改：** 建议设置 `HttpOnly` 和 `Secure` 来增强安全性。

### **代码示例**

服务端生成一个 Cookie：

```java
response.setHeader('Set-Cookie', 'userId=12345; HttpOnly; Path=/');
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

客户端每次请求都会携带：

```java
GET /dashboard HTTP/1.1
Cookie: userId=12345
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## **三、Session：服务端的“会员档案”**

### **Session 是什么？**

Session **存储在服务端**，用于**记录用户的会话信息**。客户端通过 Cookie（或 URL 参数）携带 Session ID，服务端**根据这个 ID 找到对应的用户数据**。

> **比喻：**
>  健身房前台有个“会员档案册”，每个会员进场时，只需要告诉前台自己的档案编号（Session ID），前台就能查到你是谁。

### **Session 的工作流程**

1. 用户登录时，服务端生成一个**唯一**的 Session ID。
2. 服务端将 Session ID **写入客户端的 Cookie**。
3. 客户端每次访问时携带这个 Session ID，服务端**查找到对应的用户信息**。

### **Session 的优点**

- **数据保存在服务端：** 安全性高，防止用户篡改。
- **适合存储敏感数据：** 如用户权限、购物车信息等。

### **Session 的不足**

- **依赖服务端资源：** 用户多时可能占用大量内存，服务端压力大。
- **跨域受限：** 同源策略限制了 Session 的跨域应用。

### **代码示例**

用 Java 实现 Session：

```java
// 用户登录时
HttpSession session = request.getSession();
session.setAttribute("user", user);
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

请求其他接口时获取 Session：

```java
User user = (User) session.getAttribute("user");
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## **四、Token：独立的“访问通行证”**

### **什么是 Token？**

Token 是**服务端生成的身份凭证**，通常包含**加密后的用户信息**。客户端每次请求时携带 Token，服务端验证 Token 的合法性，确认用户身份。

> **比喻：**
>  Token 就像一个一次性电子门禁卡，凭卡就能进入大楼。门禁卡本身不会暴露你的信息，但系统能识别它。

### **Token 的优点**

- **无需存储状态：** 服务端不需要保存 Token，适合分布式系统。
- **灵活性强：** 可以传递到不同的子系统，支持跨域。
- **安全性高：** 使用**加密技术**，难以伪造。

### **Token 的不足**

- **验证成本高：** 每次都需要解密 Token，性能开销较大。
- **容易泄露：** 如果 Token 被窃取，可能被滥用。

### **JWT：最流行的 Token 格式**

JWT（JSON Web Token）是一种自包含的 Token 格式，分为三部分：

1. **Header：** 描述 Token 的类型和加密算法。
2. **Payload：** **用户信息**（如 ID、角色）。
3. **Signature（服务端指定一个签名密钥）：** 加密**签名**，防止篡改。

JWT 示例：

```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VySWQiOjEyMzQ1LCJyb2xlIjoiYWRtaW4ifQ.
s5KCUO-qT-d0kPHiWUZf8CkIb1Xx9R97k5NgwMuEQHQ
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

服务端验证签名后，解密 Payload 获取用户信息。

### **代码示例**

以下是使用 Java 实现生成和验证 JWT 的代码示例，借助开源库 **jjwt (Java JSON Web Token)** 完成。

#### 生成 Token

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import java.util.Date;

public class JwtExample {
    public static void main(String[] args) {
        String secretKey = "secretKey"; // 秘钥，用于加密签名
        long expirationTime = 3600000; // 1小时（毫秒）

        // 当前时间
        Date now = new Date();

        // 生成 Token
        String token = Jwts.builder()
                .setSubject("12345") // 用户 ID 或其他标识
                .setIssuedAt(now)    // 签发时间
                .setExpiration(new Date(now.getTime() + expirationTime)) // 过期时间
                .signWith(SignatureAlgorithm.HS256, secretKey) // 使用 HMAC-SHA256 签名
                .compact();

        System.out.println("生成的 Token: " + token);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

#### 验证 Token

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;

public class JwtVerifyExample {
    public static void main(String[] args) {
        String secretKey = "secretKey"; // 与生成 Token 的秘钥一致

        // 生成的 Token 示例
        String token = "your_generated_token_here";

        try {
            // 解析 Token
            Claims claims = Jwts.parser()
                    .setSigningKey(secretKey) // 设置秘钥
                    .parseClaimsJws(token)   // 验证并解析
                    .getBody();

            // 获取 Token 中的用户 ID
            String userId = claims.getSubject();

            System.out.println("验证通过，用户 ID: " + userId);
        } catch (Exception e) {
            System.out.println("Token 验证失败: " + e.getMessage());
        }
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

#### 依赖配置（Maven 示例）

在使用以上代码前，需要在 `pom.xml` 中添加 `jjwt` 库的依赖：

```XML
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
</dependency>
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

#### 输出示例

生成 Token：

```
生成的 Token: eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NSIsImlhdCI6MTY1NjUwOTYwMCwiZXhwIjoxNjU2NTEzMjAwfQ.W_4hs3KkHhWzv5N48-LH38GxzNlRS0RNQhMB-uGcKxQ
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

验证 Token：

```
验证通过，用户 ID: 12345
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

------

## **五、三者大对比**

| 特性             | Cookie                   | Session                | Token                  |
| ---------------- | ------------------------ | ---------------------- | ---------------------- |
| **存储位置**     | 客户端                   | 服务端                 | 客户端                 |
| **依赖性**       | 依赖浏览器机制           | 依赖 Session ID        | 无需依赖               |
| **安全性**       | 容易被窃取和篡改         | 高（数据存储在服务端） | 较高（加密技术）       |
| **跨域支持**     | 较差（受 SameSite 限制） | 较差                   | 好（适合 RESTful API） |
| **典型应用场景** | 记录用户偏好、登录状态   | 管理用户会话           | 分布式系统、前后端分离 |

------

## **六、实际场景如何选择？**

1. **简单系统：**如果只是管理基本登录状态，小型应用用 **Session + Cookie** 就够了。
2. **前后端分离：**现代 Web 应用通常选用 **Token**，尤其是 **JWT**，灵活性高。
3. **高安全场景：**如果需要存储**敏感数据**（如支付信息），建议使用 **Session**，数据全在服务端。

------

## **七、总结**

Session、Cookie 和 Token 是解决登录状态管理问题的三种经典方式，各有优缺点：

- **Cookie**：适合轻量化存储，但安全性略低。
- **Session**：适合用户会话管理，安全性高但资源消耗较大。
- **Token**：灵活且适合分布式系统，是现代前后端分离的标配。

*理解它们的原理和优劣势，才能选择最适合的方案为你的系统赋能，希望这篇博客能帮助你理解这三者的关系和区别，以及各自的应用场景！*
