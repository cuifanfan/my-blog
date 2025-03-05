---
title: WebAPI：事件循环的应用
categories: JavaScript
tags: [JavaScript, WebAPI, 浏览器, 事件循环, Promise]
copyright_author: 崔帆
copyright_author_href: https://xxxxxx.com
copyright_url: https://xxxxxx.com
cover: /images/webapi.webp
top_img: /images/webapi.webp
copyright_info: 此文章版权归崔帆所有，如有转载，请注明来自原作者
---

## 1. 浏览器怎么实现setTimeout

渲染进程所有运行在主线程上的任务都要首先添加到消息队列，然后事件循环系统按照顺序执行消息队列中的任务。那么有哪些典型的事件呢？

-   当接收到HTML文档数据，渲染引擎就会将“解析DOM“事件添加到消息队列中
-   当用户改变了Web页面的窗口大小，渲染引擎就会将“重新布局”事件添加到消息队列中
-   当触发了JavaScript引擎垃圾回收机制，渲染引擎就会将”垃圾回收“任务添加到消息队列中
-   当执行一段异步JavaScript代码，也会把需要执行任务添加到消息队列中

回调函数是在指定的时间间隔内被调用，但是消息队列中的任务是按照顺序被执行的，所以定时器中的任务不能直接添加到消息队列中。那么怎么在消息循环系统的基础上添加定时器的功能呢？

其实在Chrome中，**除了正常使用的消息队列外，还有一个HashMap，其中维护了需要延迟执行的任务，包括定时器和Chromium内部需要延迟执行的任务**。当通过JavaScript创建一个定时器的时候，渲染进程会将该定时器中的回调任务添加到该HashMap中。

通过JavaScript调用setTimeout设置回调函数的时候，渲染进程会创建一个回调任务，包含了回调函数本身、当前发起时间、延迟执行时间。

```js
let delayedIncomingTask = new HashMap() // 创建延迟执行任务的HashMap
function DelayTask(callback,delayTime) {
    this.id = xxx // 指定一个id
    this.startTime = Date.now() // 当前发起时间
    this.delayTime = delayTime // 延迟执行时间
    this.cbf = callback // 回调函数
}
​
let timerTask = new DelayTask(function(){
    console.log('timerTask')
},1000) // 创建回调任务
​
delayedIncomingTask.push(timerTask) // 将该任务添加到延迟执行HashMap中
```

参考一下上述事件循环的代码：

```js
// 主线程 （Main Thread）
function MainThread() {
    ... // 之前的任务
    
    while(true) {
        let task = TaskQueue.dequeue()
        // 执行消息循环中的任务
        ProcessTask(task)
        // 执行延迟HashMap中的任务
        delayedIncomingTask()
        
        // 如果设置了退出标志，那么直接退出线程循环
        if(!keep_running) 
           break;
    }
}
```

从上述代码可以看出，渲染进程主线程处理完消息队列中的一个宏任务之后，就开始执行delayedIncomingTask函数，delayedIncomingTask根据startTime，和delayTime计算出到期的任务，然后依次执行该任务。执行完毕就进入下次循环过程。

浏览器内部实现取消定时器是通过 ID 查找到对应的任务，然后再将其从HashMap中删除。

## 2. 使用setTimeout的一些注意事项

### 2.1 当前任务执行过久，会影响到定时器任务的执行

```js
let hello = function() {
    console.log('hello world!')
}
​
function foo() {
    setTimeout(hello,0)
    for(let i = 0; i < 5000; i++) {
        let result = i*2
        console.log(result)
    }
}
```

打开Chrome的`Performance`查看一下执行情况:

![12-1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94b26749e44342759dbd244ec75e9c04~tplv-k3u1fbpfcp-watermark.image?)

执行 foo 函数所消耗的时长是 500 毫秒，这也就意味着通过setTimeout 设置的任务会被推迟到 500 毫秒以后再去执行，而设置 setTimeout 的回调延迟时间是 0。

### 2.2 setTimeout存在嵌套调用，系统会设置最短时间间隔为4ms

