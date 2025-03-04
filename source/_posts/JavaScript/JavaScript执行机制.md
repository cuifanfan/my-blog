---
title: JavaScript执行机制
categories: JavaScript
tags: [JavaScript, 原型链, 闭包, this, 作用域链, 变量提升]
copyright_author: 崔帆
copyright_author_href: https://xxxxxx.com
copyright_url: https://xxxxxx.com
cover: /images/js.webp
top_img: /images/js.webp
copyright_info: 此文章版权归崔帆所有，如有转载，请注明来自原作者
---

## 1. 原型与原型链

```
function Person() {
    
}
const person = new Person()
person.name = 'cuifanfan'
console.log(person.name) // cuifanfan
```

在这个例子中，Person 就是一个构造函数，我们使用 new 创建了一个实例对象 person。

### 1.1 prototype

每个函数都有一个`prototype`属性，就是我们经常在各种例子中看到的那个 `prototype`，它指向了一个对象，这个对象正是调用该构造函数而创建的**实例**的原型。

什么是原型呢？你可以这样理解：每一个JavaScript对象(null除外)在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型"继承"属性。

```js
function Person() {
​
}
​
// prototype是函数才会有的属性
Person.prototype.name = 'cuifanfan'
const person1 = new Person()
const person2 = new Person()
console.log(person1.name); // cuifanfan
console.log(person2.name); // cuifanfan 
```

![09-1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe472cb4f0234be08225f4d6a2dbe4ee~tplv-k3u1fbpfcp-watermark.image?)
现在你知道了函数和`prototype`的关系，那new 函数创建出的实例和`prototype`的关系该如何描述呢？所谓的“继承”又是如何实现的呢？这就要关注接下来的这个属性：

### 1.2 `__proto__`

每一个JavaScript对象(除了 null )都具有的一个属性，叫**proto**，这个属性会指向该对象的原型。

```js
function Person() {
​
}
var person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```

于是我们更新下关系图：
![09-2.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06625863697a4d779c71fa0b9dbf888e~tplv-k3u1fbpfcp-watermark.image?)

既然实例对象和构造函数都可以指向原型，那么原型是否有属性指向构造函数或者实例呢？

### 1.3 constructor

指向实例倒是没有，因为一个构造函数可以生成多个实例，但是原型指向构造函数倒是有的：`constrcutor`。每个原型都有一个`constructor`属性指向关联的构造函数。

```js
function Person() {}
console.log(Person === Person.prototype.constructor); // true
```

所以再更新下关系图：
![09-3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96fb316455c24bde97b226c2e15f8c28~tplv-k3u1fbpfcp-watermark.image?)
那么你肯定也有一个疑问，这个属性到底有什么用呢？其实这个属性可以说是一个历史遗留问题，在大部分情况下是没用的，在我的理解里，我认为他有两个作用：

-   让实例对象知道是什么函数构造了它
-   如果想给某些类库中的构造函数增加一些自定义的方法，就可以通过 `xx.constructor.method` 来扩展

综上我们已经了解了构造函数、实例原型、和实例之间的关系，接下来我们讲讲实例和原型的关系：

### 1.4 实例与原型

当读取实例的属性时，如果找不到，就会查找与对象关联的原型中的属性，如果还查不到，就去找原型的原型，一直找到最顶层为止。

```js
function Person() {}
​
Person.prototype.name = 'cuifanfan';
​
var person = new Person();
​
person.name = 'simon';
console.log(person.name) // simon
​
delete person.name;
console.log(person.name) // cuifanfan
```

### 1.5 原型的原型

在前面，我们已经讲了原型也是一个对象，既然是对象，我们就可以用最原始的方式创建它，那就是：

```js
var obj = new Object();
obj.name = 'cuifanfan'
console.log(obj.name) // cuifanfan
```

其实原型对象就是通过 Object 构造函数生成的，结合之前所讲，实例的 **proto** 指向构造函数的 prototype ，所以我们再更新下关系图：
![09-4.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a84768bd4b04fc2a465b9f76d3f115d~tplv-k3u1fbpfcp-watermark.image?)

### 1.6 原型链

那 Object.prototype 的原型呢？

null，我们可以打印：

```
console.log(Object.prototype.__proto__ === null) // true
```

然而 null 究竟代表了什么呢？

引用阮一峰老师就是：

> null 表示“没有对象”，即该处不应该有值。

