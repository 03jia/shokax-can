---
title: 【Java 新特性】常用函数式接口
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2025-01-03 16:14:45
updated:
description:
cover:
---
>  在现代 Java 开发中，**函数式接口**无疑是一种强大的工具。它们不仅让**代码更简洁**，也使开发者能够更灵活地表达逻辑。函数式接口就像电器的插头，为你的代码提供了“标准接口”。想象一下，有了插头，不同的设备都能轻松接入电源。同样，函数式接口让你可以用统一的方式编写逻辑，无论是数据处理、条件判断还是结果生成，都变得更灵活、更简洁。

------

## 一、什么是函数式接口？

函数式接口是 **仅包含一个抽象方法** 的接口。它们是 Java 8 中引入的核心特性，常与 **Lambda 表达式** 一起使用。Java 提供了 `@FunctionalInterface` 注解来显式声明一个接口为函数式接口，同时也让编译器帮助我们验证其规范性。

> 关于 **Lambda 表达式**的详细讲解可以参考我的另一篇博客

**函数式接口的基本规则：**

1. 必须**只有一个抽象方法**。
2. 可以包含多个默认方法或静态方法。
3. 使用 `@FunctionalInterface` 注解并非强制，但推荐使用。

### 形象比喻

函数式接口就像一张定制的契约（合同），规定只有一个核心任务需要实现。例如，`Runnable` 的任务是 **"做某件事情"**，具体做什么由你定义，而 Java 会帮你安排如何调用。

### 定义示例

```java
// 此自定义函数式接口接收两个参数，返回一个 int 类型的值
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

实现时：

```java
Calculator addition = (a, b) -> a + b;
System.out.println(addition.calculate(5, 3)); // 输出：8
```

------

## 二、为什么需要函数式接口？

在 Java 8 之前，我们实现逻辑常需要匿名内部类，这既冗长又不直观。**函数式接口加上 Lambda 表达式**，简直就是如虎添翼。

对比代码：

- **匿名内部类写法**：

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, World!");
    }
};
```

- **Lambda 表达式写法**：

```java
Runnable task = () -> System.out.println("Hello, World!");
```

是不是清爽简洁多了？

------

## 三、常用的函数式接口

Java 提供了一整套**常用的函数式接口**，它们就像工具箱中的各种工具，针对不同任务有不同功能。

| **接口**            | **方法签名**            | **用途**                         | **例子类比**                                                 |
| ------------------- | ----------------------- | -------------------------------- | ------------------------------------------------------------ |
| `Function<T,R>`     | `R apply(T t)`          | 将一个参数映射为另一个结果       | 就像去买东西，**一手交钱（T）一手交货（R）**                 |
| `Consumer<T>`       | `void accept(T t)`      | 消费一个参数，无返回值           | 就像**吃食物**（T），吃完就**消化了**（void）。              |
| `Supplier<T>`       | `T get()`               | 无参数，返回一个结果             | 就像一台**免费物品供应机**，按下按钮给你**物品**（T）。      |
| `Predicate<T>`      | `boolean test(T t)`     | 判断条件，返回布尔值             | 就像一台**安检机**，判断**物品**（T）是否符合安全标准（boolean 类型）。 |
| `BiFunction<T,U,R>` | `R apply(T t, U u)`     | 接收两个参数，返回一个结果       | 就像**输入两个不同类型的值**（T、U），输出操作后的任意结果（R）。 |
| `BiConsumer<T,U>`   | `void accept(T t, U u)` | 接收两个参数，无返回值           | 对**两个不同类型的值**（T、U）进行操作，不返回结果           |
| `UnaryOperator<T>`  | `T apply(T t)`          | **一元操作**，输入和输出类型相同 | 对一个值进行操作，**输入（T）和输出（T）的类型没变**。       |
| `BinaryOperator<T>` | `T apply(T t1, T t2)`   | **二元操作**，输入和输出类型相同 | 对**两个相同类型的值**进行操作，返回操作后相同类型的结果（T）。 |

------

## 四、函数式接口基础用法

### 1. `Function`：将输入映射为输出

```java
Function<String, Integer> stringToLength = s -> s.length();
System.out.println(stringToLength.apply("Hello")); // 输出：5
```

### 2. `Predicate`：判断条件是否成立

```java
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // 输出：true
System.out.println(isEven.test(3)); // 输出：false
```

### 3. `Supplier`：无条件生成结果

```java
Supplier<Double> randomSupplier = () -> Math.random();
System.out.println(randomSupplier.get());
```

