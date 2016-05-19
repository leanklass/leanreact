# Component

## 问题

优化 JSX 语法练习的 TODOMVC 页面， 通过组件化的方式拆分页面！

组件如下：

1. App 组件：整个页面的最完成组件
2. Header 组件：头部输入组件
2. TodoList 组件：列表组件
3. TodoItem 组件: 列表项
4. Footer 组件：底部操作组件

## Tips 

循环输出组件

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
