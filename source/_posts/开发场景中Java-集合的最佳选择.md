---
title: 开发场景中Java 集合的最佳选择
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2026-01-11 13:34:30
updated:
description:
cover:
---
> 在 Java 开发中，集合类是处理数据的核心工具。合理选择集合，不仅可以提高代码效率，还能让代码更简洁。本篇文章将重点探讨 **List、Set** 和 **Map** 的适用场景及优缺点，帮助你在实际开发中找到最佳解决方案。

## 一、List：有序存储的/最佳选择

### 1. ArrayList：快速查询与动态数组

**应用场景**：当你需要**频繁查询元素**，或者存储的元素数目动态变化时，`ArrayList` 是首选。例如：分页展示用户数据。

代码示例：

```java
List<String> users = new ArrayList<>();
users.add("Alice");
users.add("Bob");
System.out.println(users.get(1)); // 输出 Bob
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**：`ArrayList` 使用一个 **动态数组** 来存储元素。初始时，数组的大小是固定的，当元素超过数组的容量时，会自动扩展数组的大小。

**优点**：

- **查询效率高**：数组支持按索引快速访问元素，时间复杂度为 `O(1)`，因此 `get()` 操作非常高效。
- **内存局部性**：数组存储在连续的内存空间中，CPU 缓存友好，可以利用 CPU 的缓存机制提高访问效率。

**缺点**：

- **插入和删除效率低**：当插入或删除元素时，尤其是在中间位置时，必须移动数组中的大量元素，时间复杂度为 `O(n)`。
- **扩容操作代价高**：数组扩容时需要分配新的数组并将旧数组元素复制到新数组，操作的时间复杂度为 `O(n)`。

------

### 2. LinkedList：高效增删的双向链表

**应用场景**：需要频繁在列表中间或首尾**插入、删除数据**时，例如实现任务队列。

代码示例：

```java
LinkedList<String> tasks = new LinkedList<>();
tasks.addFirst("Task1");
tasks.addLast("Task2");
tasks.removeFirst();
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**：`LinkedList` 使用 **双向链表**，每个元素都有**两个指针**：一个指向前一个元素，一个指向下一个元素。这样可以在常数时间内插入或删除元素。

**优点**：

- **插入和删除高效**：无论是在链表的头部、中部还是尾部，插入和删除元素的时间复杂度都为 `O(1)`，因为只需要改变相关节点的指针。
- **内存使用灵活**：每个元素的内存可以分散存储，不需要连续的内存块。

