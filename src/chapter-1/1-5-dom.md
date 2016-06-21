> [书籍完整目录](https://segmentfault.com/a/1190000005136764)
# 1.5 React 与 DOM 

![图片描述][1]

在这一节中，主要的讨论范围为 React 与 DOM 相关的处理，包括：

1. 如何获取 DOM 元素
2. 如何做事件响应处理
3. 表单处理
4. style 属性

这节讲述过后，我们将会为 TODO 应用添加完整的事件响应，包括新增，删除，标记完成等。

## 1.5.1 获取 DOM 元素

上一节我们已经讲过组件的生命周期，DOM 真正被添加到 HTML 中的 hook 为 

- componentDidMount
- componentDidUpdate

在这两个 hook 函数中， 我们可以获取真正的 DOM 元素，React 提供的获取方法两种方式

### findDOMNode()

通过 ReactDOM 提供的 findDOMNode 方法， 传入参数我组件实例，eg

```html
var MyComponent = React.createClass({
    render: function() {
        return <div> .... </div>
    },
    componentDidMount: function() {
        var $root = ReactDOM.findDOMNode(this);
        console.log($root);
    }
})
```

> 需要注意的是此方法不能应用到无状态组件上

### Refs

上面的方法只能获取到 root  元素，那如果我的 DOM 有很多层级，我想获取一个子级的元素呢？
React 提供了 ref 属性来实现这种需求。

每个组件实例都有一个 `this.refs` 属性，会自动引用所有包含 ref 属性组件的 DOM, eg:

```html
var MyComponent = React.createClass({
    render: function() {
        return  <div>
                    <button ref="btn">...</button>
                    <a href="" ref="link"></a>
                </div>
    },
    componentDidMount: function() {
        var $btn = this.refs.btn;
        var $link = this.refs.link;
        console.log($btn, $link);
    }
})
```

## 1.5.2 DOM 事件 

> 官方事件文档 http://facebook.github.io/react/docs/events.html

### 绑定事件

在 React 中绑定事件的方式很简单，只需要在元素中添加事件名称的属性已经对应的处理函数，如：

```html
var MyComponent = React.creatClass({
    render: function() {
        return  <div>
                    <button onClick={this.onClick}>Click Me</button>
                </div>
    },
    onClick: function() {
        console.log('click me');
    }
});
```

事件名称和其他属性名称一样，服从驼峰式命名。

### 合成事件（SyntheticEvent）

在 React 中， 事件的处理由其内部自己实现的事件系统完成，触发的事件都叫做 合成事件（SyntheticEvent），事件系统对浏览器做了兼容，其提供的 API 与原生的事件无异。

```javascript
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget
boolean defaultPrevented
number eventPhase
boolean isTrusted
DOMEvent nativeEvent
void preventDefault()
boolean isDefaultPrevented()
void stopPropagation()
boolean isPropagationStopped()
DOMEventTarget target
number timeStamp
string type
```

和原生事件的区别在于，事件不能异步话，如：

```javascript
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  var eventType = event.type; // => "click"

  setTimeout(function() {
    console.log(event.type); // => null
    console.log(eventType); // => "click"
  }, 0);

  this.setState({clickEvent: event}); // Won't work. this.state.clickEvent will only contain null values.
  this.setState({eventType: event.type}); // You can still export event properties.
}
```

原因是在事件系统的内部实现当中， 一个事件对象可能会被重用（也就是事件做了池化 Pooling）。当一个事件响应函数执行过后，事件的属性被设置为 null， 如果想用保持事件的值的话，可以调用

```javascript
    event.persist()
```

这样，属性会被保留，并且事件也会被从池中取出。

### 事件捕获和冒泡

在 DOM2.0 事件分为捕获阶段和冒泡阶段，React 中通常我们注册的事件为冒泡事件，如果要注册捕获阶段的事件，可以在事件名称后加 Capture 如：

```
onClick
onClickCapture
```

### 支持事件列表

```javascript
粘贴板事件 {
    事件名称：onCopy onCut onPaste
    属性：DOMDataTransfer clipboardData
}

编辑事件 {
    事件名称：onCompositionEnd onCompositionStart onCompositionUpdate
    属性：string data
}

键盘事件 {
    事件名称：onKeyDown onKeyPress onKeyUp
    属性： {
        boolean altKey
        number charCode
        boolean ctrlKey
        boolean getModifierState(key)
        string key
        number keyCode
        string locale
        number location
        boolean metaKey
        boolean repeat
        boolean shiftKey
        number which
    }
}

// 焦点事件除了表单元素以外，可以应用到所有元素中
焦点事件 {
    名称：onFocus onBlur
    属性：DOMEventTarget relatedTarget
}

表单事件 {
    名称：onChange onInput onSubmit
}

鼠标事件 {
    名称：{
        onClick onContextMenu onDoubleClick onDrag onDragEnd onDragEnter onDragExit onDragLeave onDragOver onDragStart onDrop onMouseDown onMouseEnter onMouseLeave onMouseMove onMouseOut onMouseOver onMouseUp
    }
    属性：{
        boolean altKey
        number button
        number buttons
        number clientX
        number clientY
        boolean ctrlKey
        boolean getModifierState(key)
        boolean metaKey
        number pageX
        number pageY
        DOMEventTarget relatedTarget
        number screenX
        number screenY
        boolean shiftKey
    }
}

选择事件 {
    名称：onSelect
}

触摸事件 {
    名称：onTouchCancel onTouchEnd onTouchMove onTouchStart
    属性：{
        boolean altKey
        DOMTouchList changedTouches
        boolean ctrlKey
        boolean getModifierState(key)
        boolean metaKey
        boolean shiftKey
        DOMTouchList targetTouches
        DOMTouchList touches
    }
}

UI 事件 {
    名称：onScroll
    属性：{
        number detail
        DOMAbstractView view
    }
}

滚轮事件 {
    名称：onWheel
    属性：{
        number deltaMode
        number deltaX
        number deltaY
        number deltaZ
    }
}

媒体事件 {
    名称：{
        onAbort onCanPlay onCanPlayThrough onDurationChange onEmptied onEncrypted onEnded onError onLoadedData onLoadedMetadata onLoadStart onPause onPlay onPlaying onProgress onRateChange onSeeked onSeeking onStalled onSuspend onTimeUpdate onVolumeChange onWaiting
    }
}

图像事件 {
    名称：onLoad onError
}

动画事件 {
    名称：onAnimationStart onAnimationEnd onAnimationIteration
    属性：{
        string animationName
        string pseudoElement
        float elapsedTime
    }
}

渐变事件 {
    名称：onTransitionEnd
    属性： {
        string propertyName
        string pseudoElement
        float elapsedTime
    }
}
```


## 1.5.3 表单事件

在 React 中比较特殊的事件是表单事件，大多数组件都是通过属性和状态来决定的，但是表单组件如 `input`, `select`, `option` 这些组件的状态用户可以修改，在 React 中会特殊处理这些组件的事件。

### onChange 事件 

和普通 HTML 中的 onChange 事件不同， 在原生组件中，只有 input 元素失去焦点才会触发 onChange 事件， 在 React 中，只要元素的值被修改就会触发 onChange 事件。

```html
var MyComponent = React.createClass({
    getInitialState: function() {
        return {
            value: ''
        }
    },
    render: function() {
        return  <div onChange={this.onChangeBubble}>
                    <input value={this.state.value} onChange={this.onChange}/>
                </div>
    },
    onChange: function(ev) {
        console.log('change: ' + ev.target.value);
        this.setState({
            value: ev.target.value
        });
    },
    // onChange 事件支持所有组件，可以被用于监听冒泡事件
    onChangeBubble: function(ev) {
        console.log('bubble onChange event', + ev.target.value);
    }
})
```

### 交互属性

表单组件中能被用户修改的属性叫交互属性，包括：

1. `value`    => **<input>** 和 **<select>** 组件
2. `checked`  => **<input type="checkbox|radio">**
3. `selected` => **<opiton>**


### textara

在 HTML 中，textarea 的值是像如下定义的：

```html
<textarea name="" id="" cols="30" rows="10">
    some value
</textarea>
```

而在 React 中， TextArea 的使用方式同 input 组件，使用 value 来设置值

```html
var MyComponent = function() {
    render: function() {
        return <div>
                    <textarea value={...} onChange={...}/>
                </div>
    }
}
```

### select 组件

在 React 中 select 组件支持 value 值，value 值还支持多选

```html

  <select value="B">
    <option value="A">Apple</option>
    <option value="B">Banana</option>
    <option value="C">Cranberry</option>
  </select>

  <select multiple={true} value={['B', 'C']}>
    <option value="A">Apple</option>
    <option value="B">Banana</option>
    <option value="C">Cranberry</option>
  </select>

```

### 受控组件

在 React 中表单组件可分为两类，受控与非受控组件，受控组件是包含了 value 值的，如：

```html
render: function() {
    return <input type="text" value="....."/>
}
```

为什么叫受控组件？ 因为这个时候用户不能修改 input 的值， input 的值永远是 value 固定了的值。
如果去掉 value 属性，那么就可以输入值了。

那如何修改受控组件的值呢？ 如上面的例子中， 添加 onChange 事件，事件内修改 value 属性，value 属性的值会被设置到组件的 value 中。


### 非受控组件

对应受控组件，也有非受控组件，那么那种是非受控组价呢？

没有 value 值的 input 

```html
render: function() {
    return <input type="text"/>
}
```

那如果想设置默认值呢？

可以通过 defaultValue 属性来设置

```html
render: function() {
    return <input type="text" defaultValue="Default Value">
}
```

类似的对于 checkbox 有 defaultChecked 属性

> 需要注意的是，默认值只适用于第一次渲染，在重渲染阶段将不会适用。

### checkbox 和 radio 

checkbox 和 radio 比较特殊， 如果在 onChange 事件中调用了 preventDefault ，那么浏览器不会更新 checked 状态，即便事实上组件的值已经 checked 或者 unchecked 了 。

eg:

```html
var CheckBox = React.createClass({
    getInitialState: function(){
        return {
            checked: false
        }
    },
    render: function() {
        return  <div>
            <input type="checkbox" 
                checked={this.state.checked} 
                onChange={this.onChange}/>
        </div>
    },
    onChange: function(ev) {
        this.setState({
            checked: true
        });
        ev.preventDefault();
    }
})
```

这个例子里边，checked 虽然更新为 true ，但是 input 的值 checked 为 false

那应如何处理 checkbox 呢？

1. 避免调用 ev.preventDefault 就行
2. 在 setTimeout 中处理 checked 的修改
3. 使用 click 事件


## 1.5.4 style 属性

在 React 中，可以直接设置 style 属性来控制样式，不过与 HTML 不同的是， 传入的 style 值为一个对象， 对象的所有 key 都是驼峰式命名，eg:

```html
render: function() {
    var style = {
        backgroundColor: 'red',
        height: 100,
        width: 100
    }
    return <div style={style}></div>
}
```

其中还可以看到不同的地方时，为了简写宽度高度值，可以直接设置数字，对应 `100 -> 100px`。如果某些属性不需要添加 px 后缀，React 也会自动去除。

通过属性值驼峰式的原因是 DOM 内部访问 style 也是驼峰式。如果需要添加浏览器前缀瑞 `-webkit-`、`-ms-` 大驼峰(除了 ms ), 如：

```js
var divStyle = {
  WebkitTransition: 'all', // 'W' 是大写
  msTransition: 'all'      // 'ms' 为小写
};
```


### 为什么要用 inline 的样式？

在以前的前端开发方式是 样式结构和逻辑要分离， 而现在 React 中却有很多人推崇 inline 的样式。 在我看来因人而异，React 的这种模式也能做到样式模块化，样式重用（借用 Js 的特点）。并且因为 React 的实现方式，Inline 样式的性能甚至比 class 的方式高。


## 1.5.5 实例练习：完整功能的 TODO 应用

@todo

  [1]: /img/bVvUtM