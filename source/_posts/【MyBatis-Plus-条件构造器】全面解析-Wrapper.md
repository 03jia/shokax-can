---
title: 【MyBatis-Plus 条件构造器】全面解析 Wrapper
keywords: []
categories:
    - [后端,框架,MyBatis-plus]
tags:
    - MyBatis-Plus
date: 2025-01-08 21:58:15
updated:
description: "全面解析 MyBatis-Plus 的条件构造器（Wrapper），用于动态拼接 SQL 条件、链式调用与复杂逻辑处理。"
cover: "https://s2.loli.net/2026/01/12/CmFdTEKWNGphgab.jpg"
---
> 在 MyBatis-Plus 中，<font color="red"> **条件构造器**</font> 是一个强大的工具，能够帮助我们灵活地构建 SQL 查询条件，而无需手写繁琐的 SQL 语句。本文将从基础到高级，带你全面了解条件构造器的使用方法及其链式构造能力。

------

## 一、什么是条件构造器？

条件构造器是 MyBatis-Plus 提供的 **动态 SQL 拼接工具**，主要用于生成 SQL 查询中的 **`WHERE` 条件部分**，支持**动态拼接、链式调用以及条件逻辑控制**。

### 常见的条件构造器

1. **`QueryWrapper`**：用于通用的条件查询。
2. **`UpdateWrapper`**：用于更新操作的条件构造。
3. **`LambdaQueryWrapper`** 和 **`LambdaUpdateWrapper`**：使用 Lambda 表达式构造条件，避免直接写字段名称导致的拼写错误。

------

## 二、基础用法

### 常用方法一览

| 方法                      | 描述                  | 示例                                                         | 补充说明                                                     |
| ------------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `eq`                      | 等于                  | `.eq("status", 1)`                                           | 常用于精确匹配某个字段的值，例如查询某种状态的数据。         |
| `ne`                      | 不等于                | `.ne("status", 0)`                                           | 用于排除特定值的记录，例如排除状态为 0 的数据。              |
| `gt` / `ge`               | 大于 / 大于等于       | `.gt("age", 18)`                                             | 可用于筛选范围，例如筛选 18 岁以上用户。                     |
| `lt` / `le`               | 小于 / 小于等于       | `.lt("age", 60)`                                             | 结合 `gt`，实现区间查询，例如 18-60 岁之间的用户。           |
| `like` / `notLike`        | 模糊查询 / 非模糊查询 | `.like("name", "Tom")`                                       | `like` 通常用于字符串匹配，例如姓名包含某字；`notLike` 用于排除模糊匹配的记录。 |
| `likeLeft` / `likeRight`  | 左 / 右模糊查询       | `.likeLeft("name", "Tom")` / `.likeRight("name", "Tom")`     | 左模糊适用于以某个关键字结尾的情况，右模糊适用于以关键字开头的情况。 |
| `isNull` / `isNotNull`    | 字段是否为空 / 不为空 | `.isNotNull("email")`                                        | 判断字段值是否存在，适用于有些字段可能为空值的情况，例如邮箱地址。 |
| `in` / `notIn`            | 在列表中 / 不在列表中 | `.in("id", Arrays.asList(1, 2, 3))` / `.notIn("id", Arrays.asList(4, 5))` | `in` 通常用于批量匹配多个条件，例如查询多个 ID 的记录。      |
| `between` / `notBetween`  | 在区间内 / 不在区间内 | `.between("age", 18, 60)`                                    | 比 `gt` 和 `lt` 更简洁，适用于范围筛选。                     |
| `orderBy` / `orderByDesc` | 排序                  | `.orderByDesc("create_time")` / `.orderByAsc("age")`         | 支持多字段排序，例如 `.orderByAsc("age").orderByDesc("create_time")`。 |
| `groupBy`                 | 分组                  | `.groupBy("status")`                                         | 通常与聚合函数结合使用，例如统计每种状态的记录数。           |
| `having`                  | 分组后的筛选条件      | `.groupBy("status").having("count(*) > 5")`                  | 适用于对分组结果的进一步筛选，例如过滤分组中记录数少于 5 的组。 |
| `last`                    | 手动拼接 SQL 片段     | `.last("LIMIT 1")`                                           | 手动拼接自定义的 SQL 片段，但需谨慎使用，确保不会引起 SQL 注入风险。 |
| `exists` / `notExists`    | 子查询条件            | `.exists("SELECT 1 FROM user WHERE user.id = order.user_id")` | 适用于判断某记录是否存在，结合子查询使用。                   |
| `apply`                   | 自定义 SQL 条件       | `.apply("DATE(create_time) = {0}", "2025-01-01")`            |                                                              |

