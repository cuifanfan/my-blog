---
title: JavaScript中的异步解决方案：async/await
categories: JavaScript
tags: [JavaScript, Promise, 异步, Async/await]
copyright_author: 崔帆
copyright_author_href: https://xxxxxx.com
copyright_url: https://xxxxxx.com
cover: /images/callbackhell.webp
top_img: /images/callbackhell.webp
copyright_info: 此文章版权归崔帆所有，如有转载，请注明来自原作者
---
Promise虽然使得异步编程更加线性化，但是代码里面包含了大量的 then 函数，使得代码依然不是太容易阅读。基于这个原因，ES7 引入了 async/await，这是 JavaScript 异步编程的一个重大改进，**提供了在不阻塞主线程的情况下使用同步代码实现异步访问资源的能力**，并且使得代码逻辑更清晰。

```
fetch('https://www.baidu.org')
  .then((response) => {
    console.log(response)
    return fetch('https://www.baidu.org/test')
  }).then((response) => {
    console.log(response)
  }).catch((error) => {
    console.log(error)
  })
```

改为await-async:

```
async function foo() {
  try {
    let response1 = await fetch('https://www.baidu.org')
    console.log('response1')
    console.log(response1)
    let response2 = await fetch('https://www.baidu.org/test')
    console.log('response2')
    console.log(response2)
  } catch (err) {
    console.error(err)
  }
}
```

通过上面代码，你会发现整个异步处理的逻辑都是使用同步代码的方式来实现的，而且还支持 try catch 来捕获异常，这就是完全在写同步代码，所以是非常符合人的线性思维的。

因此本篇文章会介绍JavaScript 引擎是如何实现 async/await 的。如果上来直接介绍 async/await 的使用方式的话，那么你可能会有点懵，所以我们就从其最底层的技术点一步步往上讲解，从而带你彻底弄清楚 async 和 await 到底是怎么工作的。

本文我们首先介绍生成器（Generator）是如何工作的，接着讲解 Generator 的底层实现机制——协程（Coroutine）；又因为 async/await 使用了 Generator 和 Promise 两种技术，所以紧接着我们就通过 Generator 和 Promise 来分析 async/await 到底是如何以同步的方式来编写异步代码的。

## 1. **生成器** **VS** **协程**

**生成器函数是一个带星号函数，而且是可以暂停执行和恢复执行的**。我们可以看下面这段代码：

```
function* genDemo() {
  console.log(" 开始执行第一段 ")
  yield 'generator 2'
  console.log(" 开始执行第二段 ")
  yield 'generator 2'
  console.log(" 开始执行第三段 ")
  yield 'generator 2'
  console.log(" 执行结束 ")
  return 'generator 2'
}
​
console.log('main 0')
​
let gen = genDemo()
console.log(gen.next().value)
console.log('main 1')
console.log(gen.next().value)
console.log('main 2')
console.log(gen.next().value)
console.log('main 3')
console.log(gen.next().value)
console.log('main 4')
```

执行上面这段代码，观察输出结果，你会发现函数 genDemo 并不是一次执行完的，全局代码和 genDemo 函数交替执行。其实这就是生成器函数的特性，可以暂停执行，也可以恢复执行。下面我们就来看看生成器函数的具体使用方式：

1.  在生成器函数内部执行一段代码，如果遇到 yield 关键字，那么 JavaScript 引擎将返回关键字后面的内容给外部，并暂停该函数的执行。

<!---->

2.  外部函数可以通过 next 方法恢复函数的执行，并且传入next的参数会被赋值给yeild。

关于JavaScript 引擎 V8 是如何实现一个函数的暂停和恢复的，你首先要了解协程的概念。

