---
title: 【Vue 3 + Vite】快速搭建现代前端项目
keywords: []
categories:
  - []
tags:
  - null
  - null
date: 2024-12-30 14:00:00
updated:
description: 
cover:
---
> 前端开发的世界瞬息万变，每隔一段时间就会冒出新的框架和工具。但是经典永不过时，基于对经典的理解，才能更好的去上手其他框架，比如基于 **Vue** 的 **Nuxt.js** 等让我们一起来探索 **Vue 3** 和 **Vite** 这对黄金搭档，手把手搭建一个现代前端项目

## 一、为什么选择 Vue 3？

Vue 是一个广受欢迎的渐进式 JavaScript 框架，简单、灵活、功能强大。Vue 3 相较于 Vue 2 迎来了全面升级，以下是它的一些亮点：

1. **Composition API：**如果说 Vue 2 是温暖的家，Vue 3 的 Composition API 就是那间焕然一新的工作室。它带来了更好的逻辑复用和代码组织方式，特别适合大型项目。
2. **性能优化：**Vue 3 的响应式系统基于 Proxy 实现，性能更快，占用内存更小。
3. **更好的 TypeScript 支持：**TypeScript 和 Vue 3 是绝配，帮助你写出更稳定、更易维护的代码。

> 这里对 Vite 不做过多讲解，这篇主要简介怎么快速启动 Vue3+Vite 的项目，感兴趣的朋友可以去我的主页中看一下关于其详细的文章

------

## 二、Vite 是什么？为什么选它？

在传统的前端开发中，我们常用 Webpack 构建工具，但它的启动速度和构建性能往往会拖慢开发进程。Vite 则是个“小火箭”：

1. **极速冷启动：**使用原生 ES 模块，**秒级启动开发服务器**，告别漫长的“npm run dev”。
2. **热模块替换（HMR）：**修改代码时，页面自动刷新，效果立竿见影，开发体验丝滑流畅。
3. **简单易用：**配置文件直观清晰，开箱即用，开发者友好。

------

## 三、👩‍💻 实战：用 Vue 3 + Vite 快速搭建项目

### 1. 安装项目

用以下命令创建你的 Vue 项目，我们详细讲解一下每个配置项：

```
## 1.创建命令
npm create vue@latest

## 2.具体配置
## 配置项目名称
√ Project name: vue3_test
## 是否添加TypeScript支持
√ Add TypeScript?  Yes
## 是否添加JSX支持
√ Add JSX Support?  No
## 是否添加路由环境
√ Add Vue Router for Single Page Application development?  No
## 是否添加pinia环境
√ Add Pinia for state management?  No
## 是否添加单元测试
√ Add Vitest for Unit Testing?  No
## 是否添加端到端测试方案
√ Add an End-to-End Testing Solution? » No
## 是否添加ESLint语法检查
√ Add ESLint for code quality?  Yes
## 是否添加Prettiert代码格式化
√ Add Prettier for code formatting?  No
```

启动后访问 `http://localhost:5173/`，迎接你的就是 Vue 3 的默认界面了，是不是超快？

### 2. 了解 Vite 配置文件

在项目中找到 `vite.config.ts`。这是 Vite 的大脑，控制你的开发环境和构建流程。大致结构如下：

```
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
});
```

如果需要自定义环境变量，在根目录创建 `.env` 文件即可，例如： `VITE_API_URL=https://api.example.com`

------

## 四、项目启动之后的文件介绍

在使用 Vue 3 和 Vite 创建项目后，你会发现项目的**核心文件**包括 `index.html`、`main.ts` 和 `App.vue`。对于这些核心文件的理解很为重要，所以我写好了详细的注释，帮助朋友们更好地理解它们的作用和结构。

### `1. index.html`

这是项目的入口文件，也是传统意义上的 HTML 模板。Vite 使用 `index.html` 作为开发和构建的起点。

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- 网页的字符编码，确保页面正确显示多语言文本 -->
    <meta charset="UTF-8" />
    <!-- 设置视口，确保页面在移动设备上的良好展示效果 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <!-- 页面标题，会显示在浏览器标签页 -->
    <title>Vite + Vue</title>
  </head>
  <body>
    <!-- #app 是 Vue 应用挂载的根元素 -->
    <div id="app"></div>
    <!-- 加载 Vite 提供的 JavaScript 入口文件 -->
    <!-- Vite 会自动替换 `type="module"` 的引用为优化后的构建版本 -->
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

> 【**关键点**】
>
> - `#app` 是 Vue 应用的挂载点，Vue 会通过 `main.ts` 将应用绑定到这个 `div` 元素。
> - `script` 引入了 `/src/main.ts`，这个文件是 Vue 应用的启动入口。

### `2. main.ts`

因为刚才创建项目的时候选择了 TypeScript，所以是 mian.ts 文件。这是 Vue 应用的主入口文件，用于创建应用实例并挂载到 DOM 上。

```
// 引入 createApp 方法，用于创建 Vue 应用实例
import { createApp } from 'vue';
// 引入根组件
import App from './App.vue';
// 引入样式文件（如果有全局样式，可以在这里引入）
import './global.css';

// 创建 Vue 应用实例，并将其挂载到 #app 元素上
createApp(App).mount('#app');
```

> 【**关键点**】
>
> 1. **`createApp(App)：`**创建了一个 Vue 应用实例，`App` 是应用的根组件。
> 2. **`.mount('#app')：`**指定 Vue 应用实例挂载的位置（与 `index.html` 中的 `#app` 对应）。
> 3. **样式文件：**通过 `import './global.css'` 引入全局样式，你也可以使用预处理器文件如 SCSS 或 Less。

### `3. App.vue`

