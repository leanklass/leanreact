> [书籍完整目录](https://segmentfault.com/a/1190000005136764)
# 2.3 Gulp

![图片描述][1]

在前端工程化中最重要的就是流程管理，借用 gulp 可以很方便的基于流的方式定义流程任务，并将任务串联起来，本节中将详细介绍 gulp ，包括：

- gulp 介绍
    - gulp 是什么
    - gulp 能够解决哪些问题
    - gulp 核心思想和特点
- gulp 安装
- gulp 配置和 API 使用
- gulp 增量 build 


## 2.3.1 Gulp 介绍 

>  The streaming build system , Automate and enhance your workflow

Gulp 是一个基于 Node.js 的开源前端工作流构建工具，目前最新的版本为 **3.9.1** ，最新的维护分支已经到了 **4.0**，更具体一下 Gulp 是：

- **自动化工具**：Gulp 帮助解决开发过程中的流程任务自动化问题
- **平台无关工具**：Gulp 被集成进了大多数的 IDE 中，可以在 PHP, .NET, Node.js, Java 和其他的一些平台上使用 Gulp
- **构建生态系统**：Gulp 拥有完整的插件生态，到目前为止，在 npm.js 上可以搜索到 `13589 results for ‘gulp-’` ，基于这些插件几乎可以完整所有的前端构建任务。

> 我们将使用最新的版本 4.0 来配置 React 的前端工程中。

### Gulp 核心思想和特点

- **流**： Gulp 的设计核心是基于流的方式，将文件转化为抽象的流，然后通过管道的方式将任何串联起来，基于流的方式让任务处理都保存在内存当中，没有临时文件，能够提升构建的性能。
- **基于代码的任务配置**： 在 Gulp 之前，我们熟悉的任务构建工具是 Grunt，在 Grunt 的中，所有的任务都是基于配置的方式，然后 Gulp 的方式并非配置，而是通过提供极简的 API ，以代码的方式定义任务，这样在灵活性上极大的提升。
- **简洁的 API**： Gulp 在 API 的设计上极其简洁，极大的降低学习成本，同时在使用上会非常方便。
- **简单语义化的任务模块**： Gulp 的任务以插件的方式完成，插件的任务功能单一，并且语义化，让工作流的定义更加直观，易读。
- **效率**： 在 Gulp 中任务会尽可能的并发执行

### Gulp 能够解决哪些问题

通常的一个前端构建流程包括：

1. 文件清理  (gulp-clean)
2. 文件拷贝  (gulp-copy)
3. 文件转换 （gulp-webpack）
4. 文件合并 （gulp-concat)
5. 文件压缩 （gulp-minify)
6. 文件服务  （gulp-connect)
7. 文件监控  （gulp-watch)
8. css 相关
    - less，sass 转换 (gulp-less ，gulp-sass)
    - css 自动添加前缀 (gulp-autoprefixer)
9. js 相关
    - jslint (gulo-eslint)
10. html 转换
    - html 模板 (gulp-jade，gulp-ejs)
    - html prettify
    - html validator
    - html minifier 

这些构建任务在 Gulp 中都可以利用插件很容易的配置出来 

### 一个 Gulp 配置示例

Gulp 通过定义 gulpfile.js 配置文件的方式定义流程，gulp.js 会通过调用 Node.js 来执行

一个简单的流程定义文件为：

