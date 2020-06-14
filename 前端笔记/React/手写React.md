# 手写React

学习React 的底层实现原理

**实现哪些功能**

01 JSX

02  Functional components

03  Class components with state 

03 Lifecycle Hooks



**暂不实现的功能**

- Virtual DOM

  我们将使用称为snabbdom的现成虚拟DOM，有趣的是，它是Vue.js使用的虚拟DOM。[snabbdom ](https://github.com/snabbdom) 

  A virtual DOM library with focus on simplicity, modularity, powerful features and performance.



- React Hooks

  有些人在阅读本文时可能会感到失望，但嘿，我们不想咀嚼更多东西，因此让我们首先构建基本的东西，我们总是可以在此基础上加以补充。 我还计划在我们在此处构建的内容之上编写有关实现自己的React钩子和虚拟DOM的单独文章。



- Debuggablity

  这是增加任何库或框架的复杂度的关键部分之一，由于我们只是出于娱乐目的而做，因此我们可以放心地忽略React提供的可调试性功能，例如dev tools和 Profile。



## JSX

JSX是一个开放标准，不以任何方式限制于React，因此我们可以在没有React的情况下使用它，它比您想象的要容易得多。 要了解我们如何为我们的库开发JSX，让我们看看使用JSX时幕后发生的事情。

```js
const App = (
  <div>
    <h1 className="primary">QndReact is Quick and dirty react</h1>
    <p>It is about building your own React in 90 lines of JavsScript</p>
  </div>
);

// The above jsx gets converted into
/**
 * React.createElement(type, attributes, children)
 * props: it is the type of the element ie. h1 for <h1></h1>
 * attributes: it is an object containing key value pair of props passed to the element
 * children: it the array of child elements inside it
 */
var App = React.createElement(
  "div",
  null,
  React.createElement(
    "h1",
    {
      className: "primary"
    },
    "QndReact is Quick and dirty react"
  ),
  React.createElement(
    "p",
    null,
    "It is about building your own React in 90 lines of JavsScript"
  )
);
```

正如您看到的，每个JSX元素都通过@ babel / plugin-transform-react-jsx插件转换为React.createElement（…）函数调用

为了使上述转换发生，在编写JSX时React需要在您的Scope内，这就是为什么在您的Scope内尝试编写没有React的JSX时会出现奇怪的错误的原因。




让我们首先安装@ babel / plugin-transform-react-jsx插件

```js
npm install @babel/plugin-transform-react-jsx 
```



将以下配置添加到.babelrc文件

```js
{
  "plugins": [
    ["@babel/plugin-transform-react-jsx", {
      "pragma": "QndReact.createElement", // default pragma is React.createElement
      "throwIfNamespace": false // defaults to true
    }]
  ]
}
```

此后，只要Babel看到JSX，它将调用QndReact.createElement（…），但我们尚未定义该函数，因此让我们将其添加到src / qnd-react.js中。





### render函数

```js
const render = (el, rootDomElement) => {
  // logic to put el into the rootDomElement
   console.log(type, props, children);
}
```





## 函数组件

函数组件，传递的type类型就是JavaScript函数。 如果调用该函数，则将获得组件希望呈现的HTML结果。因此我们需要在 `createElement`  方法中检查type参数的类型是否为function

```js
// file: src/qnd-react.js
import { h } from 'snabbdom';

const createElement = (type, props = {}, ...children) => {
  // if type is a function then call it and return it's value
  if (typeof type == 'function') {
    return type(props);
  }

  return h(type, { props }, children);
};
```



##  Class components with state 



```js
// file: src/qnd-react.js
import { h } from "snabbdom";

const createElement = (type, props = {}, ...children) => {
  // if type is a function then call it and return it's value
  if (typeof type == "function") {
    return type(props);
  }

  return h(type, { props }, children);
};

// component base class
class Component {
  constructor() { }

  componentDidMount() { }

  setState(partialState) { }

  render() { }
}

// add a static property to differentiate between a class and a function
Component.prototype.isQndReactClassComponent = true;

// to be exported like React.createElement, React.Component
const QndReact = {
  createElement,
  Component
};

export default QndReact;

```



**给Class Component 添加 state**

让我们继续向类组件添加状态，在此之前，我们必须先清楚 每次调用this.setState（{...}）时 **如何更新DOM的职责在于react-dom包而不是React** 。**这是为了使React的核心部分（例如Component类）与平台分离，从而提高了代码的可重用性，**即在React native中，您还可以使用相同的Component类，而react-native包则负责如何更新Mobile UI。



您现在可能会问自己：当调用this.setState（{...}）时，React如何知道该怎么做，答案是react-dom通过在React上设置updater属性与React进行通信。 Dan对此也有出色的文章，您可以在这里阅读。 现在让我们制作QndReactDom来向QndReact添加updater属性

```js
import QndReact from './qnd-react';

// QndReactDom telling React how to update DOM
QndReact.__updater = () => {
  // logic on how to update the DOM when you call this.setState
}

// to be exported like ReactDom.render
const QndReactDom =  { 
  render
};

export default QndReactDom;
```

每当我们调用this.setState（{...}）时，我们都需要比较组件的oldVNode和通过在组件上调用render函数生成的组件的newVNode，为此，我们可以在__vNode属性上添加__vNode属性。 类组件，用于维护组件的当前VNode实例。

```js
// file: src/qnd-react.js

const createElement = (type, props = {}, ...children) => {
  // if type is a Class then
  // 1. create a instance of the Class
  // 2. call the render method on the Class instance
  if (type.prototype && type.prototype.isQndReactClassComponent) {
    const componentInstance = new type(props);

    // remember the current vNode instance
    componentInstance.__vNode = componentInstance.render();

    return componentInstance.__vNode;
  }
}
```



**在Component基类上实现setState函数**

```js
// component base class
class Component {
  constructor() { }

  componentDidMount() { }

  setState(partialState) {
    // update the state by adding the partial state
    this.state = {
      ...this.state,
      ...partialState
    }
    // call the __updater function that QndReactDom gave
    QndReact.__updater(this);
  }

  render() { }
}
```



**实现QndReactDom中的__updater函数**

```js
// QndReactDom telling React how to update DOM
QndReact.__updater = (componentInstance) => {
  // logic on how to update the DOM when you call this.setState

  // get the oldVNode stored in __vNode
  const oldVNode = componentInstance.__vNode;
  // find the updated DOM node by calling the render method
  const newVNode = componentInstance.render();

  // update the __vNode property with updated __vNode
  componentInstance.__vNode = reconcile(oldVNode, newVNode);
}

```