> 【比如**在中间插入**】
>
> 假设需要在链表中的某个位置 `node` 前插入新节点 `newNode`：
>
> 1. 将 `newNode` 的 `next` 指向 `node`，将 `newNode` 的 `prev` 指向 `node.prev`。
> 2. 更新 `node.prev.next` 为 `newNode`，更新 `node.prev` 为 `newNode`。
>
> 这只涉及 4 次指针操作，与链表的长度无关，因此在已定位到目标节点后，插入操作的时间复杂度为 **`O(1)`**。
>
> ![img](assets/098a06dd19514e5c895fc0cf8e0ea672-1768138492224-40.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

**缺点**：

- **查询效率低**：为了查找元素，必须从头节点或尾节点开始遍历链表，时间复杂度为 `O(n)`。
- **内存开销大**：每个元素都需要额外存储指向前后元素的指针，相较于数组，占用更多的内存。

------

## 二、Set：无重复集合的首选

### 1. HashSet：高效去重

**应用场景**：当需要存储一组**不允许重复的元素**，且对顺序没有要求时，例如用户注册时验证用户名的唯一性。

代码示例

```java
Set<String> usernames = new HashSet<>();
usernames.add("Alice");
usernames.add("Bob");
usernames.add("Alice"); // 重复的元素会被忽略
System.out.println(usernames.size()); // 输出 2
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**：`HashSet` 使用 **哈希表**（`HashMap`）【哈希表在文末有**补充讲解**】来存储元素。哈希表通过将元素的哈希码映射到表中的桶来进行存储，确保元素是唯一的。

**优点**：

- **去重高效**：哈希表能够快速判断元素是否已存在，因为它**通过哈希值进行查找**，时间复杂度为 `O(1)`。
- **查询效率高**：哈希表的查找时间复杂度为 `O(1)`，因此 `contains()` 和 `add()` 操作非常高效。

**缺点**：

- **无序存储**：哈希表**并不维护元素的顺序**，因此 `HashSet` 中的元素是无序的。
- **哈希冲突**：不同的元素可能具有相同的哈希值，哈希冲突会影响性能，但通常情况下，哈希表的设计会尽量减少冲突的概率。

------

### 2. LinkedHashSet：有序去重

**应用场景**：当需要**去重的同时保留插入顺序**，例如记录用户最近浏览的商品。

代码示例：

```java
Set<String> products = new LinkedHashSet<>();
products.add("Laptop");
products.add("Phone");
products.add("Laptop"); // 再次添加无效
System.out.println(products); // 输出 [Laptop, Phone]
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**：`LinkedHashSet` 使用一个 **哈希表** 来存储元素，并通过一个 **双向链表** 来维护元素的插入顺序。

**优点**：

- **有序存储**：由于链表的存在，`LinkedHashSet` 能够保持元素的插入顺序，访问时能够按照插入的顺序遍历元素。
- **去重高效**：与 `HashSet` 一样，哈希表提供了快速的查找和去重机制。

**缺点**：**性能略低于 HashSet**，由于还需要维护链表，`LinkedHashSet` 的操作稍微比 `HashSet` 慢，但差距通常不大。

------

### 3. TreeSet：排序与去重兼备

**应用场景**：当需要**去重的同时对元素进行排序**，例如实现排行榜或数据字典。

代码示例：

```java
TreeSet<Integer> scores = new TreeSet<>();
scores.add(50);
scores.add(80);
scores.add(70);
System.out.println(scores); // 输出 [50, 70, 80]
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**：`TreeSet` 使用 **红黑树** 来存储元素。红黑树是一种自平衡的二叉搜索树，能够确保树的深度保持在对数级别。

![img](assets/7e017ce609914c8e8449e0f32560e1d1.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

**优点**：

- **有序存储**：`TreeSet` 会自动对元素进行排序，默认按**自然顺序排序**（`compareTo(Object obj)`）或者通过传入 `Comparator` **自定义排序**）。
- **查找、插入和删除的时间复杂度为 `O(log n)`**：由于红黑树的结构特性，所有操作的时间复杂度为对数级别。

**缺点**：**性能较低，**相比哈希表，红黑树的插入、删除和查找操作的时间复杂度为 `O(log n)`，因此在大量数据操作时，性能略逊色于 `HashSet` 和 `LinkedHashSet`。

## 三、Map：键值对存储的首选

`Map` 是存储键值对的集合类，每个键唯一对应一个值。常用于快速查找和关联关系的存储。

### 1. HashMap：高效的键值映射

**应用场景**：需要**高效查找**时，例如存储用户 ID 和用户信息的映射。

代码示例：

```java
Map<Integer, String> userMap = new HashMap<>();
userMap.put(1, "Alice");
userMap.put(2, "Bob");
System.out.println(userMap.get(1)); // 输出 Alice
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**：`HashMap` 使用 **哈希表** 来存储键值对，通过键的哈希码来确定存储位置。

**优点**：

- **查找和插入高效**：查找、插入和删除操作的时间复杂度为 `O(1)`，通过哈希值直接定位位置。
- **支持键值对的存储**：每个键对应唯一的值，适合各种映射操作。

**缺点**：

- **无序存储**：哈希表中的元素是无序的，因此遍历时无法保证顺序。

------

### 2. LinkedHashMap：有序的键值映射

**应用场景**：需要既保持**插入顺序**，又能高效查找，例如实现最近访问页面的缓存。

代码示例：

```java
Map<Integer, String> accessLog = new LinkedHashMap<>();
accessLog.put(1, "HomePage");
accessLog.put(2, "ProfilePage");
accessLog.put(3, "SettingsPage");
System.out.println(accessLog); // 输出 {1=HomePage, 2=ProfilePage, 3=SettingsPage}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**：`LinkedHashMap` 使用 **哈希表** 存储元素，并通过 **双向链表** 维护元素的插入顺序。

**优点**：

- **有序存储**：保持了元素的插入顺序，遍历时能够按照插入顺序输出。
- **高效查找**：与 `HashMap` 一样，查询和插入操作的时间复杂度为 `O(1)`。

**缺点**：**内存开销较大，**需要额外的内存来存储链表指针。

------

### 3. TreeMap：有序的键值存储

**应用场景**：需要**按键排序存储键值对**，例如实现字典或排行榜。

代码示例：

```java
Map<Integer, String> sortedMap = new TreeMap<>();
sortedMap.put(3, "C");
sortedMap.put(1, "A");
sortedMap.put(2, "B");
System.out.println(sortedMap); // 输出 {1=A, 2=B, 3=C}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

- **底层结构**：`TreeMap` 使用 **红黑树** 来存储键值对，按照键的自然顺序（或通过指定的 `Comparator`）进行排序。
- **优点**：
  - **有序存储**：自动对键进行排序，适用于需要顺序访问键值对的场景。
  - **高效的查找、插入和删除**：操作时间复杂度为 `O(log n)`。
- **缺点**：**性能略低于 `HashMap` 和 `LinkedHashMap，`**由于红黑树需要维护平衡，操作的时间复杂度为对数级别，性能不如哈希表

### 4. Properties

**应用场景**

- `Properties` 常用于管理应用程序的配置信息，如数据库连接信息、语言国际化资源等。
- 它可以方便地加载和存储键值对到 `.properties` 文件中，支持流式操作。

代码示例：

```java
import java.io.*;
import java.util.Properties;

public class PropertiesExample {
    public static void main(String[] args) throws IOException {
        Properties properties = new Properties();
        
        // 设置键值对
        properties.setProperty("database.url", "jdbc:mysql://localhost:3306/mydb");
        properties.setProperty("database.user", "root");
        properties.setProperty("database.password", "password");

        // 保存到文件
        try (FileOutputStream output = new FileOutputStream("config.properties")) {
            properties.store(output, "Database Configuration");
        }

        // 从文件加载
        try (FileInputStream input = new FileInputStream("config.properties")) {
            properties.load(input);
        }

        // 打印所有属性
        properties.forEach((key, value) -> System.out.println(key + ": " + value));
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**底层结构**

- `Properties` 的基础是 `Hashtable`：
  - 底层采用线程安全的哈希表结构。
  - 键和值均为字符串类型（`String`），以适应配置文件的存储和解析需求。
  - 提供了 `load()` 和 `store()` 方法，用于流式操作，方便配置文件的读写。

**优点**

1. **简单直观**：内置方法支持直接操作配置文件，减少手动解析的复杂性；适合存储和管理小规模配置。
2. **线程安全**：继承自 `Hashtable`，所有操作均是同步的，适合简单的多线程环境。
3. **与文件系统集成良好**：提供了流式操作接口，方便将键值对直接保存为 `.properties` 文件或从文件中加载。

**缺点**

1. **性能较低**：由于继承自同步的 `Hashtable`，在现代高并发场景下不推荐使用，性能落后于 `HashMap`。
2. **局限性**：仅支持 `String` 类型的键值对，若需要存储复杂对象，需额外序列化。
3. **不适合大规模配置**：适合小型项目或简单模块的配置管理，大型系统建议采用更复杂的配置管理工具（如 Apache Commons Configuration 或 Spring）。

## 总结

| **集合类**        | **底层数据结构**             | **主要优点**                                        | **主要缺点**                                       |
| ----------------- | ---------------------------- | --------------------------------------------------- | -------------------------------------------------- |
| **ArrayList**     | 动态数组                     | 查询效率高，支持随机访问                            | 插入删除效率低，扩容代价大                         |
| **LinkedList**    | 双向链表                     | 插入删除效率高，内存灵活                            | 查询效率低，占用内存大                             |
| **HashSet**       | 哈希表                       | 查询和去重效率高                                    | 无序存储，受哈希冲突影响                           |
| **LinkedHashSet** | 哈希表 + 双向链表            | 有序存储，去重效率高                                | 内存占用较高                                       |
| **TreeSet**       | 红黑树                       | 有序存储，按自然顺序或自定义顺序排序                | 性能略低于哈希表，操作复杂度为 `O(log n)`          |
| **HashMap**       | 哈希表                       | 查找和插入效率高，支持键值对映射                    | 无序存储，受哈希冲突影响                           |
| **LinkedHashMap** | 哈希表 + 双向链表            | 有序存储，按插入顺序遍历键值对                      | 内存开销较高                                       |
| **TreeMap**       | 红黑树                       | 有序存储，按键自然顺序或自定义顺序排序              | 性能略低于哈希表，操作复杂度为 `O(log n)`          |
| **Properties**    | 哈希表（继承自 `Hashtable`） | 专为存储键值对配置设计，支持读写 `.properties` 文件 | 性能较低（继承自同步的 `Hashtable`），不适合高并发 |

## 知识点补充

### 1. 什么是哈希表？

哈希表是一种用来存储 **键值对** 的工具，它能非常快速地找到数据。你可以把它想象成一个 **带编号的储物柜**，每个柜子都有一个编号（索引），你把东西存进去时，会根据物品的特点计算出一个编号，然后直接放进对应的柜子里。取东西时也用相同的方法计算编号，直接找到对应的柜子打开拿走。

**例子**：

- 假如你有一本字典，查找某个单词（键）对应的解释（值）。
- 传统查找方法：逐页翻阅，耗时长。
- 使用哈希表：计算单词的编号，直接跳到对应的位置查看解释，速度非常快。

**总结类比**：

- 键是 “单词”，值是 “解释”。
- 哈希表通过哈希函数快速找到这个单词在哪页。

------

### 2. 哈希表是如何存数据的？

哈希表的核心在于 **哈希函数**。这个函数就像一个计算器，可以把一个键（比如一个字符串）**变成一个数字（哈希值）**。哈希值用来确定数据存储的位置。

**步骤：**

1. **计算位置**：用哈希函数把键变成一个数字，然后对储物柜的总数取模（%），确定放在哪个柜子里。假如储物柜有 10 个，"Alice" 的哈希值是 42，42 % 10 = 2，所以数据放到第 2 个柜子。
2. **存储值**：把数据放到计算出的柜子里。