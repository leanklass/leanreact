# 第四章 React 进阶

我们已经能基于 React 实现基本的交互逻辑，但是在使用 React 的过程中还是可能会有些不确定的地方或者一些特殊的功能不知道怎么实现，可能会问 React 中有没有一些 Best practices 或者 Good Pattern 可以参考的，本章会在各个维度介绍之前没有讲过的 React 特性。

- react 代码规范
- react patterns 
- react magics and tricks
    - dangerouslySetInnerHTML
    - context
    - 双向绑定
    - ref.someChild.someMethod
    - children
        - this.props.children
        - top api reactChildren
- react 动画
    - react-motion
    - rc-animate
    - transitionGroup   
- 不可变数据
    - Immutability Helpers
    - immutable.js
- 性能管理
    - 测试工具集
    - PureRenderMixin PureComponent
    - render 的时候创建了新对象  [] , {}, Function 等
    - https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f#.vru8s8e32
    - shouldComponentUpdate
    - 性能分析工具
    - key
