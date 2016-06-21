# [anti-pattern in  React]()

## Mutating Props is Bad

简单而言就是在组件中修改 props 的值，因为 React 是单向数据流的模式，在 js 中对象是通过引用传递的，修改 props 会破坏单向性和确定性 (所以可以用 Immutable.Js 来避免这种错误)

## 在 getInitialState 中使用 props 

> https://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html

在 getInitialState 中通过 props 来计算 state，anti 的原因是因为破坏了确定性原则，“source of truth” 应该只是来自于 一个地方，通过计算 state 过后增加了 source，影响编码逻辑的地方举例在组件更新props的时候，还需要计算重新计算 state

反模式例子：

```html
var MessageBox = React.createClass({
  getInitialState: function() {
    return {nameWithQualifier: 'Mr. ' + this.props.name};
  },

  render: function() {
    return <div>{this.state.nameWithQualifier}</div>;
  }
});

ReactDOM.render(<MessageBox name="Rogers"/>, mountNode);
```

优化方式：

```html
var MessageBox = React.createClass({
  render: function() {
    return <div>{'Mr. ' + this.props.name}</div>;
  }
});

ReactDOM.render(<MessageBox name="Rogers"/>, mountNode);
```

一下不是反模式

```
var Counter = React.createClass({
  getInitialState: function() {
    // naming it initialX clearly indicates that the only purpose
    // of the passed down prop is to initialize something internally
    return {count: this.props.initialCount};
  },

  handleClick: function() {
    this.setState({count: this.state.count + 1});
  },

  render: function() {
    return <div onClick={this.handleClick}>{this.state.count}</div>;
  }
});

ReactDOM.render(<Counter initialCount={7}/>, mountNode);
```


## 判断 isMounted 

在 React 中在一个组件 ummount 过后使用 setState 会出现warning提示(通常出现在一些事件注册回调函数中) ，为了避免 warning 有些人的做法是：

```html
if(this.isMounted()) { // This is bad.
  this.setState({...});
}
```

但是事实上这个是掩耳盗铃的做法，因为如果出现了错误提示就表示在组件 unmount 的时候还有组件的引用，这个时候应该是已经导致了 memory-leak 。所以解决错误的正确方法是在 componentWillUnmount 函数中取消监听：

```
class MyComponent extends React.Component {
  componentDidMount() {
    mydatastore.subscribe(this);
  }
  render() {
    ...
  }
  componentWillUnmount() {
    mydatastore.unsubscribe(this);
  }
}
```


社区里边的一些反模式:

1. 使用 mixins （Composition instead of mixins）https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.le8yopk2n
2. 在 componentWillMount 使用 setState
3. mutate dom explicitly via Jquery (和 React 的dom冲突 )
4. render with side effects （破坏一致性原则，尽量保证纯函数）
5. private states and global events （global Event 增加组件的耦合, private state 影响组件的确定性）

https://github.com/planningcenter/react-patterns 这个 repo 既介绍了一些好的pattern也介绍了一下 anti-pattern:

1. 在 jsx render 中使用复杂条件判断 -> compound-conditions   
2. 没把 render 函数中的计算剥离出来 -> Cached State in render 
3. 在 render 函数中判断是否存在某个属性 -> Existence Checking
