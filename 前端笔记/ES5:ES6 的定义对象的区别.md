# ES5/ES6 的定义对象的区别

1. `class` 声明会提升，但不会初始化赋值。`Foo` 进入暂时性死区，类似`let`、`const` 声明变量

```js
const bar = new Bar(); // it's ok
function Bar() {
  this.bar = 42;
}

const foo = new Foo();
class Foo {
  constructor(){
    this.foo = 42;
  }
}

```

1. class 声明内部会启用严格模式

```js
// 引用一个未声明的变量
function Bar() {
  bar = 42; // it's ok
}
const bar = new Bar();

class Foo {
  constructor() {
    fol = 42; // ReferenceError: fol is not defined
  }
}
const foo = new Foo();

```



3. class 的所有方法（包括静态方法和实例方法）都是不可枚举的

```js
function Bar() {
  this.bar = 42;
}
Bar.answer = function() {
  return 42;
}
Bar.prototype.print = function() {
  console.log(this.bar)
}

const barKeys = Object.keys(bar);  // ['answer']
const barProtoKeys = Object.keys(bar.prototype) // ['print']

class Foo() {
  constructor(42){
    this.foo = 42;
  }
  static answer() {
   return 42;
  }
  print() {
    console.log(this.foo)
  }
}
const fooKeys = object.keys(Foo) // []
const fooProtokeys = object.keys(Foo.prototype) //[]
```



class 的所有方法（包括静态方法和实例方法）都没有原型对象prototype，所以也没有constructor，不能使用new