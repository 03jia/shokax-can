---
title: 【MyBatis-Plus 分页插件】深入分析和实战解析
keywords: []
categories:
  - [后端,框架,MyBatis-plus]
tags:
  - 分页
  - MyBatis-Plus
date: 2025-01-12 21:28:06
updated:
description: "介绍了 MyBatis-Plus 的分页机制与 `IPage`/`Page` 的用法，说明分页在减少数据传输和提升性能中的作用。"
cover: "https://s2.loli.net/2026/01/11/s6AKfIi1G8THlOy.jpg"
---
> 分页是 Web 应用开发中的高频需求，而在 MyBatis 的生态中，<font color="red">**MyBatis-Plus 分页插件`PaginationInnerInterceptor`**</font> 和 **MyBatis 的 `PageHelper`** 是两种常见的实现方案。本文将通过 **工作机制**、**使用方法** 和 **细节剖析**，带你循序渐进地掌握这两种方式，并为你的项目选择提供指导。

------

## 一、什么是分页？

分页的核心目标是**减少数据传输量和前端渲染压力**。通过限制每次查询的结果数量，分页能够显著提升用户体验。

假设数据库表中有 100 万条数据，一次性查询所有数据并展示显然不合理。而分页查询可以按需获取，比如：

- 每页显示 10 条数据
- 用户可自由跳转页码

------

## 二、MyBatis-Plus 分页插件

MyBatis-Plus 内置分页插件 `PaginationInnerInterceptor` ，提供了简单高效的分页功能。

### 1. 工作机制

1. **拦截查询**：分页插件通过拦截 SQL 查询，在执行前修改 SQL，自动添加分页条件，例如 `LIMIT` 和 `OFFSET`。
2. **查询总记录数**：插件会为每次分页查询生成两条 SQL：
   - 第一条：查询分页数据
   - 第二条：查询总记录数（`SELECT COUNT(*)`）
3. **封装结果**：返回的数据会封装到 `IPage` 对象中，包括分页数据、总记录数、总页数等。

### 2. `IPage` 和 `Page` 的介绍

在 MyBatis-Plus 的分页功能中，**`IPage`** 是分页结果的通用接口，而 **`Page`** 是其实现类，主要用于封装分页请求和结果。

------

#### ① 两者的关系

- **`IPage`** 是一个接口，定义了分页结果的基本结构。
- **`Page`** 是 `IPage` 的实现类，既可以用来作为分页请求参数，也可以作为查询结果的载体。

#### ② `IPage` 接口方法

`IPage` 提供了多个分页相关的抽象方法，常用方法包括：

| **方法**                      | **描述**                 |
| ----------------------------- | ------------------------ |
| `getRecords()`                | 获取当前页的数据记录列表 |
| `getTotal()`                  | 获取总记录数             |
| `getPages()`                  | 获取总页数               |
| `getCurrent()`                | 获取当前页码             |
| `getSize()`                   | 获取每页记录数           |
| `setRecords(List<T> records)` | 设置当前页数据           |
| `setTotal(long total)`        | 设置总记录数             |

------

#### ③ `Page` 的主要属性

`Page` 继承了 `IPage`，并实现了所有方法，同时增加了一些属性：

##### 核心属性

| **属性**           | **类型**          | **描述**             |
| ------------------ | ----------------- | -------------------- |
| `current`          | `long`            | 当前页码             |
| `size`             | `long`            | 每页显示的记录数     |
| `total`            | `long`            | 总记录数             |
| `pages`            | `long`            | 总页数               |
| `records`          | `List<T>`         | 当前页的记录数据列表 |
| `orders`           | `List<OrderItem>` | 排序规则             |
| `optimizeCountSql` | `boolean`         | 是否优化 Count 查询  |
| `searchCount`      | `boolean`         | 是否查询总记录数     |

------

#### ④ 示例：如何使用 `Page` 和 `IPage`

##### 查询示例

```java
// 创建分页请求对象
Page<User> pageRequest = new Page<>(1, 10);

// 执行分页查询
IPage<User> result = userMapper.selectPage(pageRequest, null);

// 获取分页信息
System.out.println("当前页：" + result.getCurrent());
System.out.println("每页大小：" + result.getSize());
System.out.println("总记录数：" + result.getTotal());
System.out.println("总页数：" + result.getPages());
System.out.println("当前页数据：" + result.getRecords());
```

##### 属性扩展

通过 `Page` 对象，可以轻松获取或设置分页请求和结果：

```java
// 设置排序规则
pageRequest.setOrders(List.of(OrderItem.asc("name"), OrderItem.desc("create_time")));

// 获取排序规则
System.out.println("排序规则：" + pageRequest.getOrders());
```

> `OrderItem` 是 MyBatis-Plus 提供的一个 **排序条件封装类**，用于表示查询中的排序规则。可以通过它为分页查询添加多字段排序，并指定升序或降序。
>****
> 这里就不展开来讲述了，这个在开发中用的不多，因为可以通过`SQL`语句中来指定排序规则通常，这个类的应用场景就是简化代码和与分页功能更好的结合使用，朋友们如果想了解的话可以评论区留言，我后续再补充这部分讲解！

