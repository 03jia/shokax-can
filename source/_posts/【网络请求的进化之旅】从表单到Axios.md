---
title: 【网络请求的进化之旅】从表单到Axios
keywords: []
categories:
  - 不知怎么分类
tags:
  - HTTP
  - Axios
date: 2024-12-13 23:55:25
updated:
description: "回顾网络请求从表单提交、XHR 到 Fetch 与 Axios 的演进，说明各阶段的痛点与改进。"
cover: "https://s2.loli.net/2026/01/11/9gh7NmOcj26HYlW.jpg"
---
> 不知道有没有朋友跟我一样，现在直接用 **Axios** 封装好的网络请求方式觉得很香，轻松发送请求、处理响应，仿佛一切都很顺畅。但有时也会想，为什么我们要用这些工具？为什么一开始的网络请求方式不行呢？
>
> ------
>
> 不着急，今天我们就来聊聊网络请求的发展历程。从最初简单粗暴的表单提交，到现代化的 Axios，每一步技术的演进都在解决痛点、追求更优雅的开发体验。这不仅是技术进化的故事，也是我们提升开发效率的必修课。

## 一、表单提交，简单却原始

还记得 Web 的早期时代吗？那个时候的网络请求完全依赖 HTML 表单提交，比如：

```html
<form action="/submit" method="POST">
  <input type="text" name="username" />
  <button type="submit">Submit</button>
</form>
```

提交表单后，整个页面刷新，重新加载返回的内容。虽然这种方式简单直接，但用户体验简直一言难尽：

1. 每次请求都要刷新页面，白屏好几秒。
2. 无法动态处理复杂交互，比如用户填写错误后只能整页重新提交。
3. 前端完全依赖后端渲染，没有灵活性。

【比如】用户填写登录表单，点击提交。浏览器发送 `POST` 请求，提交用户名和密码。后端验证用户数据，跳转到另一个页面（例如用户主页）。新页面替换当前页面，用户看到登录后的界面。

整个页面刷新，用户体验不好。如果仅需要局部更新内容，也必须重新加载完整页面。前端无法独立操作数据或实现复杂交互。这就是**早期表单的局限性**。

------

## 二、XHR 和 Ajax，异步的曙光

为了提升用户体验，**XMLHttpRequest（XHR）**诞生了！XHR 是一个可以让浏览器发送**异步请求的 API**，它实现了**不刷新页面**即可与服务器通信。

在 XHR 的基础上，开发者提出了 **Ajax**（Asynchronous JavaScript and XML）这一概念。Ajax 是一种技术组合，使用 JavaScript 发起异步请求并动态更新页面内容，而无需重新加载整个页面。需要注意的是，虽然 Ajax 名称中包含“XML”，但它并不局限于使用 XML 数据，也可以处理 JSON 等格式。XML 只是当时流行的传输格式之一，但如今 JSON 更常用，XML 已逐渐淡出主流应用。

> 【注意】**Fetch（后面就会讲到）** 是一种 **现代的替代方法**，用于发送网络请求并获取响应。它是实现 **AJAX** 的一种更简洁、更现代的方式。
>
> 相比于 **XHR**，**Fetch** 使用了 **Promise**，使得异步编程更容易理解和操作，支持更简洁的语法和链式调用。

一个简单的 XHR 示例：

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', '/api/data', true);
xhr.onload = function() {
  if (xhr.status === 200) {
    console.log(JSON.parse(xhr.responseText));
  }
};
xhr.send();
```

这时，我们终于可以实现页面局部更新了，但问题也随之而来：

1. XHR 的回调地狱让代码可读性极差。
2. 缺乏统一的规范，错误处理、请求超时等功能需要手动实现。

> 我们可不能将 XHR 和 AJAX 混为一谈，我们来介绍一下他们俩的区别与联系：
>
> 1. **XHR（XMLHttpRequest）**：是一个 **JavaScript 对象**，用来在客户端与服务器之间进行 HTTP 请求。XHR 的主要功能是发起和接收 HTTP 请求，它可以在后台进行数据交换，而无需刷新页面。XHR 可以支持多种数据格式，不仅仅是 XML（尽管最初是为 XML 设计的）。
> 2. **AJAX（Asynchronous JavaScript and XML）**：是一种**技术和编程模式**，利用 **XHR** 或者 **Fetch** 来实现异步请求。AJAX 是一种技术集成，指的是通过 JavaScript 在后台与服务器交换数据并更新网页的方式。它的目标是**使网页在不重新加载整个页面的情况下动态更新内容**。

------

## 三、Fetch 的异军突起

随着前端技术的发展，**Fetch API** 横空出世，解决了 XHR 的很多问题。最大的亮点是它基于 **Promise**，代码变得更加优雅：

```javascript
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

### Fetch 的优点

- 使用 **Promise**，彻底告别**回调地狱**。
- 更现代的设计，支持更灵活的功能扩展。

但是，Fetch 也有它的不足：

1. 错误处理较繁琐，比如即使服务器返回 404，Fetch 也不会抛出异常。

2. 不支持请求超时，需要手动实现。

3. 缺乏内置的拦截器功能，无法像 XHR 那样灵活拦截请求。

   > `Fetch` 无法直接拦截请求发起前的头部或请求内容（不像 `XMLHttpRequest` 那样提供直接的方法），但是你可以在服务端代理或通过 **Service Workers** 来拦截和修改请求。

------

## 四、Axios 的闪亮登场

于是，前端开发者们想：有没有一种工具能结合 Fetch 的优雅，同时解决它的不足呢？答案就是 **Axios**！

Axios 是一个基于 Promise 的 HTTP 库，它提供了一套易用、强大的 API，让网络请求变得更简单。

### 为什么选择 Axios？

1. **默认配置**：可以设置全局 baseURL、超时时间等，避免重复配置。
2. **支持拦截器**：可以在请求和响应时统一处理，比如添加 Token 或错误日志。
3. 返回的数据会**自动被解析成 JavaScript 对象**（如果返回的是 JSON），而**Fetch** 返回一个 `Response` 对象，你需要手动调用 `.json()` 或 `.text()` 等方法来解析响应体。
4. **更友好的错误处理**：会自动抛出**非 2xx 状态**的异常。

一个简单的 Axios 示例：

```javascript
axios.get('/api/data')
  .then(response => console.log(response.data))
  .catch(error => console.error('Error:', error));
```

### Axios 的高级功能

- 创建实例管理多个 API 配置：

  ```javascript
  const apiClient = axios.create({
    baseURL: '/api',
    timeout: 5000,
  });
  
  apiClient.get('/data').then(response => console.log(response.data));
  ```

- 使用拦截器统一处理请求：

  ```javascript
  axios.interceptors.request.use(config => {
    config.headers.Authorization = 'Bearer token';
    return config;
  });
  ```


> 【因为这篇主要是介绍网络请求的发展历程嘛，所以关于 **Axios** 的知识内容就不做过多的讲解了。如果有许多朋友想多了解一下 **Axios**，可以在评论区说一声哦，我会单独写一篇来介绍一下 **Axios** 以及在开发中常用的方法，以及对它的封装】

## 五、写在最后

从传统表单到现代化的 Axios，我们见证了网络请求的演化历程。这不仅是技术的进步，也是前端开发者为提升用户体验所做的努力。了解这些基础概念，有助于我们更好地理解和运用现代工具。

所以，下次你用 Axios 时，不妨想想这段进化史，是不是更能体会它的优雅之处了呢？