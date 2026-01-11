---
title: 【Spring核心思想】面向切面编程（AOP）
keywords: []
categories:
    - [后端,框架,Spring]
tags:
    - Spring
    - AOP
date: 2024-12-22 15:52:36
updated:
description: "介绍 AOP 的核心概念与作用，讲解如何通过切面解耦横切关注点并在 Spring 中实现。"
cover:

---

> 在现代应用开发中，我们经常会遇到一些通用需求，比如日志记录、权限校验、事务管理等。这些需求**并不直接属于业务逻辑**，却必须融入系统的各个角落。如果按照传统开发方式，往往需要在每个业务代码中反复编写这些功能，结果**导致代码冗余、耦合性高，维护起来异常繁琐**。
>
> ------
>
> 那么，是否有一种方法能将这些“横切关注点”**从业务逻辑中解耦出来**呢？答案就是 **AOP（Aspect-Oriented Programming，面向切面编程）**。AOP 是一种强大的编程范式，它可以在**不修改原始代码**的情况下，通过动态代理技术，在程序运行过程中切入额外的逻辑。换句话说，AOP 让我们能够像“施魔法”一样，将日志记录、性能监控、安全检查等功能无缝地织入到系统的各个角落，而业务代码依然保持简洁干净。
>
> ------
>
> 在本篇博客中，我们将带你深入理解 AOP 的核心概念和运行机制，并通过 Spring AOP 框架的实际应用，基于实现一个“日志切面”，让你切身体会 AOP 的魅力。从理论到实战，我们将层层剖析 AOP 的核心术语，如切点、通知、切面等，并揭示其背后基于动态代理的实现原理。

## 一、AOP介绍

AOP（Aspect-Oriented Programming，面向切面编程）是一种**编程范式**，用于 **分离横切关注点**，以增强代码模块化和可维护性。

**核心概念**

- **横切关注点**：横切关注点是指散布在系统中、与业务逻辑无直接关系但对多个模块都有影响的功能，例如日志记录、事务管理、性能监控和安全校验等。
- **切面**：切面（Aspect）是横切关注点的实现部分，用于定义和应用这些通用功能。

**核心目标**

在 **不修改原有代码** 的情况下增强功能，通过**动态代理机制**，能够在业务过程中某些业务方法执行的前后或异常时切入自定义逻辑。

## 二、为什么需要用到AOP

传统的开发流程中，一些横切关注点（比如日志、事务管理等），通常会分散在每个不同的业务逻辑方法中，导致一些问题：代码复用性差，耦合度高（没有单独将一些业务逻辑无关代码抽离出来，难以维护修改），可读性差

一般一个系统当中都会有一些系统服务，例如：日志、事务管理、安全等。这些系统服务被称为：**交叉业务**，交叉业务代码在多个业务流程中反复出现，显然这个交叉业务代码**没有得到复用**。并且修改这些交叉业务代码的话，需要**修改多处**。所以需要把这些业务代码抽取出来封装为一个独立的组件，然后以横向交叉的方式应用到具体的业务流程中。

## 三、AOP的核心概念和术语

基于动态代理来实现切面功能

------

### 1. 连接点（Joinpoint）

**定义**：程序执行过程中，可以被切面增强的具体点，是一个抽象的概念。

> 【例如】方法的调用、方法的返回、异常抛出等。

**说明**：在 Spring AOP 中，连接点是指方法的执行，因为 Spring AOP 仅支持方法级别的连接点（不支持字段访问或构造函数）。

------

### 2. 切点（Pointcut）

**定义**：用于定义在哪些连接点上应用切面逻辑。是对连接点的过滤规则。

**关系**：一个切点对应多个连接点。

**实现**：在 Spring 中，切点通过表达式来定义

> 例如：`execution(* com.example.service.*.*(..))` 表示匹配 `com.example.service` 包下所有类的所有方法。

------

### 3. 通知（Advice）

**定义**：通知是切面中的具体操作，指在连接点上执行的增强逻辑代码。

**类型**：Spring AOP 提供了以下几种通知：

- **前置通知（Before Advice）**：在目标方法执行之前执行。
- **后置通知（After Returning Advice）**：在目标方法成功返回后执行。
- **环绕通知（Around Advice）**：包裹目标方法的执行，可以在方法执行前后自定义逻辑。
- **异常通知（After Throwing Advice）**：在目标方法抛出异常时执行。
- **最终通知（After Advice）**：无论目标方法是否正常执行完成，都会执行。

------

### 4. 切面（Aspect）

**定义**：切面是**切点和通知的结合**，是 AOP 的核心概念。

- **切点**：定义了**在哪里**应用增强逻辑。
- **通知**：定义了增强逻辑的**具体内容**。

> 示例：一个**日志切面**可能包含以下逻辑
>
> - 在指定的方法执行前记录日志（前置通知）。
> - 在方法抛出异常时记录错误日志（异常通知）。

------

### 5. 织入（Weaving）

**定义**：将切面逻辑动态应用到目标对象的过程。

**方式**：

- **编译时织入**：通过工具在代码编译阶段将切面逻辑织入到目标类。【例如，AspectJ 支持这种方式】
- **类加载时织入**：在目标类被类加载器加载时动态修改字节码。
- **运行时织入**：通过动态代理在程序运行时将切面逻辑织入到目标对象。【Spring AOP 使用此方式】

------

### 6. 代理对象（Proxy）

