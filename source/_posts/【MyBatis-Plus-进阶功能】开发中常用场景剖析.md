---
title: 【MyBatis-Plus 进阶功能】开发中常用场景剖析
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2025-01-10 19:00:00
updated:
description:
cover:
---
> MyBatis-Plus（MP）除了封装常见的 CRUD 操作，还提供了一些高级功能，进一步简化复杂场景下的开发工作。本文将逐一讲解 **逻辑删除**、**自动填充**、**多表关联查询**的原理与使用方式，让你快速掌握这些技巧！

------

## 一、逻辑删除

逻辑删除是指在数据库中不直接删除记录，而是通过标记（如 `is_deleted` 字段）表示数据是否有效。

### 1. 原理与配置

逻辑删除是指在数据库中不直接删除记录，而是通过标记（如 `is_deleted` 字段）来表示数据是否有效，查询时会自动**过滤掉**已标记为“删除”的记录。

### 2. 配置步骤

1. **在数据库表中添加逻辑删除字段**（如 `is_deleted`）：

   ```sql
   ALTER TABLE user ADD COLUMN is_deleted TINYINT(1) DEFAULT 0 COMMENT '逻辑删除标记';
   ```

2. **在实体类中添加逻辑删除注解**：

   ```java
   @TableLogic
   private Integer isDeleted;
   ```

3. **全局配置逻辑删除行为**（`application.yml`）：

   ```yaml
   mybatis-plus:
     global-config:
       db-config:
         logic-delete-field: is_deleted
         logic-delete-value: 1     # 逻辑已删除值
         logic-not-delete-value: 0 # 逻辑未删除值
   ```

### 3. 使用示例

#### ① 插入记录前的数据库表数据

执行以下查询语句：

```sql
SELECT * FROM user;
```

假设初始数据为空：

| id   | name | is_deleted |
| ---- | ---- | ---------- |
|      |      |            |

#### ② 插入新记录

```java
User user = new User();
user.setName("Tom");
userMapper.insert(user);
```

执行后，生成的 SQL 为：

```sql
INSERT INTO user (name, is_deleted) VALUES ('Tom', 0);
```

#### ③ 数据库表中新增记录

| id   | name | is_deleted |
| ---- | ---- | ---------- |
| 1    | Tom  | 0          |

------

#### ④ 执行逻辑删除操作

```java
userMapper.deleteById(1);
```

生成的 SQL 为：

```sql
UPDATE user SET is_deleted = 1 WHERE id = 1;
```

#### ⑤ 数据库表更新后的记录

| id   | name | is_deleted |
| ---- | ---- | ---------- |
| 1    | Tom  | 1          |

------

#### ⑥ 查询数据

默认情况下，逻辑删除记录不会出现在查询结果中：

```java
List<User> users = userMapper.selectList(null);
```

生成的 SQL 为：

```sql
SELECT id, name FROM user WHERE is_deleted = 0;
```

查询结果：

| id   | name |
| ---- | ---- |
|      |      |

若想查询所有记录（包括逻辑删除的记录），需要自定义 SQL：

```java
@Select("SELECT * FROM user")
List<User> selectAllUsers();
```

查询结果：

| id   | name | is_deleted |
| ---- | ---- | ---------- |
| 1    | Tom  | 1          |

------

通过示例可以看出，逻辑删除通过 **自动更新 `is_deleted` 字段** 和 **动态添加查询条件** 实现了逻辑上的“删除”操作，同时保留了数据的可追溯性。

### 4. 注意事项

- 逻辑删除仅在 MP 自动生成的 SQL 中生效，自定义 SQL 需手动添加条件。
- 查询时默认不包含已删除记录，若需查询已删除数据，需使用自定义 SQL。

------

## 二、自动填充

自动填充用于在插入或更新记录时，自动为某些字段赋值（如创建时间、更新时间）。

### 1. 原理与配置

实现自动填充的关键是使用 **`@TableField` 的 `fill` 属性**，并结合 **`MetaObjectHandler` 接口**完成填充逻辑。

自动填充的核心原理在于 **MyBatis-Plus 提供的元对象处理器（`MetaObjectHandler`）接口**。通过实现这个接口，MP 能够**在插入或更新数据时自动填充指定字段**。这背后实际上是一种 **拦截机制**：在执行**插入或更新操作**时，MP 会检测是否有配置的填充字段，如果有，就调用实现类的逻辑为这些字段赋值。

以下我们详细来分析一下：

1. **自动填充的触发时机**
   MP 在执行**插入（`INSERT`）或更新（`UPDATE`）操作**时，会检查**实体类**中是否有标注了 `@TableField(fill = ...)` 的字段。如果存在这些字段，则会**触发自动填充逻辑**。
2. **元对象处理器的作用**
   MP 使用 `MetaObjectHandler` 接口实现了这个逻辑，通过**重写其中的 `insertFill` 和 `updateFill` 方法**，可以定义插入或更新时如何**自动填充字段**。
3. **拦截机制**
   - MP 会在内部生成的 SQL 语句执行前，通过拦截器将指定字段的值动态插入到 SQL 语句中。
   - 这一过程对开发者透明，减少了重复编写代码的成本。
4. **配置生效**
   - MP 的核心配置会自动扫描所有实现了 **`MetaObjectHandler`** 的类，并在执行插入或更新操作时调用相关方法。
   - **无需手动绑定**，这也是为什么只需要实现类即可实现自动填充的原因。

### 2. 配置步骤

1. 在**实体类**中通过 `@TableField(fill = ...)` 指定需要填充的字段：

   ```java
   @TableField(fill = FieldFill.INSERT)
   private LocalDateTime createTime;
   
   @TableField(fill = FieldFill.INSERT_UPDATE)
   private LocalDateTime updateTime;
   ```

   > `FieldFill.INSERT`：仅在插入时填充。
   >
   > `FieldFill.INSERT_UPDATE`：在插入和更新时都会填充。