```js
setTimeout(function() {
  console.log(Date.now());
  setTimeout(function() {
    console.log(Date.now());
    setTimeout(function() {
      console.log(Date.now());
      setTimeout(function() {
        console.log(Date.now());
        setTimeout(function() {
          console.log(Date.now());
          setTimeout(function() {
            console.log(Date.now());
              // ....
          }, 1)
        }, 1)
      }, 1)
    }, 1)
  }, 1)
}, 1)
```

通过 Performance 来记录下这段代码的执行过程，如下图所示：

![12-2.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87df42d1be7c4fcea48cbb1899d7bae2~tplv-k3u1fbpfcp-watermark.image?)

嵌套调用超过五次以上，后面每次的调用最小时间间隔是 4 毫秒。这是因为在 Chrome 中，定时器被嵌套调用 5 次以上，系统会判断该函数方法被阻塞了。**如果嵌套定时器的调用时间间隔小于 4 毫秒，那么浏览器会将每次调用的时间间隔设置为4 毫秒。**

也就是说**在定时器函数里面嵌套调用定时器，也会延长定时器的执行时间**。

### 2.3 未激活的页面，setTimeout执行最小间隔是1000毫秒

如果标签不是当前的激活标签，那么定时器最小的时间间隔是 1000 毫秒，目的是为了优化后台页面的加载损耗以及降低耗电量。

### 2.4 **延时执行时间有最大值**

Chrome、Safari、Firefox 都是以 32 个 bit 来存储延时值的，32bit 最大只能存放的数字是 2147483647 毫秒，超过这个值就会溢出立即执行。

```
setTimeout(function(param) {
  console.log('hello world');
}, 2147483650)
```


![12-3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85549637b46c42e6bc93b51242e037a6~tplv-k3u1fbpfcp-watermark.image?)

### 2.5 setTimeout中的this不符合直觉

```js
const obj ={
    uname:'cuifan',
    sayHello() {
        conosle.log('hello')
    }
}
​
setTimeout(obj.sayHello)
```

输出为：

```js
undefined // this指向的是window
```

解决办法：

```
const obj ={
    uname:'cuifan',
    sayHello() {
        conosle.log('hello')
    }
}
​
setTimeout(obj.sayHello.bind(obj))
```

```
const obj ={
    uname:'cuifan',
    sayHello() {
        conosle.log('hello')
    }
}
​
setTimeout(()=>{obj.sayHello})
```

## 3. 与requestAnimationFrame的对比

在`requestAnimationFrame`之前，主要借助setTimeout和setInterval来编写动画，而动画的关键在于动画帧之间的时间间隔设置，必须**足够准确**。这个时间间隔设置很有讲究，一方面要足够小，这样动画之间帧才有连贯性。一方面要足够大，确保浏览器有足够的时间及时完成渲染。

大部分显示器的刷新率为60hz，即每秒钟`重绘`60次，大多数浏览器都会对重绘操作加以限制，使其不超过显示器的刷新频率。

setTimeout/setInterval的致命缺陷在于设定的时间并不准确，它们只是在**设定时间到达后将相应的任务添加到待执行的任务队列中**，而任务队列中前面如果还有任务尚未执行完毕，之后添加的任务就必须等待。这个等待的时间造成了原本设定的时间间隔不准。

基于上述缺陷，出现了`requestAnimationFrame`,它采用的是**系统时间间隔**（约为16.7ms），保持最佳的绘制效果与效率。使各种网页动画有一个统一的刷新机制，从而节省系统资源，提升系统性能。

MDN关于`requestAnimationFrame`的描述：

> 当你准备更新动画时你应该调用此方法。这将使浏览器在下一次重绘之前调用你传入给该方法的动画函数(即你的回调函数)。回调函数执行次数通常是每秒60次，但在大多数遵循W3C建议的浏览器中，回调函数执行次数通常与浏览器屏幕刷新次数相匹配。
>
> **注意：若你想在浏览器下次重绘之前继续更新下一帧动画，那么回调函数自身必须再次调用`window.requestAnimationFrame()`**
>
> 回调函数会被传入[`DOMHighResTimeStamp`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FDOMHighResTimeStamp)参数，[`DOMHighResTimeStamp`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FDOMHighResTimeStamp)指示当前被 `requestAnimationFrame()` 排序的回调函数被触发的时间。在同一个帧中的多个回调函数，它们每一个都会接受到一个相同的时间戳，即使在计算上一个回调函数的工作负载期间已经消耗了一些时间。该时间戳是一个十进制数，单位毫秒，最小精度为1ms(1000μs)。

