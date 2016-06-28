# react patterns

- 单向性和确定性
    - 修改 Props 
    - 在 getInitialState 中使用 props 
    - 在 componentWillMount 使用 setState
    - render with side effects （破坏一致性原则，尽量保证纯函数）
    - private states and global events （global Event 增加组件的耦合, private state 影响组件的确定性）
    - mutate dom explicitly via Jquery (和 React 的dom冲突 )
- 内存管理
    - componentDidMount + componentWillUnmount
    - 不要判断 isMounted
- 上层设计
    - 使用 container component
    - 使用 mixins （Composition instead of mixins）https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.le8yopk2n


- https://github.com/planningcenter/react-patterns#cached-state-in-render
- https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.le8yopk2n

