---
title: 手写一个Promise
categories: JavaScript
tags: [JavaScript, Promise]
copyright_author: 崔帆
copyright_author_href: https://xxxxxx.com
copyright_url: https://xxxxxx.com
cover: /images/promise.webp
top_img: /images/promise.webp
copyright_info: 此文章版权归崔帆所有，如有转载，请注明来自原作者
---
## 1. 什么是promise

promise是JavaScript中异步编程的一种新解决方案。

从语法上来说，promise是一个构造函数。

从功能上来说，promise对象封装一个异步操作并可以获取它成功或者失败的结果值。

## 2. promise的状态

初始promise对象状态为`pending`、状态只能改变为`rejected`或者`resolved`，而且只能改变一次。无论变为成功还是失败, 都会有一个结果数据。 成功的结果数据一般称为 value, 失败的结果数据一般称为 reason。

![14-1.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2731df7a6752438ebcdf1a8a19da74ce~tplv-k3u1fbpfcp-watermark.image?)

## 3. promise的基本使用

### 3.1 基本编码

```
// 1) 创建 promise 对象(pending 状态), 指定执行器函数
const p = new Promise((resolve, reject) => {
​
  // 2) 在执行器函数中启动异步任务
  setTimeout(() => {
    const time = Date.now()
    
    // 3) 根据结果做不同处理
    // 3.1) 如果成功了, 调用 resolve(), 指定成功的 value, 变为 resolved 状 态
    if (time % 2 === 1) {
      resolve('成功的值 ' + time)
    } else { // 3.2) 如果失败了, 调用 reject(), 指定失败的 reason, 变为
      // rejected 状态
      reject('失败的值' + time)
    }
  }, 2000)
})
​
// 4) 能 promise 指定成功或失败的回调函数来获取成功的 vlaue 或失败的 reason
p.then(
  value => { // 成功的回调函数 onResolved, 得到成功的 vlaue
    console.log('成功的 value: ', value)
  },
  reason => { // 失败的回调函数 onRejected, 得到失败的 reason
    console.log('失败的 reason: ', reason)
  })
```

### 3.2 封装基于定时器的异步

```
function doDelay(time) {
  // 1. 创建 promise 对象
  return new Promise((resolve, reject) => {
    // 2. 启动异步任务
    console.log('启动异步任务')
    setTimeout(() => {
      console.log('延迟任务开始执行...')
      const time = Date.now() // 假设: 时间为奇数代表成功, 为偶数代表失败
      if (time % 2 === 1) { // 成功了
        // 3. 1. 如果成功了, 调用 resolve()并传入成功的 value
        resolve('成功的数据 ' + time)
      } else { // 失败了
        // 3.2. 如果失败了, 调用 reject()并传入失败的 reason
        reject('失败的数据 ' + time)
      }
    }, time)
  })
}
const promise = doDelay(2000)
promise.then(
  value => {
    console.log('成功的 value: ', value)
  },
  reason => {
    console.log('失败的 reason: ', reason)
  },
)
```

### 3.3 封装 ajax 异步请求

```
function promiseAjax(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.onreadystatechange = () => {
      if (xhr.readyState !== 4) return
      const {status, response} = xhr
      // 请求成功, 调用 resolve(value)
      if (status >= 200 && status < 300) {
        resolve(JSON.parse(response))
      } else { // 请求失败, 调用 reject(reason)
        reject(new Error('请求失败: status: ' + status))
      }
    }
    xhr.open("GET", url)
    xhr.send()
  })
}
promiseAjax('https://api.apiopen.top2/getJoke?page=1&count=2&type=video ')
  .then(data => {
    console.log('显示成功数据', data)
  }, error => {
    alert(error.message)
  })
```

## 4. 为什么要使用promise

### 4.1 指定回调函数的方式更加灵活

以前：必须在启动异步任务前指定回调函数

promise: 启动异步任务 => 返回promie对象 => 给promise对象绑定回调函数(甚至可以在异步任务结束后指定/多个)

### 4.2 支持链式调用，解决回调地狱

回调地狱函数嵌套调用, 外部回调函数异步执行的结果是嵌套的回调执行的条件，代码可读性和可维护性差。可以用解决promise当作解决方案，终极解决方案是`async/await`

