---
title: 漫谈 Vercel Serverless 函数
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2024-12-16 23:32:09
updated:
description:
cover:
---
> 我们需要明白什么是 **Serverless**。顾名思义，Serverless 并不是没有服务器，而是 **“不需要你管理服务器”**。就像你去超市买东西，不用自己去种菜、养鸡，直接挑选、付款就好。Vercel 的 Serverless 函数也是类似的，它**帮你自动管理基础设施**，你只需专注于编写处理逻辑，Vercel 会负责其余的部分。
>
> ------
>
> Vercel 的 Serverless 函数的**工作原理**是，在你每次发出 HTTP 请求时被 **动态启动**，它就像你雇了一个“临时工”来处理你的请求，任务完成后他就离开了，不会一直待着。接下来，让我们一起看看它是如何工作的吧！

------

## 一、Vercel Serverless 函数的实现原理

### 1. **文件触发机制**

在 Vercel 上，Serverless 函数的实现与文件结构密切相关。你只需要将你的代码放进一个叫 **`api` 的文件夹**中，Vercel 会自动将每个文件转化成一个可以处理 HTTP 请求的 **API**。

比如，你有一个项目结构如下：

```
/my-project
  /api
    hello.js
```

这里 `hello.js` 就是一个 Serverless 函数文件，当你访问 `/api/hello` 路径时，Vercel 就会触发这个函数来处理请求。

### 2. **按需执行**

Vercel 的 Serverless 函数是**按需执行**的。也就是说，它只有在用户发出请求时才会被调用。当请求到达时，Vercel 会**动态分配计算资源来处理请求**，任务完成后，这些资源就会释放。

就像你去餐厅点餐，厨师只有在你点了菜后才会开始做饭，而做完了就不会再等着你，自己先休息一下去了。如果你再次点餐，厨师才会重新为你准备。

------

## 二、Vercel Serverless 函数是如何工作的？

在了解了基本概念后，我们来看看更详细的工作流程。这里的每个步骤都非常重要，帮助你理解它是如何高效工作的。

### 1. **HTTP 请求触发函数**

当你访问一个 API 路径时，比如 `/api/hello`，Vercel 会首先触发对应的 **Serverless 函数**（比如 `hello.js`）。

### 2. **Vercel 启动计算实例**

Vercel 会根据请求的类型（GET、POST等），动态启动一个 **计算实例**（就像启动一个微小的服务器）来执行你定义的函数。

### 3. **执行函数并返回结果**

这个计算实例会运行你编写的代码，处理相关业务逻辑，并将结果返回给用户。这就好比你去餐厅点餐，厨师做完饭后将菜品端上来。

### 4. **实例销毁**

当任务完成后，计算实例会自动销毁。也就是说，Vercel 会根据请求数量自动扩展或收缩计算资源，节省不必要的开销。

------

## 三、如何使用 Vercel 的 Serverless 函数？

### 步骤 1：创建项目

首先，创建一个项目并进入到你的项目文件夹：

```bash
mkdir my-vercel-project
cd my-vercel-project
```

### 步骤 2：创建 `api` 文件夹

在项目根目录下创建一个 `api` 文件夹，Vercel 会自动识别该目录下的文件并将它们转换为 API。

```bash
mkdir api
```

### 步骤 3：编写 Serverless 函数

在 `api` 文件夹下创建一个 JavaScript 文件，例如 `hello.js`，并在里面编写处理请求的逻辑。代码非常简单：

```javascript
// api/hello.js
module.exports = (req, res) => {
  res.status(200).json({ message: 'Hello from Vercel Serverless!' });
};
```

### 步骤 4：部署到 Vercel

你只需要将项目推送到 GitHub 或 GitLab，然后连接到 Vercel。Vercel 会自动检测到 `api` 文件夹中的文件，并将它们部署为 Serverless 函数。只要你推送代码，Vercel 就会立即构建并部署。

```bash
# 将代码推送到 GitHub
git add .
git commit -m "Add hello function"
git push origin main
```

> 接着，Vercel 会为你自动提供一个域名，你可以通过访问这个域名来调用 API，例如：`https://your-project-name.vercel.app/api/hello`。

------

## 四、代码示例：如何处理不同请求

你可以在 Serverless 函数中处理不同类型的请求，比如 GET、POST 等。下面是一个更复杂的示例，展示了如何处理不同的请求方式：

```javascript
// api/hello.js
module.exports = (req, res) => {
  if (req.method === 'GET') {
    res.status(200).json({ message: 'Hello from Vercel (GET)' });
  } else if (req.method === 'POST') {
    const { name } = req.body;
    res.status(200).json({ message: `Hello, ${name}! (POST)` });
  } else {
    res.status(405).json({ error: 'Method Not Allowed' });
  }
};
```

- **GET 请求**：当你发送 GET 请求时，Vercel 返回一条简单的消息。
- **POST 请求**：当你发送 POST 请求时，Vercel 会接收请求体中的数据（例如用户名），并返回个性化的消息。
- **错误处理**：如果你发送的是其他类型的请求（比如 PUT、DELETE），Vercel 会返回一个错误消息。

------

## 五、Vercel Serverless 函数的优缺点

### 优点：

- **无需管理服务器**：你不需要配置或管理服务器，Vercel 会自动帮你处理。
- **按需付费**：你只为实际使用的计算资源付费，不用担心空闲时间浪费。
- **自动扩展**：当流量增加时，Vercel 会自动扩展计算资源，减少了运维的工作量。
- **快速部署**：你只需推送代码到 GitHub，Vercel 会自动处理构建、部署等工作。

### 缺点：

- **冷启动**：虽然函数的启动很快，但在低流量场景下，Vercel 可能会经历冷启动延迟。
- **有限的执行时间**：每个函数的执行时间有限，如果函数运行时间过长，可能会被 Vercel 强制终止。

------

## 小结

Vercel 的 **Serverless 函数** 为开发者提供了一个 **无服务器** 的解决方案，可以让我们专注于业务逻辑的实现，而无需担心基础设施的管理。通过简单的目录结构和文件触发机制，Vercel 使得部署和扩展变得异常简单。而且它的按需扩展机制和按请求计费的方式，也让你能够更加高效地使用计算资源。