所以 Object.prototype.**proto** 的值为 null 跟 Object.prototype 没有原型，其实表达了一个意思。所以查找属性的时候查到 Object.prototype 就可以停止查找了。

最后一张关系图也可以更新为：
![09-5.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6ee481cc4234191b552d0215b5046f7~tplv-k3u1fbpfcp-watermark.image?)


### 1.7 补充说明

#### constructor

当获取 person.constructor 时，其实 person 中并没有 constructor 属性,当不能读取到constructor 属性时，会从 person 的原型也就是 Person.prototype 中读取，正好原型中有该属性，所以：

```js
person.constructor === Person.prototype.constructor
```

#### `__proto__`

其次是 `__proto__` ，绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于 Person.prototype 中，实际上，它是来自于 `Object.prototype`，与其说是一个属性，不如说是一个 getter/setter，当使用`obj.__proto__` 时，可以理解成返回了 `Object.getPrototypeOf(obj)`。

#### 真的是继承吗？

引用《你不知道的JavaScript》中的话，就是：

> 继承意味着复制操作，然而 JavaScript 默认并不会复制对象的属性，相反，JavaScript 只是在两个对象之间创建一个关联，这样，一个对象就可以通过委托访问另一个对象的属性和函数，所以与其叫继承，委托的说法反而更准确些。

#### `Function.__proto__` === Function.prototype?

所有对象都可以通过原型链最终找到 `Object.prototype` ，虽然 `Object.prototype` 也是一个对象，但是这个对象却不是 `Object` 创造的，而是引擎自己创建了 `Object.prototype` 。**所以可以这样说，所有实例都是对象，但是对象不一定都是实例**。

接下来我们来看 `Function.prototype` 这个特殊的对象，如果你在浏览器将这个对象打印出来，会发现这个对象其实是一个函数。

我们知道函数都是通过 `new Function()` 生成的，难道 `Function.prototype` 也是通过 `new Function()` 产生的吗？

**答案也是否定的，这个函数也是引擎自己创建的。首先引擎创建了 `Object.prototype` ，然后创建了 `Function.prototype` ，并且通过 `__proto__` 将两者联系了起来**。

**所以我们又可以得出一个结论，不是所有函数都是 `new Function()` 产生的**。有了 `Function.prototype` 以后才有了 `function Function()` ，然后其他的构造函数都是 `function Function()` 生成的。

现在可以来解释 `Function.__proto__ === Function.prototype` 这个问题了。

个人理解是：其他所有的构造函数都可以通过原型链找到 `Function.prototype` ，并且 `function Function()` 本质也是一个函数，为了不产生混乱就将 `function Function()` 的 `__proto__` 联系到了 `Function.prototype` 上。

最后补充一点：`Function.prototype`是引擎创建出来的，引擎认为不需要给这个对象添加 `prototype` 属性。

总结：

1.  所有对象都可以通过`__proto__`找到`Object.prototype`。
1.  所有函数都可以通过`__proto__`找到`Function.prototype`。
1.  `Object.prototype`和`Function.prototype`是两个特殊的对象，它们由引擎来创建。
1.  除了上面两个特殊对象，其余对象都是构造器`new`出来的。
1.  函数的`prototype`是一个对象，也就是原型。
1.  对象的`__proto__`指向原型，`__proto__`将对象和原型连接起来组成了原型链。

## 2. 词法作用域和动态作用域

### 2.1 作用域

作用域是指程序源代码中定义变量的区域。

作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。

JavaScript 采用词法作用域(lexical scoping)，也就是静态作用域。

### 2.2 静态作用域与动态作用域

因为 JavaScript 采用的是词法作用域，函数的作用域在函数定义的时候就决定了。

而与词法作用域相对的是动态作用域，函数的作用域是在函数调用的时候才决定的。

```js
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();

// 结果是 ???
```

前面我们已经说了，JavaScript采用的是静态作用域，所以这个例子的结果是 1。

### 2.3 动态作用域

也许你会好奇什么语言是动态作用域？

bash 就是动态作用域，不信的话，把下面的脚本存成例如 scope.bash，然后进入相应的目录，用命令行执行 `bash ./scope.bash`，看看打印的值是多少。

```js
value=1
function foo () {
    echo $value;
}
function bar () {
    local value=2;
    foo;
}
bar
```

### 2.4 思考题

