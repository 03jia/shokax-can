---
title: 从 Promise 到 Axios：轻松解锁异步编程
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2024-12-17 23:25:38
updated:
description:
cover:
---
> 如果你正在开发中处理异步任务，比如网络请求、文件操作，或者用户交互的处理，那么你一定接触过 **Promise** 和 **Async/Await**。它们是现代 **JavaScript 异步编程**的基石。本文将带你一步步深入了解，帮助你弄清它们的背景、解决的问题以及实际应用。希望这篇内容不仅能帮你理清思路，还能让你在开发时更得心应手。

------

## **一、从回调函数说起**

回调函数是异步编程最基础的方式。我们通过把一个函数传递给另一个函数，让它在原函数的任务完成后调用：

```javascript
setTimeout(() => {
  console.log("第一步完成");
  setTimeout(() => {
    console.log("第二步完成");
    setTimeout(() => {
      console.log("第三步完成");
    }, 1000);
  }, 1000);
}, 1000);
```

**问题来了：** 当任务需要按照一定顺序完成时，这种嵌套写法会变得非常混乱，特别是任务之间还需要处理错误或者传递结果的时候。于是，**Promise** 应运而生。

------

## **二、Promise：解救回调地狱**

### **Promise 是什么？**

Promise 是一种异步操作的“管理器”。它将任务的执行过程抽象成一个对象，包含两个核心部分：

1. **状态**：表示任务当前的进展（`pending`、`fulfilled`、`rejected`）。
2. **结果**：任务成功后会返回一个获取的数据（通过执行 **resolve()** 之后返回），失败时则返回一个原因（通过执行 **reject()** 之后返回）。

这听起来有点抽象，我们用一个简单的例子来说明：

```javascript
const fetchData = (url) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (url) {
        resolve(`数据来自：${url}`); // 成功时返回数据
      } else {
        reject("请求失败：URL 不能为空"); // 失败时返回错误原因
      }
    }, 1000);
  });
};

// 使用 Promise
fetchData("https://api.example.com")
  .then((data) => {
    console.log("成功：", data);
    return fetchData(""); // 模拟错误
  })
  .then((data) => {
    console.log("第二次成功：", data);
  })
  .catch((error) => {
    console.error("失败：", error);
  });
```

### **Promise 的几个重要点：**

1. **`resolve(value)`：** 任务成功时调用，`value` 就是返回的结果。
2. **`reject(reason)`：** 任务失败时调用，`reason` 是失败的原因。
3. `.then()` 用来处理成功的结果，而 `.catch()` 专门用来处理错误。

> **then()** 还支持链式调用，上一个处理返回的成功的结果可以传递给下一个 **then()**，可以在回调函数的**参数位置**获取

------

## **三、Async/Await：更优雅的异步处理**

Promise 虽然解决了回调嵌套的问题，但链式调用依然可能让代码变得不够直观。为了让异步代码看起来像同步代码一样易读，JavaScript 引入了 **Async/Await** 语法。

### **Async/Await 是什么？**

`async` 是用来定义一个**异步函数的关键字**，而 `await` 则用于**暂停代码的执行**，等待一个 Promise 解决后继续运行。

- **`await` 关键字的作用：**
  - `await` 用于**等待一个 Promise** 的解析（`fulfilled` 状态）或拒绝（`rejected` 状态）。
  - 如果 Promise **成功解析**，`await` 会**返回这个 Promise 的解析值（成功状态的数据）**。
  - 如果 Promise **被拒绝**，`await` 会**抛出一个异常**，需要通过 `try...catch` 捕获。
- **异步函数的返回值：**
  - 异步函数（通过 `async` 定义的函数）**本质上会返回一个 Promise**。
  - 如果异步函数内的所有操作都成功，则返回一个 **`fulfilled` 状态的 Promise**，并将 `return` 的值作为解析值。
  - 如果异步函数中抛出了错误（比如 `await` 遇到 `rejected` 的 Promise 或手动抛出异常），则返回一个 **`rejected` 状态的 Promise**，其被拒绝的原因是抛出的错误。

------

举个例子：

```javascript
const fetchDataAsync = async (url) => {
  try {
    const data = await fetchData(url); // 等待 Promise 完成
    console.log("成功：", data);
  } catch (error) {
    console.error("失败：", error); // 捕获错误
  }
};

fetchDataAsync("https://api.example.com");
```

**优势：**

- 代码更加直观，异步流程看起来像同步代码。
- 错误处理统一：`try...catch` 能捕获所有的异常。

> 在异步函数中**通过 await 执行一个能返回 promise对象 的函数**之后，如果成功获取了内容，则此**异步函数返回成功状态的数据**；如果失败，则被 **catch** 捕获到，整个异步函数将**返回一个失败的原因**

------

## **四、为什么要用 Axios？**

在实际开发中，网络请求是异步任务的典型场景。虽然浏览器原生提供了 `fetch` API，但它过于简化，比如不支持自动解析 JSON、全局错误处理等。**Axios** 是一个基于 Promise 的 HTTP 库，它让网络请求变得更加优雅。

### **Axios 的特点：**

1. 支持自动转换 JSON 数据。
2. 更加灵活的请求配置。
3. 支持全局错误处理和请求/响应拦截。
4. Axios 返回一个 promise对象

### **Axios 的使用：**

```javascript
const axios = require('axios');

// 发送 GET 请求
const fetchDataWithAxios = async () => {
  try {
    const response = await axios.get("https://api.example.com/data");
    console.log("获取的数据：", response.data);
  } catch (error) {
    console.error("网络请求失败：", error.message);
  }
};

fetchDataWithAxios();
```

### **全局错误处理与拦截器：**

```javascript
axios.interceptors.request.use(
  (config) => {
    console.log("请求发送：", config.url);
    return config;
  },
  (error) => Promise.reject(error)
);

axios.interceptors.response.use(
  (response) => response,
  (error) => {
    console.error("全局错误处理：", error.message);
    return Promise.reject(error);
  }
);
```

------

## **五、如何选择？Promise、Async/Await 和 Axios 的最佳实践**

在实际开发中，建议你灵活搭配使用它们：

1. **使用 Promise**：适用于简单的异步操作，比如加载一组图片。
2. **使用 Async/Await**：处理复杂的异步逻辑，保证代码易读性。
3. **使用 Axios**：替代原生 `fetch`，轻松管理网络请求。

比如下面的例子，我们用 Async/Await 搭配 Axios，完成一个串行任务：

```javascript
const fetchUserAndOrders = async () => {
  try {
    const user = await axios.get("https://api.example.com/user");
    console.log("用户信息：", user.data);

    const orders = await axios.get(
      `https://api.example.com/orders/${user.data.id}`
    );
    console.log("订单信息：", orders.data);
  } catch (error) {
    console.error("请求失败：", error.message);
  }
};

fetchUserAndOrders();
```

------

## **六、总结**

- **Promise** 解决了回调嵌套问题，通过 `.then()` 和 `.catch()` 提供链式调用和统一错误处理。
- **Async/Await** 让异步代码更易读，特别是任务间存在依赖时，配合 `try...catch`，可大大简化错误处理逻辑。
- **Axios** 是功能强大的网络请求库，特别适合在项目中处理复杂的 API 调用。

希望这篇内容能帮你重新梳理异步编程的知识！如果你有其他想讨论的编程问题，也欢迎随时交流！
