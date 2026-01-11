---
title: 【MyBatis-Plus 插件】并发控制机制——乐观锁
keywords: []
categories:
   - [后端,框架,MyBatis-plus]
tags:
   - MyBatis-Plus
   - 乐观锁
date: 2025-01-11 13:48:44
updated:
description: "介绍 MyBatis-Plus 的乐观锁机制及其在高并发场景下防止数据覆盖的应用与注意事项。"
cover: "https://s2.loli.net/2026/01/11/vNtLwZCVOBYngMF.jpg"
---
> 乐观锁是一种<font color='red'>**非阻塞的并发控制机制**</font>，在多线程环境中确保数据一致性。MyBatis-Plus 使用 `@Version` 注解和乐观锁插件轻松实现这一功能。在正式介绍乐观锁之前，我们先来聊一聊没有乐观锁时会出现的问题，以及它解决的痛点。

------

## 1. 没有乐观锁时的问题

在多线程或高并发场景下，如果多个线程同时读取和修改同一条数据，就可能导致**数据覆盖问题**。

### 问题场景：库存扣减

假设一个商品的库存为 10，两名用户 A 和 B 同时购买 1 个商品。
 两个线程几乎同时处理这两个操作，以下是可能发生的情况：

1. **线程 A** 查询库存，读取到值为 10。
2. **线程 B** 查询库存，也读取到值为 10。
3. **线程 A** 扣减 1，库存更新为 9。
4. **线程 B** 扣减 1，也更新库存为 9。

最终的库存变成 9，但实际上应该是 8。

> 这种情况称为 **数据覆盖问题**，即后一个线程的更新覆盖了前一个线程的结果。

------

## 3. 为什么会出现问题？

问题的根源在于以下几点：

1. **并发读取同一数据**：数据库中的库存值被多个线程同时读取，导致线程 A 和线程 B 都以初始值 10 开始操作。
2. **没有版本号校验**：更新时并未校验数据是否被其他线程修改，直接覆盖写入。
3. **并发修改失控**：高并发场景下，修改操作可能完全失控，导致数据一致性问题。

------

## 4. 乐观锁的必要性

为了解决这些问题，我们需要一种机制，能够**校验数据在更新时是否被修改过**。这就是乐观锁的核心思想。

乐观锁通过**版本号校验**，确保只有线程读取时的版本号与当前版本号一致，才能完成更新操作。如果版本号不一致，则表示数据已被修改，更新失败。

------

## 5. 场景对比：开启与不开启乐观锁

### 未开启乐观锁（问题场景重现）

假设数据库初始数据如下：

| id   | name  | stock |
| ---- | ----- | ----- |
| 1    | 商品A | 10    |

两名用户 A 和 B 几乎同时购买 1 个商品。

1. 线程 A

   查询库存：

   ```sql
   SELECT stock FROM goods WHERE id = 1; -- 返回 stock = 10
   ```

2. 线程 B

   查询库存：

   ```sql
   SELECT stock FROM goods WHERE id = 1; -- 返回 stock = 10
   ```

3. 线程 A

   执行更新：

   ```sql
   UPDATE goods SET stock = 9 WHERE id = 1;
   ```

4. 线程 B

   执行更新：

   ```sql
   UPDATE goods SET stock = 9 WHERE id = 1; -- 再次覆盖为 9
   ```

最终库存值为 9，数据出现错误。

------

### 开启乐观锁（问题解决）

开启乐观锁后，通过版本号控制更新：

1. 线程 A 查询库存与版本号：

   ```sql
   SELECT stock, version FROM goods WHERE id = 1; -- 返回 stock = 10, version = 1
   ```

2. 线程 B 查询库存与版本号：

   ```sql
   SELECT stock, version FROM goods WHERE id = 1; -- 返回 stock = 10, version = 1
   ```

3. 线程 A 执行更新（版本号匹配成功）：

   ```sql
   UPDATE goods SET stock = 9, version = 2 WHERE id = 1 AND version = 1;
   ```

4. 线程 B

   执行更新（版本号校验失败）：

   ```sql
   UPDATE goods SET stock = 9, version = 2 WHERE id = 1 AND version = 1;
   -- 更新失败，因为当前版本号为 2
   ```

