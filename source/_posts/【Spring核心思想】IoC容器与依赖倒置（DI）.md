---
title: 【Spring核心思想】IoC容器与依赖倒置（DI）
keywords: []
categories:
  - [后端,框架,Spring]
tags:
  - Spring
  - IoC
  - DI
date: 2024-12-21 11:00:00
updated:
description: "在日常开发中如何优雅地管理对象创建与依赖；介绍 Spring 的 IoC/DI 与 MyBatis 动态代理的作用。"
cover: "https://s2.loli.net/2026/01/12/qMSVNWespi3KlLz.png"
---
> 在日常开发中，我们总会面临一个问题：**如何优雅地管理对象的创建和依赖？** 你可能会写一堆代码来手动构造对象，但这种方式繁琐且难以维护。而当项目变得复杂，依赖链拉长，手动管理对象的方式很快就会捉襟见肘。
>
> ------
>
> 这时，**Spring 的 IoC 和 DI 机制**便是解放双手的利器，它让开发者专注于业务逻辑，容器则负责对象的创建与依赖管理。与此同时，**MyBatis** 的动态代理更是省去了为每个接口手动实现类的麻烦，极大地提高了效率。
>
> ------
>
> 但你有没有想过，Spring 是如何找到你的类并自动注入依赖的？MyBatis 又是如何在没有实现类的情况下完成数据库操作的？如果你也有这些疑问，那恭喜你，今天的内容正是为你准备的！

## 一、控制反转 IoC

**控制反转（Inversion of Control, IoC）** 是一种设计思想。它强调将控制权从对象本身转移到容器中。服务中心（容器）负责管理各种资源（对象和依赖）。用户（对象）需要资源时，服务中心将资源提供给它。容器控制**对象的创建、依赖注入和生命周期管理**。

**在传统编程中**，对象需要自己控制依赖的创建和管理；在 IoC 中，这些任务由容器负责。

容器根据 Bean 的依赖关系，通过 DI 注入所需的依赖。

------

## 二、依赖倒置 DI

### 1. 详细概念

**依赖注入（Dependency Injection, DI）** 是 IoC 的**具体实现方式**之一。

它通过**注入**的方式，将对象需要的依赖提供给它，而不是由对象自己去创建，Spring会按需创建相应的对象，通过构造器、Setter 方法或字段注入，把依赖传递给对象。

### 2. Spring 中 DI 的实现原理

1. **声明依赖**：使用注解（`@Component`、`@Service`、`@Repository`）将类标记为 Spring 容器管理的 Bean。
2. **注入依赖**：Spring 在启动时扫描类路径，自动检测依赖，并通过 `@Autowired` 注解注入相应的Bean。
3. **容器提供依赖**：Spring 容器会根据配置文件或注解，实例化对象并注入到需要的地方。

------

## 三、注册Bean过程：以 Spring+Mybatis 为例

### 1. Spring 是如何通过注解注册 Bean 的

Spring 通过 **组件扫描（Component Scanning）** 和 **注解识别** 将类注册为 Bean。

- **注解识别**：包括 `@Component`、`@Service`、`@Repository`、`@Controller` 等。
- **特定集成注解**：如 MyBatis 的 `@Mapper`，它告诉 Spring 将标注的接口注册为 Bean，并交由 MyBatis 动态代理生成实现类。

**注册过程**：

- Spring 启动时会扫描指定的包路径。
- 找到标注了这些注解的类或接口，并**注册到 IoC 容器**中，形成 Bean 定义。

------

### 2. MyBatis 是如何动态生成 UserMapper 的实现类的

**UserMapper 是接口，没有具体实现类**。MyBatis 会利用 `@Mapper` 注解，结合 Mapper 配置文件或注解中的 SQL 语句，**动态生成代理实现类**。

**代理类生成过程**：

1. **动态代理机制**：MyBatis 使用 JDK 动态代理，为每个 Mapper 接口生成一个代理类。
2. **InvocationHandler**：代理类拦截所有对接口方法的调用，将它们转发到 MyBatis 的核心组件（如 `SqlSession`）执行 SQL。
3. 执行 SQL：
   - 根据方法名或注解，定位 SQL 配置。
   - 使用 MyBatis 的 `Executor` 执行 SQL 并返回结果。

------

### 3. `@Autowired` 注入过程

1. **扫描 Bean**：Spring 启动时，扫描 `UserServiceImpl` 和 `UserMapper`，分别标注了 `@Service` 和 `@Mapper`，将它们**注册为 Bean**。
2. **识别依赖**：Spring 在注册 `UserServiceImpl` Bean 时，检测到其字段 `userMapper` 被标注了 `@Autowired`，即是否依赖于其他 Bean。

【**注入逻辑**】

1. **找到目标 Bean**：

   - 在 IoC 容器中，根据类型 `UserMapper` 查找对应的 Bean。
   - 如果找到多个匹配 Bean，Spring 会结合 Bean 名称或 `@Qualifier` 注解解决冲突。

2. **依赖注入**：

   - Spring 使用 Java 反射机制为 `userMapper` 字段赋值。

   - 具体实现伪代码如下：

     ```java
     // 获取字段
     Field field = UserServiceImpl.class.getDeclaredField("userMapper");
     // 使私有字段可访问
     field.setAccessible(true);
     // 将找到的 UserMapper Bean 注入到 userServiceImpl 实例
     field.set(userServiceImplInstance, userMapperBean);
     ```


------

### 4. 总结：Spring 与 MyBatis 的结合

**Spring**：

- 提供 IoC 容器，扫描 Bean，处理依赖注入。
- 通过反射将 `UserMapper` 动态代理对象注入到 `UserServiceImpl`。

**MyBatis**：

- 动态生成 `UserMapper` 的代理实现类，负责将方法调用转化为 SQL 查询。
- 代理类中通过 `InvocationHandler` 将方法调用委托给 MyBatis 的 SQL 执行器。

------

### 附加：代理类与 UserMapper 实现类的差异

> - **代理类**：
>   - 动态生成，没有手写实现代码。
>   - 通过拦截接口方法，转发到 MyBatis 核心组件处理。
> - **普通实现类**：
>   - 静态定义，需手动实现每个方法的逻辑。

**示例对比**：

```java
// 动态代理生成的代理类示例
public class UserMapperProxy implements UserMapper {
    private final SqlSession sqlSession;

    public UserMapperProxy(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public User findById(int id) {
        // 将方法调用转化为 MyBatis 的 SQL 执行
        return sqlSession.selectOne("namespace.findById", id);
    }
}

// 普通实现类（手动实现）
public class UserMapperImpl implements UserMapper {
    @Override
    public User findById(int id) {
        // 自己写逻辑，连接数据库，执行 SQL
        return executeSQL("SELECT * FROM user WHERE id = ?", id);
    }
}
```

动态代理的优势在于：

1. **代码复用性高**：只需定义接口和 SQL，无需重复写实现类。
2. **与 SQL 配置无缝对接**：方便维护和管理 SQL 语句。