### 4. `Consumer`：处理输入，不返回结果

```java
Consumer<String> logger = message -> System.out.println("Log: " + message);
logger.accept("Application started"); // 输出：Log: Application started
```

### 5. `UnaryOperator`：对输入进行一元操作

```java
UnaryOperator<Integer> increment = n -> n + 1;
System.out.println(increment.apply(5)); // 输出：6
```

### 6. `BinaryOperator`：对两个输入进行二元操作

```java
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
System.out.println(max.apply(5, 10)); // 输出：10
```

------

## 五、开发中常用场景

### 1. 使用 `Function` 进行 map 转换

```java
List<Integer> integersList = List.of(1, 2, 3);
List<Integer> mapList = integersList.stream()
    .map(i -> (i * 2))
    .toList();
```

### 2. 使用 `Function` 实现多步转换

就像流水线上的加工流程，一个任务接着另一个任务：

```java
Function<Integer, Integer> multiplyBy2 = x -> x * 2;
Function<Integer, Integer> add3 = x -> x + 3;

Function<Integer, Integer> combined = multiplyBy2.andThen(add3);
System.out.println(combined.apply(5)); // 输出：13
```

### 3. 使用 `Predicate` 实现复杂条件判断

类比：安检门，多个条件逐步筛选。

```java
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> endsWithZ = s -> s.endsWith("Z");

Predicate<String> complexCondition = startsWithA.and(endsWithZ);

System.out.println(complexCondition.test("AZ")); // 输出：true
System.out.println(complexCondition.test("BZ")); // 输出：false
```

### 4. 使用 `Predicate` 进行过滤

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println); // 输出：Alice
```

### 5. 使用 `Supplier` 延迟加载资源

```java
Supplier<String> configSupplier = () -> {
    System.out.println("Fetching configuration...");
    return "Default Config";
};

// 只有在调用 get() 时才会执行逻辑
System.out.println(configSupplier.get()); // 输出：Fetching configuration...
```

### 6. 使用 `Consumer` 记录日志

专注于记录，但不返回结果。

```java
Consumer<String> logger = message -> System.out.println("Log: " + message);
logger.accept("User logged in"); // 输出：Log: User logged in
```

### 7. 使用 `BiConsumer` 处理键值对

```java
BiConsumer<String, Integer> printOrder = (item, quantity) -> 
    System.out.println("Ordered " + quantity + "x " + item);

printOrder.accept("Apple", 3); // 输出：Ordered 3x Apple
```

### 8. 使用 `UnaryOperator` 实现批量自增

每个商品都增加同样的数量。

```java
UnaryOperator<Integer> incrementBy10 = x -> x + 10;
List<Integer> numbers = List.of(1, 2, 3, 4);
List<Integer> incrementedNumbers = numbers.stream()
    .map(incrementBy10)
    .toList();

System.out.println(incrementedNumbers); // 输出：[11, 12, 13, 14]
```

### 9. 使用 `BinaryOperator` 处理累积运算（聚合数据）

像汇总账单，每次合并两个部分，直到得出总数。

```java
BinaryOperator<Integer> sumOperator = (a, b) -> a + b;
List<Integer> nums = List.of(1, 2, 3, 4, 5);
int total = nums.stream()
    .reduce(0, sumOperator);

System.out.println(total); // 输出：15
```

## 六、总结

函数式接口让代码更简洁、更灵活，不再是过去冗长的匿名类实现。通过以下这张表格，你可以快速回忆常用的几个函数式接口的用途：

| **接口**            | **方法签名**            | **主要用途**                               |
| ------------------- | ----------------------- | ------------------------------------------ |
| `Function<T,R>`     | `R apply(T t)`          | 接受一个输入，返回一个输出，常用于映射操作 |
| `Consumer<T>`       | `void accept(T t)`      | 消费输入，无返回值，常用于执行操作         |
| `Supplier<T>`       | `T get()`               | 无输入，生成结果，常用于延迟加载           |
| `Predicate<T>`      | `boolean test(T t)`     | 判断条件，返回布尔值，用于过滤操作         |
| `UnaryOperator<T>`  | `T apply(T t)`          | 对输入执行一元操作，返回同类型结果         |
| `BinaryOperator<T>` | `T apply(T t1, T t2)`   | 对两个输入执行二元操作，返回同类型结果     |
| `BiFunction<T,U,R>` | `R apply(T t, U u)`     | 接收两个不同类型输入，返回一个输出         |
| `BiConsumer<T,U>`   | `void accept(T t, U u)` | 消费两个不同类型输入，无返回值             |
