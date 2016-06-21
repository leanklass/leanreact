# redux 异步  

在大多数的前端业务场景中，需要和后端产生异步交互，在本节中，将详细讲解 redux 中的异步方案以及一些异步第三方组件，内容有：

- redux 异步流
- redux-thunk 
- redux-promise
- redux-saga 
- 规范异步 action 


## redux 异步流 

在前面讲解的 redux 中的数据流都是同步的，如下：

`view -> actionCreator -> action -> reducer -> newState -> container component` 
但同步数据不能满足真实业务开发，真实业务大多都是异步的，

## 实现异步的方式 

本质上 redux 没有提及异步相关的概念，可以用任何原来实现异步的方式，最简单的方式就是延迟 dispatch , 如下例子中一个组件内部的代码：

```js
this.dispatch({ type: 'SYNC_SOME_ACTION'})
window.setTimeout(() => {
  this.dispatch({ type: 'ASYNC_SOME_ACTION' })
}, 1000)
```

这种方式最简单直接，但是有如下问题：

1. 如果有多个类似的 action 触发场景，异步逻辑不能重用
2. 在 javascript 性能优化中有节流的概念，也就是为了提高性能，避免重复调用，如果同时触发了 action ，那么没有一个地方能统一控制

解决办法很简单，实际上就是把异步的代码剥离出来：

someAction.js
```js
function dispatchSomeAction(dispatch, payload) {
    // ..调用控制逻辑...
    dispatch({ type: 'SYNC_SOME_ACTION'})
    window.setTimeout(() => {
      dispatch({ type: 'ASYNC_SOME_ACTION' })
    }, 1000)
}
```

实际上 dispatchSomeAction 和 actionCreator 是十分类似的

基于这种方式上面的流程就改为了：

`view -> actionCreator -> wait -> action -> reducer -> newState -> container component`

但是上面的方法有一些缺点

1. 第一：同步调用和异步调用的方式不相同，不能统一为 store.dispatch 
2. 第二：每次都需要传递 dispatch 方法

幸运的是在 redux 中通过 middleware 机制可以很容易的解决上面的问题

### 通过 middleware 实现异步

我们已经很清楚一个 middleware 的结构 ，其核心的部分为 

```js
function(action) {
    // 调用后面的 middleware
    next(action)
}
```

middleware 完全掌控了 reducer 的触发时机， 也就是 action 到了这里完全由中间件控制，不乐意就不给其他中间件处理的机会，而且还可以控制调用其他中间件的时机。 

举例来说一个异步的 ajax 请求场景，可以如下实现：

```js
function (action) {
    // async call 
    ajax({
        url: ....
        success: function(ret) {
            newAction = createNewAction(ret, action)
            next(newAction)
        }
    });
}
```

任何异步的 javascript 逻辑都可以，如： callback, Promise, setTimeout 等等。

### 第三方异步组件

上面的实现方案只是针对具体的场景设计的，那如果是如何解决通用场景下的问题呢，其实目前已经有很多第三方 redux 组件支持异步 action，其中如：

