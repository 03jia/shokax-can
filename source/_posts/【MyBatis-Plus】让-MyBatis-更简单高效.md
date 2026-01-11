---
title: 【MyBatis-Plus】让 MyBatis 更简单高效
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2025-01-01 18:44:37
updated:
description:
cover:
---
>  如果你曾经使用过 **MyBatis**，你一定知道它的强大和灵活。然而，随着项目规模的增长，手写 SQL 成为了一件既繁琐又容易出错的事。这时，**MyBatis-Plus**（简称 MP）应运而生，它为 MyBatis 增强了许多功能，帮我们极大地提高开发效率。接下来，我们就来聊聊 MyBatis-Plus 的基本介绍和使用，让你轻松上手！

------

## **一、MyBatis-Plus 是什么？**

MyBatis-Plus 是一款基于 MyBatis 的增强工具，它的口号是 “**为简化开发而生**”。简单来说，MP 在保留 MyBatis 灵活性的同时，帮我们封装了很多常用的功能，比如：

- **自动生成单表的 CRUD 操作**，再也不用手写增删改查。
- 提供强大的 **条件构造器**，动态生成查询语句。
- 内置了 **分页插件**，让分页查询变得超级简单。

> 它适合那些既想要使用 ORM 工具，又希望对 SQL 有一定掌控力的开发者。

------

## **二、MyBatis-Plus 的核心特性**

### 1. **开箱即用的 CRUD**

MP 提供了 `BaseMapper` 接口，只需继承它，你就能获得所有单表的增删改查方法。例如：

```java
UserMapper userMapper = ...; // Mapper 注入
User user = userMapper.selectById(1L); // 根据主键查询
```

就这么简单，一行代码搞定！

### 2. **强大的条件构造器**

还记得那些复杂的 WHERE 条件 SQL 吗？MP 的条件构造器能帮你轻松搞定：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("age", 18).like("name", "Jack");
List<User> users = userMapper.selectList(queryWrapper);
```

动态查询再也不用拼接字符串了，是不是很香？

### 3. **分页插件**

分页查询是开发中最常见的需求之一。MP 提供了内置的分页插件，只需简单配置，就可以用如下代码实现分页：

```java
Page<User> page = new Page<>(1, 10); // 第1页，每页10条
Page<User> result = userMapper.selectPage(page, null);
System.out.println(result.getRecords()); // 当前页数据
System.out.println(result.getTotal()); // 总记录数
```

### 4. **自动代码生成器**

如果你讨厌一遍遍写 `Entity`、`Mapper`、`Service` 的模板代码，MP 的代码生成器可以帮你一次性生成全部内容，让你专注于业务逻辑。

------

## **三、MyBatis-Plus 的快速入门**

接下来，我们用一个简单的例子来说明如何使用 MyBatis-Plus。

### 1. **引入依赖**

在你的项目中添加 MyBatis-Plus 的 Maven 依赖：

> 注意这里是以 **Spring Boot 3** 整合 MyBatis Plus，**Spring Boot 2** 的话要导入另一个包mybatis-plus-boot-starter

```XML
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.9</version>
</dependency>
```

### 2. **配置数据源**

在 `application.yml` 中配置数据源（以操作 **MySQL** 为例）和 **MyBatis-Plus**：

```XML
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test_db
    username: root
    password: 123456
  mybatis-plus:
    configuration:
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 把sql语句打印到控制台
```

### 3. **创建实体类**

假设我们的数据库中有一个 `user` 表，我们可以创建对应的实体类：

```java
@Data    // Lombok注解，生成一个JavaBean常用的方法
@TableName("user")    // 指定绑定的数据表名
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

### 4. **创建 Mapper 接口**

继承 `BaseMapper` 即可：

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

### 5. **调用 CRUD 接口**

在 `Service` 或 `Controller` 中使用：

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserMapper userMapper;

    @GetMapping("/list")
    public List<User> listUsers() {
        return userMapper.selectList(null); // 查询所有用户
    }
}
```

运行后访问 `/user/list`，你会发现用户数据已经成功查询出来了！

确实，`IService` 是 MyBatis-Plus 提供的一种更高层次的接口封装，它将常用的业务逻辑操作进一步抽象，方便我们在 Service 层使用，避免直接操作 Mapper。下面补充关于 `IService` 和 `ServiceImpl` 的使用方法。

------

### **6. 使用 IService 接口实现业务逻辑**

MyBatis-Plus 提供了 `IService` 和 `ServiceImpl`，帮我们在 Service 层简化基础逻辑操作，同时保持扩展性。

#### **① 创建 Service 接口**

继承 `IService<T>` 接口，定义业务逻辑层的接口：

```java
public interface UserService extends IService<User> {
    // 自定义业务方法（如果有）
}
```

#### **② 实现 Service 接口**

创建实现类，继承 MyBatis-Plus 提供的 `ServiceImpl` 基类，自动注入 Mapper：

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    // 可以在这里扩展自定义的业务逻辑方法
}
```

#### **③ 使用 Service**

在 Controller 或其他业务类中注入 `UserService`，调用封装的 CRUD 方法：

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/list")
    public List<User> listUsers() {
        return userService.list(); // 查询所有用户
    }

    @PostMapping("/add")
    public boolean addUser(@RequestBody User user) {
        return userService.save(user); // 新增用户
    }

    @DeleteMapping("/delete/{id}")
    public boolean deleteUser(@PathVariable Long id) {
        return userService.removeById(id); // 根据 ID 删除用户
    }
}
```



------

**【IService 的优势】**

1. **抽象通用逻辑：**将通用的 CRUD 方法（如 `save`、`removeById`、`getById`、`list` 等）封装到接口中，避免每个 Service 重复实现。
2. **Mapper 解耦：**在业务层调用 `IService` 提供的方法，无需直接操作 Mapper，增强了代码的层次性和可维护性。
3. **可扩展性：**如果有自定义业务逻辑，只需在 `UserService` 接口和 `UserServiceImpl` 类中实现，无需修改基础代码。

------

### **7. 最佳实践**

- 如果项目规模较小，直接使用 `BaseMapper` 实现 CRUD，简单快捷。
- 如果项目复杂且需要 Service 层管理业务逻辑，建议使用 `IService` 和 `ServiceImpl`，规范代码结构。

------

## **四、为什么要用 MyBatis-Plus？**

1. **手写增删改查 SQL 的疲劳感：**
    想象一下，你有 10 个表，每个表都要写四五个基础操作的 SQL，是不是一想到就想摆烂？
2. **动态拼接 SQL 的复杂性：**
    有些查询条件需要用户输入，这就意味着动态 SQL。手写拼接不仅代码冗长，而且容易出错。
3. **分页查询的重复劳动：**
    传统分页查询需要写 SQL，还要计算总数。如果有工具自动搞定，岂不是省心又省力？

------

## **五、总结**

**MyBatis-Plus** 是一款极大提升开发效率的工具，它不仅让我们摆脱了繁琐的基础代码编写，还提供了强大的功能来满足各种场景需求，我们可以在项目开发中更加专注业务逻辑，可以说是开发行云流水。通过今天的分享，相信你已经对 MP 有了初步的了解。

如果你对 MP 感兴趣，不妨动手尝试一下，亲自体验它的便捷性！之后，我会继续进一步展开来讲解开发中常用的部分，以及一些核心机制等内容，感兴趣的朋友可以关注留意一下噢！