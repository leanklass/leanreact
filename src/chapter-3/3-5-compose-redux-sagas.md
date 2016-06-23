> [书籍完整目录](https://segmentfault.com/a/1190000005136764)

# 3.5 compose redux sages


![clipboard.png](/img/bVyoVa)


基于 redux-thunk 的实现特性，可以做到基于 promise 和递归的组合编排，而 redux-saga 提供了更容易的更高级的组合编排方式（当然这一切要归功于 Generator 特性），这一节的主要内容为：

1. 基于 take Effect 实现更自由的任务编排
2. fork 和 cancel 实现非阻塞任务
3. Parallel 和 Race 任务 
4. saga 组合 yield* saga
5. channels

## 3.5.1 基于 take Effect 实现更自由的任务编排

前面我们使用过 takeEvery helper, 其实底层是通过 take effect 来实现的。通过 take effect 可以实现很多有趣的简洁的控制。

如果用 takeEvery 实现日志打印，我们可以用：

```js
import { takeEvery } from 'redux-saga'
import { put, select } from 'redux-saga/effects'

function* watchAndLog() {
  yield* takeEvery('*', function* logger(action) {
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  })
}
```

使用使用 take 过后可以改为：

```js
import { take } from 'redux-saga/effects'
import { put, select } from 'redux-saga/effects'

function* watchAndLog() {
  while (true) {
    const action = yield take('*')
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  }
}
```

while(true) 的执行并非是死循环，而只是不断的生成迭代项而已，take Effect 在没有获取到对象的 action 时，会停止执行，直到接收到 action 才会执行后面的代码，然后重新等待

take 和 takeEvery 最大的区别在于 take 是主动获取 action ，相当于 action = getNextAction() , 而 takeEvery 是消息推送。 

基于主动获取的，可以做到更自由的控制，如下面的两个例子：

完成了三个任务后，提示恭喜

```js
import { take, put } from 'redux-saga/effects'
function* watchFirstThreeTodosCreation() {
  for (let i = 0; i < 3; i++) {
    const action = yield take('TODO_CREATED')
  }
  yield put({type: 'SHOW_CONGRATULATION'})
}
```

登录和登出逻辑可以放在同一个函数内共享变量

```js
function* loginFlow() {
  while (true) {
    yield take('LOGIN')
    // ... perform the login logic
    yield take('LOGOUT')
    // ... perform the logout logic
  }
}
```

take 最不可思议的地方就是，将 **异步的任务用同步的方式来编排** ，使用好 take 能极大的简化交互逻辑处理


## 3.5.2 fork 和 cancel 实现非阻塞任务

在提非阻塞之前肯定要先要说明什么叫阻塞的代码。我们看一下下面的例子：

```js
function* generatorFunction() {
  console.log('start')

  yield take('action1')
  console.log('take action1')

  yield call('api')
  console.log('call api')

  yield put({type: 'SOME_ACTION'})
  console.log('put blabla')
}
```

因为 generator 的特性，必须要等到 take 完成才会输出 take action1, 同理必须要等待 call api 完成才会输出 call api， 这就是我们所说的阻塞。

那阻塞会造成什么问题呢？见下面的例子：

一个登录的例子（这是一段有问题的代码，可以先研究一下这段代码问题出在哪儿）

```js
import { take, call, put } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
    return token
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  }
}

function* loginFlow() {
  while (true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    const token = yield call(authorize, user, password)
    if (token) {
      yield call(Api.storeItem, {token})
      yield take('LOGOUT')
      yield call(Api.clearItem, 'token')
    }
  }
}
```

我们先来分析一下 loginFlow 的流程：

0. 通过 take effect 监听 login_request action 
1. 通过 call effect 来异步获取 token （call 不仅可以用来调用返回 Promise 的函数，也可以用它来调用其他 Generator 函数，返回结果为调用 Generator return 值） 
2. 成功（有 token）
  1 过后异步存储 token 
  2. 等待 logout action
  3. logout 事件触发后异步清除 token 
  4. 然后回到第 0 步
3. 失败(token === undefined) 回到第 0 步

其中的问题：

一个隐藏的陷阱，在调用 authorize 的时候，如果用户点击了页面中的 logout 按钮将会没有反应（此时还没有执行 take('LOGOUT')） , 也就是被 authorize 阻塞了。

redux-sage 提供了一个叫 `fork` 的 Effect，可以实现非阻塞的方式，下面我们重新设计上面的登录例子：

```js
function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  }
}

function* loginFlow() {
  while(true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    yield fork(authorize, user, password)
    yield take(['LOGOUT', 'LOGIN_ERROR'])
    yield call(Api.clearItem('token'))
  }
}
```

1. token 的获取放在了 authorize saga 中，因为 fork 是非阻塞的，不会返回值 
2. authorize 中的 call 和 loginFlow 中的 take 并行调用 
3. 这里 take 了两个 action , take 可以监听并发的 action ，不管哪个 action 触发都会执行 call(Api.clearItem...) 并回到 while 开始 
  1. 在用户触发 logout 之前, 如果 authorize 成功，那么 loginFlow 会等待 LOGOUT action 
  2. 在用户触发 logout 之前, 如果 authorize 失败，那么 loginFlow 会 take('LOGIN_ERROR') 
4. 如果在用户触发 logout 的时候，authorize 还没有执行完成，那么会执行后面的语句并回到 while 开始 

这个过程中的问题是如果用户触发 logout 了，没法停止 call api.authorize , 并会触发 LOGIN_SUCCESS 或者 LOGIN_ERROR action 。

redux-saga 提供了 cancel Effect，可以 cancel 一个 fork task 

```js
import { take, put, call, fork, cancel } from 'redux-saga/effects'

// ...

function* loginFlow() {
  while (true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    // fork return a Task object
    const task = yield fork(authorize, user, password)
    const action = yield take(['LOGOUT', 'LOGIN_ERROR'])
    if (action.type === 'LOGOUT')
      yield cancel(task)
    yield call(Api.clearItem, 'token')
  }
}
```

cancel 的了某个 generator, generator 内部会 throw 一个错误方便捕获，generator 内部 可以针对不同的错误做不同的处理

```js
import { isCancelError } from 'redux-saga'
import { take, call, put } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
    return token
  } catch(error) {
    if(!isCancelError(error))
      yield put({type: 'LOGIN_ERROR', error})
  }
}
```


##  3.5.3 Parallel 和 Race 任务 

### Parallel
基于 generator 的特性，下面的代码会按照顺序执行

```js
const users  = yield call(fetch, '/users'),
      repos = yield call(fetch, '/repos')
```

为了优化效率，可以让两个任务并行执行

```js
const [users, repos]  = yield [
  call(fetch, '/users'),
  call(fetch, '/repos')
]
```

### Race 

某些情况下可能会对优先完成的任务进行处理，一个很常见的例子就是超时处理，当请求一个 API 超过多少时间过后执行特定的任务。

eg:

```js
import { race, take, put } from 'redux-saga/effects'
import { delay } from 'redux-saga'

function* fetchPostsWithTimeout() {
  const {posts, timeout} = yield race({
    posts: call(fetchApi, '/posts'),
    timeout: call(delay, 1000)
  })

  if (posts)
    put({type: 'POSTS_RECEIVED', posts})
  else
    put({type: 'TIMEOUT_ERROR'})
}
```

这里默认使用到了 race 的一个特性，如果某一个任务成功了过后，其他任务都会被 cancel 。

## 3.5.4 yield* 组合 saga

yield* 是 generator 的内关键字，使用的场景是 yield 一个 generaor。

yield* someGenerator 相当于把 someGenerator 的代码放在当前函数执行，利用这个特性，可以组合使用 saga 
```js
function* playLevelOne() { ... }
function* playLevelTwo() { ... }
function* playLevelThree() { ... }
function* game() {
  const score1 = yield* playLevelOne()
  put(showScore(score1))

  const score2 = yield* playLevelTwo()
  put(showScore(score2))

  const score3 = yield* playLevelThree()
  put(showScore(score3))
}
```


## 3.5.5 channels 

### 通过 actionChannel 实现缓存区 

先看如下的例子：

```js
import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  while (true) {
    const {payload} = yield take('REQUEST')
    yield fork(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
```

这个例子是典型的 `watch -> fork` ，也就是每一个 REQEST 请求都会被并发的执行，现在如果有需求要求 REQUEST 一次只能执行一个，这种情况下可以使用到 actionChannel 

通过 actionChannel 修改上例子

```js
import { take, actionChannel, call, ... } from 'redux-saga/effects'

function* watchRequests() {
  //  为 REQUEST 创建一个 actionChannel 相当于一个缓冲区
  const requestChan = yield actionChannel('REQUEST')
  while (true) {
    // 重 channel 中取一个 action
    const {payload} = yield take(requestChan)
    // 使用非阻塞的方式调用 request
    yield call(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
```

channel 可以设置缓冲区的大小，如果只想处理最近的5个 action 可以如下设置

```js
import { buffers } from 'redux-saga'
const requestChan = yield actionChannel('REQUEST', buffers.sliding(5))
```

### eventChannel 和外部事件连接起来 

eventChannel 不同于 actionChannel，actionChannel 是一个 Effect ，而 eventChannel 是一个工厂函数，可以创建一个自定义的 channel 

下面创建一个倒计时的 channel 工厂

```js
import { eventChannel, END } from 'redux-saga'

function countdown(secs) {
  return eventChannel(emitter => {
      const iv = setInterval(() => {
        secs -= 1
        if (secs > 0) {
          emitter(secs)
        } else {
          // 结束 channel
          emitter(END)
          clearInterval(iv)
        }
      }, 1000);

      // 返回一个 unsubscribe 方法
      return () => {
        clearInterval(iv)
      }
    }
  )
}
```

通过 call 使用创建 channel 

```js
export function* saga() {
  const chan = yield call(countdown, value)
  try {    
    while (true) {
      // take(END) 会导致直接跳转到 finally
      let seconds = yield take(chan)
      console.log(`countdown: ${seconds}`)
    }
  } finally {
    // 支持外部 cancel saga
    if (yield cancelled()) {
      // 关闭 channel 
      chan.close()
      console.log('countdown cancelled')
    } else {
      console.log('countdown terminated')
    }
  }
}
```

### 通过 channel 在 saga 之间通信

除了 eventChannel 和 actionChannel，channel 可以不用连接任何事件源，直接创建一个空的 channel，然后手动的 put 事件到 channel 中

以上面的 watch->fork 为基础，需求改为 ，需要同时并发 3 个request 请求执行：

```js
import { channel } from 'redux-saga'
import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  // 创建一个空的 channel
  const chan = yield call(channel)
  // fork 3 个 worker saga
  for (var i = 0; i < 3; i++) {
    yield fork(handleRequest, chan)
  }
  while (true) {
    // 等待 request action
    const {payload} = yield take('REQUEST')
    // put payload 到 channel 中
    yield put(chan, payload)
  }
}

function* handleRequest(chan) {
  while (true) {
    const payload = yield take(chan)
    // process the request
  }
}
```


## 参考链接

- http://yelouafi.github.io/redux-saga/docs/advanced/index.html
- http://gajus.com/blog/2/the-definitive-guide-to-the-javascript-generators#understanding-the-execution-flow