### 1. QueryWrapper 的基本操作

示例：构造简单查询条件

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("status", 1).like("name", "Tom").orderByDesc("create_time");

List<User> users = userMapper.selectList(queryWrapper);
```

生成的 SQL：

```sql
SELECT * FROM user WHERE status = 1 AND name LIKE '%Tom%' ORDER BY create_time DESC;、
```

------

### 2. LambdaQueryWrapper 的基本操作

使用 Lambda 表达式**构建查询条件**，避免字段名硬编码导致的错误，可以传入一个**函数式接口**来动态获取查询的字段等。

示例：构造条件

```java
LambdaQueryWrapper<User> lambdaWrapper = new LambdaQueryWrapper<>();
lambdaWrapper.eq(User::getStatus, 1).like(User::getName, "Tom");

List<User> users = userMapper.selectList(lambdaWrapper);
```

**优点**：

- 避免直接使用**字符串字段名**。
- 修改实体类字段名时，条件构造器能自动适配。

------

### 3. UpdateWrapper 的基本操作

`UpdateWrapper` 用于**构造更新条件**，可以同时设置**更新内容和条件**。

示例：构造更新条件

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper.eq("id", 1).set("status", 2);

userMapper.update(null, updateWrapper);
```

生成的 SQL：

```sql
UPDATE user SET status = 2 WHERE id = 1;
```

------

## 三、高级用法

### 1. 链式调用的高级操作

条件构造器支持链式调用，能大幅提升代码的可读性。

#### 示例：根据条件动态添加查询

```java
String name = "Tom";
Integer age = null;

QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq(name != null, "name", name)
       .ge(age != null, "age", age);

List<User> users = userMapper.selectList(wrapper);
```

只会生成：

```sql
SELECT * FROM user WHERE name = 'Tom';
```

**注意**：`eq` 等方法的第一个参数为布尔值，控制条件是否拼接。

------

### 2. 嵌套查询条件

使用 `and` 或 `or` 结合嵌套条件，构造复杂查询。

#### 示例：嵌套条件

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.and(w -> w.eq("status", 1).or().eq("status", 2))
       .like("name", "Tom");

List<User> users = userMapper.selectList(wrapper);
```

生成的 SQL：

```sql
SELECT * FROM user WHERE (status = 1 OR status = 2) AND name LIKE '%Tom%';
```

------

### 3. LambdaQueryWrapper 的自定义排序

`LambdaQueryWrapper` 支持条件排序，可以动态调整排序字段。

#### 示例：动态排序

```java
boolean orderByAge = true;

LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getStatus, 1)
       .orderBy(orderByAge, true, User::getAge);

List<User> users = userMapper.selectList(wrapper);
```

**说明**：`orderBy` 的第一个参数控制是否应用排序（指定一个**条件**），第二个参数控制升序还是降序，第三个参数指定根据哪一列排序。

------

### 4. 使用条件构造器分页查询

与分页插件结合，条件构造器可以轻松实现分页查询。

#### 示例：分页查询

```java
IPage<User> page = new Page<>(1, 10); // 第1页，每页10条
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.ge("age", 18).le("age", 60);

IPage<User> result = userMapper.selectPage(page, wrapper);
```

生成的 SQL：

```sql
SELECT * FROM user WHERE age >= 18 AND age <= 60 LIMIT 0, 10;
```

------

## 四、链式条件构造器 ChainWrapper

MyBatis-Plus 提供了链式条件构造器，使查询与更新操作更加简洁流畅，主要包括 **`QueryChainWrapper` / `LambdaQueryChainWrapper`** 和 **`UpdateChainWrapper` / `LambdaUpdateChainWrapper`**。以下我们结合查询与更新操作详细讲解其使用方式。

------

### 1. 查询的链式构造器

MyBatis-Plus 提供 `query()` 和 `lambdaQuery()` 方法用于查询操作的链式调用，最终通过调用特定方法（如 `.list()` 或 `.one()`）返回结果。

#### 返回值方法

以下是 `QueryChainWrapper` / `LambdaQueryChainWrapper` 中常见的调用链终点方法及其返回值：

- **`.list()`**：返回符合条件的所有记录，类型为 `List<T>`。
- **`.one()`**：返回符合条件的单条记录，类型为 `T`。如果有多条记录，则会抛出异常。
- **`.count()`**：返回符合条件的记录数，类型为 `long`。
- **`.exists()`**：判断是否有符合条件的记录，返回 `boolean`。
- **`.map()`**：返回查询结果的 `Map` 表示，通常适用于单表查询。

```java
// 获取符合条件的所有记录
List<User> users = userService.lambdaQuery()
    .eq(User::getStatus, 1)
    .orderByDesc(User::getCreateTime)
    .list();

