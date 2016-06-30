# 4.2 react patterns

- 单向性 
    - 修改 Props 
    - Immutable data representation
- 确定性
    - 在 getInitialState 中使用 props 
    - 私有状态和全局事件 
    - render 包含 side effects
    - jQuery 修改 DOM
    - 使用无状态组件
- 内存管理
    - componentWillUnmount 取消订阅事件
    - 判断 isMounted
- 上层设计
    - 使用 container component 
    - 使用 Composition 替代 mixins
    - Composability - Presenter Pattern
    - Composability - Decorator Pattern
    - Context 传递数据 

## 4.2.1 关于

React 的框架设计是趋于函数式的，其中最主要的两点也是为什么会选择 React 的两点：

1. 单向性：数据的流动是单向的
2. 确定性：React(storeData) = view 相同数据总是渲染出相同的 view 

这两点即是特性也是设计 React 应用的基本原则，围绕这两个原则社区里边出现了一些 React 设计模式，即有好的设计模式也有应该要避免的反模式，理解这些设计模式能够帮助我们写出更优质的 React 应用，本节将围绕 **单向性、确定性、内存管理、上层设计** 来讨论这些设计模式。

> anti 表示反模式，good 表示好模式

## 4.2.2 单向性

**数据的流动是单向的**

### 修改 Props （anti）

**描述：** 组件任何地方修改 props 的值

**解释：** 

React 的数据流动是单向性的，流动的方式是通过 props 传递到组件中，而在 Javascript 中对象是通过引用传递的，修改 props 等于直接修改了 store 中的数据，导致破坏数据的单向流动特性

### 使用不可变数据 （good）

**描述：** store data 使用不可变数据

**解释：** Javascript 对象的特性是可以任意修改，而这个特性很容易破坏数据的单向性，因为人工无法永远确保数据没有被修改过，唯一的做法是使用不可变数据，用代码逻辑确保数据不能被任意修改，后面会有一个完整的小节介绍不可变数据在 React 中的应用

## 4.2.3 确定性

**React(storeData) = view 相同数据总是渲染出相同的 view**

### 在 getInitialState 中使用 props （anti）

**描述：** getInitialState 通过 props 来生成 state 数据

**解释：** 

> 官方文档 https://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html

在 getInitialState 中通过 props 来计算 state 破坏了确定性原则，“source of truth” 应该只是来自于一个地方，通过计算 state 过后增加了 truth source。这种做法的另外一个坏处是在组件更新的时候，还需要计算重新计算这部分 state。

举例：

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

需要注意的是以下这种做法并不会影响确定性

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


### 私有状态和全局事件 （anti）

**描述：** 在组件中定义私有的状态或者使用全局事件 

**介绍：** 组件中定义了私有状态和全局事件过后，组件的渲染可能会出现不一致，因为全局事件和私有状态都可以控制组件的状态，这样外部使用组件无法保证组件的渲染结果，影响了组件的确定性。另外一点是组件应该尽量保证独立性，避免和外部的耦合，使用全局事件造成了和外部事件的耦合。 


### render 函数包含 side effects （anti）

>  side effect 解释： https://en.wikipedia.org/wiki/Side_effect_(computer_science)

**描述：** render 函数包含一些 side effects 的代码逻辑，这些逻辑包括如

1. 修改 state 数据
2. 修改 props 数据
3. 修改全局变量
4. 调用其他导致 side effect 的函数

**解释：** render 函数如果包含了 side effect ，渲染的结果不再可信，所以确保 render 函数为纯函数 


### jQuery 修改 DOM （anti）

**描述：** 使用外部 DOM 框架修改或删除了 DOM 节点、属性、样式
**解释：** React 中 DOM 的结构和属性都是由渲染函数确定的，如果使用了 Jquery 修改 DOM，那么可能造成冲突，视图的修改源头增加，直接影响组件的确定性


### 使用无状态组件 （good）

**描述：** 优先使用无状态组件
**解释：** 无状态组件更符合函数式的特性，如果组件不需要额外的控制，只是渲染结构，那么应该优先选择无状态组件 


## 4.2.4 内存管理


### componentWillUnmount 取消订阅事件 (good)

**描述：** 如果组件需要注册订阅事件，可以在 componentDidMount 中注册，且必须在 ComponentWillUnmount 中取消订阅
**解释：** 在组件 unmount 后如果没有取消订阅事件，订阅事件可能仍然拥有组件实例的引用，这样第一是组件内存无法释放，第二是引起不必要的错误 


### 判断 isMounted （anti）

**描述：** 在组件中使用 isMounted 方法判断组件是否未被注销
**解释：**

React 中在一个组件 ummount 过后使用 setState 会出现warning提示(通常出现在一些事件注册回调函数中) ，避免 warning 的解决办法是：

```html
if(this.isMounted()) { // This is bad.
  this.setState({...});
}
```

但这是个掩耳盗铃的做法，因为如果出现了错误提示就表示在组件 unmount 的时候还有组件的引用，这个时候应该是已经导致了内存溢出。所以解决错误的正确方法是在 componentWillUnmount 函数中取消监听：

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


## 4.2.5 上层设计

### 使用 container component (good)

**描述：** 将 React 组件分为两类 container 、normal ，container 组件负责获取状态数据，然后传递给与之对应的 normal component，对应表示两个组件的名称对应，举例：

```
TodoListContainer => TodoList
FooterContainer => Footer
```

**解释：** 参看 redux 设计中的 container 组件，container 组件是 smart 组件，normal 组件是 dummy 组件，这样的责任分离让 normal 组件更加独立，不需要知道状态数据。明确的职责分配也增加了应用的确定性（明确只有 container 组件能够知道状态数据，且是对应部分的数据）。