```js
// 成功的回调函数
function successCallback(result) {
  console.log("声音文件创建成功: " + result);
}
// 失败的回调函数
function failureCallback(error) {
  console.log("声音文件创建失败: " + error);
}
​
/* 1.1 使用纯回调函数 */
createAudioFileAsync(audioSettings, successCallback, failureCallback)
​
/* 1.2. 使用 Promise */
const promise = createAudioFileAsync(audioSettings); // 2
setTimeout(() => {
  promise.then(successCallback, failureCallback);
}, 3000);
​
/*2.1. 回调地狱*/
doSomething(function(result) {
    doSomethingElse(result, function(newResult) {
      doThirdThing(newResult, function(finalResult) {
        console.log('Got the final result: ' + finalResult)
      }, failureCallback)
    }, failureCallback)
  }, failureCallback)
​
/*2.2. 使用 promise 的链式调用解决回调地狱*/
doSomething().then(function(result) {
    return doSomethingElse(result)
  })
  .then(function(newResult) {
    return doThirdThing(newResult)
  })
  .then(function(finalResult) {
    console.log('Got the final result: ' + finalResult)
  })
  .catch(failureCallback)
​
/*2.3. async/await: 回调地狱的终极解决方案*/
async function request() {
  try {
    const result = await doSomething()
    const newResult = await doSomethingElse(result)
    const finalResult = await doThirdThing(newResult)
    console.log('Got the final result: ' + finalResult)
  } catch (error) {
    failureCallback(error)
  }
}
```

## 5. API

### 5.1 Promise 构造函数

(1) executor 函数: 执行器 (resolve, reject) => {}

(2) resolve 函数: 内部定义成功时我们调用的函数 value => {}

(3) reject 函数: 内部定义失败时我们调用的函数 reason => {}

说明: executor 会在 Promise 内部立即同步调用,异步操作在执行器中执行

### 5.2 Promise.prototype.then()

(1) onResolved 函数: 成功的回调函数 (value) => {}

(2) onRejected 函数: 失败的回调函数 (reason) => {}

说明: 指定用于得到成功 value 的成功回调和用于得到失败 reason 的失败回调。返回一个新的 promise 对象。

### 5.3 Promise.prototype.catch()

(1) onRejected 函数: 失败的回调函数 (reason) => {}

说明: then()的语法糖, 相当于: then(undefined, onRejected)

### 5.4 Promise.resolve()

(1) value: 成功的数据或 promise 对象

说明: 返回一个成功/失败的 promise 对象

### 5.5 Promise.reject()

(1) reason: 失败的原因

说明: 返回一个失败的 promise 对象

### 5.6 Promise.all()

(1) promises: 包含 n 个 promise 的数组

说明: 返回一个新的 promise, 只有所有的 promise 都成功才成功, 只要有一个失败了就直接失败

### 5.7 Promise.race()

(1) promises: 包含 n 个 promise 的数组

说明: 返回一个新的 promise, 第一个完成的 promise 的结果状态就是最终的结果状态

## 6. 关键问题

### 6.1 如何改变 promise 的状态?

(1) resolve(value): 如果当前是 pending 就会变为 resolved

(2) reject(reason): 如果当前是 pending 就会变为 rejected

(3) 抛出异常: 如果当前是 pending 就会变为 rejected

### 6.2 一个 promise 指定多个成功/失败回调函数, 都会调用吗?

当 promise 改变为对应状态时都会调用

### 6.3 改变 promise 状态和指定回调函数谁先谁后?

(1) 都有可能, 正常情况下是先指定回调再改变状态, 但也可以先改状态再指定回调

(2) 如何先改状态再指定回调?

① 在执行器中直接调用 resolve()/reject()

② 延迟更长时间才调用 then()

(3) 什么时候才能得到数据?

① 如果先指定的回调, 那当状态发生改变时, 回调函数就会调用, 得到数据

② 如果先改变的状态, 那当指定回调时, 回调函数就会调用, 得到数据

### 6.4 promise.then()返回的新 promise 的结果状态由什么决定?

(1) 简单表达: 由 then()指定的回调函数执行的结果决定

(2) 详细表达:

① 如果抛出异常, 新 promise 变为 rejected, reason 为抛出的异常

② 如果返回的是非 promise 的任意值, 新 promise 变为 resolved, value 为返回的值

③ 如果返回的是另一个新 promise, 此 promise 的结果就会成为新 promise 的结果

### 6.5 promise 如何串连多个操作任务?

(1) promise 的 then()返回一个新的 promise, 可以开成 then()的链式调用

