# Redux

通过本文了解以下知识点

1. `redux`的设计思路及实现原理
2. `react-redux` 的设计思路和实现原理
3. `redux中间件`的设计思路和实现原理



### Redux 是什么，为什么需要Redux

React 作为一个组件化开发框架，组件间存在大量的通信，有时这些通信需要跨越多个组件，或者多个组件之间共享一套数据，简单的父子组件间传值不能满足我们的需求，自然而然地，我们需要有一个地方存取和操作这些公共状态。而redux就为我们提供了一种管理公共状态的方案。



我们思考一下如何管理公共状态：既然是公共状态，那么就直接把公共状态提取出来好了。我们创建一个store.js文件，然后直接在里边存放公共的state，其他组件只要引入这个store就可以存取共用状态了。

```js
const state = {    
    count: 0
}
```

我们在store里存放一个公共状态count，组件在import了store后就可以操作这个count。这是最直接的store，当然我们的store肯定不能这么设计，原因主要是两点：

**1. 容易误操作**

比如说，有人一个不小心把store赋值了{}，清空了store，或者误修改了其他组件的数据，那显然不太安全，出错了也很难排查，因此我们需要**有条件地**操作store，防止使用者直接修改store的数据。

**2. 可读性很差**

JS是一门极其依赖语义化的语言，试想如果在代码中不经注释直接修改了公用的state，以后其他人维护代码得多懵逼，为了搞清楚修改state的含义还得根据上下文推断，所以我们最好是给每个操作**起个名字**。



## Redux 实现

我们重新思考一下如何设计这个**公共状态管理器**，根据我们上面的分析，**我们希望公共状态既能够被全局访问到，又是私有的不能被直接修改**，思考一下，**闭包**是不是就就正好符合这两条要求，因此我们会把公共状态设计成闭包（对闭包理解有困难的同学也可以跳过闭包，这并不影响后续理解）



既然我们要存取状态，那么肯定要有**getter**和**setter**，此外当状态发生改变时，我们得进行广播，通知组件状态发生了变更。这不就和redux的三个API：`getState、dispatch、subscribe`对应上了吗。我们用几句代码勾勒出store的大致形状：


```js
export const createStore = () => {    
    let currentState = {}       // 公共状态    
    function getState() {}      // getter    
    function dispatch() {}      // setter    
    function subscribe() {}     // 发布订阅    
    return { getState, dispatch, subscribe }
}
```

#### 1. getState实现

`getState()`的实现非常简单，返回当前状态即可：

```js



```



#### 2. dispatch 实现

我们的目标是**有条件地、具名地**修改store的数据，那么我们要如何实现这两点呢？我们已经知道，在使用dispatch的时候，我们会给dispatch()传入一个action对象，这个对象包括我们要修改的state以及这个操作的名字(actionType)，根据type的不同，store会修改对应的state。

```js
export const createStore = () => {    
    let currentState = {}    
    function getState() {        
        return currentState    
    }    
    function dispatch(action) {        
        switch (action.type) {            
            case 'plus':            
            currentState = {                 
                ...state,                 
                count: currentState.count + 1            
            }        
        }    
    }    
    function subscribe() {}    
    return { getState, subscribe, dispatch }
}

```

我们把对actionType的判断写在了dispatch中，这样显得很臃肿，也很笨拙，于是我们想到把这部分修改state的规则抽离出来放到外面，这就是我们熟悉的**`reducer`。**我们修改一下代码，让reducer从外部传入：

```js
import { reducer } from './reducer'
export const createStore = (reducer) => {    
    let currentState = {}     
    function getState() {        
        return currentState    
    }    
    function dispatch(action) {         
        currentState = reducer(currentState, action)  
    }    
    function subscribe() {}    
    return { getState, dispatch, subscribe }
}
```

然后我们创建一个reducer.js文件，写我们的reducer

```js
//reducer.js
const initialState = {    
    count: 0
}
export function reducer(state = initialState, action) {    
    switch(action.type) {      
        case 'plus':        
        return {            
            ...state,                    
            count: state.count + 1        
        }      
        case 'subtract':        
        return {            
            ...state,            
            count: state.count - 1        
        }      
        default:        
        return initialState    
    }
}
```



我们可以验证一下`getState`和`dispatch`：

```js
//store.js
import { reducer } from './reducer'
export const createStore = (reducer) => {    
    let currentState = {}        
    function getState() {                
        return currentState        
    }        
    function dispatch(action) {                
        currentState = reducer(currentState, action)  
    }        
    function subscribe() {}        
    return { getState, subscribe, dispatch }
}

const store = createStore(reducer)  //创建store
store.dispatch({ type: 'plus' })    //执行加法操作,给count加1
console.log(store.getState())       //获取state
```

运行代码，我们会发现，打印得到的state是：{ count: NaN }，这是由于store里初始数据为空，state.count + 1实际上是underfind+1，输出了NaN，所以我们得先进行store数据初始化。

上面的实现还存在一些问题，store数据在调用dispath之前没有被初始化，需要在真正的dispatch 之前初始化store

```js
import { reducer } from './reducer'
export const createStore = (reducer) => {        
    let currentState = {}        
    function getState() {                
        return currentState        
    }        
    function dispatch(action) {                
        currentState = reducer(currentState, action)        
    }        
    function subscribe() {} 
  	// 初始化store
    dispatch({ type: '@@REDUX_INIT' })  //初始化store数据        
    return { getState, subscribe, dispatch }
}

const store = createStore(reducer)      //创建store
store.dispatch({ type: 'plus' })        //执行加法操作,给count加1
console.log(store.getState())           //获取state
```



#### 3.subscribe实现

增加一个observe 队列

```js
import { reducer } from './reducer'
export const createStore = (reducer) => {        
    let currentState = {}   
    // 观察者队列   
    let observers = []                  
    function getState() {                
        return currentState        
    }        
    function dispatch(action) {                
        currentState = reducer(currentState, action)
        // 在state更新之后调用 observes 里面的callback
        observers.forEach(fn => fn())        
    }        
    function subscribe(fn) {                
        observers.push(fn)        
    }        
    dispatch({ type: '@@REDUX_INIT' })  //初始化store数据        
    return { getState, subscribe, dispatch }
}
```

