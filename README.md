# Lean React
---
 
**目前本书正在撰写过程中，希望以 open source 的方式能得到更多读者的反馈**

## 本书内容

这本书我会由简单到复杂的带领大家进入 React 的世界， 其中 1 - 3 章节都是 React 的基础知识，需要提醒读者的是大多数的基础知识都可以通过 React 的官方文档学习，如果对英语敏感的读者也可以看翻译。 对比官方文档本书 1 - 3 章会循序渐进的带领大家学习 React 基础知识，其中会有些自己的见解和领悟希望能让读者更容易理解学习，每个章节都会有一个实例作业，所以读者可以同时结合官方文档和本书进行学习。

> 有 React 基础的读者可以跳过 1 - 3 章节 ， 后面的章节都是独立的，可以打乱顺序挑选阅读

文章的样例代码都在放在 https://github.com/leanklass/leanreact/ 的不同分支上，可以直接 checkout 分支按照 README 的指示运行。

### 第一章：React 入门

本章会带领大家重 0 到 1 入门 React，会涉及到 React 背景和应用范围的介绍。 然后会介绍 React 的基础知识，包括 JSX 语法和 React 组件，Flux 模式介绍等。 

- 1.1 [React 介绍](https://segmentfault.com/a/1190000005140569)
- 1.2 [JSX 语法](https://segmentfault.com/a/1190000005145610)
- 1.3 [React 组件](https://segmentfault.com/a/1190000005151182)
- 1.4 [React 组件生命周期和方法](https://segmentfault.com/a/1190000005161417)
- 1.5 [React 与 DOM](https://segmentfault.com/a/1190000005182270)
- 1.6 [Flux](https://segmentfault.com/a/1190000005348206)

### 第二章：React 工程化 

前面一章我们已经熟悉了 React 的基础，能够掌握通过 JSX 和 React 的思维来完成业务应用，但是真正的前端项目构建不仅仅是业务代码本身，我们需要搭建一整套完整的前端开发流程，也就是前端工程化。在本章中将会讲解前端工程化相关的知识，并通过 gulp，webpack 等工具搭建出一套完整的 React 前端开发环境。

- 2.1 [前端工程化概述](https://segmentfault.com/a/1190000005594760)
- 2.2 [Webpack](https://segmentfault.com/a/1190000005612506)
- 2.3 [Gulp](https://segmentfault.com/a/1190000005636680)
- 2.4 [webpack + gulp 构建完整前端工作流](https://segmentfault.com/a/1190000005657651)
- 2.5 [Webpack 进阶](https://segmentfault.com/a/1190000005666159)

### 第三章：React 与 Redux

Redux 是目前 flux 模式最流行的实现，本章节会带领大家了解 Redux 的设计概念， 阅读 Redux 的源码，以及通过实例应用讲解 Redux + React 的开发模式。

- 3.1 [redux 介绍](https://segmentfault.com/a/1190000005696767)
- 3.2 [react-redux todoApp](https://segmentfault.com/a/1190000005758244)
- 3.3 [理解 redux 中间件](https://segmentfault.com/a/1190000005766289)
- 3.4 [掌控 redux 异步](https://segmentfault.com/a/1190000005773725)
- 3.5 [compose redux sagas](https://segmentfault.com/a/1190000005776381)


### 第四章：React 进阶

我们已经能基于 React 实现基本的交互逻辑，但是在使用 React 的过程中还是可能会有些不确定的地方或者一些特殊的功能不知道怎么实现，可能会问 React 中有没有一些 Best practices 或者 Good Pattern 可以参考的，本章会在各个维度介绍之前没有讲过的 React 特性。

- 4.1 [react 代码规范](https://segmentfault.com/a/1190000005825618)
- 4.2 [react patterns](https://segmentfault.com/a/1190000005838634)
- 4.3 react magics and tricks
- 4.4 react 动画
- 4.5 不可变数据与 React  
- 4.6 性能管理

### 第五章：React 实战业务开发 

真实业务开发中会遇到很多很多的问题，本章会把大多数在真实业务开发中遇到的场景进行讲解，涉及到如具体组件的开发，表单处理，后台交互等具体开发场景问题。

### 第六章：React 与 服务端渲染

React 除了可以在浏览器端渲染以外， 还可以在服务器端渲染 HTML， 本章节会实现一个 基于 express + React 模板渲染器，通过这个渲染器渲染第一章的 HTML。

### 第七章：React 与 数据可视化

数据可视化的需求日益增加，React 同样可以胜任数据可视化的工作，本章节会带领大家通过 React 实现一些基本的图表，讲解 React 和 D3.js 如何协作。

### 第八章：React 内部实现

当深入的学习和使用过 React 后， 一定会对 React 的内部运作机制好奇，本章节会部分介绍 React 内部的一些核心工作机制， 包括 Virtual DOM 算法， 生命周期内部运作方式。 

### 第九章：React 自定义 Renderer

React 独特的地方在于， virtual dom 这种组件的组合模式可以应用于很多地方， 除了 ReactDOM 渲染器实现外，我们可以实现一个自己的渲染器， 比如 D3 渲染器， PIXI.JS 渲染器， Three.js 渲染器。

### 扩展：React 资源
### 扩展*: 各种 React 组件实现
### 扩展*：各种应用源码分析


  [1]: /img/bVvIsW
