---
title: 【Vite】解锁前端开发速度的秘密武器
keywords: []
categories:
  - [前端,构建]
tags:
- Vite
- 构建
date: 2024-12-11 18:20:04
updated:
description: "剖析 Vite 的设计理念与性能优势，说明为何它能显著提升前端开发体验和构建速度。"
cover: "https://s2.loli.net/2026/01/12/vZSexKjXBYcN9LV.png"
---

> 在前端开发的江湖中，有人偏爱 **Webpack** 的强大与稳定，有人钟情于 **Rollup** 的轻量与高效。而 **Vite**，这个后来居上的工具，却以“极致的快”和“极简的易”赢得了开发者的芳心。众所周知万事都有缘由，接下来我们就来深度剖析 Vite，看看它凭什么成为现代前端开发的宠儿。

## 一、什么是 Vite？

Vite，意为“快”（法语中读作 `/vit/`）。它由 Vue.js 的作者**尤雨溪团队**开发，因此也是因为这个先天原因，让他可以更好的配合 Vue 来使用。旨在解决传统构建工具慢、配置复杂的问题。Vite 的两大核心特点：

- **极速冷启动**：得益于原生 ESM（浏览器支持的 ES 模块）。
- **按需热更新**：基于模块化的 HMR（热模块替换），只更新需要的部分，速度飞快。

> 简单地说，Vite 是一种“快如闪电”的开发体验。

我们来看看官网中的效果图，他的功能之一是把所有需要编译的文件，不用分多个对应的编译工具，而是直接通过一个 Vite 就能变成浏览器能识别的原生 HTML、CSS、JavaScript 文件：

![img](assets/43240a707d3b411e95a57b82780aead2.gif)编辑

> **【另外，Vite还提供了模板解析功能】**
>
> 
>
> - **开发阶段**：Vite 等构建工具提供了，模块解析功能允许省略文件扩展名（如 .ts, .vue），自动处理模块导入路径
>
> ```TypeScript
>  import { useUserStore } from '@/stores/user'  // 不需要 .ts
> ```
>
> 
>
> - **构建阶段**：构建工具会自动处理并添加正确的文件扩展名，将代码编译成浏览器可以直接运行的静态资源，生成最终的生产环境文件

## 二、为什么开发需要“快”？

想象你在使用传统工具开发时，每次保存代码都要等待几秒钟。等着等着，你打开了手机刷了条短视频，结果回来发现忘了刚刚改了啥。开发效率和注意力全被拖垮了。

而 Vite 的即时响应让人直呼过瘾：修改代码，页面瞬间更新！就像和浏览器的对话一样顺滑。它通过 **原生 ESM** 和 **按需编译**，省去了传统工具动辄几分钟的**冷启动**等待。

哈哈哈，我这说的跟营销员打广告似的，但是体验过传统工具比如 Webpack 启动项目的朋友们就知道我没有夸大其词了。

## 三、Vite 的魔法：如何解析 `.vue` 文件？

对于 Vue 开发者来说，`.vue` 文件是家常便饭，但浏览器天生只认识 HTML、CSS 和 JavaScript，那 `.vue` 文件怎么办？

当浏览器请求 `.vue` 文件时，Vite 会拦截这个请求，通过内置的 **@vitejs/plugin-vue** 插件将 `.vue` 文件动态转换成浏览器可理解的 JavaScript 模块：

- 模板部分转为渲染函数。
- 脚本部分直接生成 JavaScript。
- 样式部分注入页面中的 `<style>` 标签。

这一切都在后台“悄无声息”地完成，而你只需要关注代码本身。无论是 `App.vue` 还是组件库的其他文件，都能秒速加载。

## 四、静态资源和别名配置：写代码就像在开盲盒

Vite 还为静态资源提供了优雅的解决方案。无论是图片、字体，还是视频资源，都能在项目中轻松引入。

### 1. 动态加载资源

- 小于 4kb 的资源会被内联到代码中，大文件则会自动生成带哈希值的路径，方便缓存管理。

```TypeScript
const logo = await import('@/assets/logo.png');
```

### 2. 别名支持

想象一下，不用再写一大串 `../../../assets/logo.png`，通过别名直接写成 `@/assets/logo.png`，路径整洁心情都变好了！

### **3. 配置方式**

```TypeScript
resolve: {
  alias: {
    '@': path.resolve(__dirname, 'src'),
  },
}
```

## 五、插件：你的功能扩展宝库