```html
<svg width="800" height="600">
    <circle cx="50" cy="50" r="10" fill="red" id="myCircle1"></circle>
    <circle cx="50" cy="100" r="10" fill="red" id="myCircle2"></circle>
</svg>
<script>
  const log=console.log;
  const circle1=document.getElementById('myCircle1');
  const circle2=document.getElementById('myCircle2');
​
  let start=null;
  let count=0;//统计动画共有多少帧
  function step(timestamp,circle) {
      if(!start){
          start=timestamp;
      }
      let progress=timestamp-start;
      let dx=Math.min(progress/10,600);
      circle.setAttribute('cx',dx);
      if(progress<6000){
          count++;
          // 在浏览器下次重绘之前继续更新下一帧动画
          window.requestAnimationFrame((timestamp)=>{step(timestamp,circle)});
      }
      else{
          log(progress);
          log(count);
      }
  }
​
  window.requestAnimationFrame((timestamp)=>step(timestamp,circle1));//不会阻塞后面语句执行
</script>   
复制代码
```

以上代码执行效果就是一个svg绘制的圆形在 6000ms 内水平从左向右匀速移动，动画整体耗时 progress 为 6288.444ms，动画帧数 count 为 364，每帧之间的时间间隔为 progress/count 约为 16.7 ms


![12-4.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ae2a84fd2874178b63d860309f8fcb7~tplv-k3u1fbpfcp-watermark.image?)

还有一个要注意的地方，就是 window.requestAnimationFrame(回调函数) 不会阻塞后面语句执行，所以下段代码中通过两个 window.requestAnimationFrame(回调函数) 语句可以创造两个同时进行的动画，如图2所示。（由于共有了变量 count，所以最终其值为两个动画的总帧数。）

```
 window.requestAnimationFrame((timestamp)=>step(timestamp,circle1));//不会阻塞后面语句执行
 window.requestAnimationFrame((timestamp)=>step(timestamp,circle2));
```

## 4. XMLHttpRequest是怎么实现的

### 4.1 回调函数和系统调用栈

同步回调：回调在函数内部执行

```
let sayHello = function() {
    console.log('hello world')
}
​
function foo(callback) {
    console.log('foo start')
    callback && callback()
    console.log('foo end')
}
​
foo(sayHello)
```

异步回调：回调在函数外部执行

```
let sayHello = function() {
    console.log('hello world')
}
​
function foo(callback) {
    console.log('foo start')
    setTimeout(callback,1000)
    console.log('foo end')
}
​
foo(sayHello)
```

经过前面的介绍，你已经知道了浏览器页面是通过事件循环机制来驱动的，消息队列和主线程循环机制保证了页面有条不紊地运行。

需要指出的是，当循环系统在执行一个任务的时候，需要**为这个任务维护一个系统调用栈**，类似于JavaScript的调用栈，只不过系统调用栈是由Chromium的C++来维护的。

![12-5.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b2e28d5c8824856a5e1208a4add222f~tplv-k3u1fbpfcp-watermark.image?)

这幅图记录了一个 Parse HTML 的任务执行过程，其中黄色的条目表示执行 JavaScript 的过程，其他颜色的条目表示浏览器内部系统的执行过程。

需要说明的是，**整个Parse HTML是一个完整的任务，在执行过程中的脚本解析、样式表解析都是该任务的子过程**，其下拉的长条就是执行过程中调用栈的信息。

每个任务在执行过程中都有自己的调用栈，那么同步回调就是在当前主函数的上下文中执行回调函数，这个没有太多可讲的。下面我们主要来看看异步回调过程，异步回调是指回调函数在主函数之外执行，一般有两种方式：

1.  把异步函数做成一个任务，添加到消息队列的末尾。

<!---->

2.  把异步函数添加到微任务队列中，在当前任务的末尾执行微任务。

### 4.2 XMLHttpRequest的运作机制

具体工作过程你可以参考下图：