// 获取符合条件的单条记录
User user = userService.lambdaQuery()
    .eq(User::getId, 1)
    .one();

// 获取符合条件的记录总数
long count = userService.lambdaQuery()
    .eq(User::getStatus, 1)
    .count();

// 判断是否存在符合条件的记录
boolean exists = userService.lambdaQuery()
    .eq(User::getName, "Tom")
    .exists();
```

> ==**【注意】**==
>
> - 如果查询条件返回多条记录而使用 `.one()`，会抛出异常，因此建议对主键或唯一字段使用该方法。
> - `.exists()` 的效率较高，可用于快速判断。

#### 基本查询

- **`QueryChainWrapper`**（基于字段名）：

```java
List<User> users = userService.query()
    .eq("status", 1)
    .like("name", "Tom")
    .orderByDesc("create_time")
    .list();
```

- **`LambdaQueryChainWrapper`**（基于 Lambda 表达式）：

```java
List<User> users = userService.lambdaQuery()
    .eq(User::getStatus, 1)
    .like(User::getName, "Tom")
    .orderByDesc(User::getCreateTime)
    .list();
```

#### 动态条件查询

通过链式构造器可以动态拼接查询条件，避免不必要的条件参与 SQL 构造：

```java
String name = null;
Integer age = 25;

List<User> users = userService.lambdaQuery()
    .like(name != null, User::getName, name)
    .eq(age != null, User::getAge, age)
    .list();
```

当 `name` 为 `null` 时，生成的 SQL 会自动忽略对应条件：

```sql
SELECT * FROM user WHERE age = 25;
```

#### 分页查询

结合分页对象可以快速实现分页功能：

```java
IPage<User> page = userService.lambdaQuery()
    .eq(User::getStatus, 1)
    .orderByDesc(User::getCreateTime)
    .page(new Page<>(1, 10));
```

------

### 2. 更新的链式构造器

对于更新操作，MyBatis-Plus 提供了 `update()` 和 `lambdaUpdate()` 方法，支持基于链式条件构造更新数据。

#### 基本更新

- 使用 **`LambdaUpdateChainWrapper`** 更新指定条件的数据：

```java
boolean updated = userService.lambdaUpdate()
    .eq(User::getStatus, 1)
    .set(User::getName, "Tom Updated")
    .set(User::getAge, 30)
    .update();
```

**生成的 SQL**：

```sql
UPDATE user SET name = 'Tom Updated', age = 30 WHERE status = 1;
```

#### 动态字段更新

通过布尔值控制是否更新某些字段，实现动态更新逻辑：

```java
String name = null;
Integer age = 30;

boolean updated = userService.lambdaUpdate()
    .set(name != null, User::getName, name)
    .set(age != null, User::getAge, age)
    .eq(User::getStatus, 1)
    .update();
```

如果 `name` 为 `null`，生成的 SQL 将忽略对 `name` 的更新：

```sql
UPDATE user SET age = 30 WHERE status = 1;
```

#### 批量更新

通过 `in` 条件轻松实现批量更新：

```java
boolean updated = userService.lambdaUpdate()
    .in(User::getId, Arrays.asList(1, 2, 3))
    .set(User::getStatus, 2)
    .update();
```

**生成的 SQL**：

```sql
UPDATE user SET status = 2 WHERE id IN (1, 2, 3);
```

## 五、最佳实践

1. **优先使用 `LambdaQueryWrapper`**，能避免字段名**硬编码**的潜在错误，更适合复杂的查询场景。

2. **善用动态拼接**， 利用条件构造器的布尔控制参数，可以根据条件动态生成 SQL，减少不必要的拼接逻辑。

3. **分页查询与排序结合**，在大数据量场景中，结合**分页插件和条件构造器**能显著提升查询效率。

4. **封装公共查询逻辑**，对于复杂的查询条件，可以将**条件构造封装为方法**，方便重复使用。例如：

   ```java
   public QueryWrapper<User> buildActiveUserQuery() {
       QueryWrapper<User> wrapper = new QueryWrapper<>();
       wrapper.eq("status", 1).isNotNull("email");
       return wrapper;
   }
   ```

------

## 六、总结

> MyBatis-Plus 的条件构造器为我们提供了构建 SQL 条件的强大工具。从基础的查询条件到复杂的嵌套逻辑，再到**链式调用和动态条件**，条件构造器可以极大提升开发效率。同时，灵活使用分页、排序等高级功能，可以让你的查询逻辑更清晰、更高效。在日常开发中，掌握这些技巧，将让你在 MyBatis-Plus 的使用中更加得心应手！
