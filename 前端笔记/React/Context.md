# React Context 的理解以及应用



## 前言

`Context`被翻译为上下文，在编程领域，这是一个经常会接触到的概念，React中也有。在React的官方文档中，[`Context`](https://reactjs.org/docs/context.html)被归类为高级部分(Advanced)，属于React的高级API，但官方并不建议在稳定版的App中使用Context。



不过，这并非意味着我们不需要关注`Context`。事实上，很多优秀的React组件都通过Context来完成自己的功能，比如react-redux的`<Provider />`，就是通过`Context`提供一个全局态的`store`，拖拽组件react-dnd，通过`Context`在组件中分发DOM的Drag和Drop事件，路由组件react-router通过`Context`管理路由状态等等。在React组件开发中，如果用好`Context`，可以让你的组件变得强大，而且灵活。



## 初识React Context

简单说就是，当你不想在组件树中通过逐层传递`props`或者`state`的方式来传递数据时，可以使用`Context`来实现**跨层级**的组件数据传递。



### 如何使用Context

如果要`Context`发挥作用，需要用到两种组件，一个是`Context`生产者(Provider)，通常是一个父节点，另外是一个`Context`的消费者(Consumer)，通常是一个或者多个子节点。所以`Context`的使用基于**生产者消费者模式**。

对于父组件，也就是`Context`生产者，需要通过一个静态属性`childContextTypes`声明提供给子组件的`Context`对象的属性，并实现一个实例`getChildContext`方法，返回一个代表`Context`的纯对象 (plain object) 。

```js
import React from 'react'
import PropTypes from 'prop-types'

class MiddleComponent extends React.Component {
  render () {
    return <ChildComponent />
  }
}

class ParentComponent extends React.Component {
  // 声明Context对象属性
  static childContextTypes = {
    propA: PropTypes.string,
    methodA: PropTypes.func
  }
  
  // 返回Context对象，方法名是约定好的
  getChildContext () {
    return {
      propA: 'propA',
      methodA: () => 'methodA'
    }
  }
  
  render () {
    return <MiddleComponent />
  }
}

```



而对于`Context`的消费者，通过如下方式访问父组件提供的`Context`。

```js
import React from 'react'
import PropTypes from 'prop-types'

class ChildComponent extends React.Component {
  // 声明需要使用的Context属性
  static contextTypes = {
    propA: PropTypes.string
  }
  
  render () {
    const {
      propA,
      methodA
    } = this.context
    
    console.log(`context.propA = ${propA}`)  // context.propA = propA
    console.log(`context.methodA = ${methodA}`)  // context.methodA = undefined
    
    return ...
  }
}

```

子组件需要通过一个静态属性`contextTypes`声明后，才能访问父组件`Context`对象的属性，否则，即使属性名没写错，拿到的对象也是`undefined`。



而在接下来的发行版本中，React对`Context`的API做了调整，更加明确了生产者消费者模式的使用方式。

```js
import React from 'react';
import ReactDOM from 'react-dom';

const ThemeContext = React.createContext({
  background: 'red',
  color: 'white'
});

```

通过静态方法`React.createContext()`创建一个`Context`对象，这个`Context`对象包含两个组件，`<Provider />`和`<Consumer />`。

```js
class App extends React.Component {
  render () {
    return (
      <ThemeContext.Provider value={{background: 'green', color: 'white'}}>
        <Header />
      </ThemeContext.Provider>
    );
  }
}

```

`<Provider />`的`value`相当于现在的`getChildContext()`。

```js
class Header extends React.Component {
  render () {
    return (
      <Title>Hello React Context API</Title>
    );
  }
}
 
class Title extends React.Component {
  render () {
    return (
      <ThemeContext.Consumer>
        {context => (
          <h1 style={{background: context.background, color: context.color}}>
            {this.props.children}
          </h1>
        )}
      </ThemeContext.Consumer>
    );
  }
}

```

`<Consumer />`的`children`必须是一个函数，通过函数的参数获取`<Provider />`提供的`Context`。



### 几个可以直接获取Context的地方

实际上，除了实例的`context`属性(`this.context`)，React组件还有很多个地方可以直接访问父组件提供的`Context`。比如构造方法：

- `constructor(props, context)`

比如生命周期：

- `componentWillReceiveProps(nextProps, nextContext)`
- `shouldComponentUpdate(nextProps, nextState, nextContext)`
- `componetWillUpdate(nextProps, nextState, nextContext)`