(2) 通过 then 的链式调用串连多个同步/异步任务

### 6.6 promise 异常传透?

(1) 当使用 promise 的 then 链式调用时, 可以在最后指定失败的回调,

(2) 前面任何操作出了异常, 都会传到最后失败的回调中处理

### 6.7 中断 promise 链?

(1) 当使用 promise 的 then 链式调用时, 在中间中断, 不再调用后面的回调函数

(2) 办法: 在回调函数中返回一个 pending 状态的 promise 对象

## 7. 自定义Promise

### 7.1 搭建基本结构

运行在浏览器环境：

```
(function(window) {
  function MyPromise(excutor) {
    this.PromiseState = 'pending' // 记录promise对象的状态
    this.PromiseResult = undefined // 记录promise对象的成功/失败值
    this.PromiseCallbacks = [] // 记录promise对象先指定回调后改变状态时的回调函数
    // 格式为{onResolved:xxx,onRejected:xxx}
​
    function resolve() {}
​
    function reject() {}
​
    excutor(resolve, reject)
  }
​
  MyPromise.prototype.then = function(onResolved, onRejected) {}
​
  MyPromise.prototype.catch = function(onRejected) {}
​
  MyPromise.resolve = function(value) {}
​
  MyPromise.reject = function(reason) {}
​
  MyPromise.all = function(promises) {}
  MyPromise.race = function(promises) {}
  window.MyPromise = MyPromise
})(window)
```

### 7.2 实现构造函数

改变状态：

```
function MyPromise(excutor) {
    this.PromiseState = 'pending' // 记录promise对象的状态
    this.PromiseResult = undefined // 记录promise对象的成功/失败值
    this.PromiseCallbacks = [] // 记录promise对象先指定回调后改变状态时的回调函数
    // 格式为{onResolved:xxx,onRejected:xxx}
​
    const self = this 
​
    function resolve(value) {
      if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
      self.PromiseState = 'fulfilled'
      self.PromiseResult = value
    }
​
    function reject(reason) {
      if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
      self.PromiseState = 'rejected'
      self.PromiseResult = reason
    }
​
    try {
      excutor(resolve, reject)
    } catch (e) {
      reject(e)
    }
  }
```

### 7.3 promise.then()/catch()

#### 7.3.1 先改状态后指定回调

```html
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
  <script src="./01-MyPromise.js"></script>
  <script>
    let p = new MyPromise((resolve, reject) => {
      resolve('resolve')
    })
    p.then(value => {
      console.log(value);
    })
  </script>
</body>
​
</html>
```

then的返回值为初始为pending状态的promise，根据回调函数的返回值修改该promise对象的状态：

① 回调函数返回值为promise对象，此 promise 的结果成为新 promise 的结果

② 返回值为普通值，resolve(普通值)

③ 回调函数中抛出异常，reject(错误对象)

```js
MyPromise.prototype.then = function(onResolved, onRejected) {
  return new MyPromise((resolve, reject) => {
    // 先指定状态后调用回调
    if (this.PromiseState === 'fulfilled') {
      try {
        let resolveReturn = onResolved() // 记录then中第一个回调的返回值
          // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
        if (resolveReturn instanceof MyPromise) {
          resolveReturn.then(resolve,reject)
        } else {
          // 2. 返回值为普通值（包装为Promise成功状态）
          resolve(resolveReturn)
        }
      } catch (e) {
        // 3.回调函数中出现错误，返回拒绝的Promise
        reject(e)
      }
    } else if (this.PromiseState === 'rejected') {
      try {
        let resolveReturn = onRejected() // 记录then中第一个回调的返回值
          // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
        if (resolveReturn instanceof MyPromise) {
          resolveReturn.then(resolve,reject)
        } else {
          // 2. 返回值为普通值（包装为Promise成功状态）
          resolve(resolveReturn)
        }
      } catch (e) {
        // 3.回调函数中出现错误，返回拒绝的Promise
        reject(e)
      }
    } else {
​
    }
  })
}
```

抽离公共代码：

