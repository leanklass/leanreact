# React 工程

前面一章我们已经熟悉了 React 的基础，能够掌握通过 JSX 和 React 的思维来完成业务应用，但是真正的前端项目构建不仅仅是业务代码本身，我们需要搭建一整套完整的前端开发流程，也就是前端工程化。在本章中将会讲解前端工程化相关的知识，并通过 gulp，webpack 等工具搭建出一套完整的 React 前端开发环境。

2.1 前端工程化概述
2.2 Webpack 
2.3 Gulp 
2.4 构建 React 工程
2.5 深入 Webpack

## 2.2 webpack
- webpack 介绍
    - webpack 是什么
    - 为什么引入新的打包工具
    - webpack 核心思想
- webpack 安装
- webpack 使用
    - 命令行调用
    - 配置文件
- webpack 配置参数
    - entry 和 output
    - 单一入口
    - 多个入口
    - 多个打包目标
- webpack 支持 Jsx 和 Es6
- webpack loaders
    - loader 定义
    - loader 功能
    - loader 配置
    - 使用 loader
- webpack 开发环境与生产环境
- webpack 分割 vendor 代码和应用业务代码
- webpack develop server
    - 安装 webpack-dev-server
    - 启动 webpack-dev-server
    - 代码监控
    - 自动刷新
    - 热加载 （hot module replacement)
    - 在 webpack.config.js 中配置 webpack develop server 
