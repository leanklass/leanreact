> [书籍完整目录](https://segmentfault.com/a/1190000005136764)

# 3.1 开始使用 redux 


![clipboard.png](/img/bVx39E)


前面我们介绍了 flux 架构以及其开源实现 redux，在这一节中，我们将完整的介绍 redux:

- redux 介绍
    - redux 是什么
    - redux 概念
    - redux 三原则
- redux Stores
- redux Action
- redux Reducers
- redux 数据流动


## 3.1.1 redux 介绍

### redux 是什么

> Redux is a predictable state container for JavaScript apps
> 译：Redux 是为 Javascript 应用而生的可预估的状态容器

定义有些抽象，简单来讲 redux 可以理解为基于 flux 和其他一些思想（Elm，函数式编程）发展出来的前端应用架构库，作为一个前端数据状态容器体现，并可以在 React 和其他任何前端框架中使用。

### redux 概念

- **store**: 同 flux，应用数据的存储中心
- **action**: 同 flux ，应用数据的改变的描述
- **reducer**: 决定应用数据新状态的函数，接收应用之前的状态和一个 action 返回数据的新状态。 
- **middleware**: redux 提供中间件的方式，完成一些 flux 流程的自定义控制，同时形成其插件体系

### redux 三原则 

**单一的 store**

区别于 flux 模式中可以有多个 state，整个应用的数据以树状结构存放在一个 state 对象中。

```js
console.log(store.getState())
/* Prints
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
*/
```

**state 只读**

state 包含的应用数据不能随意修改，修改数据的唯一方式是 dispatch action，action 描述要修改的信息(这和 flux 架构上是一致的，不过在设计上更加严格，见后面的 reducer 设计)。

```js
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
})

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
})
```

**数据的改变由纯函数生成**

在 redux 中，应用数据的改变路径为:

1. store.dispatch(action) 
2. newState = reducer(previousState, action)
3. reducer 为纯函数

纯函数是函数是编程的思想，只要参数相同，函数返回的结果永远是一致的，并且不会对外部有任何影响（不会改变参数对象的数据）。也就是说 reducer 每次必须返回一个新状态，新状态由旧状态和 action 数据决定。

## 3.1.2 安装 redux

安装 redux 核心和 react-redux 集成进 react

```shell
$ npm install --save redux
$ npm install --save react-redux
```

## 3.1.3 redux store

在 redux 中 store 作为一个单例，存放了应用的所有数据，对外提供了如下的 API：

1. 获取数据 `store.getState()`
2. 通过触发 action 来更新数据 `store.dispatch(action)`
3. pubsub 模式提供消息订阅和取消 `store.subscribe(listener)` 

### 创建并初始化 store 

redux 提供 `createStore` 方法来创建一个 store  

```js
/**
 * [createStore description]
 * @param  {[type]} reducer      [reducer 函数]
 * @param  {[type]} initialState [应用初始化状态]
 * @return {[type]}              [description]
 */
function createStore(reducer, initialState) {
    // ....
}
```

创建一个 store ：

```js
var redux = require('redux');
var todoAppReducer = require('./reducers');
var initalState = {
    todos: []
};
var store = redux.createStore(todoAppReducer, initialState);
```

### 触发 action 

redux 修改应用数据的唯一方式为 `store.dispatch(action)`

eg:

```js
store.dispatch({
  type: 'ADD_TODO',
  title: 'new todo'
})
```

### 消息订阅和取消

为了让用户监听应用数据改变，redux 在 store 集成了 pubsub 模式

**订阅**

```js
// 每次应用数据改变，输出最新的应用数据
store.subscribe(function(){
  console.log(store.getState())
})

// 如果新增了 todo 会触发上面的订阅函数
store.dispatch({
  type: 'ADD_TODO',
  title: 'new todo'
})
```

**取消**

subscribe 返回的函数即为取消函数

```js
var unsubscribe = store.subscribe(function(){
  console.log(store.getState())
})

// ....

unsubscribe();
```

### 设计应用数据结构
 
所有数据都存放在一个 store 中，以 todoApp 为例子，state 的数据结构可能为

```js
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

当应用变大的时候，数据结构可能没有这么简单，这时候需要找到一个较好的结构来设计应用数据，下面是两个 redux 在设计 state 上的 tip:

1. **业务数据和 UI 状态数据分离**，尽量避免 UI 状态数据放在 store 中，即便放在 store 中也好和业务数据分离。
2. **避免嵌套**，在一个复杂的场景，数据对象可能很深，出现多层，那在设计的时候可以通过 id 的方式来引用，可以参考 [normalizr](https://github.com/paularmstrong/normalizr) 的简化方式 


## 3.1.4 redux action

我们已经知道 action 就是数据修改的描述信息，不过在实际使用过程中需要理解下面的这些规范：

0. action 描述数据结构
1. action 类型常量 
2. action creator 

### action 描述数据结构

redux 对 action 对象的数据结构做了简单规范，每个对象必须包含一个字段 **type**，应用通过 type 来识别 action 类型，其他字段不做任何限制。

eg：
```js
{
  type: "ADD_TODO",
  text: 'Build my first Redux app'
}

{
  type: "REMVOE_TODO",
  index: 1
}

{
  type: "TOGGLE_TODO",
  id: 'a1s2d1'
}
```


### action 类型常量

为了项目的规范，通常把 action type 定义为名称常量，放在一个单独的文件中管理，这在大型项目中是一个很好的习惯。

eg:

```
var ADD_TODO = "ADD_TODO"

{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```


### action creator

在 flux 模式小节已经介绍过，为了规范化 action 通过 action creator 来创建

```js
function addTodo(text) {
  return {
    type: ADD_TODO,
    text: text
  }
}
```

原来的 dispatch 的使用改为了

```js
// 旧的方式
store.dispatch({
    type: ADD_TODO,
    text: text
})

// 新的方式
store.dispatch(addTodo(text))
```


## 3.1.5 redux reducer

reducer 应该是最为陌生的概念，理解 reducer 是理解 redux 的关键，牢记 **reducer 决定应用数据的改变** 

### reducer 基础

```js
/**
 * [reducer description]
 * @param  {[type]} previewsState [之前状态]
 * @param  {[type]} action        [redux action]
 * @return {[type]} newState      [新状态]
 */
function reducer(previewsState, action) {
    var newState;
    switch(action.type) {
        case ..
        case ..
        default:
            newState = previewsState
    }
    return newState;
}
```

首先 reducer 是一个纯函数，接收到之前的应用状态和 action 并返回一个新状态，为了每次返回一个新状态，可以通过 Object.assign() 方法返回一个新的对象，也可以使用 Immutable.js （后面的章节会讲解 Immutable.js)。

```js
function todoApp(state, action) {
    state = initialState
    switch (action.type) {
        case SET_VISIBILITY_FILTER:
          return Object.assign({}, state, {
            visibilityFilter: action.filter
          })
        default:
          return state
    }
}
```

redux 会在初始的时候调用一次 reducer （这时参数 previewsState 为 undefined）， 可以借用这个机会返回应用初始状态数据。

eg:

```js
// reducers/todoApp.js

var initialState = {todos: ...};

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }
  // 返回默认值
  return state
}

