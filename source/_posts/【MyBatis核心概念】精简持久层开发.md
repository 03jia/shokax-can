---
title: 【MyBatis核心概念】精简持久层开发
keywords: []
categories:
    - [后端,框架,MyBatis]
tags:
    - MyBatis
    - 持久层
date: 2024-12-23 11:00:00
updated:
description: "概述 MyBatis 的核心概念与设计理念，说明其如何帮助精简持久层开发工作。"
cover: "https://s2.loli.net/2026/01/12/e3DkEzfA41MnJKu.jpg"
---
> 在 Java 开发中，**与数据库的交互**是不可或缺的一部分。传统的 JDBC 操作虽然功能强大，但却伴随着冗长的代码、复杂的资源管理和难以维护的 SQL 嵌入。而 MyBatis 作为一款轻量级的**持久层框架**，凭借灵活的 SQL 管理、便捷的对象映射和出色的扩展性，成为了解决这些痛点的理想选择。
>
> ------
>
> 本文将带你从零认识 MyBatis，从它的核心概念、配置文件、接口调用到 Mapper 的工作机制，帮助你轻松掌握持久层开发的高效方法。

## 一、MyBatis 是什么，为什么需要它？

### MyBatis 的定义

MyBatis 是一个优秀的持久层框架，用于简化 Java 程序与数据库之间的交互，特别是在传统 JDBC 操作冗长且繁琐的情况下，为开发者提供了更灵活的 SQL 管理方式。

### 相较于 JDBC 的优势

传统 JDBC 通常需要以下步骤：

1. 手动加载驱动。
2. 建立数据库连接。
3. 编写 SQL。
4. 处理结果集。
5. 关闭连接和释放资源。

**示例：传统 JDBC 代码**：

```java
public class JdbcExample {
    public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            // 加载驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            // 建立连接
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "password");
            // 编写 SQL
            String sql = "SELECT * FROM user WHERE id = ?";
            ps = conn.prepareStatement(sql);
            ps.setInt(1, 1);
            // 执行查询
            rs = ps.executeQuery();
            while (rs.next()) {
                System.out.println(rs.getString("name"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 释放资源
            try { if (rs != null) rs.close(); } catch (Exception e) {}
            try { if (ps != null) ps.close(); } catch (Exception e) {}
            try { if (conn != null) conn.close(); } catch (Exception e) {}
        }
    }
}
```

### MyBatis 的改进

1. **减少重复代码**：MyBatis **管理连接和资源释放**，开发者只需关注业务逻辑。
2. **SQL 与代码耦合**：传统 JDBC 的 SQL 嵌在代码中，难以维护；MyBatis 将 SQL 与代码分离，方便修改。
3. **对象映射支持**：MyBatis 能将查询结果**自动映射为 Java 对象**，无需手动解析 `ResultSet`。
4. **动态 SQL**：通过条件动态拼接 SQL，适应复杂场景。
5. **性能优化难**：MyBatis 支持一级、二级缓存机制，提升查询性能。

------

## 二、MyBatis 如何与数据库交互？

> 对于这个**核心工作机制**，这里先大概有些了解，因为怕一开始就讲的太复杂耽误大家看下文，感兴趣的朋友可以看我另一篇博客有详细的讲解！

### 1.`SqlSessionFactory`：

- **作用**：创建 `SqlSession` 对象，负责管理 MyBatis 的核心配置和生命周期。
- **工作机制**：`SqlSessionFactory` 从配置文件读取环境信息，初始化数据源和 Mapper 配置。
- **实现方式**：通过 `SqlSessionFactoryBuilder` 构建。

```java
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
    .build(Resources.getResourceAsStream("mybatis-config.xml"));
```

### 2.`SqlSession`：

- **作用**：执行 SQL 语句，如查询、插入、更新、删除等。
- 工作机制：
  1. 获取 SQL 配置。
  2. 与数据库交互。
  3. 将结果映射为 Java 对象。
- **注意事项**：`SqlSession` 线程不安全，使用完后需关闭。

### 3.`Mapper` 接口：

- **作用**：将 SQL 操作封装为接口方法，通过动态代理调用实际的 SQL。
- **工作机制**：`SqlSession` 动态代理实现 Mapper 接口，直接执行对应 SQL。

------

## 三、MyBatis 的配置文件

### `mybatis-config.xml` 的作用

- 配置 MyBatis 的全局运行环境。
- 包括数据源、事务管理、Mapper 关联等。

### 配置文件常见节点

1. **`<environments>`**：配置不同环境（开发、测试、生产）的数据库信息。

2. **`<mappers>`**：指定 Mapper 文件或包。

   > 如果你的项目结构遵循 MyBatis 推荐的标准规则，并且在配置中采用了 **扫描包**（如使用了 `@MapperScan` 注解或者在 Spring 配置中进行包扫描），那么可以省略 `<mappers>` 标签。

**配置示例**

```java
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test"/>
                <property name="username" value="root"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/example/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

------

## 四. 基于注解的 MyBatis 程序

1. `@Select`：定义查询语句。
2. `@Insert`、`@Update`、`@Delete`：对应数据库的增删改操作。
3. `@Param`：绑定方法参数到 SQL 中的占位符。

**代码示例**：

```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM user WHERE id = #{id}")
    User selectUserById(@Param("id") int id);
}
```

------

## 五、Mapper 是如何工作的

### Mapper 接口的本质

- 是 MyBatis 动态代理的结果。
- Mapper 接口的方法调用时，MyBatis 会通过代理类解析方法名，并找到对应的 SQL 执行。

### 注解和 Mapper 配合使用

> - 如果某些 **SQL 较复杂**，可以将注解**与 XML 配合**使用。
> - 注解处理**简单 SQL**，XML 负责复杂**动态 SQL**。

**示例**：

```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM user WHERE id = #{id}")
    User selectUserById(@Param("id") int id);
}
```

------

> 通过本文的讲解，我们深入了解了 MyBatis 作为一个轻量级持久层框架如何有效简化 Java 与数据库的交互。从基础的 JDBC 操作到 MyBatis 的配置文件解析，再到 Mapper 接口和注解的结合使用，MyBatis 无疑为开发者提供了一种更加灵活、简洁且高效的数据库操作方式。

