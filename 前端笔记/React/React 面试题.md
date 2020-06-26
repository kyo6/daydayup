## React特点

1. 高效、虚拟DOM，最大限度地减少与DOM的交互：
   - 浏览器在渲染网页时，会先将HTML文档解析并构建DOM树，然后与CSSOM树生成RenderObject树，最后渲染成页面。浏览器中渲染引擎和JavaScript引擎是分离的，渲染引擎会提供一些接口给JavaScript调用，它们二者通信是通过桥接的，性能其实是很差的。
   - 以往，为了优化性能通常采用的办法是减少DOM操作次数。而React提出了一个新的思路就是虚拟DOM：组件的HTML结构不再是直接生成DOM，而是映射生成虚拟的JavaScript DOM结构，React通过diff算法将最小变更写入DOM中，从而减少DOM的实际次数，提升性能。
2. 服务器端渲染
   - React提供开箱即用的服务器端渲染，服务器端渲染解除了服务器端对浏览器的依赖，它会将“可视”部分先渲染，然后再交给客户端做渲染。
3. 组件化编码
   - React 的一切都是基于组件的。可以通过定义一个组件，然后在其他的组件中，可以像HTML标签一样引用它。说得通俗点，组件其实就是自定义的标签；通过 React 构建组件，使得代码更加容易得到复用，能够很好的应用在大项目的开发中。
4. 声明式设计
   - React采用声明范式，函数式编程，可以轻松描述应用。
5. 灵活
   - React可以与已知的库或框架很好地配合。
6. JSX
   - JSX 是 JavaScript 语法的扩展。React开发不一定使用 JSX ，但我们建议使用它。
7. 单向响应的数据流
   - React 实现了单向响应的数据流，数据是自顶向下单向流动的，即从父组件到子组件，这种原则让组件之间的关系变得简单可预测；props作为外部接口、state作为内部状态。



### 请说一说Forwarding Refs有什么用

是父组件用来获取子组件的dom元素的，为什么有这个API，原因如下

```js
// 例如有一个子组件和父组件，代码如下
子组件为：
class Child extends React.Component{
    constructor(props){
      super(props);
    }
    render(){
      return <input />
    }
 }

// 父组件中，ref：
class Father extends React.Component{
  constructor(props){
    super(props);
    this.myRef=React.createRef();
  }
  componentDidMount(){
    console.log(this.myRef.current);
  }
  render(){
    return <Child ref={this.myRef}/>
  }
}

```

此时父组件的this.myRef.current的值是Child组件，也就是一个对象，如果用了React.forwardRef，也就是如下

```js
// 子组件
const Child = React.forwardRef((props, ref) => (
    <input ref={ref} />
));
// 父组件
class Father extends React.Component {
    constructor(props) {
        super(props);
        this.myRef = React.createRef();
    }
    componentDidMount() {
        console.log(this.myRef.current);
    }
    render() {
        return <Child ref={this.myRef} />
    }
}
```

此时父组件的this.myRef.current的值是input这个DOM元素



### 什么是受控组件和非受控组件

受状态控制的组件，必须要有onChange方法，否则不能使用 受控组件可以赋予默认值（官方推荐使用 受控组件） 实现双向数据绑定

```js
class Input extends Component{
    constructor(){
        super();
        this.state = {val:'100'}
    }
    handleChange = (e) =>{ //e是事件源
        let val = e.target.value;
        this.setState({val});
    };
    render(){
        return (<div>
            <input type="text" value={this.state.val} onChange={this.handleChange}/>
            {this.state.val}
        </div>)
    }
}

```

非受控也就意味着我可以不需要设置它的state属性，而通过ref来操作真实的DOM

```js
class Sum extends Component{
    constructor(){
        super();
        this.state =  {result:''}
    }
    //通过ref设置的属性 可以通过this.refs获取到对应的dom元素
    handleChange = () =>{
        let result = this.refs.a.value + this.b.value;
        this.setState({result});
    };
    render(){
        return (
            <div onChange={this.handleChange}>
                <input type="number" ref="a"/>
                {/*x代表的真实的dom,把元素挂载在了当前实例上*/}
                <input type="number" ref={(x)=>{
                    this.b = x;
                }}/>
                {this.state.result}
            </div>
        )
    }
}

```