这是应用的根组件，是所有 Vue 组件的起点。它包含 `template`、`script` 和 `style` 三部分。

```
<template>
  <!-- 根模板，默认提供一个简单的内容 -->
  <div id="app">
    <h1>Vite + Vue</h1>
    <p>Welcome to your Vue 3 + Vite project!</p>
  </div>
</template>

<script setup>
// 这里是 `<script setup>` 语法，它是 Vue 3 的新特性，用于编写更简洁的组件逻辑
// 不需要 `export default`，也不用显式定义 `setup` 函数
</script>

/* 局部样式，作用于本组件 */
<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

> 【**关键点**】
>
> 1. `template` 部分
>
>    - 定义组件的 HTML 结构和内容。
>    - `#app` 是根元素，组件的其他部分会插入到这里。
>
> 2. `script setup` 部分
>
>    - 使用 Vue 3 的 `<script setup>` 语法，直接编写组件逻辑。
>    - 如果需要使用响应式数据，可以引入 Vue 的 `ref` 或 `reactive` 方法。
>
> 3. `style` 部分
>
>    - 默认情况下，CSS 样式只作用于本组件（通过生成作用域 CSS 实现）。
>
>    - 如果想局部生效，可以添加 
>
>      ```
>      style
>      ```
>
>       标签的 
>
>      ```
>      scoped
>      ```
>
>       属性： 	
>
>      ```
>      <style scoped>
>      ```
>
>      

------

> 【**小结**】
>
> 1. `index.html` 是静态模板，`#app` 是 Vue 的挂载点，`<script>` 引入项目主入口。
> 2. `main.ts` 用于创建和启动 Vue 应用实例，并将其绑定到 `#app`。
> 3. `App.vue` 是根组件，作为整个应用的起点，可以在里面实现更多复杂的逻辑和布局。

------

## 五、开发初体验：父子组件通信

下面我们来实现一个基础的父子组件通信示例，分别通过 **`props`** 和 **`emit`** 进行数据传递。

### 1. 父组件 `App.vue`

```
<template>
  <div>
    <!-- 向子组件传递数据 message -->
    <Home :message="parentMessage" @reply="handleReply" />
  </div>
</template>

<script setup>
// 父组件的数据
const parentMessage = "Hello from Parent!";

// 父组件的方法，用于接收子组件的事件
const handleReply = (childMessage) => {
  console.log("Received from child:", childMessage);
};
</script>

<style scoped>
/* 父组件的样式，仅影响本组件 */
div {
  font-family: Arial, sans-serif;
  margin: 20px;
}
</style>
```

> **【注释说明】**
>
> 1. **`<Home :message="parentMessage" />`**
>    - 使用 `props` 将数据从父组件传递给子组件。
>    - 这里的 `message` 是子组件接收的 `prop` 名，`parentMessage` 是父组件的变量。
> 2. **`@reply="handleReply"`**
>    - 通过事件监听 `reply`，父组件会调用 `handleReply` 方法，处理子组件传来的数据。
> 3. **数据和方法**
>    - `parentMessage` 是父组件的变量，用于传递给子组件。
>    - `handleReply` 是父组件的方法，用于接收子组件通过事件传递的数据。

### 2. 子组件 `Home.vue`

```
<template>
  <div>
    <!-- 显示父组件传递过来的数据 -->
    <p>Parent says: {{ message }}</p>
    <!-- 点击按钮触发事件，向父组件发送数据 -->
    <button @click="sendReply">Reply to Parent</button>
  </div>
</template>

<script setup>
// 子组件接收来自父组件的 props
defineProps({
  message: String, // 定义 message 是一个字符串类型的 prop
});

// 子组件向父组件发送事件
const sendReply = () => {
  // 使用 $emit 将数据发送给父组件
  emit("reply", "Hello from Child!");
};

// 声明 emit 方法，告诉 Vue 子组件可能会触发哪些事件
defineEmits(["reply"]);
</script>

<style scoped>
/* 子组件样式，仅作用于本组件 */
div {
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px;
  text-align: center;
}
button {
  background-color: #42b983;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
button:hover {
  background-color: #369c7d;
}
</style>
```

> **【注释说明】**
>
> 1. **`defineProps`**
>    - 定义子组件需要从父组件接收的 `props`，这里的 `message` 类型是 `String`。
>    - 父组件传递的 `message` 会被自动注入到子组件。
> 2. **`defineEmits`**
>    - 定义子组件可以触发的事件，方便与父组件通信。
>    - `reply` 是事件名称，`emit("reply", "Hello from Child!")` 会触发这个事件并传递数据。
> 3. **`sendReply` 方法**
>    - 子组件通过点击按钮调用 `sendReply` 方法，触发 `reply` 事件，将数据发送给父组件。

### 3. 运行效果

1. **父组件显示子组件传来的数据**
   - 父组件的控制台会输出：`"Received from child: Hello from Child!"`。
2. **页面内容**
   - 父组件传递的信息会显示在子组件中：`Parent says: Hello from Parent!`。
   - 点击按钮，子组件通过事件将数据传递给父组件。

### 4. 小结

- **`props`**：用于父组件向子组件传递数据。
- **`emit`**：用于子组件向父组件发送事件和数据。

> 这种通信方式是 Vue 中最基础、最常用的父子组件交互模式。如果有更复杂的场景，比如跨层级组件通信，可以考虑使用 Vuex 或 Pinia等其他方式，我打算后续专门写一篇文章来讲解**多种组件通信**，因为实在是太多种通信方式了，一会儿父子传一会儿爷孙传一会儿兄弟传，刚开始学习的时候都要给我绕晕了，不知道有没有朋友跟我一样的困扰，所以我打算整理总结一篇出来。