> [书籍完整目录](https://segmentfault.com/a/1190000005136764)

# 3.2 Redux TodoApp



![clipboard.png](/img/bVykcv)




上一节讲完了 redux 中的概念，但是仍然没有和 react 联系起来，这一节
将利用 redux 在 react 中实现完整的 todolist：

- 在 react 使用 redux
- 通过 Provider 连接 react 和 redux store
- 创建 action creators
- 创建 reducer
- 创建 Container Component
- 床架 Dummy Component

## 3.2.1 在 react 使用 redux

redux 可以和很多第三方的框架结合起来使用，为了在 react 中使用 redux，可以通过 `react-redux` 

安装 `react-redux`

```shell
$ npm install --save react-redux
```


## 3.2.2 通过 Provider 连接 react 和 redux store

react-redux 提供了一个叫 Provider 的组件，将 react 和 react-redux 结合的方式是用 Provider 嵌套应用的 App 组件，并将 redux store 作为属性传递到 Provider 组件之中。

index.js

```html
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

/** store 数据结构 sample
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
let store = createStore(todoApp)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

> 这里使用到了 React 的 Context ，App 下面的所有组件可以利用 context 获取传入到 Provider 中的 store 


## 3.2.3 创建 action creators

actions/index.js

```js
let nextTodoId = 0
export const addTodo = (text) => {
  return {
    type: 'ADD_TODO',
    id: nextTodoId++,
    text
  }
}

export const setVisibilityFilter = (filter) => {
  return {
    type: 'SET_VISIBILITY_FILTER',
    filter
  }
}

export const toggleTodo = (id) => {
  return {
    type: 'TOGGLE_TODO',
    id
  }
}
```

## 3.2.4 创建 reducer

首先创建根 reducer ,通过 `redux.combineReducers` 方法将其他 reducer 结合起来，每个数据 key 都需要实现一个对应的 reducer

reducer/index.js
```js
import { combineReducers } from 'redux'
import todos from './todos'
import visibilityFilter from './visibilityFilter'

const todoApp = combineReducers({
  todos,
  visibilityFilter
})

export default todoApp
```

接着是 todos reducer , 需要注意的地方是其中使用 Object.assign 方法保证每次都是返回新的
对象

reducer/todos.js
```js
const todo = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        id: action.id,
        text: action.text,
        completed: false
      }
    case 'TOGGLE_TODO':
      if (state.id !== action.id) {
        return state
      }

      return Object.assign({}, state, {
        completed: !state.completed
      })

    default:
      return state
  }
}

const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        todo(undefined, action)
      ]
    case 'TOGGLE_TODO':
      return state.map(t =>
        todo(t, action)
      )
    default:
      return state
  }
}

export default todos
```

最后是 visibilityFilter 

reducers/visibibityFilter.js
```js
const visibilityFilter = (state = 'SHOW_ALL', action) => {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

export default visibilityFilter
```


## 3.2.5  Container Component

在介绍 flux 的时候介绍过组件分两个类型，smart component 和 dummy component，在 redux 中 Container Component 就是 smart component 

### 作用 

在 react 应用中，store 的数据只有 container component 能知晓，container component 会将知晓的数据传递给 dummy components ，除此之外 action 的触发方法也会由它传递给 dummy components 

### connect 方法

react-redux 提供了一个叫 connect 的方法，可以将一个组件变为 container component

```js
const ContainerComponent = connect(
    /**
     * 方法将 store 作为参数，返回有个 {key: Value} 对象，key 作为属性传递给 DummyComponent 
     * @type {[type]}
     */
    mapStateToProps: Function,
    /**
     * 方法传递 store.dispatch 作为参数，返回一个{key: Function} 对象，key 作为属性传递给 DummyComponent 
     * @type {[type]}
     */
    mapDispatchToProps: Function
)(DummyComponent)

```

> 其本质就是获取react context 中的 store，并将 store 中的数据作为属性传递到原来的组件中

### 创建 todolist 的 container component

todolist 会分为三个 container，有个负责 todolist，一个负责 filter，一个为添加 todo，首先是 todolist。

containers/VisibleTodoList.js
```js
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```


containers/FilterLink.js
```js
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink
```

containers/AddTodo.js
```
import React from 'react'
import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = ({ dispatch }) => {
  let input

  return (
    <div>
      <form onSubmit={e => {
        e.preventDefault()
        if (!input.value.trim()) {
          return
        }
        dispatch(addTodo(input.value))
        input.value = ''
      }}>
        <input ref={node => {
          input = node
        }} />
        <button type="submit">
          Add Todo
        </button>
      </form>
    </div>
  )
}
AddTodo = connect()(AddTodo)

export default AddTodo
```

## 3.2.6 负者展现的 Dummy Components

components/App.js: App.js 连接所有的 Container Components
```
import React from 'react'
import Footer from './Footer'
import AddTodo from '../containers/AddTodo'
import VisibleTodoList from '../containers/VisibleTodoList'

const App = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
)

export default App
```

components/TodoList.js: 展现 todos 列表
```
import React, { PropTypes } from 'react'
import Todo from './Todo'

const TodoList = ({ todos, onTodoClick }) => (
  <ul>
    {todos.map(todo =>
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
    )}
  </ul>
)

TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  onTodoClick: PropTypes.func.isRequired
}

export default TodoList
```

components/Todo.js
```
import React, { PropTypes } from 'react'

const Todo = ({ onClick, completed, text }) => (
  <li
    onClick={onClick}
    style={{
      textDecoration: completed ? 'line-through' : 'none'
    }}
  >
    {text}
  </li>
)

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  completed: PropTypes.bool.isRequired,
  text: PropTypes.string.isRequired
}

export default Todo
```

components/Link.js

```
import React, { PropTypes } from 'react'

const Link = ({ active, children, onClick }) => {
  if (active) {
    return <span>{children}</span>
  }

  return (
    <a href="#"
       onClick={e => {
         e.preventDefault()
         onClick()
       }}
    >
      {children}
    </a>
  )
}

Link.propTypes = {
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func.isRequired
}

export default Link
```

components/Footer.js

```
import React from 'react'
import FilterLink from '../containers/FilterLink'

const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
)

export default Footer
```