```javascript
var gulp = require('gulp');
var less = require('gulp-less');
var babel = require('gulp-babel');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var rename = require('gulp-rename');
var cleanCSS = require('gulp-clean-css');
var del = require('del');

var paths = {
  styles: {
    src: 'src/styles/**/*.less',
    dest: 'assets/styles/'
  },
  scripts: {
    src: 'src/scripts/**/*.js',
    dest: 'assets/scripts/'
  }
};

/**
 * 并非所有的任务都是基于流，例如删除文件
 * 一个 gulpfile 只是一个 Node 程序，在 gulpfile 中可以使用任何 npm 中的模块或者其他 Node.js 程序
 */
function clean() {
  // del 也可以和 `gulp.src` 一样可以基于模式匹配的文件路径定义方式 
  return del([ 'assets' ]);
}

/*
 * 通过 Javascript 函数的方式定义任务
 */
function styles() {
  return gulp.src(paths.styles.src)
    .pipe(less())
    .pipe(cleanCSS())
    // 传递一些配置选项到 stream 中
    .pipe(rename({
      basename: 'main',
      suffix: '.min'
    }))
    .pipe(gulp.dest(paths.styles.dest));
}

/**
 * 编译 coffee 文件，然后压缩代码，然后合并到 all.min.js
 * 并生成 coffee 源码的 sourcemap
 */
function scripts() {
  return gulp.src(paths.scripts.src, { sourcemaps: true })
    .pipe(babel())
    .pipe(uglify())
    .pipe(concat('main.min.js'))
    .pipe(gulp.dest(paths.scripts.dest));
}

/**
 * 监控文件，当文件改变过后做对应的任务
 * @return {[type]} [description]
 */
function watch() {
  gulp.watch(paths.scripts.src, scripts);
  gulp.watch(paths.styles.src, styles);
}

/*
 * 使用 CommonJS `exports` 模块的方式定义任务
 */
exports.clean = clean;
exports.styles = styles;
exports.scripts = scripts;
exports.watch = watch;

/*
 * 确定任务是以并行还是串行的方式定义任务
 */
var build = gulp.series(clean, gulp.parallel(styles, scripts));

/*
 * 除了 export 的方式，也可以使用 gulp.task 的方式定义任务
 */
gulp.task('build', build);

/*
 * 定义默认任务，默认任务可以直接通过 gulp 的方式调用
 */
gulp.task('default', build);
```

## 2.3.2 Gulp 安装

```shell
$ cd your-project

// 安装最新版本的 gulp-cli
$ npm install gulpjs/gulp-cli -g

// 安装最新版本的 gulp 4.0
$ npm install gulpjs/gulp.git#4.0 --save-dev

// 检查 gulp 版本
$ gulp -v

---
[10:48:35] CLI version 1.2.1
[10:48:35] Local version 4.0.0-alpha.2
```

## 2.3.3 Gulp 配置与 API

### 任务定义

定义任务有两种方法

第一种方法为 Node.js 模块 exports 的方式:

```javascript
function someTask() {
   ...
}
exports.someTask = SomeTask
```

第二种方法为调用 gulp.task API 的方式

```javascript
function someTask() {
   ...
}

// api 定义方式 1
gulp.task('someTask', someTask)

// ap1 定义方式 2
gulp.task(function someTask() {
    ...
});

// 获取
var someTask = gulp.task('someTask')
```

**任务内容**

通常一个任务会以如下方式定义
```javascript
function someTask() {
    return  gulp.src(...)            // 流的输入
                .pipe(someplugin())  // 插件处理流
                .pipe(someplugin2()) // 插件处理流
                .dest(...)           // 输出流
 }
```

**任务的异步**

task 的执行时异步的，可以基于回调函数 或 promise 或 stream 等方式

**回调函数**

```javascript
var del = require('del');
// 传入 done  回调函数
gulp.task('clean', function(done) {
  del(['.build/'], done);
});
```

**返回流**

```javascript
gulp.task('somename', function() {
  return gulp.src('client/**/*.js')
    .pipe(minify())
    .pipe(gulp.dest('build'));
});
```

**返回 Promise**

```javascript
var Promise = require('promise');
var del = require('del');

gulp.task('clean', function() {
  return new Promise(function (resolve, reject) {
    del(['.build/'], function(err) {
      if (err) {
        reject(err);
      } else {
        resolve();
      }
    });
  });
});
```

**返回子进程**

```javascript
gulp.task('clean', function() {
  return spawn('rm', ['-rf', path.join(__dirname, 'build')]);
});
```