```
MyPromise.prototype.then = function(onResolved, onRejected) {
  let self = this
​
  function callback(type) {
    try {
      let resolveReturn = type(self.PromiseResult) // 记录then中第一个回调的返回值
        // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
      if (resolveReturn instanceof MyPromise) {
        resolveReturn.then(resolve,reject)
      } else {
        // 2. 返回值为普通值（包装为Promise成功状态）
        resolve(resolveReturn)
      }
    } catch (e) {
      // 3.回调函数中出现错误，返回拒绝的Promise
      reject(e)
    }
  }
  return new MyPromise((resolve, reject) => {
    // 先指定状态后调用回调
    if (this.PromiseState === 'fulfilled') {
      callback(onResolved)
    } else if (this.PromiseState === 'rejected') {
      callback(onRejected)
    } else if (this.PromiseState === 'pending') {}
  })
}
```

#### 7.3.2 先指定回调后改变状态

```
<script>
  let p = new MyPromise((resolve, reject) => {
    setTimeout(function() {
      resolve('resolve')
    })
  }, 1000)
  p.then(value => {
    console.log(value);
  })
</script>
```

此时代码执行流会进入`else if (this.PromiseState === 'pending') {}`中：

```
MyPromise.prototype.then = function(onResolved, onRejected) {
  let self = this
​
  return new MyPromise((resolve, reject) => {
    function callback(type) {
      try {
        let resolveReturn = type(self.PromiseResult) // 记录then中第一个回调的返回值
          // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
        if (resolveReturn instanceof MyPromise) {
          resolveReturn.then(resolve,reject)
        } else {
          // 2. 返回值为普通值（包装为Promise成功状态）
          resolve(resolveReturn)
        }
      } catch (e) {
        // 3.回调函数中出现错误，返回拒绝的Promise
        reject(e)
      }
    }
​
    // 先指定状态后调用回调
    if (this.PromiseState === 'fulfilled') {
      callback(onResolved)
    } else if (this.PromiseState === 'rejected') {
      callback(onRejected)
    } else if (this.PromiseState === 'pending') {
      this.PromiseCallbacks.push({
        onResolved,
        onRejected
      })
    }
  })
}
```

等状态修改完毕后会去执行指定的回调：

```
function MyPromise(excutor) {
  this.PromiseState = 'pending' // 记录promise对象的状态
  this.PromiseResult = undefined // 记录promise对象的成功/失败值
  this.PromiseCallbacks = [] // 记录promise对象先指定回调后改变状态时的回调函数
  // 格式为{onResolved:xxx,onRejected:xxx}
​
  const self = this
​
  function resolve(value) {
    if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
    self.PromiseState = 'fulfilled'
    self.PromiseResult = value
​
    // 先指定回调后改变状态，修改完状态后就来执行相关的回调
    self.PromiseCallbacks.forEach(callback => {
      callback.onResolved(self.PromiseResult)
    })
  }
​
  function reject(reason) {
    if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
    self.PromiseState = 'rejected'
    self.PromiseResult = reason
​
    // 先指定回调后改变状态，修改完状态后就来执行相关的回调
    self.PromiseCallbacks.forEach(callback => {
      callback.onRejected(self.PromiseResult)
    })
  }
​
  try {
    excutor(resolve, reject)
  } catch (e) {
    reject(e)
  }
}
```

但是问题来了，我们怎么改变then返回的promise对象的状态呢？此时返回的promise对象为pending状态。这需要回调的执行必须在then方法内才行。修改代码如下：

```
MyPromise.prototype.then = function(onResolved, onRejected) {
  let self = this
​
  return new MyPromise((resolve, reject) => {
    function callback(type) {
      try {
        let resolveReturn = type(self.PromiseResult) // 记录then中第一个回调的返回值
          // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
        if (resolveReturn instanceof MyPromise) {
          resolveReturn.then(resolve,reject)
        } else {
          // 2. 返回值为普通值（包装为Promise成功状态）
          resolve(resolveReturn)
        }
      } catch (e) {
        // 3.回调函数中出现错误，返回拒绝的Promise
        reject(e)
      }
    }
​
    // 先指定状态后调用回调
    if (this.PromiseState === 'fulfilled') {
      callback(onResolved)
    } else if (this.PromiseState === 'rejected') {
      callback(onRejected)
    } else if (this.PromiseState === 'pending') {
      this.PromiseCallbacks.push({
        onResolved() {
          callback(onResolved)
        },
        onRejected() {
          callback(onRejected)
        }
      })
    }
  })
}
```