让我们看一个《JavaScript权威指南》中的例子：

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```

猜猜两段代码各自的执行结果是多少？

两段代码都会打印：`local scope`。原因也很简单，因为JavaScript采用的是词法作用域，函数的作用域基于函数创建的位置。

但是在这里真正值得思考的是：

虽然两段代码执行的结果一样，但是两段代码究竟有哪些不同呢？

如果要回答这个问题，就要牵涉到很多的内容，词法作用域只是其中的一小部分，需要你继续向下阅读。

## 3. 变量提升

你觉得下面这段代码输出的结果是什么？

```js
 showName()
 console.log(myname)

 var myname = 'cuifanfan'

 function showName() {
   console.log('函数 showName 被执行');
 }
```

实际执行结果却并非顺序执行：

```js
函数 showName 被执行
undefined
```

通过上面的执行结果，你应该已经知道了函数或者变量可以在定义之前使用，那如果使用没有定义的变量或者函数，JavaScript 代码还能继续执行吗?

答案是否定的，JavaScript 引擎会报错。

但同样的方式，变量和函数的处理结果为什么不一样？比如上面的执行结果，提前使用的showName 函数能打印出来完整结果，但是提前使用的 myname 变量值却是undefined。要解释这两个问题，你就需要先了解下什么是变量提升。

**所谓的变量提升，是指在 JavaScript 代码执行过程中，JavaScript 引擎把变量的声明部分和函数的声明部分提升到代码开头的“行为”。变量被提升后，会给变量设置默认值，这个默认值就是我们熟悉的 undefined**。

下面我们来模拟下实现：

```js
// 声明部分，可以看出，函数变量提升优先级高于变量
function showName() {
  console.log('函数 showName 被执行');
}

var myname = undefined

