

# html中的css、javascript、dom之间的解析和相互阻塞关系

### JavaScript 会阻塞 CSS、DOM 吗

- **javascript 加载会阻塞 css 解析和渲染**
- **javascript 加载会阻塞 dom 解析和渲染**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script>
    console.log("start loaded");
    function h () {
        console.log(document.querySelectorAll('h1'))
      }
      setTimeout(h, 0)
  </script>
  <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.js"></script>
  <style>
    h1 {
      color: red !important
    }
  </style>
   <script>
    console.log("end loaded");
    setTimeout(h,0)
  </script>
</head>
<body>
  <h1>测试阻塞加载</h1>
<ul>
  <li><b>javascript 加载会阻塞 css 解析和渲染</b> </li>
  <li><b>javascript 加载会阻塞 dom 解析和渲染</b></li>
</ul>
</body>
</html>

```

在加载到jquery 文件后，会先下载远程的 jquery 并且执行他，他会阻塞 dom 的解析和渲染，css 解析和渲染，一直是白屏，等 jquery执行完成了才接着解析 Dom 和 cssom 并且渲染，console.log 打印 h1 标签也是空数组。如下图所示：

![img](https://mmbiz.qpic.cn/mmbiz_gif/gicdSwTiccre1xVsj0UrAjjrjRQ4llicc9aAwVfIIol3ryWH9tEJGEfZwK9uO7TPux70mMLxnB53AwtVC4MCeIh1g/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)



### css 加载会阻塞 JavaScript 的加载和执行、会阻塞 Dom 的解析和渲染？

- css加载不会阻塞DOM树的解析
-  css加载会阻塞DOM树的渲染
- css加载会阻塞后面js语句的执行、而js加载会阻塞 dom 解析和渲染。所以如果 css 后面有 js 没有执行完，DOM树也不会解析。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script>
    console.log("before css");
    function h () {
        console.log(document.querySelectorAll('h1'))
      }
      setTimeout(h, 0)
  </script>
  <link href="https://cdn.bootcss.com/bootstrap/4.0.0-alpha.6/css/bootstrap.css" rel="stylesheet">
  <style>
    h1 {
      color: red !important
    }
  </style>
   
</head>
<body>
  <h1>测试CSS阻塞加载</h1>
<ul>
  <li><b>css加载不会阻塞DOM树的解析</b> </li>
  <li><b>css加载会阻塞DOM树的渲染</b></li>
  <li><b>css加载会阻塞后面js语句的执行、而js加载会阻塞 dom 解析和渲染。所以如果JS 没有执行完DOM树就不会解析</b></li>

</ul>
</body>
</html>

```



### 原理分析

那么为什么会出现上面的现象呢？我们从浏览器的渲染过程来解析下。

不用浏览器使用的内核不同，所以他们的渲染过程也是不一样的。目前主要有两个：

**webkit渲染过程**

![img](https://segmentfault.com/img/remote/1460000018130508?w=624&h=289)

**Gecko渲染过程**

![img](https://segmentfault.com/img/remote/1460000018130509)

从上面两个流程图我们可以看出来，浏览器渲染的流程如下：

1. HTML解析文件，生成DOM Tree，解析CSS文件生成CSSOM Tree
2. 将Dom Tree和CSSOM Tree结合，生成Render Tree(渲染树)
3. 根据Render Tree渲染绘制，将像素渲染到屏幕上。



从流程我们可以看出来

1. DOM解析和CSS解析是两个并行的进程，所以这也解释了为什么CSS加载不会阻塞DOM的解析。
2. 然而，由于Render Tree是依赖于DOM Tree和CSSOM Tree的，所以他必须等待到CSSOM Tree构建完成，也就是CSS资源加载完成(或者CSS资源加载失败)后，才能开始渲染。因此，CSS加载是会阻塞Dom的渲染的。
3. 由于js可能会操作之前的Dom节点和css样式，因此浏览器会维持html中css和js的顺序。因此，样式表会在后面的js执行前先加载执行完毕。所以css会阻塞后面js的执行。





### **DOMContentLoaded**

对于浏览器来说，页面加载主要有两个事件，一个是DOMContentLoaded，另一个是onLoad。而onLoad没什么好说的，就是等待页面的所有资源都加载完成才会触发，这些资源包括css、js、图片视频等。

而DOMContentLoaded，顾名思义，就是当页面的内容解析完成后，则触发该事件。那么，正如我们上面讨论过的，css会阻塞Dom渲染和js执行，而js会阻塞Dom解析。那么我们可以做出这样的假设

1. 当页面只存在css，或者js都在css前面，那么DomContentLoaded不需要等到css加载完毕。
2. 当页面里同时存在css和js，并且js在css后面的时候，DomContentLoaded必须等到css和js都加载完毕才触发。



我们先对第一种情况做测试：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>css阻塞</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script>
      document.addEventListener('DOMContentLoaded', function() {
        console.log('DOMContentLoaded');
      })
    </script>
    <link href="https://cdn.bootcss.com/bootstrap/4.0.0-alpha.6/css/bootstrap.css" rel="stylesheet">
  </head>
  <body>
  </body>
</html>
```

从动图我们可以看出来，css还未加载完，就已经触发了DOMContentLoaded事件了。因为css后面没有任何js代码。



接下来我们对第二种情况做测试，很简单，就在css后面加一行代码就行了

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>css阻塞</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script>
      document.addEventListener('DOMContentLoaded', function() {
        console.log('DOMContentLoaded');
      })
    </script>
    <link href="https://cdn.bootcss.com/bootstrap/4.0.0-alpha.6/css/bootstrap.css" rel="stylesheet">

    <script>
      console.log('到我了没');
    </script>
  </head>
  <body>
  </body>
</html>
```



我们可以看到，只有在css加载完成后，才会触发DOMContentLoaded事件。因此，我们可以得出结论：

1. 如果页面中同时存在css和js，并且存在js在css后面，则DOMContentLoaded事件会在css加载完后才执行。
2. 其他情况下，DOMContentLoaded都不会等待css加载，并且DOMContentLoaded事件也不会等待图片、视频等其他资源加载。

<!DOCTYPE html>
<html lang="en">
  <head>
    <title>css阻塞</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script>
      document.addEventListener('DOMContentLoaded', function() {
        console.log('DOMContentLoaded');
      })
    </script>
    <link href="https://cdn.bootcss.com/bootstrap/4.0.0-alpha.6/css/bootstrap.css" rel="stylesheet">
  </head>
  <body>
  </body>
</html>





### 参考文档

#### [css加载会造成阻塞吗](https://segmentfault.com/a/1190000018130499)

#### [html中的css、javascript、dom之间的解析和相互阻塞关系](https://mp.weixin.qq.com/s/uxmtVKJUjFIvpVO_-eArXQ)

