> [书籍完整目录](https://segmentfault.com/a/1190000005136764)
# 2.1 前端工程化概述

 ![图片描述][1]

在前端开发的初始阶段，开发者通常只用关 html, css, javascript。但是现代化的前端开发已经不仅仅是业务代码本身，真正的前端开发环境涉及很多方面的需求，如：

1. 开发需求
2. 共享需求
3. 性能需求
4. 部署需求

这些东西我们都统一的叫做前端工程化，为了简化前端工程化的配置，出现了很多优秀的工具比如：

1. 前端工作流工具：Gulp，Grunt，Broccoli
2. 前端 Javascript 模块编译工具：Babel，Browserify，Webpack
3. 前端开发系列工具： livereload，数据 mock，代码监控，代码检查

在本节中我们对涉及到这些概念进行简单的概述介绍。

## 2.1.1 前端开发需求 

在开始一个前端项目的时候，通常我们会进行技术选型，然后定义代码规范，已经配合后端和业务进行项目的目录规划，我把这些相关的需求都归为前端开发需求。

### 代码规范 

开发通常来说是一个团队的事，在大型的前端开发团队中，通常会定义团队的代码开发规范，也可以遵循一些开源的规范：[airbnb style guide](https://github.com/airbnb/javascript)。代码规范可以提高代码的可阅读性和避免一些低级错误。为了将代码规范的检查放到前端开发工程中，可以利用 jslint 在提交代码之前对代码进行自动的 code lint。

### Javascript 预处理

**Javascript-* 语言**

前端开发的核心是 Javascript，但是因为 Javascript 语言在设计上的种种不足，为了优化语言本身的问题，出现了很多试图替代 Javascript 的语言， 这其中如：

1. Coffeescript 
2. Livescript
3. Typescript
4. React Jsx 
5. Dart
6. Elm

这些语言在语法上都具有相应的优势，如 Typescript 中提供静态语法的一些强类型特性，Coffeescript, Livescript 提供现代化语言的语法糖特性，专门针对 xml 优化的 JSX 。这些语言最终都会编译为 Javascript。

**ES6**

因为浏览器的实现大多还是 ES5 的标准，为了使用最新的 ES6 语法，通常的做法是采用 Babel 将 ES6 编译为 ES5。

**CommonJS**

前端开发一直在追求模块化，这个过程中出现了 AMD, CommonJs, Umd 这些模块定义模式，简单而言：

- **AMD（Asynchronous Module Definition）:** 异步模块定义，可以异步的加载或依赖其他模块，支持的库如 Require.js, Sea.js 。
- **CommonJs :** 可以将 Javascript 按照 Node 模块的方式定义
- **UMD:** 为了兼容 AMD 还是 CommonJsx 风格，出现了 UMD

AMD 例子：

```javascript
// myFubc.js
define(['jquery'], function ($) {
    function myFunc(){};
    return myFunc;
});
```

CommonJs 例子：

```javascript
var $ = require('jquery');
function myFunc(){};
module.exports = myFunc;
```

UMD 例子：

```javascript
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS-like
        module.exports = factory(require('jquery'));
    } else {
        // Browser globals (root is window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    function myFunc(){};
    return myFunc;
}));
```

其中，CommonJs 的风格是需要编译的，最终会将多个模块编译为一个 UMD 方式定义的模块。 

一个前端项目可能同时用了 Es6 的语法和 CommonJs 的规范，以及 JSX 语言。 所以前端开发需要 Javascript 的编译过程，编译的过程大多是在开发的时候，这个过程就是 Javascript 预处理。

### Css 预处理

和 Javascript 类似，也出现了一些 css-* 语言来优化 css 的开发过程，这些语言如：

- less 
- sass

这些 css-* 语言同样最终都会被编译为原生的 css, 这个过程叫 Css 的预处理。

除了这类需求以外，因为浏览器的兼容问题，导致不得不在 css 上写一些浏览器兼容的 hack, 为了自动解决这些 hack, 出现了一些工具如：

- Autoprefixer
- Compass 

在预处理流程中可以加入这些预处理工具。

### 文件处理 

通常一个前端项目会分有一个 src 目录和 dist 目录， src 放置源码，dist 放置编译后的代码。所以在前端工程的流程中会涉及到文件的拷贝，删除，移动等流程。

### 开发效率

通常的前端开发过程是，修改前端代码，调用命令编译代码，然后浏览器 F5 刷新。这个过程可以做到自动化，通过代码监控工具，指定要监控的目录和文件，如果对应文件有改变，调用编译工具编译源码，然后通过 livereload 自动刷新浏览器。

### 数据 mock

现代化前端项目开发大多是前后端分离的方式，也就是后端基本是提供 API 服务，在真实开发环境中，通常的问题是，后端 API 极其不稳定或者没开发，为了不阻碍前端的开发，通常的做法是，前后端先约定 API 接口定义，然后前端根据定义 mock 接口。

对于大公司来说，可能有 mock 平台来实现这一功能，但对于小公司小项目来说，可能只能自己实现，我们可以利用一些开源的数据 mock 工具来在前端工程中实现。

### 域名代理

对于大型线上前端应用来说，mock 数据远远不够，进入一个页面可能需要多方的后端依赖，只能在真实环境中通过域名代理的方式实现开发和调试。

## 2.1.2 共享需求

对于公司和业务，快速高效的实现业务是终极追求，这对前后端都是挑战。在前端团队中，能够形成基础组件库和业务组件库是一种必然需求。

所以在设计前端项目架构的时候，一定要考虑业务的组件化和可共享性。

- Base 基础代码共享
- 通用工具方法共享
- 基础交互组件共享
- 业务组件共享

React 天然提供了组件的结构，只要在组件的开发过程中，注意组件的独立性，很容易形成可重用的组件。

## 2.1.3 性能需求

web 应用的特点是，所有源码的加载都需要通过网络，所以优化源码的体积是提升首屏加载时间的关键，涉及到的一些点：

1. Javascript, Css 代码压缩
2. Javascript, Css 代码合并 
3. 图片压缩
4. Css 图片精灵或雪碧图（css sprit）

这些过程都可以在前端工程的 build 过程中实现。

## 2.1.4 部署需求

一个前端工程通常是多人维护的，所以会用代码管理工具(默认 git) 来管理源码，然后将开发流程和部署流程和 git 结合起来。

1. 多人分支协作流程：用 git flow 来管理代码分支 
2. 代码自动发布：git hook


## 2.1.5 前端工作流工具

为了解决前端工程中复杂的流程，出现了很多开源前端流程处理工具。这些工作流工具不仅仅是其本身，都是一个流程生态体系，每个工具都涉及到对应的插件库，几乎我们能想到的前端工程问题都有对用的插件能够解决。

### Grunt

Grunt 是最先流行起来的前端工程，其核心思想是基于配置的工作流模式，定义一个配置文件，声明工作流各个环节的相关配置，调用 grunt 就能完成打包编译

### Broccoli

同样是工作流工具，但其核心是以 tree 的基础结构，提供极其高效稳定的工作流。

### Gulp 

目前 Gulp 几乎已经取缔 Grunt ，成为前端的默认流程工具，其核心思想是基于内存的流的方式，提供高效的性能，极简的 API，定义不同的 task，然后将 task 串联起来。

## 2.1.6 前端 Javascript 编译工具

上面我们已经讲到了 Javascript 的预处理，这里详细介绍一下几个预处理工具

### Babel

> Use next generation JavaScript, Today

babel 的核心标语就是，现在就开始使用下一代的 Javascript , 我们可以在工程中使用 Es6, Jsx, 甚至是 Es7 的语法，最终这些语法都可以被编译为 Es5。

### Browserify 

Browserify 是最先出现的 CommonJs 编译工具，使得我们可以像写 Node 模块一样写前端代码，Browserify 可以 build 使用 npm 中的所有模块（只要 模块支持在前端中运行）

### Webpack 

Webpack 是支持 CommonJs 和 AMD 的模块编译工具，逐渐替代 Browserify, 基于 AMD 的好处就是代码可以异步话，这是 Browserify 无法做到的 


### 最后

在本章中不会涉及到所有工具，会根据 React 的开发方式配置一个最合适的前端工程环境。主要是

0. JSX + Es6 + CommonJs 
1. Webpack + Babel 做 Javascript 预处理
2. Gulp 做工作流 


  [1]: /img/bVxDHn