------

### 3. 配置分页插件

#### ① 添加依赖

如果项目中已经引入了 MyBatis-Plus，只需确保版本支持分页即可。以下为 Maven 依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3</version>
</dependency>
```

#### ② 启用分页插件

在配置类中添加分页插件：

```java
@Configuration
public class MyBatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
}
```

------

### 4. 使用方法

#### ① 分页查询

MyBatis-Plus 提供了 `Page` 对象，作为分页参数和结果的载体：

```java
public void queryWithMPPage() {
    // 创建分页对象（当前页，每页条数）
    Page<User> page = new Page<>(1, 10);

    // 调用分页查询方法
    IPage<User> result = userMapper.selectPage(page, null);

    // 打印分页结果
    System.out.println("总记录数：" + result.getTotal());
    System.out.println("总页数：" + result.getPages());
    result.getRecords().forEach(System.out::println);
}
```

#### ② 自定义条件查询

结合 `QueryWrapper` 添加条件：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("status", 1).like("name", "Tom");

Page<User> page = new Page<>(1, 5);
IPage<User> result = userMapper.selectPage(page, queryWrapper);
```

------

### 5. 复杂 SQL 配合 XML

确实！结合 `XML` 使用分页功能是实际开发中常见的一种需求，尤其在需要处理复杂 SQL 时。以下是对这一部分内容的详细讲解：

在一些复杂查询场景下，分页通常需要配合手写 SQL 来完成，例如多表联查、动态条件等。MyBatis-Plus 支持在 `Mapper` 方法中传入 `Page` 参数，并在 `XML` 中书写具体的 SQL。

#### ① 步骤详解

1. **定义 Mapper 方法**：在 `Mapper` 接口中定义一个带分页参数的方法：

   ```java
   List<User> selectUsers(Page<User> page, @Param("name") String name);
   ```

   - **第一个参数**：`Page` 对象，MyBatis-Plus 自动识别它为分页参数。
   - **其他参数**：可以传入查询条件，如这里的 `name`。

2. **在 XML 中书写分页 SQL**：在对应的 `Mapper.xml` 文件中书写 SQL，示例如下：

   ```xml
   <select id="selectUsers" resultType="User">
       SELECT *
       FROM user
       WHERE name LIKE CONCAT('%', #{name}, '%')
   </select>
   ```

   **注意**：这里没有 `LIMIT` 语句，MyBatis-Plus 会自动在 SQL 上拼接分页参数。

3. **调用 Mapper 方法**：在服务层调用时传入 `Page` 和其他查询条件：

   ```java
   Page<User> page = new Page<>(1, 10); // 页码1，每页10条
   List<User> users = userMapper.selectUsers(page, "Tom");
   
   // 获取分页数据
   List<User> records = page.getRecords();
   System.out.println("总记录数：" + page.getTotal());
   ```

------

为什么能实现分页？是否拦截了方法？没错，MyBatis-Plus 是通过 **插件机制** 实现分页功能的，分页的核心在于 **MyBatis 插件拦截器**。

#### ② 工作机制

1、**拦截器原理**：MyBatis 提供了插件机制，允许开发者拦截以下四种方法：

- `Executor` 的 `query` 方法（用于查询操作）。
- `Executor` 的 `update` 方法（用于更新操作）。
- 其他如 `prepare`、`parameterize` 等。

MyBatis-Plus 的分页功能就是通过 **拦截 `Executor.query` 方法**，在 SQL 执行之前对其进行修改。

2、**执行过程**

- 当 Mapper 方法被调用时，MyBatis 会生成对应的 SQL。
- 插件会在 SQL 执行之前，判断是否存在 `Page` 参数。
- 如果存在，插件会：
  - 自动**生成 `COUNT` 查询**，用于获取总记录数。
  - 自动**为 SQL 添加分页参数**（如 `LIMIT` 和 `OFFSET`）。
- 插件会将分页结果和总记录数**封装回 `Page` 对象**。

3、**XML 适配**：因为 `Mapper` 中的方法和 `XML` 是一一对应的，所以只要方法参数中包含了 `Page` 对象，MyBatis-Plus 的分页插件就能捕获并处理。对于开发者来说，这个过程完全透明，无需手动拼接分页逻辑。

------

#### ③ 示例扩展：复杂 SQL 的分页

假如有一个需要多表联查的场景：

```xml
<select id="selectUserOrders" resultType="UserOrderDTO">
    SELECT u.id, u.name, o.order_id, o.amount
    FROM user u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.name LIKE CONCAT('%', #{name}, '%')
</select>
```

##### 调用