**返回 RxJS observable**

```javascript
var Observable = require('rx').Observable;

gulp.task('sometask', function() {
  return Observable.return(42);
});
```

### 流的入口 gulp.src

```javascript
/**
 * @param globs [String | Array]
 * @param options [Object {
 *          // 默认: process.cwd()
 *          // 描述: 工作目录
 *          cwd: String,   
 *          
 *          // 默认：在模式匹配之前的路径 a/b/ ** / *.js 路径为 a/b/
 *          // 描述：gulp.dest 目录会添加 base 目录
 *          base: String | Number,
 *          ...
 * }]
 */
gulp.src(globs[, options])
```

gulp.src 方法是流的入口，方法的方法返回的结果为一个 [Vinyl files](https://github.com/wearefractal/vinyl-fs) 的 [node stream](http://nodejs.org/api/stream.html) ，可以被 [piped](http://nodejs.org/api/stream.html#stream_readable_pipe_destination_options) 到别的插件中。

```javascript
gulp.src('client/templates/*.jade')
  .pipe(jade())
  .pipe(minify())
  .pipe(gulp.dest('build/minified_templates'));
```

### 匹配模式

gulp.src 的参数 globs 中的 glob 是一种匹配模式，可以使用 \*\*，\* 这些通配符来匹配文件，globs 参数可以为一个 glob 匹配字符串，也可以是 glob 匹配字符串数组

假定我们的项目目录结构如下：

```shell
.
└── src
    ├── d1
    │   ├── d1-1
    │   │   └── f1-1-1.js
    │   └── f1-1.js
    ├── f1.js
    ├── f2.js
    └── f3.js
```

下面是一些匹配的示例：

#### `src/*.js` 

**匹配结果：** 

```shell
src/f1.js src/f2.js src/f3.js
```

**匹配策略：** 

匹配 src 一级目录下面的所有 js 文件，同 

```shell
$ ls src/*.js
```

>`*` 表示匹配文件名称中的 0 个或者多个字符，`*`  不匹配 `.` 开头的文件

#### `src/**/*.js`

**匹配结果：**

```shell
src/d1/d1-1/f1-1-1.js src/d1/f1-1.js        src/f1.js             src/f2.js             src/f3.js
```

**匹配策略：**

匹配 src 下面的所有 js 文件，同 

```shell
$ ls src/**/*.js
```

> ** 表示匹配所有子目录和当前目录，不包括 symlinked 的目录 (如果要包含需要 options 中传入 follow: true)

#### `src/{d1/*.js,*.js}`

**匹配结果：**

```shell
src/d1/f1-1.js src/f1.js      src/f2.js      src/f3.js
```

**匹配策略：** 

匹配 src/d1 一级目录下面的 js 和 src 一级目录下面的 js，同：

```shell
$ ls src/{d1/*.js,*.js}
```

> `{}` 内添加 `,` 可以分割多个匹配

#### 其他匹配模式

- `[...]`: 同正则表达式中的中括号，匹配其中的任意字符，如果字符的开头包好为 `!`或`^` 表示不匹配其中的任何字符
- `!(pattern|pattern|pattern)`: 匹配任意不满足其中的文件
- `?(pattern|pattern|pattern)`: 匹配 0 个或者 1 个 
- `+(pattern|pattern|pattern)`: 匹配 1 个或者多个 
- `*(pattern|pattern|pattern)`: 匹配 0 个或者多个 
- `@(pattern|pattern|pattern)`: 匹配 1 个

gulp 的匹配使用了 node-glob 更多匹配模式可参考 https://github.com/isaacs/node-glob , gulp.src 还可以通过传递 options 配置 glob 的匹配参数，


### 流的出口 gulp.dest

```javascript
/**
 * @param path [String]
 * @param options [Object {
 *          // 默认: process.cwd()
 *          // 描述: 如果提过的 output 目录是相对路径，会将 cwd 作为 output 目录
 *          cwd: String,   
 *          
 *          // 默认：file.stat.mode
 *          // 描述：文件的八进制权限码如 "0744", 如果没有回默认进程权限
 *          mode: String | Number,
 *          
 *          // 默认：process.mode
 *          // 描述：目录的八进制权限码
 *          dirMode: String | Number,
 *
 *          // 默认：true
 *          // 描述：相同路径如果存在文件是否要被覆盖
 *          overwrite： Boolean
 *          ....
 * }]
 */
gulp.dest(globs[, options])
```

gulp.dest 可以理解为流的出口，会基于传入的 path 参数和流的 base 路径导出文件。

```javascript
// 匹配 'client/js/somedir/somefile.js'   

// base 为 client/js
// 导出 为 'build/somedir/somefile.js'
gulp.src('client/js/**/*.js')
  .pipe(minify())
  .pipe(gulp.dest('build'));  

// base 为 client
// 导出 为 'build/js/somedir/somefile.js'
gulp.src('client/js/**/*.js', { base: 'client' })
  .pipe(minify())
  .pipe(gulp.dest('build'));  // 'build/js/somedir/somefile.js'
```


### 任务的并行与串行

在工作流管理中，有些任务需要串行执行，有些任务可能需要并行执行，Gulp 提供了两个 API 来解决此问题：

1. gulp.parallel : 并行执行
2. gulp.series:    串行执行 

eg:

```javascript
gulp.task('one', function(done) {
  // do stuff
  done();
});

gulp.task('two', function(done) {
  // do stuff
  done();
});

// 并行任务，任务执行完成可以添加回调函数
gulp.task('parallelTask', gulp.parallel('one', 'two', function(done) {
  done();
}));

// 串行任务
gulp.task('seriesTask', gulp.series('one', 'two', function(done) {
  done();
}));
```

### 文件监控

gulp 提供的文件监控 API： gulp.watch 

```javascript
/**
 * @param globs [String | Array] 需要监控的文件 globs 
 * @param opts [Object]  https://github.com/paulmillr/chokidar 的配置参数，
 */
gulp.watch(globs[, opts][, fn])
```

使用示例:

```javascript
var watcher = gulp.watch('js/**/*.js', gulp.parallel('concat', 'uglify'));
watcher.on('change', function(path, stats) {
  console.log('File ' + path + ' was changed');
});

watcher.on('unlink', function(path) {
  console.log('File ' + path + ' was removed');
});
```

## 2.3.4 gulp 增量 build  

每次执行构建任务的时候，为了减少构建时间，可以采用增量构建的方式，在 Gulp 中，可以利用一些插件过滤 stream，找出其中修改过的文件。

- [gulp-changed](https://github.com/sindresorhus/gulp-changed) 
- [gulp-newer](https://github.com/tschaub/gulp-newer)

以 gulp-newer 为例:

```javascript
function images() {
  var dest = 'build/img';
  return gulp.src(paths.images)
    .pipe(newer(dest))  // 找出新增加的图像
    .pipe(imagemin({optimizationLevel: 5}))
    .pipe(gulp.dest(dest));
}
```

在某些情况过滤掉 stream 过后还需要还原原来的 stream ，比如文件 transform 过后还需要文件合并，这种时候可以利用一下这两个插件：

- [gulp-cached](https://github.com/wearefractal/gulp-cached)
- [gulp-remember](https://github.com/wearefractal/gulp-cached)

```javascript
function scripts() {
  return gulp.src(scriptsGlob)
    .pipe(cache('scripts'))         // 和 newer 类似，过滤出改变了的 scripts
    .pipe(header('(function () {')) // 文件添加 header
    .pipe(footer('})();'))          // 文件添加 footer
    .pipe(remember('scripts'))      // 找出所有的 scripts
    .pipe(concat('app.js'))         // 将所有文件合并
    .pipe(gulp.dest('public/'))
}
```


  [1]: /img/bVxOv6