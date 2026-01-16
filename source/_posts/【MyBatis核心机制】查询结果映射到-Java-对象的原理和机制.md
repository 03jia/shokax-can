---
title: 【MyBatis核心机制】查询结果映射到 Java 对象的原理和机制
keywords: []
categories:
    - [后端,框架,MyBatis]
tags:
    - MyBatis
    - ORM
date: 2024-12-25 20:30:37
updated:
description: "介绍 MyBatis 查询结果映射的原理，说明反射与映射规则如何将结果集转换为 Java 对象。"
cover: "https://s2.loli.net/2026/01/12/gnx5jopdcbJAtCV.jpg"
---
> MyBatis 的**查询结果映射（Result Mapping）**功能，像是一个桥梁，连接了数据库中的结果集和 Java 中的对象。通过它，SQL 查询**返回的表数据**被自动转换为我们熟悉的 **Java 对象**。这一过程的背后，依靠的是 **反射机制** 和 **映射规则**。接下来，我们一起来看看它的工作原理以及如何实现各种映射。

------

## **一、自动映射的简单逻辑**

自动映射就像“机智的助理”，它会尝试让数据库的列名和 Java 对象的字段名一一匹配。

> - 这是一种较为**通用**且灵活的方式，可以随意命名。
> - 但是开发中如果遵循一些规则的话，开发起来会更舒服，比如后面会提到的**小驼峰命名和下划线映射**，这就需要程序员遵循好命名规范了。

**如何匹配？** MyBatis 利用反射找到对象的字段名，然后逐一检查数据库的列名。如果两者一致（或者通过别名 `AS` 一致），MyBatis 会自动将列的值赋给字段。例如：

```sql
SELECT user_id AS id, user_name AS name FROM users;
```

```java
public class User {
    private int id;
    private String name;
    // Getters and setters...
}
```

> `user_id` 匹配 `id`（通过别名），`user_name` 匹配 `name`，值自动填充。

**代码示例**：

```java
@Select("SELECT id, name FROM user WHERE id = #{id}")
User selectUserById(int id);
```

自动映射的好处是省时省力，但如果列名和字段名不一致时，就需要更灵活的手动映射。

------

## **二、手动映射——显式定义规则**

当“助理”搞不定时，你需要用 `resultMap` 手动告诉 MyBatis 如何映射。

**使用场景**：列名和字段名差别很大，比如列名是 `user_id`，字段名却是 `uid`。

**如何操作？**手动定义映射关系，让每一列清晰对应到 Java 对象字段。

**示例代码**：

```XML
<resultMap id="userResultMap" type="com.example.User">
    <id column="user_id" property="id"/>
    <result column="user_name" property="name"/>
</resultMap>

<select id="selectUserById" resultMap="userResultMap">
    SELECT user_id, user_name FROM user WHERE user_id = #{id}
</select>
```

**工作流程**：

1. 执行 SQL，获得 `ResultSet`。
2. 按 `resultMap` 的规则逐列映射。
3. 使用反射给 Java 对象字段赋值。

------

## **三、注解映射的灵活使用**

如果你喜欢代码清晰明了，可以用注解来定义映射规则。比如：

```java
@Results({
    @Result(property = "id", column = "user_id", id = true),
    @Result(property = "name", column = "user_name")
})
@Select("SELECT user_id, user_name FROM user WHERE user_id = #{id}")
User selectUserById(int id);
```

这里的 `@Result` 注解实现了类似 `resultMap` 的功能，让代码更紧凑。

------

## **四、高级映射——嵌套对象和集合**

如果查询结果不仅是简单字段，还包含嵌套对象或集合，MyBatis 也能处理。

1. **嵌套对象**： 当结果中包含复杂结构时，可以用 `<association>` 指定嵌套对象的映射。

   ```XML
   <resultMap id="orderResultMap" type="com.example.Order">
       <id column="order_id" property="id"/>
       <result column="order_date" property="date"/>
       <association property="user" javaType="com.example.User">
           <id column="user_id" property="id"/>
           <result column="user_name" property="name"/>
       </association>
   </resultMap>
   
   <select id="selectOrderById" resultMap="orderResultMap">
       SELECT o.order_id, o.order_date, u.user_id, u.user_name
       FROM orders o
       JOIN users u ON o.user_id = u.user_id
       WHERE o.order_id = #{id}
   </select>
   ```

   > 查询结果中的用户信息被映射到 `Order` 对象中的 `User` 字段。

2. **集合映射**： 如果查询返回一个对象包含子对象集合，可以用 `<collection>` 定义集合的映射规则。

   ```XML
   <resultMap id="departmentResultMap" type="com.example.Department">
       <id column="dept_id" property="id"/>
       <result column="dept_name" property="name"/>
       <collection property="employees" ofType="com.example.Employee">
           <id column="emp_id" property="id"/>
           <result column="emp_name" property="name"/>
       </collection>
   </resultMap>
   
   <select id="selectDepartmentWithEmployees" resultMap="departmentResultMap">
       SELECT d.dept_id, d.dept_name, e.emp_id, e.emp_name
       FROM department d
       JOIN employee e ON d.dept_id = e.dept_id
   </select>
   ```


------

## **五、数据类型的自动转换**

MyBatis 会根据数据库的列类型和 Java 字段类型自动完成数据转换。例如：

**基本数据类型和包装类**：

- `VARCHAR` → `String`
- `INT` → `Integer` 或 `int`
- `BIGINT` → `Long` 或 `long`
- `DECIMAL` → `BigDecimal`
- `DATE` → `java.sql.Date` 或 `java.util.Date`
- `TIME` → `java.sql.Time`
- `TIMESTAMP` → `java.sql.Timestamp`

------

## 六、**小驼峰与下划线的自动映射**

MyBatis 提供自动映射功能，可以将数据库中的**下划线命名**（`snake_case`）的列名映射为 Java 类中**小驼峰命名**（`camelCase`）的字段名。

------

### **如何启用自动映射**

在 MyBatis 配置文件中开启 `mapUnderscoreToCamelCase` 设置即可：

```XML
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

------

### **自动映射规则**

- 数据库列名中的下划线会被移除，并将其后一个字母转为大写。
- 例如： 
  - `user_id` → `userId`
  - `created_at` → `createdAt`

------

**【示例】**

数据库查询：

```sql
SELECT user_id, created_at FROM users;
```

Java 类：

```java
public class User {
    private Integer userId;
    private Date createdAt;
}
```

SQL 查询结果会自动映射到 Java 对象中，无需额外配置。

------

> **【注意事项】**
>
> 1. 字段名不符合下划线或小驼峰规则时，可能需要手动映射。
> 2. 启用后可能略微增加运行时解析开销，字段较多时建议优化配置。

## **七、查询结果映射的核心流程总结**

1. **解析结果集**：MyBatis 执行 SQL，获得 `ResultSet`。
2. **根据映射规则处理**：通过自动映射、`resultMap` 或注解匹配列名和字段名。
3. **对象实例化**：利用反射创建对象。
4. **赋值**：调用字段的 `set` 方法，将列的值赋值到字段。
5. **复杂对象处理**：递归映射嵌套对象或填充集合。

------

## **总结**

> MyBatis 的查询结果映射灵活性很高，既能满足简单场景，也能应对复杂的数据结构需求。理解其映射原理和机制，可以帮助我们更高效地完成数据库和 Java 对象之间的转换。