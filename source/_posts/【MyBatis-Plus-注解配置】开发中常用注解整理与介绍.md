---
title: 【MyBatis-Plus 注解配置】开发中常用注解整理与介绍
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2026-01-11 13:51:48
updated:
description:
cover:
---
> 不知道朋友们会不会在`SpringBoot`中集成`MyBatis-Plus`的时候，总是这个注解那个注解，都不知道哪些是`MyBatis-Plus`的了，搞得晕乎乎的，所以我整理了一份`MyBatis-Plus`开发中常用的注解，相信看完你就知道哪些注解是`MyBatis-Plus`提供的了，以后在开发中就能够更加清晰了

### 1. @TableName

**作用**：指定实体类对应的数据库表名。

**用法**：

```java
@TableName("user")
public class User {
    private Long id;
    private String name;
}
```

**属性**：

- `value`：表名。
- `schema`：数据库 schema 名称。
- `keepGlobalPrefix`：是否保留全局表前缀
- `resultMap`：自定义结果映射。
- `excludeProperty`：查询和操作时排除的字段。
- `autoResultMap`：是否启用自动映射结果（解决复杂类型字段问题）。

**场景**：实体类与数据库表名不一致时使用。

```java
@TableName(value = "sys_user", schema = "public", autoResultMap = true)
public class User {
    @TableId
    private Long id;

    @TableField("nickname")
    private String name;

    private Integer age;
}
```

**示例说明**：

- 数据库表名为 `sys_user`，而实体类为 `User`，通过 `@TableName` 指定映射关系。
- 指定了 `schema`（数据库模式），适用于多模式的数据库。

------

### 2. @TableId

**作用**：指定主键字段，支持主键生成策略。

**用法**：

```java
@TableId(value = "id", type = IdType.AUTO)
private Long id;
```

**属性**：

- `value`：数据库主键列名。
- `type`：主键生成策略（`IdType.AUTO`, `IdType.ASSIGN_ID`, `IdType.NONE`, 等）。

------

### 3. @TableField

**作用**：指定实体字段与数据库字段的映射关系。

**用法**：

```java
@TableField("user_name")
private String name;
```

**属性**：

- `value`：数据库字段名。
- `exist`：是否存在于表中。
- `condition`：自定义查询条件。
- `update`：自定义更新逻辑。
- `insertStrategy`、`updateStrategy`、`whereStrategy`：
  - 插入/更新/条件时字段的策略控制，支持：
    - `DEFAULT`：遵循全局配置。
    - `NOT_NULL`：非空时操作。
    - `NOT_EMPTY`：非空字符串或非空对象时操作。
    - `ALWAYS`：总是操作。
- `typeHandler`：指定自定义类型处理器

------

### 4. @TableLogic

**作用**：标注逻辑删除字段，MyBatis-Plus 自动处理逻辑删除。

**用法**：

```java
@TableLogic
private Integer deleted;
```

**说明**：配合全局配置 `logic-delete-value` 和 `logic-not-delete-value` 使用。

------

### 5. @InterceptorIgnore

**作用**：忽略某些拦截器的处理，比如分页拦截器。

**用法**：

```java
@InterceptorIgnore(tenantLine = "true", blockAttack = "true")
public List<User> getAllUsers() {
    return userMapper.selectList(null);
}
```

**说明**：忽略租户线（`tenantLine`）拦截器，查询时不添加租户条件。

****

### 6. @EnumValue

**作用**：标注枚举类的字段，用于数据库值与枚举映射。

**用法**：

```java
public enum Status {
    @EnumValue
    ACTIVE(1),
    INACTIVE(0);

    private final int code;

    Status(int code) {
        this.code = code;
    }
}

@TableField("status")
private Status status;
```

**示例说明**：

- 数据库中的 `status` 字段值为 `0` 或 `1`，自动映射为 `Status` 枚举。
- `@EnumValue` 标注的字段是数据库与枚举之间的映射字段。

****

### 7. @Version

**作用**：标注乐观锁字段。

**用法**：

```java
@Version
private Integer version;
```

**示例说明**：

- 插件会拦截更新操作并附加 `version` 条件：`WHERE version = 当前版本`。
- 如果版本号不匹配，更新会失败，确保数据一致性。

****

### 8. @OrderBy

**场景**：为查询结果指定默认排序规则。

```java
@OrderBy(asc = true, sort = 1)
private String name;

@OrderBy(asc = false, sort = 2)
private Integer age;
```

**示例说明**：

- 查询结果会先按 `name` 升序排序，再按 `age` 降序排序。
- 适合需要默认排序的场景，避免每次查询都显式指定排序。