Vite 的插件机制就像给开发加了外挂。你可以用官方插件支持 Vue，也可以通过社区插件生成 SVG 图标，甚至自己开发插件。以下是一些必备插件：

- **@vitejs/plugin-vue：Vue 支持** Vue 开发必装插件，解析 `.vue` 文件，支持 `<script setup>`、Vue 3 的各种语法特性。
- **vite-plugin-compression：生产环境压缩** 自动压缩资源，提升页面加载速度。
- **vite-plugin-pwa：PWA 支持** 一键为项目添加渐进式 Web 应用能力。

> 开发中，插件不仅提升效率，还能让你更加专注于业务逻辑。

我们以**@vitejs/plugin-vue插件**为例，介绍一下如何在vite项目中使用插件

- 安装：

```bash
npm install @vitejs/plugin-vue --save-dev
```

- 使用：

```TypeScript
import vue from '@vitejs/plugin-vue';
export default defineConfig({
  plugins: [vue()],
});
```

## 六、TypeScript：更安全的开发方式

Vite 天然支持 TypeScript，让代码更规范、更安全。以下是 Vite 中使用 TS 的几个技巧：

### 1. 定义 Props 和 Emit

```TypeScript
defineProps<{ title: string }>();
const emit = defineEmits<(event: 'submit') => void>();
```

### 2. 全局类型声明：

```TypeScript
declare module '*.vue' {
  import { DefineComponent } from 'vue';
  const component: DefineComponent<{}, {}, any>;
  export default component;
}
```

### 3. 类型检查插件

使用 `vite-plugin-checker` 集成 TS 类型检查：

```TypeScript
import checker from 'vite-plugin-checker';
export default defineConfig({
  plugins: [checker({ typescript: true })],
});
```

## 七、Vite 的预打包机制详解

在使用 Vite 开发项目时，预打包（Pre-Bundling）是一个非常重要的优化机制。它的主要目的是加快依赖的加载速度和提高开发体验。以下是 Vite 预打包的详细工作流程：

### 1. 什么是预打包？

预打包是指 Vite 在项目启动时，使用 `esbuild` 对第三方依赖（如 npm 包）进行一次性打包。这些打包的文件会存储在本地缓存中，以便后续直接加载，而不需要每次解析。

### 2. 为什么需要预打包？

- **减少网络请求**： 浏览器在解析 ESM 模块时，需要发起大量的网络请求，尤其是遇到模块嵌套时。通过预打包，Vite 将多个模块合并为一个，从而减少请求数量。
- **提高解析效率**： 通过将第三方依赖预先转换为浏览器可直接使用的 ESM 格式，减少开发服务器在运行时的开销。

### 3. 预打包的具体流程

- **收集依赖**： Vite 在启动时，会扫描项目中的依赖关系，找出第三方模块（如 `node_modules` 中的包）。
- **调用** `esbuild`： 使用 `esbuild` 将这些模块快速打包为 ESM 格式的文件。
- **存储缓存**： 打包后的文件会存储在项目的 `node_modules/.vite` 目录下。

### 4. 如何手动控制预打包？

Vite 提供了灵活的配置项，允许开发者手动调整预打包行为：

- **强制重新预打包**： 使用命令 `npx vite --force`，可以清除缓存并强制重新打包所有依赖。

- **优化选项**： 在 `vite.config.ts` 中，使用 `optimizeDeps` 选项调整预打包行为：

  ```TypeScript
  export default defineConfig({
    optimizeDeps: {
      include: ['lodash', 'axios'], // 手动指定需要预打包的依赖
      exclude: ['some-large-package'], // 排除不需要预打包的依赖
    },
  });
  ```


> 【**预打包的注意事项**】
>
> - **开发体验**： 如果项目中依赖数量较多，初次启动时间可能会稍长，但之后的开发过程中会极大提升速度。
> - **依赖变更**： 如果更新了依赖版本，可能需要清理 `.vite` 缓存目录，并重新预打包。
>
> ------
>
> 通过预打包机制，Vite 不仅加快了开发服务器的响应速度，还让模块的加载更加高效，为开发者带来了极致的流畅体验。

## 八、Vite 的常用配置文件详解

在 Vite 项目中，`vite.config.ts` 是核心配置文件，我们应该要能熟练搭配其配置项开发项目，以更加高效的开发项目以及优化项目，我们来附上常用配置项及其功能的注释：