- [redux-thunk](https://github.com/gaearon/redux-thunk)
- [redux-promise](https://github.com/acdlite/redux-promise)
- [redux-saga](http://yelouafi.github.io/redux-saga/)

这些组件都有很好的扩展性，完全能满足我们开发异步流程的场景，下面来一一介绍

## redux-thunk

### redux-thunk 介绍

redux-thunk 是 redux 官方文档中用到的异步组件，实质就是一个 redux 中间件，thunk 听起来是一个很陌生的词语，先来认识一下什么叫 thunk

> A thunk is a function that wraps an expression to delay its evaluation.

简单来说一个 thunk 就是一个封装表达式的函数，封装过后可以延迟执行表达式

```js
// 1 + 2 立即被计算 = 3
let x = 1 + 2;

// 1 + 2 被封装在了 foo 函数内
// foo 可以被延迟执行
// foo 就是一个 thunk 
let foo = () => 1 + 2;
```

redux-thunk 是一个通用的解决方案，其核心思想是让 action 可以变为一个 thunk ，简单来讲就是 middleware 中判断 action 为一个函数，就做特殊处理，函数的参数为 dispatch。

原本的 dispatch 代码为：

```js
this.dispatch({type: 'SOME_ACTION'})
```

现在变为了：

```js
this.dispatch(function (dispatch){
    setTimeout(() => {
       dispatch({type: 'THUNK_ACTION'}) 
    }, 1000)
})
```

又因为为之间已经讲过，这样的设计会导致异步逻辑放在了组件中，解决办法就是通过 actionCreator 或者叫 thunkActionCreator:

```js
//actions/someThunkAction.js
export function createThunkAction(payload) {
    return function(dispatch) {
        setTimeout(() => {
           dispatch({type: 'THUNK_ACTION', payload: payload}) 
        }, 1000)
    }
}

// someComponent.js
this.dispatch(createThunkAction(payload))
```

### 安装

```shell
$ npm install redux-thunk
```

### 使用

第一步: 添加 thunk 中间件
```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```

第二步：实现一个 thunk action creator

```js
//actions/someThunkAction.js
export function createThunkAction(payload) {
    return function(dispatch) {
        setTimeout(() => {
           dispatch({type: 'THUNK_ACTION', payload: payload}) 
        }, 1000)
    }
}
```

第三步：组件中 dispatch thunk

```js
this.dispatch(createThunkAction(payload));
```

### thunk 源码 

说了这么多，redux-thunk 是不是做了很多工作，实现起来很复杂，那我们来看看 thunk 中间件的实现

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

这么简单，不是在忽悠吧，没忽悠，真只有 14 行，但是这简短的实现却能让异步流程变得极其简单，怎么做到的，我们来分析一下：

1. 判断如果是 function 那么执行 action(dispatch, getState, ...)
    1. action 也就是一个 thunk 
    2. 执行 action 相当于执行了异步逻辑
    2. 把执行的结果作为返回值直接返回 
    3. 直接返回并没有调用其他中间件，也就意味着中间件的执行在这里停止了
    4. 可以对返回值做处理（后面会讲如果返回值是 Promise 的情况）
2. 如果不是函数直接调用其他中间件并返回

理解了这个过后是不是对 redux-thunk 的使用思路变得清晰了

### thunk 的组合 

根据 redux-thunk 的特性，可以做出很有意思的事情，利用返回值为 Promise 可以实现多个 thunk 的组合编排。

编排例子：
```js
function thunkC() {
    return function(dispatch) {
        dispatch(thunkB())
    }
}
function thunkB() {
    return function (dispatch) {
        dispatch(thunkA())
    }
}
function thunkA() {
    return function (dispatch) {
        dispatch({type: 'THUNK_ACTION'})
    }
}
```

Promise 例子

```js
function ajaxCall() {
    return fetch(...);
}

function thunkC() {
    return function(dispatch) {
        dispatch(thunkB(...))
        .then(
            data => dispatch(thunkA(data)),
            err  => dispatch(thunkA(err))
        )
    }
}
function thunkB() {
    return function (dispatch) {
        return ajaxCall(...)
    }
}

function thunkA() {
    return function (dispatch) {
        dispatch({type: 'THUNK_ACTION'})
    }
}
```

## redux-promise

另外一个 redux 文档中提到的异步组件为 redux-promise, 我们直接分析一下其源码

```js
import { isFSA } from 'flux-standard-action';

function isPromise(val) {
  return val && typeof val.then === 'function';
}

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action)
        ? action.then(dispatch)
        : next(action);
    }

    return isPromise(action.payload)
      ? action.payload.then(
          result => dispatch({ ...action, payload: result }),
          error => {
            dispatch({ ...action, payload: error, error: true });
            return Promise.reject(error);
          }
        )
      : next(action);
  };
}
```

大概的逻辑就是：

1. 如果不是标准的 flux action，那么判断是否是 promise, 是执行 action.then(dispatch)，否执行 next(action) 
2. 如果是标准的 flux action, 判断 payload 是否是 promise，是的话获取 payload 数据，获取数据过后再次 dispatch(action) , 否执行 next(action) 


结合 redux-promise 可以利用 es7 的 async 和 await 语法，简化异步的 promiseActionCreator 的设计， eg:

```js
export default async (payload) => {
  const result = await somePromise;
  return {
    type: "PROMISE_ACTION",
    payload: result.someValue;
  }
}
```

如果可以对 async 不是很熟悉可以简单看一下

async 关键字可以总是返回一个 Promise 的 resolve 结果或者 reject 结果
```js
async function foo() {
    if(true)
        return 'Success!';
    else
        throw 'Failure!';
}

// 等价于下面
 
function foo() {
    if(true)
        return Promise.resolve('Success!');
    else
        return Promise.reject('Failure!');
}

```

在 async 关键字中可以使用 await 关键字，其目的是 await 一个 promise， 等待 promise resolve 和 reject 

eg：

```js
async function foo(aPromise) {
    const a = await new Promise(function(resolve, reject) {
            // This is only an example to create asynchronism
            window.setTimeout(
                function() {
                    resolve({a: 12});
                }, 1000);
        })
    console.log(a.a)
    return  a.a
}

// in console
> foo() 
> Promise {_c: Array[0], _a: undefined, _s: 0, _d: false, _v: undefined…}
> 12
```

可以看到在控制台中，先返回了一个 promise，然后输出了 12


## redux-saga

@todo



http://redux.js.org/docs/advanced/AsyncActions.html