module.exports = todoApp;
```


总结需要注意的点：

1. 纯函数特性，不能修改 previewsState，不能调用异步 API，无论什么时候，传入相同的参数都能立即返回相同的结果（不能调用 Math.random, Data.now 这些函数，因为会导致不同的结果）
2. 默认返回 previewsState (在 action 不会得到处理的情况)
3. 处理 state 为 undefined 的情况

### reducer 组合 

一个 todo 应用可能有很多操作，在真实场景上面的 todoApp reducer 可能膨胀为如下的结构：

```js
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
          if(index === action.index) {
            return Object.assign({}, todo, {
              completed: !todo.completed
            })
          }
          return todo
        })
      })
    default:
      return state
  }
}
```

这时候可以对 todoApp reducer 做拆分，将它拆分为多个不同的 reducer，todoApp reducer 的作用只是组合其他 reducer 。

拆分后可以如下：

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}

module.exports = todoApp
```

todoApp 不再负责整个应用状态的修改，而是将责任分配给了其他的 reducer， 每个子 reducer 相互独立，负责各自的责任，这个时候应用数据的改变不是靠一个 reducer 改变的，而是由多个 reducer 组合起来，这是 redux 应用的基础，叫做 **reducer 组合**。 

可以发现 reducer 的组合和状态数据的结构是相同的，都可以理解为树状结构。 state 根对象有一个根 reducer （createState 传入的 reducer）这里是 todoApp , state 对象下面的第一级数据对应不用的 reducer，visibilityFilter 和 todos , 理解起来这也是一个典型的 **分而治之策略**。

对于 reducer 的组合，redux 提供了 `combineReducers()` 方法来简化，上面的 todoApp 可以简化为：

```js
var todoApp = redux.combineReducers({
  visibilityFilter: visibilityFilter,
  todos: todos
})
```

这样的写法和上面的写法作用完全相同


## 3.1.5 redux 数据流动

redux 继承 flux 最核心的地方是 **数据的单向流动**，
上面已经详细介绍了 redux 的各个概念，下面将这些概念结合起来，看看数据怎么在 redux 中流动的。

**第一步：调用 `store.dispatch(action)`**

可以在任何地方触发 dispatch，例如 UI 交互，API 调用

**第二步： Redux store 调用 rootReducer**

redux 收到 action 过后，调用根 reducer 并返回最新的状态数据。（根 reducer 内部组合其他 reducer 返回部分的最新状态）

**第三步：接收新状态并 publish 给订阅者**

当 rootReducer 返回最新的状态后，通知订阅函数 `store.subscribe(listener)` 。在 React 中，可以订阅状态更新，在订阅函数中获取最新的状态过后，修改根组件的数据:

```js
component.setState(newState)
```
