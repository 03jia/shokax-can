---
title: 【MyBatis-Plus ORM】数据库和实体类映射
keywords: []
categories:
  - [后端,框架,MyBatis-plus]
tags:
  - MyBatis-Plus
  - ORM
date: 2025-01-09 10:00:00
updated:
description: "说明 MyBatis-Plus 在主键生成与实体类映射方面的策略与常见实践。"
cover: "https://s2.loli.net/2026/01/11/1nFywYhRWPf9Usl.jpg"
---

>在开发中，数据库和 <font color=red>**Java 对象的映射（ORM）**</font>是一个绕不开的话题，而 MyBatis-Plus（MP）作为一款优秀的 <font color='red'>**ORM 工具**</font>，帮我们简化了繁琐的数据库操作。本文将从数据库基础、表与实体映射、复杂对象映射、自定义 SQL 等角度，深入探讨 MP 的数据库映射功能。

------

## 一、数据库设计基本知识

在开始 ORM 映射之前，理解数据库设计的基本原则至关重要：

### 1. 索引

- 索引可以提高查询效率，例如主键索引、唯一索引、复合索引等。

- 在创建实体时，可以用注解标注需要索引的字段：

  ```java
  @TableField("username")
  @TableIndex(type = IndexType.UNIQUE)
  private String username;
  ```

### 2. 主键与外键

- **主键**：标识表中每一行记录的唯一性。
- **外键**：用来建立表与表之间的关系。

### 3. 范式

数据库范式（如第一范式、第三范式）是设计良好表结构的重要参考。

**Tip**: 保持表结构简单，但不妨碍业务扩展。

------

## 二、表与实体的映射关系

MP 提供了丰富的注解，帮助开发者高效完成数据库表与 Java 对象的映射。

### 1. 基本映射

MP 默认将表名与实体名直接映射，但我们可以通过注解自定义：

- **表名映射**：使用 `@TableName` 注解将**实体类与数据库表**关联：

  ```java
  @TableName("user")
  public class User {
      @TableId(value = "id", type = IdType.AUTO)
      private Long id;
  
      @TableField("username")
      private String name;
  }
  ```

- **字段名映射**：数据库字段名通常为下划线风格，Java 属性名为驼峰风格。当字段名与属性名不一致时，MP 默认自动处理这种映射，也可以通过 `@TableField` 自定义：

  ```java
  @TableField("email_address")
  private String emailAddress;
  ```

### 2. 自定义主键生成策略

主键生成是 ORM 映射中重要的一环，当往数据库添加字段的时候，此id会根据指定的主键生成策略来进行生成对应的值。MP 支持多种主键生成策略：

- `IdType.AUTO`：
  - 适用于主键为自增类型（如 MySQL 的 `AUTO_INCREMENT`）
  - 在插入数据时，**不需要为主键赋值**，由数据库根据自增策略生成。
  - 此策略仅在数据库支持自增主键的情况下有效，需要在数据库中保证主键是自增的。
- `IdType.ASSIGN_ID`：
  - MyBatis-Plus 默认的主键生成策略，使用 **雪花算法** 生成全局唯一 ID。
  - 在插入数据时，MP 自动生成主键值，并在 SQL 中直接插入该值。
  - 数据库表的主键类型为 `BIGINT`，需要全局唯一标识符。
- `IdType.INPUT`：
  - 手动输入主键值
  - 需要在**插入数据前手动设置主键值**，否则插入操作将失败。
  - 适用于特定业务场景（如主键由外部系统生成）。

示例：

在实体类中通过注解设置：
```java
@TableId(value = "id", type = IdType.AUTO)
private Long id;
```
全局设置主键策略（`application.yaml` 配置文件中）
```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: ASSIGN_ID  # 全局默认使用雪花算法
      # 可选值：
      # AUTO: 数据库自增
      # NONE: 无状态
      # INPUT: 手动输入
      # ASSIGN_ID: 雪花算法
      # ASSIGN_UUID: UUID
```
------

