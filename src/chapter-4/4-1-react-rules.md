# react 代码规范

- 关于
- 组件定义
- render jsx 
- 命名

## 关于

在代码的设计上，每个团队可能都有一定的代码规范和模式，好的代码规范能够提高代码的可读性便于协作沟通，好的模式能够上层设计上避免不必要的 bug 出现。本节会参考社区提供一些 React 的规范和优秀的设计模式。


## 基础规范

1. 统一全部采用 Es6
2. 组件文件名称采用大驼峰命名

## 组件结构定义

**总体规则：** stateless(Function) 优先于 Es6 Class 优先于 React.createClass；
**书写规则：** 规范组件内部方法的定义顺序

- Ordering for `class extends React.Component`:

1. optional `static` methods
1. `constructor`
1. `getChildContext`
1. `componentWillMount`
1. `componentDidMount`
1. `componentWillReceiveProps`
1. `shouldComponentUpdate`
1. `componentWillUpdate`
1. `componentDidUpdate`
1. `componentWillUnmount`
1. *clickHandlers or eventHandlers* like `onClickSubmit()` or `onChangeDescription()`
1. *getter methods for `render`* like `getSelectReason()` or `getFooterContent()`
1. *Optional render methods* like `renderNavigation()` or `renderProfilePicture()`
1. `render`

以 Es6 Class 定义的组件为例；

```js
const defaultProps = {
  name: 'Guest'
};
const propTypes = {
  name: React.PropTypes.string
};
class Person extends React.Component {

  // 构造函数
  constructor (props) {
    super(props);
    // 定义 state
    this.state = { smiling: false };
    // 定义 eventHandler
    this.handleClick = this.handleClick.bind(this);
  }

  // 生命周期方法
  componentWillMount () {},
  componentDidMount () {},
  componentWillUnmount () {},
  

  // getters and setters
  get attr() {}

  // handlers
  handleClick() {},

  // render
  renderChild() {},
  render () {},

}

/**
 * 类变量定义
 */
Person.defaultProps = defaultProps;

/**
 * 统一都要定义 propTypes
 * @type {Object}
 */
Person.propTypes = propTypes;
```

## 命名规范 

- 组件名称：大驼峰
- 属性名称：小驼峰
- 事件处理函数：`handleSomething` 
- 自定义事件属性名称：`onSomething={this.handleSomething}`
- key: 不能使用数组 index ，构造或使用唯一的 id
- 组件方法名称：避免使用下划线开头的命名

## jsx 书写规范 

- 自闭合
```
// bad
<Foo className="stuff"></Foo>

// good
<Foo className="stuff" />
```

- 属性对齐
```
// bad
<Foo superLongParam="bar"
     anotherSuperLongParam="baz" />

// good
<Foo
    superLongParam="bar"
    anotherSuperLongParam="baz"
/>

// if props fit in one line then keep it on the same line
<Foo bar="bar" />
```

- 返回 
```
// bad
render() {
  return <MyComponent className="long body" foo="bar">
           <MyChild />
         </MyComponent>;
}

// good
render() {
  return (
    <MyComponent className="long body" foo="bar">
      <MyChild />
    </MyComponent>
  );
}

// good, when single line
render() {
  const body = <div>hello</div>;
  return <MyComponent>{body}</MyComponent>;
}
```

## eslint-react 

规范可以使用 eslint-react 插件来强制实施

> 更多 react 代码规范可参考 https://github.com/airbnb/javascript/tree/master/react