![12-6.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f99d57ecd044a17b9ace1b645a207f4~tplv-k3u1fbpfcp-watermark.image?)

渲染进程会将请求发送给网络进程，然后网络进程负责资源的下载，等网络进程接收到数据之后，就会利用 IPC 来通知渲染进程；**渲染进程接收到消息之后，会将xhr 的回调函数封装成任务并添加到消息队列中**，等主线程循环系统执行到该任务的时候，就会根据相关的状态来调用对应的回调函数。

下面是封装的一个XMLHttpRequest请求函数：

```
 function GetWebData(URL) {
      /**
       * 1: 新建 XMLHttpRequest 请求对象
       */
      let xhr = new XMLHttpRequest()
        /**
         * 2: 注册相关事件回调处理函数
         */
      xhr.onreadystatechange = function() {
        switch (xhr.readyState) {
          case 0: // 请求未初始化
            console.log(" 请求未初始化 ")
            break;
          case 1: //OPENED
            console.log("OPENED")
            break;
          case 2: //HEADERS_RECEIVED
            console.log("HEADERS_RECEIVED")
            break;
          case 3: //LOADING 
            console.log("LOADING")
            break;
          case 4: //DONE
            if (this.status == 200 || this.status == 304) {
              console.log(this.responseText);
            }
            console.log("DONE")
            break;
        }
      }
      xhr.ontimeout = function(e) {
        console.log('ontimeout')
      }
      xhr.onerror = function(e) {
          console.log('onerror')
        }
        /**
         * 3: 打开请求
         */
      xhr.open('Get', URL, true); // 创建一个 Get 请求, 采用异步
      /**
       * 4: 配置参数
       */
      xhr.timeout = 3000 // 设置 xhr 请求的超时时间
      xhr.responseType = "text" // 设置响应返回的数据格式
      xhr.setRequestHeader("X_TEST", "time.geekbang") // 添加自己专用的请求头属性
​
      /**
       * 5: 发送请求
       */
      xhr.send();
}
```

responseType的几种格式:


![12-7.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75635891e22d45668c15fd03bc162ac1~tplv-k3u1fbpfcp-watermark.image?)

### 4.3 XMLHttpRequest使用过程的问题

#### 4.3.1 跨域

```
<!DOCTYPE html>
<html lang="en">
​
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
​
<body>
  <script>
    function callOtherDomain(url) {
      // 1.创建xhr请求对象
      let xhr = new XMLHttpRequest()
        // 2.打开请求
      xhr.open('GET', url, true)
        // 3.回调函数
      xhr.onreadystatechange = function() {
          // 0 未初始化 1 opened 2 HEADERS_RECEIVED 3 Loading 4 Received 
          if (this.readyState === 4) {
            if (this.status >= 200 && this.status < 300) {
              console.log(this.responseText);
            }
          }
        }
        // 4.发送请求
      xhr.send()
    }
    callOtherDomain('http://www.baidu.com')
  </script>
</body>
​
</html>
```


![12-8.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5bf00726f744da4a585364aa4cdf7b2~tplv-k3u1fbpfcp-watermark.image?)

狭义的同源就是指，域名、协议、端口均为相同。不同则为跨域

#### 4.3.2 HTTPS 内容混合

HTTPS 混合内容是 HTTPS 页面中包含了**不符合 HTTPS 安全要求的内容**，比如包含了 HTTP 资源，通过 HTTP 加载的图像、视频、样式表、脚本等，都属于混合内容。

通常，如果 HTTPS 请求页面中使用混合内容，浏览器会针对 HTTPS 混合内容显示警告，用来向用户表明此 HTTPS 页面包含不安全的资源。


![12-9.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1dc1fdfe2c7495aa128a38c6b267605~tplv-k3u1fbpfcp-watermark.image?)

通过 HTML 文件加载的混合资源，虽然给出警告，但大部分类型还是能加载的。

而使用 XMLHttpRequest 请求时，浏览器认为这种请求可能是攻击者发起的，会阻止此类危险的请求，比如下图：

![12-10.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9ef5a7b6c6b45b0b199452d8fae56fe~tplv-k3u1fbpfcp-watermark.image?)