**定义**：通过 **AOP 增强后的目标对象**，表现为目标对象的一个**代理（增强）版本**。

**特点**：

- 对外表现为目标对象的实例。
- 内部包含切面逻辑，用于执行增强代码。

**实现方式**：

- **JDK 动态代理**：针对实现了接口的目标对象生成代理类。
- **CGLIB 动态代理**：针对没有接口的目标对象，通过字节码技术生成子类代理。

------

### 7. 目标对象（Target）

**定义**：目标对象是被织入切面逻辑的对象，即**未增强之前的原始对象**。

**说明**：AOP 的核心是增强目标对象的功能，同时保持目标对象的核心业务逻辑不变。

------

### AOP 七大术语的关系图

1. **目标对象** 是**原始**的业务类。
2. **切点** 定义了哪些方法或**位置**需要增强。
3. **通知** 是增强的具体**逻辑**。
4. **切面** 是切点和通知的集合。
5. **织入** 是将通知应用到目标对象的**过程**。
6. **代理对象** 是**织入后**的目标对象。
7. **连接点** 是程序执行过程中**可被增强的具体位置**。

------

### 实例示例：日志记录切面

**目标对象**

```java
@Service
public class UserService {
    public void addUser() {
        System.out.println("Adding user...");
    }
}
```

**切面定义**

```java
@Aspect
@Component
public class LoggingAspect {
    
    /* 先定义切点在定义通知 */
    // 切点定义
    @Pointcut("execution(* com.example.service.UserService.*(..))")
    public void userServiceMethods() {}

    // 前置通知
    @Before("userServiceMethods()")
    public void logBefore() {
        System.out.println("Logging before method execution...");
    }
    
    /* 也可以切点和通知合并，前置通知，直接嵌入切点表达式 */
    @Before("execution(* com.example.service.UserService.*(..))")
    public void logBefore() {
        System.out.println("Logging before UserService method...");
    }
}
```

**执行流程**

1. **切点匹配**：`userServiceMethods` 定义的切点会匹配 `UserService` 的所有方法。

   > 切点方法名称完全是开发者定义的，Spring 只关心切点表达式是否正确。

2. **通知增强**：在 `addUser` 方法执行前，调用 `logBefore` 方法输出日志。

3. **织入代理**：Spring AOP 生成 `UserService` 的代理对象，代理对象负责调用增强逻辑和目标方法。

   > 当 Spring 检测到**某个 Bean 被 AOP 增强时**，会**用代理对象替代原始对象**，将代理对象放入 IOC 容器中。因此，通过 `@Autowired` 或手动从容器中获取的 Bean，其实是**代理对象**，而不是目标对象本身。
   >
   > 【大致流程】Spring在**创建Bean的初期**，通过构造函数或者工厂方法创建目标对象的实例之前会**先检查是否需要增强**（如是否有匹配到的**切点表达式**），如果检测到**需要增强**则为目标对象**生成代理对象以及具有相应需要增强的方法**，之后将代理对象**注入ICO容器中注册**，当外部调用方法的时候，代理对象通过**拦截器**拦截目标对象的方法以及处理增强逻辑。

**执行结果**：

- 方法调用前，打印日志信息。
- 实现完全解耦，日志逻辑与业务逻辑分离。

```XML
Logging before method execution...
Adding user...
```

## 四、核心机制

Spring AOP 基于动态代理来实现切面功能：

1. **JDK 动态代理**
   - 用于代理实现了接口的类。
   - 动态生成代理类，通过接口调用实际目标对象，并在调用前后添加增强逻辑。
2. **CGLIB 动态代理**
   - 用于代理没有实现接口的类。
   - 动态生成子类，覆盖目标类的方法，并在方法执行前后添加增强逻辑。

## 五、Spring AOP 的实际应用

1. 添加依赖

   ```XML
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```

2. 定义切面类

   ```java
   import org.aspectj.lang.annotation.*;
   
   @Aspect
   @Component
   public class LoggingAspect {
   
       // 切入点：拦截所有 controller 包下的方法
       @Pointcut("execution(* com.example.controller..*(..))")
       public void controllerMethods() {}
   
       // 前置通知
       @Before("controllerMethods()")
       public void logBefore() {
           System.out.println("Method execution started...");
       }
   
       // 环绕通知
       @Around("controllerMethods()")
       public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
           System.out.println("Before method execution");
           Object result = joinPoint.proceed(); // 执行目标方法
           System.out.println("After method execution");
           return result;
       }
       
        // 后置通知
       @After("controllerMethods()")
       public void logAfter() {
           System.out.println("Method execution finished...");
       }
   }
   ```

## 六、动态代理的原理

以 `环绕通知` 为例，动态代理的执行流程如下：

1. 目标对象的某个方法被调用。
2. Spring AOP 的代理对象捕获调用。
3. 根据切入点表达式，判断是否需要增强。
4. 如果需要，执行切面逻辑（如 `Before Advice`），然后调用目标方法。
5. 方法返回后，执行切面逻辑（如 `After Advice`）。

代理类的代码结构类似：

```java
public class ProxyUserService implements UserService {
    private final UserService target;

    public ProxyUserService(UserService target) {
        this.target = target;
    }

    @Override
    public void someMethod() {
        // 前置通知逻辑
        System.out.println("Before method");

        // 调用目标方法
        target.someMethod();

        // 后置通知逻辑
        System.out.println("After method");
    }
}
```

>  ***希望这篇博客能对你理解AOP有所启发，欢迎一起交流！***
