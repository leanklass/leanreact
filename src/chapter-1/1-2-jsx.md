> [书籍完整目录](https://segmentfault.com/a/1190000005136764)
# 1.2 JSX 语法
![图片描述][1]
> 官方文档 https://facebook.github.io/react/docs/jsx-in-depth.html

JSX 语法听上去很讨厌，但当真正使用的时候会发现，JSX 的写法在组件的组合和属性的传递上提供了非常灵活的解决方案。   

在学习本节的时候，希望读者在阅读的同时能够实际编码体验 JSX ，写代码的意思是真的要写.代.码。

## 1.2.1 准备 React 运行环境

为了快速开始 JSX 的学习，我们可以通过如下几种方式快速进入 React 开发环境

### 方式一：Babel REPL

**[Babel REPL](https://babeljs.io/repl/)**

直接在 REPL 中写 JSX 语法，可以实时的查看编译后的结果。

### 方式二：JSFiddle
 
在线模式 [React Fiddle](https://jsfiddle.net/reactjs/69z2wepo/)

### 方式三：本地开发

第一步：打开编辑器，新建一个 `hello-react.html` 文件 
第二步：粘贴 Hello, world 代码

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello React!</title>
    <script src="http://facebook.github.io/react/js/react.js"></script>
    <script src="http://facebook.github.io/react/js/react-dom.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
        var Hello = React.createClass({
          render: function render() {
            return <div>Hello {this.props.name}</div>;
          }
        });

        ReactDOM.render(
          <Hello name="World" />,
          document.getElementById('example')
        );
    </script>
  </body>
</html>
```

### 方式四：clone Github hello-world 分支代码

https://github.com/leanklass/leanreact/tree/hello-world 

## 1.2.2 JSX 语法

### JSX 本质 

创建 JSX 语法的本质目的是为了使用基于 xml 的方式表达组件的嵌套，保持和 HTML 一致的结构，语法上除了在描述组件上比较特别以外，其它和普通的 Javascript 没有区别。 并且最终所有的 JSX 都会编译为原生 Javascript。

需要提醒读者的是，React 的很多例子都是通过 ES6 来写的， 但这并不是 JSX 语法，后面我们会有单独的一小节讲解 ES6 的基本语法，不过目前为止我们先将跟多精力放在 JSX 上。

### xml 基本规则 

JSX 构建组件的规则和 xml 规则一致

#### 嵌套规则

标签可以任意的嵌套

eg:

```html
function render() {
  return <p>
           text content 
           <ul>
             <li>....</li>
             <li>....</li>
           </ul>
         </p>
}

```

#### 标签闭合

标签必须严格闭合，否则无法编译通过

自闭合：
```html
function render() {
  return <input type="text"/>
}
```

标签闭合: 
```html
function render() {
  return <p>....</p>
}
```

### JSX 组件

JSX 组件分为 HTML 组件和 React 组件

HTML 组件就是 HTML 中的原生标签， 如： 
```html
function render() {
  return  <p> hello, React World </p>
}

function render() {
  return <ul> 
            <li>list item 1</li>
            <li>list item 2</li>
         </ul>
}
```

React 组件就是自定义的组件，如

```html
// 定义一个自定义组件
var CustomComponnet = React.createClass({
  render: function() {
    return <div> custom component </div>
  }
});

// 使用自定义组件
function render() {
    return <p> <CustomComponent/> </p>
}

```

### 组件属性

和 html 一样，JSX 中组件也有属性，传递属性的方式也相同

对于 HTML 组件
```html
 function render() {
   return  <p title="title" >hello, React, world </p>
 }
```

如果是 React 组件可以定义自定义属性，传递自定义属性的方式：
```html
  function render() {
    return <p> <CustomComponent customProps="data"/> </p>
  }
}
```

属性即可以是字符串，也可以是任意的 Javascript 变量
, 传递方式是将变量用花括号， eg:
```html
  function render() {
    var data = {a: 1, b:2};
    return <p> <CustomComponent customProps={data}/> </p>
  }
```

需要注意的地方上，属性的写法上和 HTML 存在区别，在写 JSX 的时候，所有的属性都是驼峰式的写法，如：

```html
function render() {
    return  <div className="...">
                <label htmlFor="..."></label>
                <input onChange="..."/>
            </div>
}
```

而原生的写法为:
```html
    <div class="...">
        <label for="..."></label>
        <input onchange="..."/>
    </div>
```

主要是出于标准的原因，驼峰式是 Javascript 的标准写法，并且 React 底层是将属性直接对应到原生 DOM 的属性，而原生 DOM 的属性是驼峰式的写法，这里也可以理解为什么类名不是 class 而是 className 了, 又因为 class 和 for 还是 js 关键字，所以 jsx 中:

- **class => className**
- **for   => htmlFor**

除此之外比较特殊的地方是 `data-*` 和 `aria-*` 两类属性是和 HTML 一致的。

### JSX 花括号

#### 显示文本

很多情况，我们需要将 JS 中的文本直接显示，做法和显示变量属性一样，用花括号

```html
  function render() {
    var text = "Hello, World"
    return <p> {text} </p>
  }
```

#### 运算

花括号里边实际上除了变量以外，可以是一段 JS 表达式，我们可以利用花括号做简单的运算：

```html
  funtion render() {
    var text = text;
    var isTrue = false;
    var arr = [1, 2, 3];
    return <p>
      {text}
      {isTrue ? "true" : "false"}

      {arr.map(function(it) {
        return <span> {it} </span>
      })}

      </p>
  }
```

### JSX 注释

注释的写法如下：

```html
  function render() {
    return <p>
            /* 这里是注释内容 */
           </p>
  }

```

### 限制规则

render 方法返回的组件必须是有且只有一个根组件，错误情况的例子

```html
  // 无法编译通过，JSX 会提示编译错误
  function render() {
    return (<p> .... </p>
           <p> .... </p>)
  }
```


###  组件命名空间  

JSX 可以通过命名空间的方式使用组件, 通过命名空间的方式可以解决相同名称不同用途组件冲突的问题。

如：

```html
  function render() {
    return <p>
           <CustomComponent1.SubElement/>
           <CustomComponent2.SubElement/>
           </p>
  }
```


## 1.2.3 理解 JSX

###  JSX 的编译方式
JSX 的写法最终会被编译成原生的 Javascript。 如果你愿意的话，也可以直接写编译后的 JS, 不过最好是写 JSX, 因为 JSX 的目的就是为了简化写法，并保持和 HTML 相同的开发体验。

JSX 具体的编译方式有两种

1. 在 HTML 中引入 babel 编译器, 如上 Hello World 程序中一样。
2. 离线编译 JSX，通过 babel 编译 JSX，细节我们将在第二章中讲解。

### JSX 到 JS 的转化 

Hello World 程序转化为 JS 的代码如下：

```javascript
  var Hello = React.createClass({
    displayName: 'Hello',
    render: function() {
      return React.createElement("div", null, "Hello ", this.props.name);
    }
  });

  ReactDOM.render(
    React.createElement(Hello, {name: "World"}),
    document.getElementById('container')
  );
```

可以看出：

<Hello .../>  <=> React.createElement(Hello,  ....) 

xml 的写法实际上是调用 React 的工厂方法 createElement。

当理解了 JSX 最终会编译为 JS 就可以理解 JSX 的一些特性如命名空间，组件实际上就是一个 Javascript 对象，命名空间下的组件相当于对象的属性

## 1.2.4 实例练习：通过数据渲染 TODOMVC 代办事项列表 

[TODOMVC](http://todomvc.com) 以代办事项列表为需求模型，包含了各种框架的实现。 本例子的目的为了让读者能够切身的体会 JSX 的使用方法。

### 问题需求

根据一个 JSON 对象，用 React JSX 的方式渲染出 [TODOMVC](http://todomvc.com/examples/react/#/) 页面:

 ![图片描述][2]

JSON 对象如下：

```javascript
var todolist = {
    name: "todos",
    todos: [{
            completed: false,
            title: 'finish exercise'
        }, {
            completed: false,
            title: 'lean jsx'
        }, {
            completed: true,
            title: 'lean react'
        }]
}
```

修改 hello world index.html 中的代码，为了简化问题， 我们可以直接复制 TODOMVC 中的 HTML 和 CSS 。

### Tips 

1. class 要写为 className 
2. input 标签未闭合 
3. 数组遍历过后要添加 key 属性，否则会提示 error 信息（在组件章节会讲解）
4. [html 转 JSX工具](http://facebook.github.io/react/html-jsx.html)， Facebook 提供了 html 转 JSX 组件的工具，可以直接复制 html 转为 JSX 组件

### 参考答案 

https://github.com/leanklass/leanreact/tree/jsx 

  [1]: /img/bVvKLR
  [2]: /img/bVvLki