2. **实现 `MetaObjectHandler` 接口**。只需创建一个类并实现 `MetaObjectHandler`，就会**自动扫描**并加载这个类的配置，无需额外声明。记得通过`@Component`注解标注为组件，**SpringBoot** 才能够扫描到

   ```java
   @Component
   public class MyMetaObjectHandler implements MetaObjectHandler {
   
       @Override
       public void insertFill(MetaObject metaObject) {
           this.strictInsertFill(metaObject, "createTime", LocalDateTime::now, LocalDateTime.class);
           this.strictInsertFill(metaObject, "updateTime", LocalDateTime::now, LocalDateTime.class);
       }
   
       @Override
       public void updateFill(MetaObject metaObject) {
           this.strictUpdateFill(metaObject, "updateTime", LocalDateTime::now, LocalDateTime.class);
       }
   }
   ```

   > `strictInsertFill` 和 `strictUpdateFill` 是 MP 提供的方法，用于安全地为指定字段赋值。
   >
   > MP 会自动调用 `insertFill` 和 `updateFill`，而开发者无需显式调用。

### 3. 使用示例

#### ① 插入数据

插入或更新时，无需手动设置 `createTime` 和 `updateTime`：

```java
User user = new User();
user.setName("Tom");
userMapper.insert(user);
```

执行后，自动填充的 SQL 为：

```sql
INSERT INTO user (name, create_time, update_time) 
VALUES ('Tom', '2025-01-01 12:00:00', '2025-01-01 12:00:00');
```

#### ② 更新数据

```java
User user = userMapper.selectById(1);
user.setName("Jerry");
userMapper.updateById(user);
```

执行后，自动填充的 SQL 为：

```sql
UPDATE user 
SET name = 'Jerry', update_time = '2025-01-01 12:05:00' 
WHERE id = 1;
```

------

## 三、多表关联查询

虽然 MyBatis-Plus 不直接支持多表联查，但我们可以通过以下方式实现：

### 1. 数据库表设计

假设我们有以下两张表：

**用户表（`user`）**

| 字段名 | 类型        | 描述   |
| ------ | ----------- | ------ |
| id     | INT         | 主键   |
| name   | VARCHAR(50) | 用户名 |
| email  | VARCHAR(50) | 邮箱   |

**订单表（`order`）**

| 字段名     | 类型           | 描述     |
| ---------- | -------------- | -------- |
| id         | INT            | 主键     |
| user_id    | INT            | 用户 ID  |
| order_time | DATETIME       | 下单时间 |
| amount     | DECIMAL(10, 2) | 订单金额 |

### 2. 实体类定义

**用户实体类（`User`）**

```java
@Data
public class User {
    private Integer id;
    private String name;
    private String email;

    // 关联的订单集合
    private List<Order> orders;
}
```

**订单实体类（`Order`）**

```java
@Data
public class Order {
    private Integer id;
    private Integer userId;
    private LocalDateTime orderTime;
    private BigDecimal amount;
}
```

### 3. XML 配置

**自定义 SQL 和返回结果映射**

通过 MyBatis 的 XML 配置实现多表联查：

**UserMapper.xml**

```xml
<mapper namespace="com.example.mapper.UserMapper">

    <!-- 多表联查，查询用户及其订单 -->
    <select id="getUserWithOrders" resultMap="userWithOrdersMap">
        SELECT 
            u.id AS userId,
            u.name AS userName,
            u.email AS userEmail,
            o.id AS orderId,
            o.order_time AS orderTime,
            o.amount AS orderAmount
        FROM user u
        LEFT JOIN order o ON u.id = o.user_id
        WHERE u.id = #{userId}
    </select>

    <!-- 结果映射 -->
    <resultMap id="userWithOrdersMap" type="com.example.entity.User">
        <id property="id" column="userId" />
        <result property="name" column="userName" />
        <result property="email" column="userEmail" />
        <collection property="orders" ofType="com.example.entity.Order">
            <id property="id" column="orderId" />
            <result property="orderTime" column="orderTime" />
            <result property="amount" column="orderAmount" />
        </collection>
    </resultMap>

</mapper>
```

### 4. Mapper 接口

```java
public interface UserMapper extends BaseMapper<User> {
    User getUserWithOrders(@Param("userId") Integer userId);
}
```

------

### 5. 测试联查

**Service 层调用**

```java
@Autowired
private UserMapper userMapper;

public User getUserWithOrders(Integer userId) {
    return userMapper.getUserWithOrders(userId);
}
```

**测试用例**

```java
@Test
public void testGetUserWithOrders() {
    User user = userMapper.getUserWithOrders(1);
    System.out.println("用户信息：" + user.getName());
    System.out.println("订单信息：");
    user.getOrders().forEach(order -> {
        System.out.println("订单 ID：" + order.getId());
        System.out.println("订单金额：" + order.getAmount());
    });
}
```

### 6. 查询结果

假设 `user` 表中有以下记录：

| id   | name | email             |
| ---- | ---- | ----------------- |
| 1    | 张三 | zhangsan@mail.com |

`order` 表中有以下记录：

| id   | user_id | order_time          | amount |
| ---- | ------- | ------------------- | ------ |
| 1    | 1       | 2025-01-01 12:00:00 | 100.00 |
| 2    | 1       | 2025-01-01 15:00:00 | 200.00 |

运行测试后输出：

```
用户信息：Alice
订单信息：
订单 ID：1
订单金额：100.00
订单 ID：2
订单金额：200.00
```