**协程是一种比线程更加轻量级的存在**。你可以把协程看成是跑在线程上的任务，一个线程上可以存在多个协程，但是在线程上同时只能执行一个协程，比如当前执行的是 A 协程，要启动 B 协程，那么 A 协程就需要将主线程的控制权交给 B 协程，这就体现在 A 协程暂停执行，B 协程恢复执行；同样，也可以从 B 协程中启动 A 协程。通常，**如果从 A 协程启动 B 协程，我们就把 A 协程称为B 协程的父协程**。

最重要的是，**协程不是被操作系统内核所管理，而完全是由程序所控制**（也就是在用户态执行）。这样带来的好处就是性能得到了很大的提升，不会像线程切换那样消耗资源。

下面的“协程执行流程图”，可以让你更好地理解协程是怎么执行的：


![15-1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d0eea7b5c9f421ca58de2f473b5e517~tplv-k3u1fbpfcp-watermark.image?)

从图中可以看出来协程的四点规则：

1.  通过调用生成器函数 genDemo 来创建一个协程 gen，创建之后，gen 协程并没有立即执行。

<!---->

2.  要让 gen 协程执行，需要通过调用 gen.next。

<!---->

3.  当协程正在执行的时候，可以通过 yield 关键字来暂停 gen 协程的执行，并返回主要信息给父协程。

<!---->

4.  如果协程在执行期间，遇到了 return 关键字，那么 JavaScript 引擎会结束当前协程，并将 return 后面的内容返回给父协程。

不过，对于上面这段代码，你可能又有这样疑问：父协程有自己的调用栈，gen 协程时也有自己的调用栈，当 gen 协程通过 yield 把控制权交给父协程时，V8 是如何切换到父协程的调用栈？当父协程通过 gen.next 恢复 gen 协程时，又是如何切换 gen 协程的调用栈？

要搞清楚上面的问题，你需要关注以下两点内容。

第一点：gen 协程和父协程是在主线程上交互执行的，并不是并发执行的，它们之前的切换是通过 yield 和 gen.next 来配合完成的。

第二点：当在 gen 协程中调用了 yield 方法时，JavaScript 引擎会保存 gen 协程当前的调用栈信息，并恢复父协程的调用栈信息。同样，当在父协程中执行 gen.next 时，JavaScript 引擎会保存父协程的调用栈信息，并恢复 gen 协程的调用栈信息。为了直观理解父协程和 gen 协程是如何切换调用栈的，你可以参考下图：

![15-2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5e96a5e84df441a820c5e00dee8df84~tplv-k3u1fbpfcp-watermark.image?)

其实在 JavaScript 中，生成器就是协程的一种实现方式，这样相信你也就理解什么是生成器了。那么接下来，我们使用生成器和Promise 来改造开头的那段 Promise 代码。改造后的代码如下所示：

```
//foo 函数
function* foo() {
  let response1 = yield Promise.resolve(111)
  console.log('response1')
  console.log(response1)
  let response2 = yield Promise.resolve(222)
  console.log('response2')
  console.log(response2)
}
​
// 执行 foo 函数的代码
​
let gen = foo()
let count = 0
​
function getGenPromise(gen) {
  return gen.next(++count).value
}
​
getGenPromise(gen).then((response) => {
  console.log('response1')
  console.log(response)
  return getGenPromise(gen)
}).then((response) => {
  console.log('response2')
  console.log(response)
  return getGenPromise(gen)
})
```

1.  首先执行的是let gen = foo()，创建了 gen 协程。
1.  然后在父协程中通过执行 gen.next 把主线程的控制权交给 gen 协程。
1.  gen 协程获取到主线程的控制权后，就调用 fetch 函数创建了一个 Promise 对象response1，然后通过 yield 暂停 gen 协程的执行，并将 response1 返回给父协程。
1.  父协程恢复执行后，调用 response1.then 方法等待请求结果。
1.  等通过 fetch 发起的请求完成之后，会调用 then 中的回调函数，then 中的回调函数拿到结果之后，通过调用 gen.next 放弃主线程的控制权，将控制权交 gen 协程继续执行下个请求。