线程 B 的更新失败，库存值保持正确。

------

> **【背景小结】**
>
> 没有乐观锁时，多个线程可能覆盖彼此的修改，导致数据一致性问题，特别是在**库存扣减**、**订单状态更新**等高并发场景中尤为明显。
>
> 开启乐观锁后，通过**版本号校验**，有效解决了**并发修改**导致的覆盖问题，为数据一致性提供了保障。这也是乐观锁在实际开发中不可或缺的重要功能之一。

## 6.乐观锁的使用

### 6.1 乐观锁的核心原理

1. **版本号控制**： 每条记录增加一个 `version` 字段，初始值通常为 1。
2. 更新时**校验版本号**：更新操作会在 SQL 条件中加入 version检查，并对 `version` 自增：
   - 如果当前 `version` 匹配，则更新成功，并将 `version` 加 1。
   - 如果 `version` 不匹配，说明数据已被其他线程修改，更新失败。
3. **并发**场景：
   - 线程 A 和 B 同时读取一条记录。
   - A 修改并提交，`version` 由 1 变为 2。
   - B 修改时发现 `version` 不为 1，更新失败。

------

### 6.2 配置步骤

#### ① 添加版本字段

在实体类中定义 `version` 字段并使用 `@Version` 注解：

```java
@Data
public class User {
    private Long id;
    private String name;

    @Version
    private Integer version; // 版本号字段
}
```

> **注意**：
>
> - `version` 字段类型支持 `Integer`、`Long`、`Date`、`Timestamp`。
> - 数据库中需添加 `version` 列。

------

#### ② 配置乐观锁插件

在 `Spring Boot` 项目中，向 MyBatis-Plus 添加乐观锁插件：

```java
@Configuration
public class MyBatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

> **提示**：
>
> - `MybatisPlusInterceptor` 是 MP 的核心拦截器，`OptimisticLockerInnerInterceptor` 是乐观锁插件。
> - 确保插件已注册，否则乐观锁功能无法生效。

------

### 6.3 使用示例

#### ① 数据准备

假设数据库中 `user` 表的初始数据如下：

| id   | name | version |
| ---- | ---- | ------- |
| 1    | Tom  | 1       |

#### ② 模拟并发更新

模拟两个线程 A 和 B 同时修改同一条记录：

- **线程 A 操作**：

  ```java
  User userA = userMapper.selectById(1); // 查询数据
  userA.setName("Jerry");               // 修改
  userMapper.updateById(userA);         // 提交
  ```

  **生成的 SQL**：

  ```sql
  UPDATE user SET name = 'Jerry', version = 2 WHERE id = 1 AND version = 1;
  ```

  更新成功，`version` 更新为 2。

- **线程 B 操作**：

  ```java
  User userB = userMapper.selectById(1); // 查询数据
  userB.setName("Tommy");                // 修改
  userMapper.updateById(userB);          // 提交
  ```

  **生成的 SQL**：

  ```sql
  UPDATE user SET name = 'Tommy', version = 2 WHERE id = 1 AND version = 1;
  ```

- **执行结果**：更新失败，因为 `version = 1` 的条件不再满足。

------

#### ③ 更新成功与失败的判断

MyBatis-Plus 的 `updateById` 方法会返回更新的行数：

- **成功**：返回 1。
- **失败**：返回 0。

可以通过以下代码判断更新结果：

```java
int result = userMapper.updateById(user);
if (result == 0) {
    System.out.println("更新失败，数据可能已被其他线程修改！");
} else {
    System.out.println("更新成功！");
}
```

------

### 6.4 常见问题与注意事项

1. **更新失败的处理**：
   - 提示用户数据已过期，要求重新加载。
   - 自动重试更新逻辑（根据需求决定）。
2. **字段类型匹配**：确保数据库 `version` 字段的类型与实体类一致。
3. **只对数据更新操作生效**：乐观锁仅影响更新操作，对新增或删除无影响。

------

### 6.5 实践总结

适用场景：**高并发业务**，避免**数据覆盖**（如订单状态更新、库存扣减）。

注意：如果业务中版本号字段不适用，乐观锁可能不合适。

通过乐观锁，开发者可以轻松实现数据一致性的保障，尤其在高并发场景下，其**非阻塞特性**是关键优势。