// 可执行代码部分
showName()
console.log(myname)
myname = 'cuifanfan'
```

通过这段模拟的变量提升代码，相信你已经明白了可以在定义之前使用变量或者函数的原因——**函数和变量在执行之前都提升到了代码开头**。

### 3.1 JavaScript代码执行流程

从概念的字面意义上来看，“变量提升”意味着变量和函数的声明会在物理层面移动到代码的最前面，正如我们所模拟的那样。但，这并不准确。**实际上变量和函数声明在代码里的位置是不会改变的，而且是在编译阶段被 JavaScript 引擎放入内存中**。

一段JavaScript 代码在执行之前需要被 JavaScript 引擎编译，**编译**完成之后，才会进入**执行**阶段。

### 3.2 预编译阶段

大致流程你可以参考下图：
![09-6.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf99ae62bf4643489a11dc93c71dc84a~tplv-k3u1fbpfcp-watermark.image?)

从上图可以看出，输入一段代码，经过编译后，会生成两部分内容：**执行上下文（Execution context）和可执行代码**。

**执行上下文是 JavaScript 执行一段代码时的运行环境**，比如调用一个函数，就会进入这个函数的执行上下文，确定该函数在执行期间用到的诸如 this、变量、对象以及函数等。

在执行上下文中存在一个**变量环境的对象**（Viriable Environment），该对象中保存了变量提升的内容，比如上面代码中的变量myname 和函数 showName，都保存在该对象中。

你可以简单地把变量环境对象看成是如下结构：

```js
VariableEnvironment:
  myname -> undefined, 
  showName ->function : {console.log(myname)
```

我们可以一行一行来分析上述代码：

> 第 1 行和第 2 行，由于这两行代码不是声明操作，所以 JavaScript 引擎不会做任何处理；
>
> 第 3 行，这行是经过 var 声明的，因此 JavaScript 引擎将在环境对象中创建一个名为 myname 的属性，并使用undefined初始化；
>
> 第 4 行，JavaScript 引擎发现了一个通过 function 定义的函数，所以它将函数定义存储到堆 (HEAP）中，并在环境对象中创建一个 showName 的属性，然后将该属性值指向堆中函数的位置。

这样就生成了变量环境对象。接下来 JavaScript 引擎会把声明以外的代码**编译为字节码**(之前都是预编译的操作)，至于字节码的细节，你可以类比如下的模拟代码：

```js
showName()
console.log(myname)
myname = 'cuifanfan'
```

现在有了执行上下文和可执行代码了，那么接下来就到了执行阶段了。

### 3.3 执行阶段

JavaScript 引擎开始执行“可执行代码”，按照顺序一行一行地执行：

> 1.  当执行到 showName 函数时，JavaScript 引擎便开始在变量环境对象中查找该函数，由于变量环境对象中存在该函数的引用，所以 JavaScript 引擎便开始执行该函数，并输出“函数 showName 被执行”结果。
> 1.  接下来打印“myname”信息，JavaScript 引擎继续在变量环境对象中查找该对象，由于变量环境存在 myname 变量，并且其值为 undefined，所以这时候就输出undefined。
> 1.  接下来执行第 3 行，把“cuifanfan”赋给 myname 变量，赋值后变量环境中的myname 属性值改变为“cuifanfan”，变量环境如下所示：

```
1 VariableEnvironment:
2 myname -> " cuifanfan ", 
3 showName ->function : {console.log(myname)
```

好了，以上就是一段代码的编译和执行流程。实际上，编译阶段和执行阶段都是非常复杂的，包括了词法分析、语法解析、代码优化、代码生成等，这些内容会在后续介绍。

另外，**一段代码如果定义了两个相同名字的函数，那么最终生效的是最后一个函数**。

## 4. 执行上下文栈

### 4.1 可执行代码

前面我们讲到，当一段代码被执行时，JavaScript 引擎先会对其进行编译，并创建执行上下文。但是并没有明确说明到底什么样的代码才算符合规范。其实很简单，就三种，**全局代码、函数代码、eval代码**。

1.  当 JavaScript 执行全局代码的时候，会编译全局代码并创建全局执行上下文，在整个页面的生存周期内，全局执行上下文只有一份。
1.  当**调用**一个函数的时候，函数体内的代码会被编译，并创建函数执行上下文，一般情况下，函数执行结束之后，创建的函数执行上下文会被销毁。
1.  当使用 eval 函数的时候，eval 的代码也会被编译，并创建执行上下文。

### 4.2 函数调用

```js
1 var a = 2
2 function add(){
3 	var b = 10
4 	return a+b
5 }
6 add()
```

执行到函数 add() 之前，JavaScript 引擎会为上面这段代码创建**全局执行上下文**，包含了声明的函数和变量。此时还没有创建函数执行上下文，函数只有被调用才会进行编译、创建执行上下文。

![09-7.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e5b53d0d81e403abb5b026fd1da28c3~tplv-k3u1fbpfcp-watermark.image?)

执行上下文准备好之后，便开始执行全局代码，当执行到 add 这儿时，JavaScript 判断这是一个函数调用，那么将执行以下操作:

1.  首先，从**全局执行上下文**中，取出 add 函数代码。
1.  其次，对 add 函数的这段代码进行编译，并创建**该函数的执行上下文**和**可执行代码**。
1.  最后，执行代码，输出结果。

### 4.3 调用栈

在调用add函数的时候，我们就有了两个执行上下文，接下来问题来了，我们写的函数多了去了，如何管理创建的那么多执行上下文呢？所以 JavaScript 引擎创建了**执行上下文栈（** Execution context stack，ECS）来管理执行上下文。当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出。

好了，现在你应该知道了**调用栈是 JavaScript 引擎追踪函数执行的一个机制**，当一次有多个函数被调用时，通过调用栈就能够追踪到哪个函数正在被执行以及各函数之间的调用关系。

**如何利用浏览器查看调用栈的信息**

你可以打开“开发者工具”，点击“Source”标签，选择 JavaScript 代码的页面，然后在第 3 行加上断点，并刷新页面。你可以看到执行到 add 函数时，执行流程就暂停了，这时可以通过右边“callstack”来查看当前的调用栈的情况，如下图：

![09-8.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3780ec7435b34a0f93abd1bdcadbe1ba~tplv-k3u1fbpfcp-watermark.image?)
你还可以使用 console.trace() 来输出当前的函数调用关系:


![09-9.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac96b6dc29574c0b8408638d51dad8f1~tplv-k3u1fbpfcp-watermark.image?)

现在我们已经了解了执行上下文栈是如何处理执行上下文的,所以让我们看看上节最后的问题：

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

```js
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```

两段代码执行的结果一样，但是两段代码究竟有哪些不同呢？答案就是执行上下文栈的变化不一样。让我们模拟第一段代码：

```
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop();
ECStack.pop();
```

```
ECStack.push(<checkscope> functionContext);
ECStack.pop();
ECStack.push(<f> functionContext);
ECStack.pop();
```

## 5. 块级作用域与词法环境

在 ES6 之前，ES 的作用域只有两种：全局作用域和函数作用域。相较而言，其他语言则都普遍支持**块级作用域**。块级作用域就是使用一对大括号包裹的一段代码，比如函数、判断语句、循环语句，甚至单独的一个{}都可以被看作是一个块级作用域。

和 Java、C/C++ 不同，**ES6 之前是不支持块级作用域的**，因为当初设计这门语言的时候，并没有想到 JavaScript 会火起来，所以只是按照最简单的方式来设计。没有了块级作用域，再把作用域内部的变量统一提升无疑是最快速、最简单的设计，不过这也直接导致了函数中的变量无论是在哪里声明的，在编译阶段都会被提取到执行上下文的变量环境中，所以这些变量在整个函数体内部的任何地方都是能被访问的，这也就是 JavaScript 中的变量提升。

### 5.1 变量提升带来的问题

#### 5.1.1 变量容易在不被察觉的情况下被覆盖掉

```js
var myname = " 极客时间 "
function showName(){
  console.log(myname);
 if(0){
   var myname = " 极客邦 "
}
  console.log(myname);
} 
showName()
```

打印结果为undefined，是不是很奇怪？其实就是变量提升造成的后果。

![09-10.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ae36814fc384c16a409f35518a9a0c7~tplv-k3u1fbpfcp-watermark.image?)
在函数执行过程中，JavaScript 会优先从当前的执行上下文中查找变量，由于变量提升，当前的执行上下文中就包含了变量 myname，而值是 undefined，所以获取到的 myname 的值就是 undefined。

#### 5.1.2 本应销毁的变量没有及时销毁

```js
function foo() {
  for (var i = 0; i < 7; i++) {}
  console.log(i);
}
foo()
```

最后打印结果为7，原因就是i没有销毁。这依旧和其他支持块级作用域的语言表现是不一致的，所以必然会给一些人造成误解。

### 5.2 **JavaScript** 是如何支持块级作用域的

为了解决这些问题，**ES6 引入了 let 和const 关键字**，从而使 JavaScript 也能像其他语言一样拥有了块级作用域。不过你是否有过这样的疑问：“在同一段代码中，ES6 是如何做到既要支持变量提升的特性，又要支持块级作用域的呢？”

你已经知道 **JavaScript 引擎是通过变量环境实现函数级作用域的**，那么 ES6 又是如何在函数级作用域的基础之上，实现对块级作用域的支持呢？你可以先看下面这段代码：

```js
function foo() {
  var a = 1
  let b = 2
  {
    let b = 3
    var c = 4
    let d = 5
    console.log(a)
    console.log(b)
  }
  console.log(b)
  console.log(c)
  console.log(d)
}
foo()
```

**第一步是编译并创建执行上下文**，如图：


![09-11.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77b31e2f26f546aa823bed2b4c99143f~tplv-k3u1fbpfcp-watermark.image?)

通过上图我们可以发现：

> 1.  函数内部通过 var 声明的变量，在编译阶段全都被存放到**变量环境**里面了。
> 1.  通过 let 声明的变量，在编译阶段会被存放到**词法环境**(Lexical Environment)中
> 1.  在函数的作用域内部，通过 let 声明的变量并没有被存放到词法环境中。

这里提一下，从第二点看出，let其实也是有变量提升的。只是let声明的变量，在被赋值前不能够访问。这也就是所谓的“暂时性死区”。

接下来，**第二步继续执行代码**，当执行到代码块里面时，变量环境中 a 的值已经被设置成了 1，词法环境中 b 的值已经被设置成了 2，这时候函数的执行上下文就如下图所示：


![09-12.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d9ae7e7abe944bebf0f96dcc010f2f0~tplv-k3u1fbpfcp-watermark.image?)

其实，在词法环境内部，维护了一个**小型栈结构**，栈底是函数最外层的变量，进入一个作用域块后，就会把该作用域块内部的变量压到栈顶；当作用域执行完成之后，该作用域的信息就会从栈顶弹出，这就是词法环境的结构。需要注意下，我这里所讲的变量是指通过 let 或者 const 声明的变量。

当执行到作用域块中的console.log(a)这行代码时，就需要在词法环境和变量环境中查找变量 a 的值了。

具体查找方式是：沿着词法环境的栈顶向下查询，如果在词法环境中的某个块中查找到了，就直接返回给 JavaScript 引擎，如果没有查找到，那么继续在变量环境中查找。

当作用域块执行结束之后，其内部定义的变量就会从词法环境的栈顶弹出，最终执行上下文如下图所示：

![09-13.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f62c9c8adbd49ce94240b27513744b0~tplv-k3u1fbpfcp-watermark.image?)
## 6. 变量对象与活动对象

对于每个执行上下文，都有四个重要的组成部分：

-   变量环境(以前也叫做变量对象)
-   词法环境
-   作用域链
-   this

之前，我们介绍了**预编译阶段**变量环境的生成，但是比较笼统，现在来探究一下其中的细节。

开始之前，有一点需要明确，就是**全局执行上下文的变量对象就是全局对象，在客户端，通常可用window引用**。

在函数执行上下文中，我们用活动对象(Activation Object, AO)来表示变量对象。

活动对象和变量对象其实是一个东西，只是变量对象是规范上的或者说是引擎实现上的。**不可在 JavaScript 环境中访问**，只有到当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，所以才叫 Activation Object ，而只有被激活的变量对象，也就是活动对象上的各种属性才能被访问。

**变量对象在预编译阶段生成，而活动对象是在进入函数上下文时被创建的**。活动对象通过函数的 arguments 属性初始化。arguments 属性值是 Arguments 对象。

## 7. 作用域链与JavaScript执行代码执行流程

### 7.1 作用域链

当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链。

下面，让我们以一个函数的创建和激活两个时期来讲解作用域链是如何创建和变化的。

### 7.2 函数创建

之前说过，函数的作用域在函数定义的时候就决定了，即词法作用域。

这是因为函数有一个内部属性 [[scope]]，**函数在创建的时候就会保存所在执行上下文的作用域链(outer)到[[scope]]特性**。你可以理解此时[[scope]] 就是所有父变量对象的层级链，但是注意：[[scope]] 并不代表完整的作用域链！

比如：

```js
function foo() {
    function bar() {
        ...
    }
}
```

函数创建时，各自的[[scope]]为：

```js
foo.[[scope]] = [
  globalContext.VO
];

bar.[[scope]] = [
    fooContext.AO,
    globalContext.VO
];
```

### 7.3 函数调用

函数一经调用，才会开始进行编译。编译之前会先进行预编译操作生成VO，预编译具体步骤如下：

1.  把函数形参和函数中的变量声明作为VO的键，并用undefined初始化。
1.  用函数实参的值初始化VO中的形参变量，做到形参实参相统一。
1.  把函数声明作为键加入VO对象，用undefined初始化，如果和变量声明冲突，就替换。

预编译阶段生成了函数执行上下文，接着把声明以外的代码编译为字节码(先生成AST，再进行编译)。并把函数执行上下文压入调用栈。

### 7.4 函数激活

当进入函数执行上下文的时候，函数激活，VO活化为AO，并将AO添加到作用域链的前端。

```js
outer = [AO].concat([[Scope]]);
```

至此，作用域链创建完毕。接着开始执行可执行代码。随着函数的执行，修改 AO 的属性值，待函数执行完毕，函数执行上下文便从调用栈中弹出。

### 7.5 JavaScript执行代码流程总结

以这段代码为例：

```js
var scope = "global scope";
function checkscope(uname){
    var scope2 = 'local scope';
    function checksocpe1() {}
    return scope2;
}
checkscope('cuifanfan');
```

首先进行词法、语法分析，进行预编译，创建全局执行上下文，生成全局对象，并将全局上下文压入调用栈。之后运行可执行代码调用`checkscope`函数。

1.  函数被创建的时候，已经保存作用域链到内部属性[[scope]]。

```js
checkscope.[[scope]] = [
    globalContext.AO
];
```

2.  函数调用，对函数进行预编译，创建VO对象和函数执行上下文，接着对其他非声明部分进行编译，并把函数执行上下文压入调用栈。

    > 创建VO步骤：(1)把函数形参和变量声明作为键，用undefined初始化 (2)实参形参相统一 (3)把内部函数声明作为键，用undefined初始化，如果和变量声明冲突，就进行替换。

```js
VO: {
    arguments: {
        uname: 'cuifanfan',
        length: 1
    },
    scope2: undefined,
    checkscope1: undefined
}
```

```
ECStack = [
    checkscopeContext,
    globalContext
];
```

3.  函数激活，进入函数执行上下文，活化AO，并将AO添加到作用域链的前端，作用域链生成完毕。

```
checkscopeContext.outer = [AO, globalContext.AO]
```

4.  执行函数，随着函数的执行，修改 AO 的属性值。

```js
checkscopeContext: {
  AO: {
    arguments: {
        uname: 'cuifanfan',
        length: 1
    },
    scope2: 'local scope',
    checkscope1: function() {}
  },
  outer: [AO, globalContext.AO]   
}
```

5.  函数执行完毕，函数上下文从执行上下文栈中弹出

```
ECStack = [
    globalContext
];
```

## 8. JavaScript中的this

**在对象内部的方法中使用对象内部的属性是一个非常普遍的需求**。但是 JavaScript 的作用域机制并不支持这一点，基于这个需求，JavaScript 又搞出来另外一套**this 机制**。

前面提到过，对于每个执行上下文，都由四个部分组成：


![09-14.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff2750a713514ed1b0156a5a12190c95~tplv-k3u1fbpfcp-watermark.image?)
执行上下文主要分为三种——全局执行上下文、函数执行上下文和 eval 执行上下文，所以对应的 this 也只有这三种——全局执行上下文中的 this、函数中的 this 和 eval 中的 this。不过由于 eval 我们使用的不多，所以本文我们对此就不做介绍了，如果你感兴趣的话，可以自行搜索和学习相关知识。

那么接下来我们就重点讲解下**全局执行上下文中的 this**和**函数执行上下文中的 this**。

### 8.1 全局执行上下文中的this

在控制台中输入console.log(this)来打印出来全局执行上下文中的 this，最终输出的是 window 对象。所以你可以得出这样一个结论：全局执行上下文中的 this 是指向window 对象的。这也是 this 和作用域链的唯一交点，作用域链的最底端包含了 window对象，**全局执行上下文中的 this 也是指向 window 对象**。

### 8.2 函数上下文中的this

先看下面这段代码：

```js
function foo() {
 console.log(this)
} 
foo()
```

我们在 foo 函数内部打印出来 this 值，执行这段代码，打印出来的也是 window 对象，这说明在默认情况下调用一个函数，其执行上下文中的 this 也是指向 window 对象的。估计你会好奇，那能不能设置执行上下文中的 this 来指向其他对象呢？答案是肯定的。通常情况下，有下面三种方式来设置函数执行上下文中的 this 值。

#### 8.2.1 通过函数的方法设置

类似的有call、apply、bind，自行学习即可，这里不多赘述。

#### 8.2.2 通过对象调用的方法设置

```js
var myObj = {
  name: "cuifanfan",
  showThis: function() {
    console.log(this)
  }
}
myObj.showThis()
```

执行这段代码，你可以看到，最终输出的 this 值是指向 myObj 的。所以，你可以得出这样的结论：**使用对象来调用其内部的一个方法，该方法的 this 是指向对象本身的**。

所以通过以上两个例子的对比，你可以得出下面这样两个结论： **（1）在全局环境中调用一个函数，函数内部的 this 指向的是全局变量 window。（2）通过一个对象来调用其内部的一个方法，该方法的执行上下文中的 this 指向对象本身。**

#### 8.2.3 通过构造函数设置

```js
function CreateObj() {
  this.name = "cuifanfan"
}
var myObj = new CreateObj()
```

new的过程中其实发生了这些操作：

1.  在内存中开辟一块空间创建一个新对象
1.  把这个新对象内部的[[Prototype]]特性被赋值为构造函数的 prototype 属性(搭上原型链)
1.  this 指向新对象
1.  执行构造函数内的代码
1.  如果构造函数返回非空对象，则返回该对象；否则，返回刚创建的新对象

简易版new如下：

```js
function objectFactory() {
  var obj = {}
  let Constrcutor = [].shift.call(arguments)
  obj.__proto__ = Constrcutor.prototype
  let result = Constrcutor.apply(obj, arguments)
  return typeof result === 'object' ? result : obj
}
```

### 8.3 this的缺陷和应对方案

#### 8.3.1 **嵌套函数中的** this不会从外层函数中继承

```js
var myObj = {
  name: "cuifanfan",
  showThis: function() {
    console.log(this)
    function bar() {
      console.log(this)
    }
    bar()
  }
}
myObj.showThis()
```

这种情况可以在 showThis 函数中**声明一个变量 self 用来保存 this**，然后在 bar 函数中使用 self。

```
var myjsObj = {
  name: "simon",
  showThis: function() {
    console.log(this)
    var self = this
​
    function bar() {
      self.name = "cuifanfan"
    }
    bar()
  }
}
myObj.showThis()
console.log(myObj.name)
console.log(window.name)
```

这个方法的的本质是**把 this 体系转换为了作用域的体系**。

#### 8.3.2 **普通函数中的** **this** **默认指向全局对象** window

上面我们已经介绍过了，在默认情况下调用一个函数，其执行上下文中的 this 是默认指向全局对象 window 的。不过这个设计也是一种缺陷，因为在实际工作中，我们并不希望函数执行上下文中的 this默认指向全局对象，因为这样会打破数据的边界，造成一些误操作。如果要让函数执行上下文中的 this 指向某个对象，最好的方式是通过 call 方法来显示调用。

最后补充一点：**箭头函数在执行时比块级作用域的内容多，比函数执行上下文的内容少，砍掉了很多函数执行上下文中的组件(比如this)，不过在箭头函数在执行时也是有变量环境的，因为还要支持变量提升**。
## 9. 闭包

### 9.1 什么是闭包

```
function foo() {
  var myName = "cuifanfan"
  let test1 = 1
  const test2 = 2
  var innerBar = {
    getName: function() {
      console.log(test1)
      return myName
    },
    setName: function(newName) {
      myName = newName
    }
  }
  return innerBar
}
var bar = foo()
bar.setName("simon")
bar.getName()
console.log(bar.getName())
```

代码执行到`return innerBar`的时候，调用栈的情况为：


![09-15.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a4c90510d944b8baf8c5ad7352aaa9c~tplv-k3u1fbpfcp-watermark.image?)

**根据词法作用域的规则，内部函数 getName 和 setName 总是可以访问它们的外部函数foo 中的变量**，所以当 innerBar 对象返回给全局变量 bar 时，虽然 foo 函数已经执行结束，但是 getName 和 setName 函数依然可以使用 foo 函数中的变量 myName 和test1。所以当 foo 函数执行完成之后，其整个调用栈的状态如下图所示：


![09-16.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d405c1ee3c64212bef7b3d776740c02~tplv-k3u1fbpfcp-watermark.image?)
foo 函数执行完成之后，其执行上下文从栈顶弹出了，但是由于返回的setName 和 getName 方法中使用了 foo 函数内部的变量 myName 和 test1，所以这两个变量依然保存在内存中。这像极了 setName 和 getName 方法背的一个专属背包，无论在哪里调用了 setName 和 getName 方法，它们都会背着这个 foo 函数的专属背包。之所以是**专属**背包，是因为除了 setName 和 getName 函数之外，其他任何地方都是无法访问该背包的，我们就可以把这个背包称为 foo 函数的**闭包**。

**在 JavaScript 中，根据词法作用域的规则，内部函数总是可以访问其外部函数中声明的变量，当通过调用一个外部函数返回一个内部函数后，即使该外部函数已经执行结束了，但是内部函数引用外部函数的变量依然保存在内存中，我们就把这些变量的集合称为闭包**。

JavaScript 引擎会沿着“当前执行上下文–>foo 函数闭包–> 全局执行上下文”的顺序来查找 myName 变量，你可以参考下面的调用栈状态图：


![09-17.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/777a42c9ed564c2897bab85f636641ce~tplv-k3u1fbpfcp-watermark.image?)

你也可以通过“开发者工具”来看看闭包的情况，打开 Chrome 的“开发者工具”，在bar 函数任意地方打上断点，然后刷新页面，可以看到如下内容：
![09-18.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bacfdf26fb64a138a12ae628e4ed305~tplv-k3u1fbpfcp-watermark.image?)
从图中可以看出来，当调用 bar.getName 的时候，右边 Scope 项就体现出了作用域链的情况：Local 就是当前的 getName 函数的作用域，Closure(foo) 是指 foo 函数的闭包，最下面的 Global 就是指全局作用域，从“Local–>Closure(foo)–>Global”就是一个完整的作用域链。

### 9.2 闭包是怎么回收的

通常，如果引用闭包的函数是一个全局变量，那么闭包会一直存在直到页面关闭；但如果这个闭包以后不再使用的话，就会造成内存泄漏。如果引用闭包的函数是个局部变量，等函数销毁后，在下次 JavaScript 引擎执行垃圾回收时，判断闭包这块内容如果已经不再被使用了，那么 JavaScript 引擎的垃圾回收器就会回收这块内存。

**因此，如果该闭包会一直使用，那么它可以作为全局变量而存在；但如果使用频率不高，而且占用内存又比较大的话，那就尽量让它成为一个局部变量**。
