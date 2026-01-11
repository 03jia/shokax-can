---
title: Vite 打包构建的产物
keywords: []
categories:
  - [前端,构建]
tags:
  - Vite
  - 打包
date: 2024-12-12 13:57:54
updated:
description: "说明 Vite 构建前后项目结构与生成的产物，并讨论开发与生产环境的资源路径问题。"
cover:
---
> 当我们谈到现代前端工具时，Vite 是一个不可忽视的名字。它以极快的开发速度和高效的生产构建而闻名。不知道朋友们有没有跟我有一样好奇，当 Vite 将你的代码打包时，它究竟会生成什么样的文件？又是如何智能地找到入口文件和资源文件的？或者是在开发环境中遇到了很多路径问题报错，然后不知道出什么问题了，找 bug 半天一天就过去了……不说了，想起来就痛苦😭，还是先来讲一下 **Vite 构建前后的项目结构有什么区别**，以及在**开发环境和生产环境部署的时候，如何找到这些文件路径**的，为什么开发环境这样写路径，他到了生产环境就也能找到这个路径下的资源呢？

## 一、Vite 构建前的项目结构

在开发环境中，你的项目可能会长这样：<br>
![Alt](https://s2.loli.net/2026/01/11/yzHRvAcFmpLwugJ.png)

```markdown
├── node_modules/        # 相关包文件夹
├── public/              # 静态资源文件，原封不动地复制到 dist
├── src/                 # 源代码文件夹
│   ├── assets/          # 样式、图片等资源
│   ├── components/      # 组件
│   ├── App.vue          # 根组件
│   ├── main.ts          # 应用入口文件
└── index.html            # 项目入口文件
├── vite.config.ts       # Vite 配置文件
└── package.json         # 项目配置
└── ......               # 其他配置
```

### 开发环境中的路径是如何工作的？

在开发环境中，Vite 使用的是原生的 ES 模块支持。你的文件路径（如 `/src/main.ts` 或 `/src/assets/logo.png`）会直接按需加载，不需要打包。Vite 内部有一个基于 **esbuild** 的高速开发服务器，处理这些模块的解析。

比如你在 `main.ts` 中导入了一张图片：

```ts
import logo from './assets/logo.png';
```

在开发环境下，你的主机域名加开发服务器的指定的端口号指向这个**项目文件夹的根目录**，Vite 会直接将 `logo` 解析为一个动态生成的 URL，比如：

```
http://localhost:3000/src/assets/logo.png
```

这就是为什么在开发环境中，你可以直接使用相对路径，而不需要担心编译、打包等问题。

## 二、Vite 构建后的产物

Vite 会将代码打包成适合部署的静态文件，这些文件最终生成在 `dist/` 目录下，可以直接部署到生产环境中。主要包括：

1.**HTML 文件**：

入口文件，如 `index.html`，用于引导加载其他资源。

示例：

 ```html
  <script type="module" src="/assets/entry.123abc.js"></script>
 ```

2.**CSS 文件**：

 - 样式被提取为独立的文件，例如 `style.456def.css`，便于浏览器缓存和优化加载。

3.**JavaScript 文件**：

  - **入口文件**：`entry.[hash].js` 是应用的主入口。
  - **分包模块**：通过代码分割生成的按需加载模块，例如 `chunk.[hash].js`。
  - **Vendor 文件**：常用的外部库（如 Vue 或 React）被单独打包为 `vendor.[hash].js`，提高缓存效率。

4.**静态资源文件**：

 每一个图片、字体文件等默认会被优化后存放在 `assets/` 目录中，文件名通常带有哈希值（例如 `logo.789ghi.png`）。

  **开发环境**

   - 静态资源直接通过原生 ESM 加载。

   - 路径相对于项目根目录或配置的 `root`。例如：

  ```plaintext
 /src/assets/images/logo.png
  ```

 **生产环境**

 静态资源被打包到 `dist/assets` 目录，并带有哈希值。例如：

  ```plaintext
 /assets/logo.456def.png
  ```

> 这里不是我忘了写`images`路径，而是打包之后图片会脱离去原来的文件层级，直接存放在`assets`下。

HTML 文件会更新引用路径以匹配 `base` 配置（默认为 `/` 或自定义路径）。

****

当我们运行 `npm run build`，Vite 会生成一个 `dist` 文件夹，这就是打包后的产物。它的结构通常如下：

![image-20241211145233177](https://s2.loli.net/2026/01/11/lwaqnQSjHBhNtF4.png)

```
├── dist/
│   ├── assets/            # 打包后的静态资源文件
│   │   ├── index.xxx.js
│   │   ├── index.xxx.css
│   │   ├── bg.xxx.jpg
│   ├── index.html         # 打包后的入口 HTML 文件
```

### 1. 静态资源的优化

- 所有的 JS 和 CSS 文件都会被合并、压缩，并生成带有哈希值的文件名（如 `index.abcd1234.js`、`index.abcd1234.css`），用来避免缓存问题。

  给各位看看压缩后的产物，我们不需要看懂，只需要知道他帮我们做了一些优化，比如将一些函数合并了，还有变成立即执行函数，以及对空行空格等进行了压缩之类的：

- 图片资源同样会被优化，比如压缩体积或者直接内嵌成 base64。

### 2. 入口 HTML 文件

构建后的 `index.html` 文件是整个应用的入口，它会自动注入打包后的 JS 和 CSS：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link rel="stylesheet" href="/assets/index.abcd1234.css">
</head>
<body>
  <div id="app"></div>
  <script type="module" src="/assets/vendor.abcd1234.js"></script>
</body>
</html>
```

### 3. 路径的处理

在生产环境下，Vite 会根据你的 `base` 配置，调整文件路径。默认情况下，所有资源路径都是相对根路径的（`/assets/...`）。如果你的项目部署在子路径下，比如 `https://example.com/my-app/`，需要在 `vite.config.ts` 中配置 `base`：

```ts
export default defineConfig({
  base: '/my-app/',
});
```

------

## 三、为什么开发路径能在生产环境正常工作？

在开发环境中，Vite 使用的是原生路径加载，`src` 中的资源可以直接通过相对路径引用。而在生产环境中，Vite 的打包工具会自动调整路径，把相对路径转换成生产环境可以解析的绝对路径（如 `assets` 文件夹）。

在生产模式下，Vite 通过以下方式分析依赖关系：

- **扫描 HTML 文件**：读取 `<script>` 和 `<link>` 标签。
- **递归解析**：构建依赖图，找到所有被引用的资源文件。
- **打包优化**：将这些文件打包为高效、可缓存的静态资源。

同时，Vite 提供了内置的路径别名功能。例如你可以在 `vite.config.ts` 中配置 `@` 指向 `src`：

```ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  resolve: {
    alias: {
      '@': '/src',
    },
  },
});
```

这样，你在开发中可以直接用 `@` 引用模块：

```ts
import Component from '@/components/MyComponent.vue';
```

而在生产构建时，Vite 会自动解析这些别名，让它们适配打包后的结构。

## 四、部署生产环境时的注意事项

### 1. 确保正确的资源路径

如果你的项目部署在非根路径下，一定要在 `base` 中配置正确的路径，否则资源文件可能加载失败。

### 2. 正确配置服务器

生产环境通常会使用 Nginx、Node.js 等作为静态文件服务器。确保你的 `dist` 文件夹中的 `index.html` 和 `assets` 文件夹都可以被正确访问。

Nginx 示例配置：

```nginx
server {
  listen 80;
  server_name your-domain.com;

  location / {
    root /path/to/your/dist;
    index index.html;
  }
}
```

------

## 五、总结

Vite 在开发环境下，项目文件根目录就是资源访问的根路径；在生产部署环境下，通过配置文件中的 `base` 配置项中指定的路径为资源访问根路径，默认为"/"，就是将你的那个**dist**文件夹作为根路径

Vite 的打包过程将你的开发代码转化成高性能的生产代码，包含优化后的 HTML、JS 和资源文件。在开发环境和生产环境之间，Vite 通过路径的智能解析和调整，帮你省去了很多繁琐的配置工作。但同时，我们也需要注意正确配置 `base` 和部署路径，以确保资源能在生产环境中正常加载。

希望大家看完这篇文章后，再遇到类似路径问题，能够冷静分析，快速找到解决方法！😊