### createElement 和 cloneElement 有什么区别？

React.createElement():JSX 语法就是用 React.createElement()来构建 React 元素的。它接受三个参数，第一个参数可以是一个标签名。如 div、span，或者 React 组件。第二个参数为传入的属性。第三个以及之后的参数，皆作为组件的子组件。

```
React.createElement(
    type,
    [props],
    [...children]
)
```

React.cloneElement()与 React.createElement()相似，不同的是它传入的第一个参数是一个 React 元素，而不是标签名或组件。新添加的属性会并入原有的属性，传入到返回的新元素中，而就的子元素奖杯替换。

```
React.cloneElement(
  element,
  [props],
  [...children]
)
```



### 如果你创建了类似于下面的 Twitter 元素，那么它相关的类定义是啥样子的？

```js
<Twitter username='tylermcginnis33'>
  {(user) => user === null
    ? <Loading />
    : <Badge info={user} />}
</Twitter>
import React, { Component, PropTypes } from 'react'
import fetchUser from 'twitter'
// fetchUser take in a username returns a promise
// which will resolve with that username's data.
class Twitter extends Component {
  // finish this
}
```

如果你还不熟悉回调渲染模式（Render Callback Pattern），这个代码可能看起来有点怪。这种模式中，组件会接收某个函数作为其子组件，然后在渲染函数中以 props.children 进行调用：

```js
import React, { Component, PropTypes } from 'react'
import fetchUser from 'twitter'
class Twitter extends Component {
  state = {
    user: null,
  }
  static propTypes = {
    username: PropTypes.string.isRequired,
  }
  componentDidMount () {
    fetchUser(this.props.username)
      .then((user) => this.setState({user}))
  }
  render () {
    return this.props.children(this.state.user)
  }
}
```

这种模式的优势在于将父组件与子组件解耦和，父组件可以直接访问子组件的内部状态而不需要再通过 Props 传递，这样父组件能够更为方便地控制子组件展示的 UI 界面。譬如产品经理让我们将原本展示的 Badge 替换为 Profile，我们可以轻易地修改下回调函数即可：

```js
<Twitter username='tylermcginnis33'>
  {(user) => user === null
    ? <Loading />
    : <Profile info={user} />}
</Twitter>
```



### 4、你对Fiber架构了解吗



### 何为高阶组件(higher order component)

高阶组件是一个以组件为参数并返回一个新组件的函数。HOC 运行你重用代码、逻辑和引导抽象。最常见的可能是 Redux 的 connect 函数。除了简单分享工具库和简单的组合，HOC 最好的方式是共享 React 组件之间的行为。如果你发现你在不同的地方写了大量代码来做同一件事时，就应该考虑将代码重构为可重用的 HOC。



### 刚刚说到块级作用域，谈谈它和闭包的区别，就是说一般什么时候使用闭包？

- 回答:
  1. 闭包使用的作用的话主要是为了**获取函数内部的变量**和**将变量保存下来，使其不被垃圾回收器回收，供给之后使用**；
  2. 使用的话一般是为了封装作用域，即代替全局变量，避免全局变量污染；
  3. 因为使用闭包，会将变量保存不被回收器回收，所以应该尽量避免使用，防止造成内存泄漏
- 答的不是太全面，这里补充：
- **闭包的应用场景**
  1. 使用闭包代替全局变量
  2. 函数外或在其他函数中访问某一函数内部的参数
  3. 在函数执行之前为要执行的函数提供具体参数
  4. 在函数执行之前为函数提供只有在函数执行或引用时才能知道的具体参数
  5. 为节点循环绑定click事件，在事件函数中使用当次循环的值或节点，而不是最后一次循环的值或节点
  6. 暂停执行
  7. 包装相关功能

### 说到回收器，说说如何判断一个变量可回收，如果我需要使用闭包，该如何避免你所说的内存泄漏？

- 感觉有点给自己挖坑了，（回收机制、内存泄漏不是太懂啊~~~~~~）
- 回答：
  1. 一个变量的可回收性，一般需要根据浏览器js引擎的回收器机制判断，大概有两种方式（当时只说到了一种）；
  2. 即**引用计数**:引用计数的含义是跟踪记录每个变量被引用的次数，当期的引用次数为0时表回收
  3. 一般手动清除是将它赋值为nul