## 三、XML 中配置复杂映射

### 1. 基本数据类型映射

对于普通表字段到实体属性的映射，MP 提供了简单直观的方式。

通过 `resultMap` 解决复杂结果集的映射问题。

指定了id之后，如果查询数据中包含了多个id的值，会自动合并，映射为一个对象集合

#### a. 基本映射规则

MP **默认遵循数据库字段名和实体类属性名之间的映射规则**：

- 数据库字段名：下划线命名。
- Java 属性名：驼峰命名。

示例表结构（`user` 表）：

```sql
CREATE TABLE user (
    id BIGINT PRIMARY KEY,
    username VARCHAR(50),
    age INT
);
```

实体类定义：

```java
@Data
public class User {
    private Long id;          // 对应数据库字段 id
    private String username;  // 对应数据库字段 username
    private Integer age;      // 对应数据库字段 age
}
```

#### b. 自定义字段映射：

通过 `column` 属性来指定表字段

```xml
<resultMap id="userMap" type="User">
    <id property="id" column="id"/>
    <result property="username" column="user_name"/>
    <result property="age" column="age"/>
</resultMap>
```

------

### 2. 嵌套对象映射

嵌套对象映射用于处理对象属性本身是另一个复杂对象的场景，例如主表与子表关联。

#### a. 表和实体类

假设有以下**表结构**：

-  `user`表：

   ```sql
   CREATE TABLE user (
        id BIGINT PRIMARY KEY,
        name VARCHAR(50)
    ); 
   ```

- `address`表：

  ```sql
      CREATE TABLE address (
        id BIGINT PRIMARY KEY,
        user_id BIGINT,
        city VARCHAR(50),
        FOREIGN KEY (user_id) REFERENCES user(id)
      );
  ```

****

**实体类**定义：

```java
@Data
public class User {
    private Long id;
    private String name;
    private Address address; // 嵌套对象
}

@Data
public class Address {
    private Long id;
    private Long userId;
    private String city;
}
```

#### b. XML 映射

在 XML 文件中通过 `<association>` 定义嵌套对象映射：

```xml
<resultMap id="userWithAddressMap" type="User">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <association property="address" javaType="Address">
        <id property="id" column="address_id"/>
        <result property="city" column="city"/>
    </association>
</resultMap>

<select id="selectUserWithAddress" resultMap="userWithAddressMap">
    SELECT u.id, u.name, a.id AS address_id, a.city
    FROM user u
    LEFT JOIN address a ON u.id = a.user_id
</select>
```

------

### 3. 集合类型映射

当查询结果中存在一个字段包含 **多个值**（例如多个标签 `tag`），并且在实体类中将该字段指定为 **集合类型**（如 `List<String>`），MyBatis-Plus 会根据配置自动将这些值合并并映射为集合。

这种场景常见于 **一对多** 或 **多对多** 的查询中，主要通过 **`<collection>` 元素** 实现。

#### a. 基本示例

① 假设有以下数据：

**`article_tag`表**

| article_id | tag     |
| ---------- | ------- |
| 1          | Spring  |
| 1          | MyBatis |
| 2          | Java    |

------

② 实体类定义

```
public class Article {
    private Long id;               // 文章 ID
    private String title;          // 文章标题
    private List<String> tags;     // 标签集合
}
```

------

③ Mapper 配置（XML 映射文件）

使用 `<collection>` 将多行数据的 `tag` 字段自动映射为一个集合：

```xml
<resultMap id="ArticleResultMap" type="com.example.Article">
    <id property="id" column="article_id" />
    <result property="title" column="title" />
    <collection property="tags" ofType="java.lang.String">
        <result column="tag" />
    </collection>
</resultMap>

<select id="selectArticlesWithTags" resultMap="ArticleResultMap">
    SELECT 
        a.id AS article_id, 
        a.title, 
        t.tag 
    FROM 
        article a
    LEFT JOIN 
        article_tag t 
    ON 
        a.id = t.article_id
</select>
```

