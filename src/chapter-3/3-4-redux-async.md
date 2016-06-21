# redux 异步  

在大多数的前端业务场景中，需要和后端产生异步交互，在本节中，将详细讲解 redux 中的异步方案以及一些异步第三方组件，内容有：

- redux 异步流
- redux-thunk 
- redux-saga 
- 规范异步 action 


## redux 异步流 

在前面讲解的 redux 中的数据流都是同步的，如下：

`view -> actionCreator -> action -> reducer -> newState -> container component` 

但同步数据不能满足真实业务开发，真实业务大多都是异步的，上面的流程需要改为：

`view -> actionCreator -> action -> wait -> reducer -> newState -> container component`

幸运的是在 redux 中通过 middleware 机制可以很容易的实现异步数据流。

### 那具体怎样做到？

我们已经很清楚一个 middleware 的结构 ，其核心的部分为 

```js
function(action) {
    // 调用后面的 middleware
    next(action)
}
```

在 middleware 中完全掌控了 reducer 的触发时机， 也就是 action 到了这里完全由中间件控制，不乐意就不给其他中间件处理的机会，而且还可以控制调用其他中间件的时机。 

举例来说一个异步的 ajax 请求场景，可以如下实现：

```js
function(action) {
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

其实目前已经有很多第三方 redux 组件支持异步 action，其中如：

- [redux-thunk](https://github.com/gaearon/redux-thunk)
- [redux-promise](https://github.com/acdlite/redux-promise)
- [redux-saga](http://yelouafi.github.io/redux-saga/)

这些组件都有很好的扩展性，完全能满足我们开发异步流程的场景


## redux-thunk

redux-thunk 是 redux 官方文档中用到的异步组件


## 规范异步 action

http://redux.js.org/docs/advanced/AsyncActions.html