```TypeScript
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path from 'path'

export default defineConfig({
  // 插件配置，用于扩展 Vite 的功能
  plugins: [vue()], // 加载 Vue.js 插件，支持 .vue 文件

  // 项目根目录，默认为 process.cwd()【用于返回当前工作目录的路径。】
  root: './src', // 指定项目的根目录，默认为当前文件所在目录

  // 用于指定应用的公共基础路径（Base URL）。
  base: '/',

  // 开发服务器配置
  server: {
    // 监听地址
    host: '0.0.0.0', //设置为 0.0.0.0 可使局域网访问
    // 端口号，默认 5173
    port: 3000, 
    // 是否在浏览器中自动打开
    open: true, 
    // 是否启用 HTTPS
    https: false,
    // 配置代理规则
    proxy: {
      '/api': {
        // 代理目标地址
        target: 'https://api.example.com',
        // 是否更改请求源，修改为目标服务器的域名或 IP 地址
        changeOrigin: true,
        // 重写路径
        rewrite: (path) => path.replace(/^\/api/, ''), 
      },
    },
  },

  // 构建配置
  build: {
    // 输出目录，默认是 dist
    outDir: 'dist',
    // 静态资源目录，存放在指明文件夹名中
    assetsDir: 'assets',
    // 是否生成 sourcemap 文件
    sourcemap: false,
    // 指定压缩工具，支持 'esbuild' 和 'terser'
    minify: 'esbuild', 
    // 自定义 Rollup 构建配置
    rollupOptions: {
      // 自定义入口文件
      input: './src/main.js',
      output: {
        manualChunks: {
          // 将 Vue 分离到 vendor 文件
          vendor: ['vue'],
        },
      },
    }, 
  },

  // 别名配置
  resolve: {
    alias: {
      // 配置 @ 为 /src 路径
      '@': path.resolve(__dirname, 'src'),
      // 配置组件路径别名
      components: '/src/components', 
    },
    // 导入时省略的文件扩展名
    extensions: ['.js', '.ts', '.vue', '.json'],
  },

  // 设置 CSS 预处理器选项和模块化配置，方便全局样式管理。
  css: {
    preprocessorOptions: {
      // 自动引入全局 SCSS 变量
      scss: {
        additionalData: `@import "@/styles/variables.scss";`, 
      },
    },
    modules: {
      // 默认所有 CSS 模块化
      scopeBehaviour: 'local', 
      // 自定义类名生成规则
      generateScopedName: '[name]__[local]__[hash:base64:5]', 
    },
  },

  // 环境变量配置
  envDir: './env', //指定存放环境变量文件的目录，默认是根目录
  // 指定环境变量的前缀，默认为 VITE_
  envPrefix: ['VITE_'],
  
  // 预打包配置，加快开发服务器启动速度。
  optimizeDeps: {
    include: ['lodash'], // 手动指定需要预打包的依赖
    exclude: ['some-large-lib'], // 排除不需要预打包的依赖
  },
});
```

> **为了让大家更清晰的区分一些路径配置项，我们来细说一下几个配置项的区别：**
>
> - **`root`**：用于指定项目的根目录。Vite 会在这个目录下查找项目的源文件（如 `index.html`、`src` 目录等）。默认值是 `process.cwd()`，即当前工作目录。【指定**项目的根目录**，影响 **Vite** 查找项目文件的路径。】
>
> - **`base`**：它通常用于部署应用时，指定应用的根路径。如果你的应用部署在服务器的某个子路径下（例如 `https://example.com/my-app/`），你可以将 base 设置为 `/my-app/`。Vite 会将所有的静态资源路径（如 CSS、JS、图片等）都加上 `/my-app/` 前缀。例如，原本的 `/assets/logo.png` 会变成 `/my-app/assets/logo.png`。【指定**应用的公共基础路径**，影响静态资源的路径前缀。】
> - **`resolve.alias`**：用于为模块路径设置别名，使得在导入模块时可以使用更简洁的路径。例如，你可以将 `@` 设置为 `src` 目录的别名，这样在导入 `src` 目录下的文件时，可以直接使用 `@` 开头，而不需要使用相对路径。通过设置别名，可以避免使用冗长的相对路径，例如从 `../../../src/components/Button` 简化为 `@/components/Button`。

## 九、总结：为什么选择 Vite？

如果 Webpack 是“**前端世界的工厂**”，Rollup 是“**模块化世界的匠人**”，那么 Vite 就像“**灵活高效的服务员**”。它专注于开发阶段的速度与体验，轻便、友好、快速响应。

开发用 Vite，你会感受到一种从未有过的流畅感。代码改一点，页面瞬间更新；项目大一点，启动依然飞快，让我们从繁重的工具中解放出来。