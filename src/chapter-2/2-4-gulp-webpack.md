> [书籍完整目录](https://segmentfault.com/a/1190000005136764)
# 2.4 webpack + gulp 构建完整前端工作流

![图片描述][1]

在前面的两个小节中已经完整的讲了 webpack 和 gulp 相关的内容，本小节中将会结合二者构建一个完整的前端工作流，内容目录为：

- 前端工程结构和目标
- 前端工程目录结构
- gulp clean
- gulp copy
- gulp less
- gulp autoprefixer
- gulp webpack 
- gulp eslint
- gulp watch
- gulp connect 和 livereload 
- gulp mock server
- gulp test

## 2.4.1 前端工程结构和目标

React 在大多数情况被当做当 SPA （单页面应用）的框架，实际上在真实业务开发过程中，非单页面应用的业务框架居多。所以我们在构建前端工程的时候，以多个页面的方式维护。下面定义前端工程的目标：

### 基础技术

1. react(es6 + jsx)
2. less
3. gulp + webpack

### 应用模式 

多页面应用：以多页面应用方式，能同时构建多个页面

### 样式结构 

在样式的架构上的一些基本需求:

1. 基于 less 或者 sass 或者其他样式语言
2. 基础库
3. 共享变量和工具类

样式的设计上，大可归为两种方式：

1. **独立样式**：样式的开发和其他代码尽量分离独立；
2. **Inline Style**：样式通过 javascript 变量维护，或者通过工具将 css 转化为 javascript，再应用到 React 的 style 中； 

样式文件在工程中的位置也可以分为两种：

1. **逻辑样式隔离**：javascript 文件和样式文件在不同的工程目录，这是传统的样式逻辑分离设计，符合大多数的业务场景；
2. **component pod**：也就是一个组件的目录结构包括样式，逻辑和模板，在 React 中模板和逻辑是在一起的，也就是一个组件包括一个 `component.js` 和 `component.css` 。 这种模式的目的是出于组件的 **独立性**，所以基于 pod 的组件好处是能够更好的共享，但坏处是和不方便共享变量和工具类（共享就会产生耦合，也就违背了 pod 的目的）

所以在样式的设计上，我们应用如下这些设计：

1. 基于 Less 
2. Less 相关的基础库和公共变量独立出来，变量主要是用于主题设置
3. 样式统一放在 style 目录下面，业务组件需要共享变量，以非 pod 的方式设计样式，放置在 style 目录独立文件中，文件名称和组件名相同
4. 需要独立的组件以 npm 的方式维护，样式以 pod 和 inline style 的方式设计

### 兼容的第三方库引入方式

在第三方库的引入上，可能以 bower_components 的方式，可能是自己公司内部维护的第三方库和基础组件，也可能是 npm 组件，所以为了兼容这些第三方库的引入，确定一下规范:

1. vendor 目录下面放在第三方库，包括样式和逻辑，为了优化编译速度，这些目录的文件在编译的时候只做合并，避免和业务代码的编译做过多的耦合（vendor 库文件通常比较大）
2. bower_components 中的库同 vendor 
3. npm 中的第三方库做代码分割，统一打包到 vendor.bundle.js 中

### 配置自动化

业务代码可能在不断增加，在工程 build 的时候，尽量以 glob 的方式匹配文件，避免增加一个业务文件就需要修改配置

### 高效的编译

代码编译的时间如果太长，会极大的影响开发体验，所以在编译的时候要考虑提高编译的效率：

1. 避免全局编译
2. 增量编译：利用上一节介绍的 gulp-cached 和 gulp-remember
3. 只在 prodution 的时候才做代码压缩优化

### 后端数据 mock 和代理

能够支持数据 mock 和代理功能

## 2.4.2 前端工程目录结构

基于这些目标定义如下工程结构：

```
.
├── package.json                 
├── README.md                    
├── gulpfile.js                  // gulp 配置文件
├── webpack.config.js            // webpack 配置文件
├── doc                          // doc  目录：放置应用文档
├── test                         // test 目录：测试文件
├── dist                         // dist 目录：放置开发时候的临时打包文件
├── bin                          // bin  目录：放置 prodcution 打包文件
├── mocks                        // 数据 mock 相关  
├── src                          // 源文件目录
│   ├── html                     // html 目录 
│   │   ├── index.html
│   │   └── page2.html
│   ├── js                       // js 目录 
│   │   ├── common               // 所有页面的共享区域，可能包含共享组件，共享工具类
│   │   ├── home                 // home 页面 js 目录
│   │   │   ├── components
│   │   │   │   ├── App.js
│   │   │   ├── index.js         // 每个页面会有一个入口，统一为 index.js
│   │   ├── page2                // page2 页面 js 目录
│   │   │   ├── components
│   │   │   │   ├── App.js
│   │   │   └── index.js
│   └── style                    // style 目录
│       ├── common               // 公共样式区域
│       │   ├── varables.less    // 公共共享变量
│       │   ├── index.less       // 公共样式入口
│       ├── home                 // home 页面样式目录    
│       │   ├── components       // home 页面组件样式目录
│       │   │   ├── App.less 
│       │   ├── index.less       // home 页面样式入口
│       ├── page2                // page2 页面样式目录
│       │   ├── components       
│       │   │   ├── App.less
│       │   └── index.less       
├── vendor
│   └── bootstrap
└── └── jquery
```

## 2.4.3 安装基础依赖

目录创建好过后，进入项目目录，安装 webpack ，gulp，react 相关的基础依赖

// react 相关
```shell
$ cd project
$ npm install react react-dom --save
```

// webpack 相关
```shell
$ npm install webpack-dev-server webpack --save-dev
$ npm install babel-loader babel-preset-es2015 babel-preset-stage-0 babel-preset-react babel-polyfill --save-dev
```

// gulp 相关
```shell
$ npm install gulpjs/gulp-cli -g
$ npm install gulpjs/gulp.git#4.0 --save-dev
$ npm install gulp-util del gulp-rename gulp-less gulp-connect connect-rest@1.9.5  --save-dev
```

## 2.4.4 创建 gulpfile.js

创建 gulpfile.js 并全局定义打包源文件和打包目标相关的配置

```javascript
// gulpfile.js
var gulp = require("gulp");
var gutil = require("gulp-util");
var src = {
  // html 文件
  html: "src/html/*.html",                          
  // vendor 目录和 bower_components
  vendor: ["vendor/**/*", "bower_components/**/*"], 
  // style 目录下所有 xx/index.less
  style: "src/style/*/index.less",                 
  // 图片等应用资源
  assets: "assets/**/*"                             
};

var dist = {
  root: "dist/",
  html: "dist/",
  style: "dist/style",
  vendor: "dist/vendor",
  assets: "dist/assets"
};

var bin = {
  root: "bin/",
  html: "bin/",
  style: "bin/style",
  vendor: "bin/vendor",
  assets: "bin/assets"
};
```

## 2.4.5 清理和拷贝任务

在每次启动 build 的时候，需要先清除之间的 build 结果，然后将最新的文件如 html, assets, vendor 这些文件拷贝到 dist 目录中

```js
var del = require("del");

/**
 * clean build dir
 */
function clean(done) {
  del.sync(dist.root);
  done();
}

/**
 * [cleanBin description]
 * @return {[type]} [description]
 */
function cleanBin(done) {
  del.sync(bin.root);
  done();
}

/**
 * [copyVendor description]
 * @return {[type]} [description]
 */
function copyVendor() {
  return gulp.src(src.vendor)
    .pipe(gulp.dest(dist.vendor));
}

/**
 * [copyAssets description]
 * @return {[type]} [description]
 */
function copyAssets() {
  return gulp.src(src.assets)
    .pipe(gulp.dest(dist.assets));
}

/**
 * [copyDist description]
 * @return {[type]} [description]
 */
function copyDist() {
  return gulp.src(dist.root + '**/*')
    .pipe(gulp.dest(bin.root));
}

/**
 * [html description]
 * @return {[type]} [description]
 */
function html() {
    return gulp.src(src.html)
      .pipe(gulp.dest(dist.html))
}
```


## 2.4.6 样式转换

样式使用了 gulp-less 插件，其中需要注意的一点是，less 文件如果出错可能会将整个 build 进程结束，需要添加 error 时候的处理函数，同时也能自定义的输出 error 相关的信息，样式的转换使用了 autoprefixer 代码补全

```js
var less = require('gulp-less');
var autoprefixer = require('gulp-autoprefixer');

/**
 * [style description]
 * @param  {Function} done [description]
 * @return {[type]}        [description]
 */
function style() {
    return gulp.src(src.style)
      .pipe(cached('style'))
      .pipe(less())
      .on('error', handleError)
      .pipe(autoprefixer({
        browsers: ['last 3 version']
      }))
      .pipe(gulp.dest(dist.style))
}

exports.style = style;

/**
 * [handleError description]
 * @param  {[type]} err [description]
 * @return {[type]}     [description]
 */
function handleError(err) {
  if (err.message) {
    console.log(err.message)
  } else {
    console.log(err)
  }
  this.emit('end')
}
```

执行 gulp style 测试

```shell
$ gulp style
[21:00:35] Starting 'style'...
[21:00:36] Finished 'style' after 87 ms
```

home/index.less 

```css
/**
 * before
 */
body {
  background: white;
  color: #333;
  transform: rotate(45deg);
  display: flex;
}

/**
 * after
 */
body {
  background: white;
  color: #333;
  -webkit-transform: rotate(45deg);
      -ms-transform: rotate(45deg);
          transform: rotate(45deg);
  display: -webkit-flex;
  display: -ms-flexbox;
  display: flex;
}
```

如果需要做样式的 lint 可以通过 ，gulp-stylelint 来实现。

更多信息可参考：

> gulp-autoprefixer: https://www.npmjs.com/package/gulp-autoprefixer
> gulp-stylelint: https://github.com/stylelint/stylelint


## 2.4.7 webpack 配置 

首先创建 webpack.config.js，这里使用的一个技巧是使用 glob 动态添加 entry，让配置做到自动化。

```js
// webpack.config.js
var webpack = require("webpack");
const glob = require('glob');

var config = {
  entry: {
    vendor: [
      'react',
      'react-dom'
    ]
  },
  output: {
    path: __dirname + '/dist/js/',
    filename: '[name].js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: { presets: [ 'es2015', 'stage-0', 'react' ] }
      }
    ]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.bundle.js')
  ]
};

/**
 * find entries
 */
var files = glob.sync('./src/js/*/index.js');
var newEntries = files.reduce(function(memo, file) {
  var name = /.*\/(.*?)\/index\.js/.exec(file)[1];
  memo[name] = entry(name);
  return memo;
}, {});

config.entry = Object.assign({}, config.entry, newEntries);

/**
 * [entry description]
 * @param  {[type]} name [description]
 * @return {[type]}      [description]
 */
function entry(name) {
  return './src/js/' + name + '/index.js';
}

module.exports = config;
```

在 gulpfile 中定义 webpack 任务，因为 webpack.config.js 为一个 node 模块，可直接引入，
又因为在 production 环境和 development 环境的模式是不同的，可以定义两个不同的任务：

```js
/**
 * [webpack 相关的依赖]
 * @type {[type]}
 */
var webpack = require("webpack");
var WebpackDevServer = require("webpack-dev-server");
var webpackConfig = require("./webpack.config.js");

/**
 * [webpackDevelopment description]
 * @param  {Function} done [description]
 * @return {[type]}        [description]
 */
var devConfig, devCompiler;

devConfig = Object.create(webpackConfig);
devConfig.devtool = "sourcemap";
devConfig.debug = true;
devCompiler = webpack(devConfig);

function webpackDevelopment(done) {
  devCompiler.run(function(err, stats) {
    if (err) {
      throw new gutil.PluginError("webpack:build-dev", err);
      return;
    }
    gutil.log("[webpack:build-dev]", stats.toString({
      colors: true
    }));
    done();
  });
}

/**
 * [webpackProduction description]
 * production 任务中添加了压缩和打包优化组件，且没有 sourcemap
 * @param  {Function} done [description]
 * @return {[type]}        [description]
 */
function webpackProduction(done) {
  var config = Object.create(webpackConfig);
  config.plugins = config.plugins.concat(
    new webpack.DefinePlugin({
      "process.env": {
        "NODE_ENV": "production"
      }
    }),
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.UglifyJsPlugin()
  );

  webpack(config, function(err, stats) {
    if(err) throw new gutil.PluginError("webpack:build", err);
    gutil.log("[webpack:production]", stats.toString({
      colors: true
    }));
    done();
  });
}
```


## 2.4.8 javascript lint

为了能够 lint Es6 和 jsx 的 javascript ，可以基于 Eslint 来实现，Eslint 的基本配置：

**安装相关依赖**

```shell
$ npm install eslint eslint-loader eslint-plugin-react  --save-dev
```

**添加 .eslintrc 配置文件**

```json
{
  // Extend existing configuration
  // from ESlint and eslint-plugin-react defaults.
  "extends": [
    "eslint:recommended", 
    "plugin:react/recommended"
  ],
  // Enable ES6 support. If you want to use custom Babel
  // features, you will need to enable a custom parser
  // as described in a section below.
  "parserOptions": {
    "ecmaVersion": 6,
    "sourceType": "module"
  },
  "env": {
    "browser": true,
    "node": true
  },
  // Enable custom plugin known as eslint-plugin-react
  "plugins": [
    "react"
  ],
  "rules": {
    // Disable `no-console` rule
    "no-console": 0,
    // Give a warning if identifiers contain underscores
    "no-underscore-dangle": 1,
    // Default to single quotes and raise an error if something
    // else is used
    "quotes": [2, "single"]
  }
}
```

**修改 webpack.config.js 配置**

在 webpack.config.js 中添加 preloaders (preloader 会在其他 loader 前应用)

```js
module: {
  preLoaders:[{
    test: /\.js$/, 
    loader: "eslint-loader", 
    exclude: /node_modules/
  }]
},
eslint: {
  configFile: './.eslintrc'
}
```

**测试**

修改 index.js 添加如下的代码块：

```js
const d = a ? b : c;
```

运行 webpack 测试

```shell
$ webpack

ERROR in ./src/js/home/index.js
......./src/js/home/index.js
  1:7   error  'd' is defined but never used  no-unused-vars
  1:11  error  'a' is not defined             no-undef
  1:15  error  'b' is not defined             no-undef
  1:19  error  'c' is not defined             no-undef

✖ 4 problems (4 errors, 0 warnings)
```

更多的 eslint 的定义可以参考官网：http://eslint.org

## 2.4.9 自动刷新和数据 mock

代码的自动刷新用到了 gulp-connect 插件，并通过 connect-rest 模块实现 rest 接口的数据 mock。 

```js
/**
 * [connectServer description]
 * @return {[type]} [description]
 */
function connectServer(done) {
  connect.server({
      root: dist.root,
      port: 8080,
      livereload: true,
      middleware: function(connect, opt) {
        return [rest.rester({
          context: "/"
        })]
      }
  });
  mocks(rest);
  done();
}
```

mocks 目录下面定义了一个 index.js 如下：

```js
/**
 * [mocks]
 * @param  {[type]} app [description]
 * @return {[type]}     [description]
 */
module.exports = function(app) {
    app.get("rest", function(req, content, callback) {
        setTimeout(function() {
            callback(null, {
                a: 1,
                b: 2
            });
        }, 500)
    })
}
```

connect-rest 不仅可以做数据 mock 的 rest 接口，同时也能实现 proxy 转发。更多可参见 https://github.com/imrefazekas/connect-rest/tree/v2

> 需要注意的是 connect-rest 用到的版本为 1.9.5, 高版本不兼容。

## 2.4.10 代码监控 

为了能够监控文件改变能实现自动刷新，还需要通过定义 watch 任务，监控文件的改变。 这里使用到了一个 trick，只监控 dist 目录的文件，如果该目录文件改变了，使用 pipe 的方式调用 connect.reload(), 直接调用不会自动刷新

```js
/**
 * [watch description]
 * @return {[type]} [description]
 */
function watch() {
  gulp.watch(src.html, html);
  gulp.watch("src/**/*.js", webpackDevelopment);
  gulp.watch("src/**/*.less", style);
  gulp.watch("dist/**/*").on('change', function(file) {
    gulp.src('dist/')
      .pipe(connect.reload());
  });
}

```

## 2.4.11 任务编排

最后将之前定义的任务通过 parallel 和 series 方法进行编排， 默认任务为开发任务，build 任务为 production 任务

```js
/**
 * default task
 */
gulp.task("default", gulp.series(
  clean, 
  gulp.parallel(copyAssets, copyVendor, html, style, webpackDevelopment), 
  connectServer, 
  watch
));

/** 
 * production build task
 */
gulp.task("build", gulp.series(
  clean, 
  gulp.parallel(copyAssets, copyVendor, html, style, webpackProduction), 
  cleanBin, 
  copyDist, 
  function(done) {
    console.log('build success');
    done();
  }
));
```

## 2.4.12 webpack-dev-server vs gulp-connect

在 webpack 小节也介绍过 webpack-dev-server 可以实现自动刷新并能实现局部的热加载 ，那为什么不使用 webpack-dev-server 而是使用 gulp-connect？

我的观点是对于一般的项目来说两者都可以使用，甚至可以只使用 webpack 就能完整工程构建任务，但是引入了 gulp 过后，能够更加清晰可控的编排任务，通过使用 gulp-connect 能够很方便的通过中间件的方式实现数据 mock，并且也能和 gulp.watch 整合。


## 附件：完整代码

// webpack.config.js
```js
var webpack = require("webpack");
const glob = require('glob');
var config = {
    entry: {
        vendor: ['react', 'react-dom']
    },
    output: {
        path: __dirname + '/dist/js/',
        filename: '[name].js'
    },
    module: {
        loaders: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel',
            query: {
                presets: ['es2015', 'stage-0', 'react']
            }
        }],
        preLoaders:[{
            test: /\.js$/, 
            loader: "eslint-loader", 
            exclude: /node_modules/
        }],
    },
    plugins: [
        new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.bundle.js')
    ],
    eslint: {
        configFile: './.eslintrc'
    }
};
/**
 * find entries
 */
var files = glob.sync('./src/js/*/index.js');
var newEntries = files.reduce(function(memo, file) {
    var name = /.*\/(.*?)\/index\.js/.exec(file)[1];
    memo[name] = entry(name);
    return memo;
}, {});
config.entry = Object.assign({}, config.entry, newEntries);
/**
 * [entry description]
 * @param  {[type]} name [description]
 * @return {[type]}      [description]
 */
function entry(name) {
    return './src/js/' + name + '/index.js';
}
module.exports = config;
```

// gulpfile.js
```js
/**
 * [gulp description]
 * @type {[type]}
 */
var gulp = require("gulp");
var gutil = require("gulp-util");
var del = require("del");
var rename = require('gulp-rename');
var less = require('gulp-less');
var autoprefixer = require('gulp-autoprefixer');
var cached = require('gulp-cached');
var remember = require('gulp-remember');

var webpack = require("webpack");
var WebpackDevServer = require("webpack-dev-server");
var webpackConfig = require("./webpack.config.js");

var connect = require('gulp-connect');
var rest = require('connect-rest');
var mocks = require('./mocks');

/**
 * ----------------------------------------------------
 * source configuration
 * ----------------------------------------------------
 */

var src = {
  html: "src/html/*.html",                          // html 文件
  vendor: ["vendor/**/*", "bower_components/**/*"], // vendor 目录和 bower_components
  style: "src/style/*/index.less",                  // style 目录下所有 xx/index.less
  assets: "assets/**/*"                             // 图片等应用资源
};

var dist = {
  root: "dist/",
  html: "dist/",
  style: "dist/style",
  vendor: "dist/vendor",
  assets: "dist/assets"
};

var bin = {
  root: "bin/",
  html: "bin/",
  style: "bin/style",
  vendor: "bin/vendor",
  assets: "bin/assets"
};

/**
 * ----------------------------------------------------
 *  tasks
 * ----------------------------------------------------
 */

/**
 * clean build dir
 */
function clean(done) {
  del.sync(dist.root);
  done();
}

/**
 * [cleanBin description]
 * @return {[type]} [description]
 */
function cleanBin(done) {
  del.sync(bin.root);
  done();
}

/**
 * [copyVendor description]
 * @return {[type]} [description]
 */
function copyVendor() {
  return gulp.src(src.vendor)
    .pipe(gulp.dest(dist.vendor));
}

/**
 * [copyAssets description]
 * @return {[type]} [description]
 */
function copyAssets() {
  return gulp.src(src.assets)
    .pipe(gulp.dest(dist.assets));
}

/**
 * [copyDist description]
 * @return {[type]} [description]
 */
function copyDist() {
  return gulp.src(dist.root + '**/*')
    .pipe(gulp.dest(bin.root));
}

/**
 * [html description]
 * @return {[type]} [description]
 */
function html() {
    return gulp.src(src.html)
      .pipe(gulp.dest(dist.html))
}

/**
 * [style description]
 * @param  {Function} done [description]
 * @return {[type]}        [description]
 */
function style() {
    return gulp.src(src.style)
      .pipe(cached('style'))
      .pipe(less())
      .on('error', handleError)
      .pipe(autoprefixer({
        browsers: ['last 3 version']
      }))
      .pipe(gulp.dest(dist.style))
}

exports.style = style;

/**
 * [webpackProduction description]
 * @param  {Function} done [description]
 * @return {[type]}        [description]
 */
function webpackProduction(done) {
  var config = Object.create(webpackConfig);
  config.plugins = config.plugins.concat(
    new webpack.DefinePlugin({
      "process.env": {
        "NODE_ENV": "production"
      }
    }),
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.UglifyJsPlugin()
  );

  webpack(config, function(err, stats) {
    if(err) throw new gutil.PluginError("webpack:build", err);
    gutil.log("[webpack:production]", stats.toString({
      colors: true
    }));
    done();
  });
}


/**
 * [webpackDevelopment description]
 * @param  {Function} done [description]
 * @return {[type]}        [description]
 */
var devConfig, devCompiler;

devConfig = Object.create(webpackConfig);
devConfig.devtool = "sourcemap";
devConfig.debug = true;
devCompiler = webpack(devConfig);

function webpackDevelopment(done) {
  devCompiler.run(function(err, stats) {
    if (err) {
      throw new gutil.PluginError("webpack:build-dev", err);
      return;
    }
    gutil.log("[webpack:build-dev]", stats.toString({
      colors: true
    }));
    done();
  });
}

/**
 * webpack develop server
 */
// devConfig.plugins = devConfig.plugins || []
// devConfig.plugins.push(new webpack.HotModuleReplacementPlugin())
// function webpackDevelopmentServer(done) {
//   new WebpackDevServer(devCompiler, {
//    contentBase: dist.root,
//     lazy: false,
//     hot: true
//   }).listen(8080, 'localhost', function (err) {
//     if (err) throw new gutil.PluginError('webpack-dev-server', err)
//     gutil.log('[webpack-dev-server]', 'http://localhost:8080/')
//  reload();
//  done();
//   });
// }

/**
 * [connectServer description]
 * @return {[type]} [description]
 */
function connectServer(done) {
  connect.server({
      root: dist.root,
      port: 8080,
      livereload: true,
      middleware: function(connect, opt) {
        return [rest.rester({
          context: "/"
        })]
      }
  });
  mocks(rest);
  done();
}

/**
 * [watch description]
 * @return {[type]} [description]
 */
function watch() {
  gulp.watch(src.html, html);
  gulp.watch("src/**/*.js", webpackDevelopment);
  gulp.watch("src/**/*.less", style);
  gulp.watch("dist/**/*").on('change', function(file) {
    gulp.src('dist/')
      .pipe(connect.reload());
  });
}

/**
 * default task
 */
gulp.task("default", gulp.series(
  clean, 
  gulp.parallel(copyAssets, copyVendor, html, style, webpackDevelopment), 
  connectServer, 
  watch
));

/** 
 * production build task
 */
gulp.task("build", gulp.series(
  clean, 
  gulp.parallel(copyAssets, copyVendor, html, style, webpackProduction), 
  cleanBin, 
  copyDist, 
  function(done) {
    console.log('build success');
    done();
  }
));

/**
 * [handleError description]
 * @param  {[type]} err [description]
 * @return {[type]}     [description]
 */
function handleError(err) {
  if (err.message) {
    console.log(err.message)
  } else {
    console.log(err)
  }
  this.emit('end')
}

/**
 * [reload description]
 * @return {[type]} [description]
 */
function reload() {
  connect.reload();
}
```


// .eslintrc

```json
{
  // Extend existing configuration
  // from ESlint and eslint-plugin-react defaults.
  "extends": [
    "eslint:recommended", 
    "plugin:react/recommended"
  ],
  // Enable ES6 support. If you want to use custom Babel
  // features, you will need to enable a custom parser
  // as described in a section below.
  "parserOptions": {
    "ecmaVersion": 6,
    "sourceType": "module"
  },
  "env": {
    "browser": true,
    "node": true
  },
  // Enable custom plugin known as eslint-plugin-react
  "plugins": [
    "react"
  ],
  "rules": {
    // Disable `no-console` rule
    "no-console": 0,
    // Give a warning if identifiers contain underscores
    "no-underscore-dangle": 1,
    // Default to single quotes and raise an error if something
    // else is used
    "quotes": [2, "single"]
  }
}
```


  [1]: /img/bVxUol