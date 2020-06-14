# 浏览器是怎么渲染 html 的

https://mp.weixin.qq.com/s/C4KIsNAAkGcypAJeG-Cswg



![img](https://mmbiz.qpic.cn/mmbiz_png/gicdSwTiccre3MZjscrFiaGqSuZlqjEFHiaU6xm2yGtPdnLibZYMbeTYjUsutq00GBOVEibptbncPKuWWYNkF16h8iamQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_png/gicdSwTiccre3MZjscrFiaGqSuZlqjEFHiaU6xm2yGtPdnLibZYMbeTYjUsutq00GBOVEibptbncPKuWWYNkF16h8iamQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



当我们在浏览器中输入一个网址的时候，他是怎么请求资源，并且把我们的页面绘制出来的，大概可以分为六步，

其中又可以细分，下面我大概说一个 6大步骤：

1. 浏览器首先通过 HTTP 协议或者HTTPS 协议，向服务器请求页面资源，当然这其中也可能有缓存什么的。

2. 把请求回来的 HTML 代码经过解析，构建成 DOM 树
3. 计算 DOM 树上的 CSS 属性，生成CSSOM树（CSS OBject Model）
4. 将DOM树和 CSSOM 树合并成一个渲染树（render Tree）
5. 渲染树的每个元素包含的内容都是计算过的，它被称之为布局layout。浏览器使用一种流式处理的方法，只需要一次pass绘制操作就可以布局所有的元素；（Layout(回流)）
6. 将渲染树的各个节点绘制到屏幕上，这一步被称为绘制painting。Painting(重绘)
7. 按照合理的树序合并图层然后显示到屏幕上Composite（渲染层合并）

![img](https://mmbiz.qpic.cn/mmbiz_png/gicdSwTiccre3MZjscrFiaGqSuZlqjEFHiaUuG5V0rSSjGIp4yGnkJRTfTpcjH5n5fP808Jp8SQmGMlF9hYHfgcSLw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**js async 和 defer 的区别**

![IMG_5234](/Users/liuqiang/Downloads/IMG_5234.PNG)