```java
Page<UserOrderDTO> page = new Page<>(1, 10); // 每页10条
List<UserOrderDTO> userOrders = userMapper.selectUserOrders(page, "Tom");

// 获取分页结果
System.out.println("总记录数：" + page.getTotal());
page.getRecords().forEach(System.out::println);
```

##### 插件自动生成的 SQL

1. 查询总记录数：

   ```sql
   SELECT COUNT(*)
   FROM user u
   LEFT JOIN orders o ON u.id = o.user_id
   WHERE u.name LIKE '%Tom%';
   ```

2. 查询分页数据：

   ```sql
   SELECT u.id, u.name, o.order_id, o.amount
   FROM user u
   LEFT JOIN orders o ON u.id = o.user_id
   WHERE u.name LIKE '%Tom%'
   LIMIT 0, 10;
   ```

****

>==**【小结一下】**==
>
>通过 XML 和 MyBatis-Plus 的分页插件结合，可以轻松实现复杂 SQL 的分页功能：
>
>- 不需要手动拼接分页逻辑。
>- 依赖 MyBatis 插件机制，分页参数的拼接完全透明。
>- 支持动态传入分页条件和查询参数，灵活性极高。
>
>这种方式既**保留了 `XML` 的灵活性，又简化了分页逻辑**，开发效率和代码可读性大幅提升

### 6. `PaginationInnerInterceptor` 优势与特点

| **特点**           | **说明**                                             |
| ------------------ | ---------------------------------------------------- |
| **无侵入性**       | 基于插件实现，无需修改原有方法                       |
| **功能全面**       | 支持条件查询、排序等功能                             |
| **与 MP 无缝衔接** | 自动适配 MP 的其他功能（如逻辑删除、乐观锁）         |
| **性能优化**       | 插件内置了多种优化策略，如防止分页过大导致的性能问题 |

------

## 三、`PageHelper` 分页

`PageHelper` 是专为 MyBatis 设计的分页插件，功能强大且适配性广。

### 1. 工作机制

1. **动态注入分页参数**：使用 `PageHelper.startPage` 方法时，会动态生成分页条件。
2. **拦截执行 SQL**：与 MyBatis-Plus 类似，PageHelper 会修改 SQL，添加分页语句。
3. **结果封装**：返回的结果由 `PageInfo` 对象封装，包含分页数据、总记录数等。

------

### 2. 配置 `PageHelper`

#### ① 添加依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.6</version>
</dependency>
```

#### ② 配置文件中启用 PageHelper

```yaml
pagehelper:
  helper-dialect: mysql
  reasonable: true
  support-methods-arguments: true
  params: count=countSql
```

------

### 3. 使用方法

#### ① 分页查询

在执行查询前调用 `PageHelper.startPage`：

```java
public void queryWithPageHelper() {
    // 设置分页参数（页码，页面大小）
    PageHelper.startPage(1, 10);

    // 调用普通查询方法
    List<User> users = userMapper.selectAll();

    // 封装分页结果
    PageInfo<User> pageInfo = new PageInfo<>(users);
    System.out.println("总记录数：" + pageInfo.getTotal());
    System.out.println("当前页数据：" + pageInfo.getList());
}
```

#### ② 条件查询

`PageHelper` 本身不提供条件构造器，但可以结合自定义 SQL 使用：

```java
PageHelper.startPage(1, 5);
List<User> users = userMapper.queryWithConditions("Tom", 1);
PageInfo<User> pageInfo = new PageInfo<>(users);
```

------

### 4. 优势与特点

| **特点**         | **说明**                                       |
| ---------------- | ---------------------------------------------- |
| **适配性强**     | 可用于任何 MyBatis 项目，无需依赖 MyBatis-Plus |
| **支持复杂查询** | 可以结合自定义 SQL 或复杂逻辑                  |
| **灵活性高**     | 可动态设置分页参数                             |
| **侵入性较低**   | 不改变原有 Mapper 方法                         |

------

## 四、两种分页方式对比

| **对比项**   | **`PaginationInnerInterceptor`**             | **`PageHelper`**                   |
| ------------ | ------------------------------------- | -------------------------------- |
| **集成难度** | 简单，MyBatis-Plus 自带分页功能       | 需额外引入依赖                   |
| **分页参数** | 使用 `Page` 对象，封装清晰            | 动态调用 `startPage`，侵入性稍高 |
| **适用场景** | 推荐使用 MyBatis-Plus 项目            | 推荐在普通 MyBatis 项目中使用    |
| **扩展性**   | 更适合 MyBatis-Plus，支持动态条件构造 | 适用于多种场景，支持多表复杂查询 |
| **性能优化** | 内置多种优化策略                      | 无内置优化机制                   |

------

## 五、总结与选择建议

- **`PaginationInnerInterceptor `**：如果你的项目已经使用了 MyBatis-Plus，分页插件是更推荐的选择，它能够与 MP 的其他功能无缝配合，简单高效。
- **`PageHelper`**：如果你的项目是普通 MyBatis 项目，PageHelper 是一个成熟且灵活的解决方案，支持复杂场景。
