> [书籍完整目录](https://segmentfault.com/a/1190000005136764)

![图片描述][1]

# 2.5 webpack 进阶

- 配置分离
- code splitting  异步加载
- 理解 webpack chunk  
- webpack 调试

## 2.5.1 配置分离

在大型项目中，可能 webpack.config.js 会变得越来越臃肿，这个时候可以利用做 webpack-merge 插件。将配置定义在一个目录下面的不同文件中，然后通过 webpack-merge 来合并成最终的配置。

> webpack-merge 详见 https://www.npmjs.com/package/webpack-merge

## 2.5.2 code splitting 异步加载

代码分割不仅仅是提取业务的公共代码，更应该关注的是实现代码的按需加载。 通过在模块中定义 split point ，webpack 会在打包的时候自动的分割代码块。定义 split point 的方式有两种

### CommonJs 方式

`require.ensure` 方法：

```js
/**
 * @param dependencies [Array] 模块依赖的数组
 * @param callback [Function]
 */
require.ensure(dependencies, callback)
```

eg:
// 模块 index.js
```js
/**
 * [description]
 * @param  {[type]} ) { const      async [description]
 * @return {[type]}   [description]
 */
require.ensure(['./async'], function() {
    const async = require('./async');
    console.log(async.default)
});
```

// 模块 async.js
```js
export default {
    a: 1
}
```

webpack 打包过后会生成三个文件

```
index.js 1.1.js vendor.common.js
```

index.js 的内容为

```js
webpackJsonp([0],[
/* 0 */
/***/ function(module, exports, __webpack_require__) {

    'use strict';
    __webpack_require__.e/* nsure */(1, function () {
      var async = __webpack_require__(1);
      console.log(async.default);
    });

/***/ }
]);
//# sourceMappingURL=home.js.map
```

1.1.js 的内容为

```js
webpackJsonp([1],[
/* 0 */,
/* 1 */
/***/ function(module, exports) {

    "use strict";
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    /**
     * async module
     */
    
    exports.default = {
      a: 1
    };

/***/ }
]);
//# sourceMappingURL=1.1.js.map
```

webpackJsonp 方法定义在 vendor.bundle.js 中:

```js
window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules) {
/******/        // add "moreModules" to the modules object,
/******/        // then flag all "chunkIds" as loaded and fire callback
/******/        var moduleId, chunkId, i = 0, callbacks = [];
/******/        for(;i < chunkIds.length; i++) {
/******/            chunkId = chunkIds[i];
/******/            if(installedChunks[chunkId])
/******/                callbacks.push.apply(callbacks, installedChunks[chunkId]);
/******/            installedChunks[chunkId] = 0;
/******/        }
/******/        for(moduleId in moreModules) {
/******/            modules[moduleId] = moreModules[moduleId];
/******/        }
/******/        if(parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules);
/******/        while(callbacks.length)
/******/            callbacks.shift().call(null, __webpack_require__);
/******/        if(moreModules[0]) {
/******/            installedModules[0] = 0;
/******/            return __webpack_require__(0);
/******/        }
/******/    };
```

所以在 html 中应该先加载 vendor.bundle.js

```html
<script src="js/vendor.bundle.js"></script>
<script src="js/index.js"></script>
```


在使用异步加载的时候需要注意一下两点：

1. 配置 output.publicPath: publicPath 为脚本在服务器中的文字，只有定义了 publicPath 才能通过 webpackJsonp 获取；
2. 是对于异步的 CommonJs 模块引入只能通过 require 的方式引用，不能通过 Es6 import 的方式引入；


### AMD 方式

AMD 的方式也可以实现异步加载，这和使用 require.js 的使用方式基本相同，在定义模块的时候需要按照 AMD 的规范来定义

```js
/**
 * @param dependencies [Array] 模块依赖
 * @param callback [Function]
 */
require(dependencies, callback)
```

eg:

```js
// 定义模块
define('modula-a', ['module-c'], function(c) {
    //
    return ...
})

// 依赖模块
require(["module-a", "module-b"], function(a, b) {
    console.log(a, b);
});
```

这时候的 require 实现是异步的方式，只有依赖的模块加载完成并执行回调，才会执行模块的 callback，依赖模块的回调结果会作为参数传入 a, b 中。

## 2.5.3 代码块 Chunk 

### Chunk 是什么？

webpack 中 Chunk 实际上就是输出的 .js 文件，可能包含多个模块，主要的作用是为了优化异步加载。

### Chunk 包含了哪些内容？

对于同步的情况，一个 Chunk 会递归的把模块中所有的依赖都加入到 Chunk 中。

对于异步的情况，在每一个 split point 上所有依赖的模块会打包进一个新的 chunk，和同步一样，依赖也是递归的，如果子模块依赖其他模块也会加入到 chunk 中，依赖的回调函数中 require 的其他模块也会打包进 chunk 中，以下公式表示 chunk 内容：

**chunk content = recursive(ensure 依赖) + recursive(callback 依赖)**

### Chunk 分类

**Entry Chunk** 

入口代码块包含了 webpack 运行时需要的一些函数，如 `webpackJsonp`, `__webpack_require__` 等以及依赖的一系列模块。

**Normal Chunk** 

普通代码块没有包含运行时需要的代码，只包含模块代码，其结构有加载方式决定，如基于 CommonJs 异步的方式可能会包含 webpackJsonp 的调用。 The chunk also contains a list of chunk id that it fulfills.

**Initial chunk**

与入口代码块对应的一个概念是入口模块（module 0），如果入口代码块中包含了入口模块 webpack 会立即执行这个模块，否则会等待包含入口模块的代码块，包含入口模块的代码块其实就是 initial chunk。 以上面的 CommonJs 异步加载为例:

```html
<!-- 入口 Chunk, 未包含入口模块 -->
<script src="js/vendor.bundle.js"></script>
<!-- 包含入口模块的 Initial Chunk，执行入口模块 -->
<script src="js/index.js"></script>
```

### Commons chunk 与 CommonsChunkPlugin

之前我们已经利用 CommonsChunkPlugin 来分割公共代码如 `react`, `react-dom` 到 vendor.bundle.js 中，这里介绍相关的原理。

以下面的配置为例

```js
var webpack = require("webpack");
module.exports = {
    entry: { a: "./a", b: "./b" },
    output: { filename: "[name].js" },
    plugins: [ new webpack.optimize.CommonsChunkPlugin("init.js") ]
}
```

当有多个入口的时候，CommonsChunkPlugin 会把 a，b 模块公共依赖的模块抽离出来，并加上 webpack 运行时代码，形成一个新的代码块，这个代码块类型为 entry chunk。a,b 两个入口会形成两个单独的代码块，这两个代码块为 initial chunk。

在 html 中，可以如下加载:

```html
<!-- entry chunk -->
<script src="init.js"></script>
<!-- inital chunk a -->
<script src="a.js"></script>
<!-- initial chunk b -->
<script src="b.js"></script>
```

>  更多 CommonChunkPlugin 的参数参见 https://webpack.github.io/docs/list-of-plugins.html#commonschunkplugin 

### require.include 

可以使用 require.include 方法，直接引入模块，如下例子：

```js
require.ensure(["./file"], function(require) {
  require("./file2");
});

// is equal to

require.ensure([], function(require) {
  require.include("./file");
  require("./file2");
});
```

这个方法可以实现一些分块的优化，当一个 chunk-parent 可能会异步引用多个 chunk-child 而这些 chunk-child 可能都包含了 moduleA, 那么可以在 chunk-parent 中 require.include('moduleA') 就可以避免重复加载 moduleA。

### 给异步 Chunk 命名

根据 split point 生成出来的 chunk 名称都是数字，可以在 split point 上定义 chunk 名称：

```js
/**
 * @param chunkName [String] chunk 名称
 */
require.ensure(dependencies, callBack, chunkName)
```

也可以在 webpack.config.js 中配置修改 output.chunkName 来修改 chunk 名称

### Chunk 优化

在一些特殊的场景可以利用如下这些插件来完成 Chunk 的优化，

- LimitChunkCountPlugin
- MinChunkSizePlugin
- AggressiveMergingPlugin

## 调试 webpack

在配置 webpack 的过程中，可以利用 webpack 提供的一些工具和参数来调试。

### 命令行参数

通过调用

```shell
$ webpack [--params,...] 
```

1. `--progress`: 能够看到打包进度
2. `--json`:  可以将打包结果输出为 json 
3. `--display-chunks`: 可以看到打包出来的 chunk 信息

> 更多的参数见 http://webpack.github.io/docs/cli.html#progress

### webpack analyse

> analyse 地址：https://webpack.github.io/analyse/ 

可以通过 analyse 网站分析 webpack 的编译结果，如下图，可以分析 chunks, modules, Assets 等。

![图片描述][2]


  [1]: /img/bVxWgk
  [2]: /img/bVxWcV