### 使用 Composition 替代 mixins （good）

**描述：** 使用组件的组合的方式（高阶组件）替代 mixins 实现为组件增加附加功能 
**解释：** 

mixins 的设计主要目的是给组件提供插件机制，大多数情况使用 mixin 是为了给组件增加额外的状态。但是使用 mixins 会带来一些额外的坏处：

1. mixins 通常需要依赖组件定义特定的方法，如 getSomeMixinState ，而这个是隐式的约束
2. 多个 mixins 可能会导致冲突
3. mixins 通常增加了额外的状态数据，而 react 的设计应该是要避免过多的内部状态
4. mixins 可能会影响 shouldComponentUpdate 的逻辑, mixins 做了很多数据合并的逻辑

另外一点是在新版本的 React 中，mixins 将会是废弃的 feature，在 es6 class 定义组件也不会支持 mixins。

举个例子，一个订阅 fluxstore 的 mixin 为：

```js
function StoreMixin(store) {
  var Mixin = {
    getInitialState() {
      return this.getStateFromStore(this.props);
    },
    componentDidMount() {
      store.addChangeListener(this.handleStoreChanged)
      this.setState(this.getStateFromStore(this.props));
    },
    componentWillUnmount() {
      store.removeChangeListener(this.handleStoreChanged)
    },
    handleStoreChanged() {
      if (this.isMounted()) {
        this.setState(this.getStateFromStore(this.props));
      }
    }
  };
  return Mixin;
}
```

使用

```js
const TodolistContainer = React.createClass({
  mixins: [StoreMixin(AppStore)],
  getStateFromStore(props) {
    return {
      todos: AppStore.get('todos');
    }
  }
})
```

转换为组件的组合方式为：

```js
function connectToStores(Component, store, getStateFromStore) {
  const StoreConnection = React.createClass({
    getInitialState() {
      return getStateFromStore(this.props);
    },
    componentDidMount() {
        store.addChangeListener(this.handleStoreChanged)
    },
    componentWillUnmount() {
        store.removeChangeListener(this.handleStoreChanged)
    },
    handleStoreChanged() {
      if (this.isMounted()) {
        this.setState(getStateFromStore(this.props));
      }
    },
    render() {
      return <Component {...this.props} {...this.state} />;
    }
  });
  return StoreConnection;
};
```

使用方式：

```js
class Todolist extends React.Component {
    render() {
        // ....
    }
}
TodolistContainer = connectToStore(Todolist, AppStore, props => {
    todos: AppStore.get('todos')
})
```


### Presenter Pattern 

**描述：** 利用 children 可以作为函数的特性，将数据获取和数据表现分离成为两个不同的组件

如下例子：

```
class DataGetter extends React.Component {
  render() {
    const { children } = this.props
    const data = [ 1,2,3,4,5 ]
    return children(data)
  }
}

class DataPresenter extends React.Component {
  render() {
    return (
      <DataGetter>
        {data =>
          <ul>
            {data.map((datum) => (
              <li key={datum}>{datum}</li>
            ))}
          </ul>
        }
      </DataGetter>
    )
  }
}

const App = React.createClass({
  render() {
    return (
      <DataPresenter />
    )
  }
})
```

**解释：** 将数据获取和数据展现分离，同时利用组件的 children 可以作为函数的特性，让数据获取和数据展现都可以作为组件使用 

### Decorator Pattern

**描述：** 父组件通过 cloneElement 方法给子组件添加方法和属性

cloneElement 方法：

```js
ReactElement cloneElement(
  ReactElement element,
  [object props],
  [children ...]
)
```

如下例子：

```
const CleverParent = React.createClass({
  render() {
    const children = React.Children.map(this.props.children, (child) => {
      return React.cloneElement(child, {
        // 新增 onClick 属性
        onClick: () => alert(JSON.stringify(child.props, 0, 2))
      })
    })
    return <div>{children}</div>
  }
})

const SimpleChild = React.createClass({
  render() {
    return (
      <div onClick={this.props.onClick}>
        {this.props.children}
      </div>
    )
  }
})

const App = React.createClass({
  render() {
    return (
      <CleverParent>
        <SimpleChild>1</SimpleChild>
        <SimpleChild>2</SimpleChild>
      </CleverParent>
    )
  }
})
```

**解释：** 通过这种设计模式，可以应用到一些自定义的组件设计，提供更简洁的 API 给第三方使用，如 facebook 的 FixedDataTable 也是应用了这种设计模式


### Context 传递数据

**描述：** Context 是 React 文档中没有提到，但是相信大多数已经用到的功能，其目的是让所有组件共享相同的上下文，避免数据的逐级传递。 Context 是大多数 flux 库共享 store 的基本方法。

使用方法：

```js

/**
 * 初始化定义 Context 的组件
 */
class Chan extends React.Component {
  getChildContext() {
    return {
      environment: "grandma's house"
    }
  }
}

// 设置 context 类型
Chan.childContextTypes = {
  environment: React.PropTypes.string
};

/**
 * 子组件获取 context 
 */
class ChildChan extends React.Component {
  render() {
    const ev = this.context.environment;
  }
}
/**
 * 需要设置 contextTypes 才能获取
 */
ChildChan.contextTypes = {
  environment: React.PropTypes.string
};
```


**解释：** 通常情况下 Context 是为基础组件提供的功能，一般情况应该避免使用，否则滥用 Context 会影响应用的确定性。


## 参考链接

- https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.hvbsii4zd
- http://www.zhubert.com/blog/2016/02/05/react-composability-patterns/
- https://medium.com/@learnreact/context-f932a9abab0e#.wn00ktlde
