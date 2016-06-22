上面使用过 takeEvery helper, 其实底层是通过 take effect 来实现的。通过 take effect 可以实现很多有趣的简洁的控制。

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

使用 take 过后可以改为：

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

其他的两个例子：

```js
import { take, put } from 'redux-saga/effects'
/**
 * 完成了三个任务后，提示恭喜信息
 */
function* watchFirstThreeTodosCreation() {
  for (let i = 0; i < 3; i++) {
    const action = yield take('TODO_CREATED')
  }
  yield put({type: 'SHOW_CONGRATULATION'})
}
```

```js
/**
 * 登录和登出逻辑可以放在同一个函数内共享变量
 */
function* loginFlow() {
  while (true) {
    yield take('LOGIN')
    // ... perform the login logic
    yield take('LOGOUT')
    // ... perform the logout logic
  }
}
```
