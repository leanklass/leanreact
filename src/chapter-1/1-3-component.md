> [书籍完整目录](https://segmentfault.com/a/1190000005136764)
# 1.3 React 组件

![图片描述][1]

## 1.3.1 React 组件介绍

在 React 中组件是第一元素，是 React 的基础，一个 React 应用就是基于 React 组件的组合而成。
前面的 JSX 练习过后，大家应该对 React 组件不陌生了，在这一节我们将温习以及深入学习 React 组件。

## 1.3.2 创建一个 React 组件

创建一个 React 组件的方法为，调用 React.createClass 方法，传入的参数为一个对象，对象必须定义一个 render 方法，render 方法返回值为组件的渲染结构，也可以理解为一个组件实例（React.createElement 工厂方法的返回值），返回值有且只能为一个组件实例，或者返回 null/false，当返回值为 null/false 的时候，React 内部通过 <noscript/> 标签替换。

eg:

```HTML
    var MyComponnet = React.createClass({
        render: function() {
            return <p>....</p>;
        }
    });
```

### 组件命名空间 

可以看出 React.createClass 生成的组件类为一个 Javascript 对象。 当我们想设置命名空间组件时，可以在组件下面添加子组件：

eg:

```HTML
    MyComponent.SubComponent = React.createClass({...});
    MyComponnet.SubComponent.Sub = React.createClass({....});
```

在组件较多的情况下，可以借助命名空间优化组件维护结构以及解决组件名称冲突问题。

### 无状态组件

除了可以通过 React.createClass 来创建组件以外，组件也可以通过一个普通的函数定义，函数的返回值为组件渲染的结果。

eg:

```html
    function StatelessComponent(props) {
        return  <div> Hello {props.name} </div>
    }
```

无状态组件能够优化性能，因为其内部不会维护状态，React 内部不会有一个对应的组件实例，并且也没有生命周期 hook。


## 1.3.3 将组件渲染到 DOM 中

当创建好了组件过后，为了将组件渲染到 DOM 中显示出来，需要两个步骤

1. 在 HTML 中定义一个元素，设置 id 属性
2. JSX 中调用 ReactDOM.render 方法， 第一个参数为 组件，第二个为刚才定义的 DOM 元素

eg:

```html
<!-- 定义的 DOM 元素 -->
<div id="example"></div>
<script type="text/babel">
    // 自定义组件 
    var MyComponnet = React.createClass({
        render: function() {
            return <p>....</p>;
        }
    });
    // 调用 render 方法
    ReactDOM.render(
    <MyComponnet/>,
    document.getElementById('example')
    );
</script>
```

对于组件的渲染，可能涉及到的一些问题：

- **Q1: 只能 render 到一个元素吗？**
- **Q2: 在程序运行时能够动态的调用 render 吗？**
- **Q3: 修改了数据过后，还需要重新调用 render 方法吗？**

这里要先提一下 React 的设计初衷，React 在开发时候的目标就是简单精巧，可以和其他框架结合起来使用。简单而言我们可以当 React 是一个渲染数据对象到 DOM 中的 Javascript 函数工具类，工具类当然可以多次使用。 

那么对于上面的问题：

- **A1:** 不是，React 可以渲染到多个元素，任意位置的元素。
- **A2:** 可以，比如以一个弹出层组件为例，我们希望创建一个 append 到 body 最后的组件来实现全屏遮罩， 那么我们可以动态的创建一个 div append 到 body 最后，然后将弹出层 render 到那个 div 内。
- **A3:** 假设每次数据改变都重新调用 render 方法，那么每次 render 带来的结果是重新创建一个顶级组件实例，以及子组件实例。 如果只调用 render 一次，将数据的变更放在组件内部，那么就不会重复创建顶级组件。


## 1.3.4 组件状态 State

![图片描述][2]

React 中每个组件可以存储自己的当前状态， 以一个 switch 开关组件为例，开关的状态可以存储在组件内部。

React 的渲染结果是由组件属性和状态共同决定的，状态和属性的区别是，状态维护在组件内部，属性是由外部控制，我们先介绍组件状态相关细节：

控制状态的 API 为：

1. this.state：组件的当前状态
2. getInitialState：获取组件的初始状态，在组件加载的时候会被调用一次，返回值赋予 **this.state** 作为初始值
3. this.setState：
    - 组件状态改变时，可以通过 this.setState 修改状态
    - setState 方法支持按需修改，如 state 有两个字段，仅当 setState 传入的对象包含字段 key 才会修改属性
    - 每次调用 setState 会导致重渲染调用 render 方法
    - 直接修改 state 不会重渲染组件

eg:

```html
    var Switch = React.createClass({
        // 定义组件的初始状态，初始为关
        getInitialState: function() {
            return {
                open: false
            }
        },
        // 通过 this.state 获取当前状态
        render: function() {
            console.log('render switch componnet');
            var open = this.state.open;
            return <label className="switch"> 
                        <input type="checkbox" checked={open}/> 
                    </label>
        },
        // 通过 setState 修改组件状态
        // setState 过后会 React 会调用 render 方法重渲染
        toggleSwitch: function() {
            var open = this.state.open;
            this.setState({
                open: !open
            });
        }
    })
```


## 1.3.5 组件属性 Props

前面已经提到过 React 组件可以传递属性给组件，传递方法和 HTML 中无异, 可以通过 this.props 获取组件属性

属性相关的 API 为:

1. this.props: 获取属性值
2. getDefaultProps: 获取默认属性对象，会被调用一次，当组件类创建的时候就会被调用，返回值会被缓存起来，当组件被实例化过后如果传入的属性没有值，会返回默认属性值
4. this.props.children：子节点属性
3. propTypes: 属性类型检查

以一个代办事项的列表项组件为例:

```html
    // props.name 表示代办事项的名称
    var TodoItem = React.createClass({
        render: function() {
            var props = this.props;
            return <div className="todo-item">
                        <span className="todo-item__name">{props.name}</span>
                    </div>
        }
    });

    ReactDOM.render(
        <TodoItem name="代办事项1"/>, 
         document.getElementById('example'));
```

### children 属性

组件属性中会有一个特殊属性 children ，表示子组件， 还是以上面一个组件为例子，我们可以换一种方式定义 name:

```html
    var TodoItem = React.createClass({
        render: function() {
            var props = this.props;
            return <div className="todo-item">
                        <span class="todo-item__name">{props.children}</span>
                    </div>
        }
    });

    ReactDOM.render(
        <TodoItem>代办事项1</TodoItem>, 
         document.getElementById('example')); 
```

> 需要注意的是，children 只能为一个元素，不能为多个组件

### 属性类型检查 

为了保证组件传递属性的正确性， 我们可以通过定义 propsType 对象来实现对组件属性的严格校验：

```html
    var MyComponent = React.createClass({
        propTypes: {
            optionalArray: React.PropTypes.array,
            optionalBool: React.PropTypes.bool,
            optionalFunc: React.PropTypes.func,
            optionalNumber: React.PropTypes.number,
            optionalObject: React.PropTypes.object,
            optionalString: React.PropTypes.string,
            // 任何可以被渲染的包括，数字，字符串，组件，或者数组
            optionalNode: React.PropTypes.node,
            // React 元素
            optionalElement: React.PropTypes.element,
            // 枚举
            optionalEnum: React.PropTypes.oneOf(['News', 'Photos']),
            // 任意一种类型
            optionalUnion: React.PropTypes.oneOfType([
              React.PropTypes.string,
              React.PropTypes.number,
              React.PropTypes.instanceOf(Message)
            ]),
            // 具体类型的数组
            optionalArrayOf: React.PropTypes.arrayOf(React.PropTypes.number),
            // 具体类型的对象
            optionalObjectOf: React.PropTypes.objectOf(React.PropTypes.number),
            // 符合定义的对象
            optionalObjectWithShape: React.PropTypes.shape({
              color: React.PropTypes.string,
              fontSize: React.PropTypes.number
            }),
            requiredFunc: React.PropTypes.func.isRequired,
            requiredAny: React.PropTypes.any.isRequired,
            // 自定义校验
            customProp: function(props, propName, componentName) {}
        }
    });
```

### 属性传递的单向性

我们已经提到过 React 的单向数据流模式，数据的流动管道就是 props，流动的方向就是组件的层级自定向下的方向。所以一个组件是不能修改自身的属性的，组件的属性一定是通过父组件传递而来（或者默认属性）。

### 无状态组件属性

对于无状态组件，可以添加 **.propTypes** 和 **.defaultProps** 属性到函数上。

## 1.3.6 组件的嵌套组合

在 1.2 节的 JSX 实例子中，当我们循环输出 todo 列表的时候，React 会提示对于循环输出的组件，需要有一个唯一的 key 属性。这个问题的原因在于 React 的调和机制（Reconciliation）上。

### 什么叫调和？

在每次数据更新过后，React 会重新调用 render 渲染出新的组件结构，新的结构应用到 DOM 中的过程就叫做调和过程。

### 为什么需要调和？

想一想，假设我们有一个输入组件，这个时候我们正聚焦在输入框中，当修改值过后触发事件导致了数据改变，数据改变导致了重渲染， 这个时候输入框被替换成了新的 DOM。 这个过程对用户来说应该是无感知的，所以那原来的聚焦状态应该被保存， 那怎么做到的呢？ DOM 都被替换了，输入状态，选择状态为什么还能保存。 我们先不急着知道 How，目前只需要知道这就是调和过程，后面我们会在 React 实现原理章节进行详细介绍。 

除了保存状态以外，调和过程还做了很多 DOM 优化。 比如输出一个数组的时候，数据新增加或者减少了一下，或者数组项值改变了，实际上我们没有必要删除原来的 DOM 结构，只需要修改 DOM 的值或者删除 DOM 就能实现重渲染。

这就是为什么要有 key 属性，key 属性能够帮助定位 DOM 与数组元素的关系，在重渲染的时候能够实现渲染优化。 


## 1.3.7 实例练习：通过组件化的方式优化之前的待办事项列表

### 问题

优化 JSX 语法练习的 TODOMVC 页面， 通过组件化的方式拆分页面！

组件如下：

1. App 组件：整个页面的最完成组件
2. Header 组件：头部输入组件
2. TodoList 组件：列表组件
3. TodoItem 组件: 列表项
4. Footer 组件：底部操作组件

### Tips 

循环输出组件的方式

方式一：先计算出组件
```html
 function render() {
    var todos = this.props.todos;
    var $todos = todos.map(function(todo) {
        return <Todo data={todo}/>
    });
    return <div>
        {$todos}
    </div>
 }
```

方式二：{} 内直接计算

```html
 function render() {
    var todos = this.props.todos;
    return <div>
        {todos.map(function(todo) {
            return <Todo data={todo}/>
        })}
    </div>
 }  
```

### 参考答案 
https://github.com/leanklass/leanreact/tree/component

  [1]: /img/bVvLOW
  [2]: /img/bVvL33