```
function MyPromise(excutor) {
  this.PromiseState = 'pending' // 记录promise对象的状态
  this.PromiseResult = undefined // 记录promise对象的成功/失败值
  this.PromiseCallbacks = [] // 记录promise对象先指定回调后改变状态时的回调函数
  // 格式为{onResolved:xxx,onRejected:xxx}
​
  const self = this
​
  function resolve(value) {
    if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
    self.PromiseState = 'fulfilled'
    self.PromiseResult = value
​
    // 先指定回调后改变状态，修改完状态后就来执行相关的回调
    self.PromiseCallbacks.forEach(callback => {
      callback.onResolved()
    })
  }
​
  function reject(reason) {
    if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
    self.PromiseState = 'rejected'
    self.PromiseResult = reason
​
    // 先指定回调后改变状态，修改完状态后就来执行相关的回调
    self.PromiseCallbacks.forEach(callback => {
      callback.onRejected()
    })
  }
​
  try {
    excutor(resolve, reject)
  } catch (e) {
    reject(e)
  }
}
```

catch:

```
MyPromise.prototype.catch = function(onRejected) {
  return this.then(undefined, onRejected)
}
```

写到这里，完善一下MyPromise的then方法相较于原promise没有的两个功能：`异常穿透`和`值传递`

```
<script>
let m1 = new Promise((resolve, reject) => {
  resolve(111)
})
m1.then().then(value => {
  console.log(value);
})
​
let m2 = new MyPromise((resolve, reject) => {
  resolve(222)
})
m2.then().then(value => {
  console.log(value);
}, reason => {
  console.log(reason);
})
</script>
```

输出为:

```
TypeError: type is not a function
    at callback (01-MyPromise.js:45)
    at 01-MyPromise.js:61
    at new MyPromise (01-MyPromise.js:33)
    at MyPromise.then (01-MyPromise.js:42)
    at 02-手写promise.html:86
111
```

m1会发生值传递，而m2不会。原因是m2中回调为undefined的时候，callback中调用`type(self.PromiseResult)`会发生错误，被捕捉后第一个then返会拒绝的MyPromise对象。

修改上述then代码如下:

```
MyPromise.prototype.then = function(onResolved, onRejected) {
  let self = this
​
  // 值传递
  if (typeof onResolved !== 'function') onResolved = value => value
​
  // 异常穿透
  if (typeof onRejected !== 'function') onRejected = reason => reason
  return new MyPromise((resolve, reject) => {
    function callback(type) {
      try {
        let resolveReturn = type(self.PromiseResult) // 记录then中第一个回调的返回值
          // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
        if (resolveReturn instanceof MyPromise) {
          resolveReturn.then(resolve, reject)
        } else {
          // 2. 返回值为普通值（包装为Promise成功状态）
          resolve(resolveReturn)
        }
      } catch (e) {
        // 3.回调函数中出现错误，返回拒绝的Promise
        reject(e)
      }
    }
​
    // 先指定状态后调用回调
    if (this.PromiseState === 'fulfilled') {
      callback(onResolved)
    } else if (this.PromiseState === 'rejected') {
      callback(onRejected)
    } else if (this.PromiseState === 'pending') {
      this.PromiseCallbacks.push({
        onResolved() {
          callback(onResolved)
        },
        onRejected() {
          callback(onRejected)
        }
      })
    }
  })
}
```

### 7.4 Promise.resolve()/reject()

```
MyPromise.resolve = function(value) {
  return new MyPromise((resolve, reject) => {
    if (value instanceof MyPromise) {
      value.then(resolve, reject)
    } else {
      resolve(value)
    }
  })
}
​
MyPromise.reject = function(reason) {
  return new Promise((undefined, reject) => {
    reject(reason)
  })
}
```

### 7.5 Promise.all/race()

```
MyPromise.all = function(promises) {
  return new MyPromise((resolve, reject) => {
    let fulfilledCount = 0 // 记录状态为fulfilled的promise对象的梳理
    let fulfilledResults = [] // 记录成功值
    promises.forEach((promise, index) => {
      promise.then(value => {
        fulfilledResults[index] = value
        if (++fulfilledCount === promises.length) {
          resolve(fulfilledResults)
        }
      }, reason => {
        reject(reason)
      })
    })
  })
}
​
MyPromise.race = function(promises) {
  return new MyPromise((resolve, reject) => {
    promises.forEach(promise => {
      promise.then(resolve, reject)
    })
  })
}
```

### 7.6 Promise.resolveDelay()/rejectDelay()

返回一个延迟指定时间才确定结果的 promise 对象