------

④ 查询结果

```java
List<Article> articles = articleMapper.selectArticlesWithTags();
```

假设查询结果为：

```json
[
  {
    "id": 1,
    "title": "学习 MyBatis",
    "tags": ["Spring", "MyBatis"]
  },
  {
    "id": 2,
    "title": "Java 基础",
    "tags": ["Java"]
  }
]
```

#### b. 对象类型集合

以用户为例和订单表为例：

① 表和实体类

假设有以下**表结构**：

- `user`表：

  ```sql
  CREATE TABLE user (
      id BIGINT PRIMARY KEY,
      name VARCHAR(50)
  );
  ```

- `order`表：

  ```sql
  CREATE TABLE order (
      id BIGINT PRIMARY KEY,
      user_id BIGINT,
      order_number VARCHAR(50),
      FOREIGN KEY (user_id) REFERENCES user(id)
  );
  ```

****

② **实体类**定义：

```java
@Data
public class User {
    private Long id;
    private String name;
    private List<Order> orders; // 集合类型
}

@Data
public class Order {
    private Long id;
    private Long userId;
    private String orderNumber;
}
```

****

③ **XML** 映射

在 XML 文件中通过 `<collection>` 定义集合映射：

```xml
<resultMap id="userWithOrdersMap" type="User">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderNumber" column="order_number"/>
    </collection>
</resultMap>

<select id="selectUserWithOrders" resultMap="userWithOrdersMap">
    SELECT u.id, u.name, o.id AS order_id, o.order_number
    FROM user u
    LEFT JOIN order o ON u.id = o.user_id
</select>
```

> **【注意事项】**
>
> - `<collection>` 的 `ofType` 属性指定集合中元素的类型。
> - 确保 SQL 查询中包含子表数据，否则集合为空。

------

## 四、自定义 SQL

MP 提供了两种自定义 SQL 的方式：**XML 文件**和**注解方式**。

### 1. 使用注解自定义 SQL

在 `Mapper` 接口中直接使用注解书写简单 SQL：

```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    @Select("SELECT * FROM user WHERE name = #{name}")
    User findByName(@Param("name") String name);
}
```

- 优点：简洁、轻量化。
- 缺点：不适合复杂查询。

### 2. 使用 XML 文件自定义 SQL

对于复杂 SQL，建议使用 XML 文件。MP 通过 `Mapper` 文件与 XML 配置绑定：

- `UserMapper.java`

  ```java
  @Mapper
  public interface UserMapper extends BaseMapper<User> {
      User selectUserWithOrders(Long userId);
  }
  ```

- `UserMapper.xml`

  ```xml
  <resultMap id="userResultMap" type="User">
      <id property="id" column="id"/>
      <result property="name" column="name"/>
      <collection property="orders" ofType="Order">
          <id property="id" column="order_id"/>
          <result property="orderNumber" column="order_number"/>
      </collection>
  </resultMap>
  
  <mapper namespace="com.example.mapper.UserMapper">
      <select id="selectUserWithOrders" resultType="User">
          SELECT u.*, o.*
          FROM user u
          LEFT JOIN order o ON u.id = o.user_id
          WHERE u.id = #{userId}
      </select>
  </mapper>
  ```

------

## 五、写在最后

### 总结

- MP 将实体类与数据库表映射打通，使得开发效率提升。
- 对于复杂场景，XML 自定义 SQL 是不可或缺的利器。
- **主键生成**、**字段映射**等基础功能覆盖面广，足够应对大多数项目需求。

### 建议

1. 优先使用 MP 提供的内置方法，减少代码冗余。
2. **复杂业务逻辑**时，灵活使用**自定义 SQL**（尤其是 XML 文件）。
3. 对于**关联查询**，合理使用 `resultMap` 和**嵌套映射**。

*希望这篇文章能帮你更好地理解 MP 的 ORM 映射功能，并在实际项目中得心应手！ 😊*
