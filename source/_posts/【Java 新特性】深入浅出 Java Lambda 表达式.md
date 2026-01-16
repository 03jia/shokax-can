```
title: 【Java 新特性】常用函数式接口
keywords: []
categories:
    - [后端,语言,Java]
tags:
	- Java
    - Lambda
date: 2025-01-04 20:04:39
updated:
description: "讲解 Java Lambda 表达式的语法与应用场景，展示如何用函数式风格简化代码。"
cover: "https://s2.loli.net/2026/01/11/s6AKfIi1G8THlOy.jpg"
```

>  Java 8 的 **Lambda 表达式**是一次编程方式的革命，让代码更加简洁、高效。本文将从基础语法入手，逐步深入讲解 Lambda 表达式的常用用法、进阶场景以及开发中的实战案。

------

## 一、 **什么是 Lambda 表达式？**

Lambda 表达式是一种匿名函数，旨在减少冗长的代码。其核心在于将功能作为参数传递。

简单来说，它让代码从这样的写法：

```java
new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, Lambda!");
    }
};
```

变得简洁如斯：

```java
() -> System.out.println("Hello, Lambda!");
```

------

## 二、**Lambda 表达式的基础语法**

Lambda 表达式的基本语法如下：

```java
(parameters) -> expression
(parameters) -> { statements; }
```

### **1. 无参数，无返回值**

```java
() -> System.out.println("Hello, World!");
```

- `()`：无参数
- `->`：表示 Lambda 的分隔符
- `System.out.println("Hello, World!")`：**单行表达式**作为 Lambda 体。

### **2. 有参数，无返回值**

接收一个参数 `x`，并打印。

```java
(x) -> System.out.println(x);
```

### **3. 有多个参数，有返回值**

返回两个参数的和，省略了 `{}` 和 `return`。

```java
(x, y) -> x + y
```

### **4. 多行代码**

如果 Lambda 体有**多行代码**，必须用 `{}` 包裹，并显式使用 `return` 返回值：

```java
(x, y) -> {
    int sum = x + y;
    System.out.println("Sum: " + sum);
    return sum;
}
```

------

## 三、**Lambda 表达式的常用场景**

### 场景 1：代替匿名内部类

Lambda 表达式常用于简化匿名类的实现。

**传统写法**：

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running...");
    }
};
```

**Lambda 简化**：

```java
Runnable runnable = () -> System.out.println("Running...");
```

### 场景 2：配合函数式接口

Lambda 表达式的核心是与 **函数式接口** 搭配使用。函数式接口是只包含一个抽象方法的接口，例如 `java.util.function` 包中的接口：

| **接口**        | **描述**                 | **示例**                     |
| --------------- | ------------------------ | ---------------------------- |
| `Consumer<T>`   | 接收一个参数，无返回值   | `x -> System.out.println(x)` |
| `Supplier<T>`   | 无参数，返回一个值       | `() -> "Hello"`              |
| `Function<T,R>` | 接收一个参数，返回一个值 | `x -> x.length()`            |
| `Predicate<T>`  | 接收一个参数，返回布尔值 | `x -> x > 0`                 |

**示例：过滤集合**：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.stream()
       .filter(x -> x % 2 == 0)  // 筛选偶数
       .forEach(System.out::println);
```

------

## **四、方法引用与构造器引用**

Lambda 表达式的简化之道之一就是方法引用和构造器引用。

### 1. 方法引用

方法引用是 Lambda 表达式的一种简化形式，它通过 `::` 运算符直接引用已有的方法。

| **类型**           | **示例**                    | **说明**              |
| ------------------ | --------------------------- | --------------------- |
| 静态方法引用       | `ClassName::staticMethod`   | `Math::max`           |
| 实例方法引用       | `instance::instanceMethod`  | `System.out::println` |
| 任意对象的方法引用 | `ClassName::instanceMethod` | `String::toUpperCase` |
| 构造器引用         | `ClassName::new`            | `ArrayList::new`      |

**示例：静态方法引用**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
numbers.forEach(System.out::println); // 等价于 x -> System.out.println(x)
```

**示例：实例方法引用**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(String::toUpperCase); // 每个字符串转为大写
```

### 2. 构造器引用

通过构造器引用，可以动态创建对象实例。

**示例：无参构造器**

```java
Supplier<List<String>> supplier = ArrayList::new;
List<String> list = supplier.get(); // 创建一个新的 ArrayLit
```

**示例：有参构造器**

```java
Function<String, Integer> converter = Integer::new;
Integer number = converter.apply("123"); // 将字符串 "123" 转为 Integer
```

------

## 五、**开发中的实战案例**

### 1. 集合排序

使用 Lambda 表达式快速排序：

```java
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
names.sort((a, b) -> a.compareTo(b)); // 按字典序排序
```

进一步简化为方法引用：

```java
names.sort(String::compareTo);
```

### 2. 线程池任务

Lambda 在多线程中非常适合，尤其是简化任务提交：

```java
ExecutorService executor = Executors.newCachedThreadPool();
executor.submit(() -> System.out.println("Task running in thread!"));
```

### 3. 文件操作

使用 Lambda 简化文件行处理：

```java
Files.lines(Paths.get("data.txt"))
     .filter(line -> line.startsWith("Error")) // 筛选以 "Error" 开头的行
     .forEach(System.out::println);
```

------

## 六、**总结表格**

| **概念**        | **语法示例**                 | **应用场景**                  |
| --------------- | ---------------------------- | ----------------------------- |
| Lambda 基本语法 | `(x, y) -> x + y`            | 替代匿名类、简化代码          |
| Consumer 接口   | `x -> System.out.println(x)` | 消费一个值，例如打印、保存    |
| Supplier 接口   | `() -> "Hello"`              | 提供一个值，例如默认值        |
| Function 接口   | `x -> x.length()`            | 转换一个值，例如字符串长度    |
| Predicate 接口  | `x -> x > 0`                 | 判断条件，例如筛选集合        |
| 静态方法引用    | `Math::max`                  | 引用已有静态方法              |
| 实例方法引用    | `System.out::println`        | 引用已有对象的方法            |
| 构造器引用      | `ArrayList::new`             | 动态创建实例，例如 `Supplier` |