```
MyPromise.resolveDelay = function(value, time) {
  return new MyPromise((resolve, reject) => {
    setTimeout(function() {
      if (value instanceof MyPromise) {
        value(resolve, reject)
      } else {
        resolve(value)
      }
    })
  }, time)
}
​
MyPromise.rejectDelay = function(reason, time) {
  return new MyPromise((undefined, reject) => {
    setTimeout(function() {
      reject(reason)
    }, time)
  })
}
```

### 7.7 完整代码

#### 7.7.1 ES5版本

最后由于回调是异步执行的，所以给每个回调的执行添加setTimeout模拟异步操作

```
(function (window) {
  function MyPromise(excutor) {
    this.PromiseState = 'pending'; // 记录promise对象的状态
    this.PromiseResult = undefined; // 记录promise对象的成功/失败值
    this.PromiseCallbacks = []; // 记录promise对象先指定回调后改变状态时的回调函数,格式为{onResolved:xxx,onRejected:xxx}

    var self = this;

    function resolve(value) {
      // promise对象的状态只能修改一次
      if (self.PromiseState !== 'pending') return;
      self.PromiseState = 'fulfilled';
      self.PromiseResult = value;

      // 先指定回调后改变状态，修改完状态后就来执行相关的回调
      self.PromiseCallbacks.forEach(function () {
        setTimeout(function () {
          callback.onResolved();
        });
      });
    }

    function reject(reason) {
      // promise对象的状态只能修改一次
      if (self.PromiseState !== 'pending') return;
      self.PromiseState = 'rejected';
      self.PromiseResult = reason;

      // 先指定回调后改变状态，修改完状态后就来执行相关的回调

      self.PromiseCallbacks.forEach(function (callback) {
        setTimeout(function () {
          callback.onRejected();
        });
      });
    }

    try {
      excutor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }

  MyPromise.prototype.then = function (onResolved, onRejected) {
    var self = this;

    // 值传递
    if (typeof onResolved !== 'function') {
      onResolved = function (value) {
        return value;
      };
    }

    // 异常穿透
    if (typeof onRejected !== 'function') {
      onRejected = function (reason) {
        return reason;
      };
    }

    return new MyPromise(function (resolve, reject) {
      function callback(type) {
        try {
          var resolveReturn = type(self.PromiseResult); // 记录then中第一个回调的返回值
          // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
          if (resolveReturn instanceof MyPromise) {
            resolveReturn.then(resolve, reject);
          } else {
            // 2. 返回值为普通值（包装为Promise成功状态）
            resolve(resolveReturn);
          }
        } catch (e) {
          // 3.回调函数中出现错误，返回拒绝的Promise
          reject(e);
        }
      }

      // 先指定状态后调用回调
      if (this.PromiseState === 'fulfilled') {
        setTimeout(function () {
          callback(onResolved);
        });
      } else if (this.PromiseState === 'rejected') {
        setTimeout(function () {
          callback(onRejected);
        });
      } else if (this.PromiseState === 'pending') {
        this.PromiseCallbacks.push({
          onResolved() {
            callback(onResolved);
          },
          onRejected() {
            callback(onRejected);
          }
        });
      }
    });
  };

  MyPromise.prototype.catch = function (onRejected) {
    return this.then(undefined, onRejected);
  };

  MyPromise.resolve = function (value) {
    return new MyPromise((resolve, reject) => {
      if (value instanceof MyPromise) {
        value.then(resolve, reject);
      } else {
        resolve(value);
      }
    });
  };

  MyPromise.reject = function (reason) {
    return new MyPromise((undefined, reject) => {
      reject(reason);
    });
  };

  MyPromise.all = function (promises) {
    return new MyPromise((resolve, reject) => {
      var fulfilledCount = 0; // 记录状态为fulfilled的promise对象的梳理
      var fulfilledResults = []; // 记录成功值
      promises.forEach((promise, index) => {
        promise.then(
          function (value) {
            fulfilledResults[index] = value;
            if (++fulfilledCount === promises.length) {
              resolve(fulfilledResults);
            }
          },
          function (reason) {
            reject(reason);
          }
        );
      });
    });
  };

  MyPromise.race = function (promises) {
    return new MyPromise(function (resolve, reject) {
      promises.forEach(function (promise) {
        promise.then(resolve, reject);
      });
    });
  };

  MyPromise.resolveDelay = function (value, time) {
    return new MyPromise(function (resolve, reject) {
      setTimeout(function () {
        if (value instanceof MyPromise) {
          value(resolve, reject);
        } else {
          resolve(value);
        }
      });
    }, time);
  };

  MyPromise.rejectDelay = function (reason, time) {
    return new MyPromise((undefined, reject) => {
      setTimeout(function () {
        reject(reason);
      }, time);
    });
  };

  window.MyPromise = MyPromise;
})(window);
```