以上就是协程和 Promise 相互配合执行的一个大致流程。不过通常，我们把执行生成器的代码封装成一个函数，并把这个执行生成器代码的函数称为**执行器**（可参考著名的 co 框架），如下面这种方式：

```
function* foo() {
  let response1 = yield Promise.resolve(111)
  console.log('response1')
  console.log(response1)
​
  let response2 = yield Promise.resolve(222)
  console.log('response2')
  console.log(response2)
}
co(foo());
```

通过使用生成器配合执行器，就能实现使用同步的方式写出异步代码了，这样也大大加强了代码的可读性。

## 2. async/await

虽然生成器已经能很好地满足我们的需求了，但是程序员的追求是无止境的，这不又在ES7 中引入了 async/await，这种方式能够彻底告别执行器和生成器，实现更加直观简洁的代码。其实 async/await 技术背后的秘密就是 Promise 和生成器应用，往低层说就是微任务和协程应用。

### 2.1 async

根据 MDN 定义，async 是一个通过**异步执行**并**隐式返回 Promise** 作为结果的函数。

关于异步执行的原因，我们一会儿再分析。这里我们先来看看是如何隐式返回 Promise的，你可以参考下面的代码：

```
async function foo() {
  return 2
}
console.log(foo()) // Promise {<resolved>: 2}
```

### 2.2 await

```
async function foo() {
  console.log(1)
  let a = await 100
  console.log(a)
  console.log(2)
}
console.log(0)
foo()
console.log(3)
```

观察上面这段代码，你能判断出打印出来的内容是什么吗？这得先来分析 async 结合await 到底会发生什么。在详细介绍之前，我们先站在协程的视角来看看这段代码的整体执行流程图:


![15-3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba28ee79df6846f489b1dbd1c8bc48b9~tplv-k3u1fbpfcp-watermark.image?)

结合上图，我们来一起分析下 async/await 的执行流程。

1.  首先，执行`console.log(0)`这个语句，打印出来 0。

1.  接着就是执行 foo 函数，由于 foo 函数是被 async 标记过的，所以当进入该函数(协程)的时候，JavaScript 引擎会保存当前的调用栈等信息，然后执行 foo 函数中的`console.log(1)`语句，并打印出 1。

1.  接下来就到`let a = await 100`，这里是我们分析的重点J，avaScript 引擎在背后为我们默默做了太多的事情：

    (1) JavaScript引擎会默认创建一个 Promise 对象 **_promise**，也就是对await后的值调用`Promise.resolve(100)`。因此_promise对象的resolve(100)任务会被加入当前宏任务的微任务队列。

    (2) 然后 JavaScript 引擎会暂停当前协程的执行，将主线程的控制权转交给父协程执行，同时**会将 _promise 对象返回给父协程。**

    (3) 主线程的控制权已经交给父协程了，这时候父协程会调用 _promise.then 来监控 _promise 状态的改变。

    (4) 接下来继续执行父协程的流程，执行console.log(3)。随后父协程将执行结束，在结束之前，会进入微任务的检查点，然后执行微任务队列，微任务队列中有`resolve(100)`的任务等待执行，执行到这里的时候，会触发 _promise.then 中的回调函数。

    (5) 该回调函数被激活以后，会将主线程的控制权交给 foo 函数的协程，并同时将 value 值传给该协程，赋值给await

    (6) foo 协程激活之后，会把刚才的 value 值赋给了变量 a，然后 foo 协程继续执行后续语句，执行完成之后，将控制权归还给父协程。

最后，留一个课后作业给你，分析下这段代码的输出是什么？

```
async function foo() {
  console.log('foo')
}
async function bar() {
  await foo()
  console.log('bar end')
}
console.log('script start')
setTimeout(function() {
  console.log('setTimeout')
}, 0)
bar();
new Promise(function(resolve) {
  console.log('promise executor')
  resolve();
}).then(function() {
  console.log('promise then')
})
console.log('script end')
```

