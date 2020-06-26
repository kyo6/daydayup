## React 16 新架构

### React16 解决了哪些痛点

- 16.0 让组件支持返回任何数组类型，从而解决了数组问题；推出createPortal API 解决弹窗问题； 推出componentDidCatch新钩子划分出错误组件和边界组件，每个边界组件都能修复下方组件错误一次，再次出错转交更上层边界组件来处理，解决异常处理问题。
- 16.2 推出Fragment组件，可以看作是数组的一种语法糖
- 16.3 推出createRef与forwardRef解决Ref 在Hoc 中传递问题，推出new Context API，解决HOC的context传递问题
- 而性能问题，从16.0开始一直是由内部机制来保证，涉及到批量更新及基于时间分片的先来更新。





## setState 是同步还是异步的

- 生命周期和事件中会触发React的批处理机制，造成异步的假象

  在 React 的生命周期和合成事件中，React 仍然处于他的更新机制中，这是无论调用多少次setState都不会立即执行更新，而是将要更新的存入`_pendingStateQueue`，将要更新的组件存入`dirtyComponent`。

React 的setState本身并不是异步的，而是React的批处理机制给人一种异步的假象。

- **在异步代码和原生事件中不会触发React的批处理机制**，所以setState是同步的

最佳实践：

setState 的第二个参数接受一个函数，该函数会在React的批处理机制完成后调用，所以你想在setState后立即获取更新后的值，可以在回调函数中获取。





## React 如何实现自己的事件机制

React事件并没有绑定在真实Dom节点上，而是通过事件代理，在最外层的document上对事件进行统一的分发。



### 原生事件 和 React 事件的区别

- React 事件是驼峰命名，而不是全部小写
- 传入函数而不是字符串作为事件处理程序
- 不能返回false来阻止默认行为，必须明确调用preventDefault



### 为什么React事件要自己绑定this

事件函数在react源码中是直接调用的，并没有指定调用的组件，所以如果不进行手动绑定获取到的this是不准确的。



### React合成事件是什么？

React 根据W3C 规范定义了每个事件处理函数的参数，即合成事件。

事件处理程序将传递SyntheticEvent 的实例，它是一个跨浏览器原生事件包装器。它具有与浏览器原生事件相同的接口。包括stopPropagation() 和	preventDefault()，在所有浏览器中他们的工作方式都相同。

- React合成的SyntheticEvent采用了**事件池**，这样做可以大大节省内存，而不会频繁的创建和销毁事件对象。

- 可以跨浏览器兼容。



### React和原生事件的执行顺序是什么？

React 的所有事件都通过document进行统一分发，当真实Dom触发事件冒泡到document后会对React事件进行处理。

所以原生事件会先执行，然后执行React合成事件，最后执行真正在docunent上挂载的事件。