#### 7.7.2 class版本

```
class MyPromise {
  constructor(excutor) {
    this.PromiseState = 'pending' // 记录promise对象的状态
    this.PromiseResult = undefined // 记录promise对象的成功/失败值
    this.PromiseCallbacks = [] // 记录promise对象先指定回调后改变状态时的回调函数
    // 格式为{onResolved:xxx,onRejected:xxx}
​
    const self = this
​
    function resolve(value) {
      if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
      self.PromiseState = 'fulfilled'
      self.PromiseResult = value
​
      // 先指定回调后改变状态，修改完状态后就来执行相关的回调
      self.PromiseCallbacks.forEach(callback => {
        setTimeout(function() {
          callback.onResolved()
        })
      })
    }
​
    function reject(reason) {
      if (self.PromiseState !== 'pending') return // promise对象的状态只能修改一次
      self.PromiseState = 'rejected'
      self.PromiseResult = reason
​
      // 先指定回调后改变状态，修改完状态后就来执行相关的回调
      self.PromiseCallbacks.forEach(callback => {
        setTimeout(function() {
          callback.onRejected()
        })
      })
    }
​
    try {
      excutor(resolve, reject)
    } catch (e) {
      reject(e)
    }
  }
​
  then(onResolved, onRejected) {
    let self = this
​
    // 值传递
    if (typeof onResolved !== 'function') onResolved = value => value
​
    // 异常穿透
    if (typeof onRejected !== 'function') onRejected = reason => reason
    return new MyPromise((resolve, reject) => {
      function callback(type) {
        try {
          let resolveReturn = type(self.PromiseResult) // 记录then中第一个回调的返回值
            // 1. 返回值为MyPromise对象(根据该对象状态修改返回Promise对象的状态)
          if (resolveReturn instanceof MyPromise) {
            resolveReturn.then(resolve, reject)
          } else {
            // 2. 返回值为普通值（包装为Promise成功状态）
            resolve(resolveReturn)
          }
        } catch (e) {
          // 3.回调函数中出现错误，返回拒绝的Promise
          reject(e)
        }
      }
​
      // 先指定状态后调用回调
      if (this.PromiseState === 'fulfilled') {
        setTimeout(function() {
          callback(onResolved)
        })
      } else if (this.PromiseState === 'rejected') {
        setTimeout(function() {
          callback(onRejected)
        })
      } else if (this.PromiseState === 'pending') {
        this.PromiseCallbacks.push({
          onResolved() {
            callback(onResolved)
          },
          onRejected() {
            callback(onRejected)
          }
        })
      }
    })
  }
​
  catch (onRejected) {
    return this.then(undefined, onRejected)
  }
​
  static resolve(value) {
    return new MyPromise((resolve, reject) => {
      if (value instanceof MyPromise) {
        value.then(resolve, reject)
      } else {
        resolve(value)
      }
    })
  }
​
  static reject(reason) {
    return new MyPromise((undefined, reject) => {
      reject(reason)
    })
  }
​
  static resolveDelay(value, time) {
    return new MyPromise((resolve, reject) => {
      setTimeout(function() {
        if (value instanceof MyPromise) {
          value(resolve, reject)
        } else {
          resolve(value)
        }
      })
    }, time)
  }
​
  static rejectDelay(reason, time) {
    return new MyPromise((undefined, reject) => {
      setTimeout(function() {
        reject(reason)
      }, time)
    })
  }
​
  static all(promises) {
    return new MyPromise((resolve, reject) => {
      let fulfilledCount = 0 // 记录状态为fulfilled的promise对象的梳理
      let fulfilledResults = [] // 记录成功值
      promises.forEach((promise, index) => {
        promise.then(value => {
          fulfilledResults[index] = value
          if (++fulfilledCount === promises.length) {
            resolve(fulfilledResults)
          }
        }, reason => {
          reject(reason)
        })
      })
    })
  }
​
  static race(promises) {
    return new MyPromise((resolve, reject) => {
      promises.forEach(promise => {
        promise.then(resolve, reject)
      })
    })
  }
}
```