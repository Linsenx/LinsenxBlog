---
title: JavaScript深入学习(一) -- 事件循环
date: 2019-09-17 18:25:55
categories: JavaScript深入学习
tags: [JavaScript, Event Loop]
thumbnail: http://image.linsenx.com/blog/2019-09-19-daniel-leone-v7daTKlZzaw-unsplash.jpg
---

这是深入学习JavaScript系列的第一篇，该系列的写作目标主要是让我自己对JavaScript的运作机制有一个更深刻的理解，事件循环也是JavaScript中比较难懂的一个知识点，仅以个人理解和网络上的资料作为参考，如果存在错误之处，欢迎指正。

<!--more-->

## 1.执行栈

要谈事件循环，我们得先从JavaScript运行机制讲起，我们首先要了解执行栈的概念。

> Q: 什么是执行栈？
>
> A: 栈是一种先进后出的数据结构，而执行栈可以理解为是存储当前函数调用的栈。

下图为执行栈的可视化演示：

![](http://image.linsenx.com/blog/2019-09-20-155541.gif)

从上图我们可以看到，JavaScript脚本的执行就是往执行栈中压入函数，之后根据先入后出的原则依次执行函数后将其从栈中弹出，并将返回值传递给之后的函数。



## 2.事件循环

> 我们先引入[任务](https://html.spec.whatwg.org/multipage/webappapis.html#concept-task)的概念，可以将任务理解为函数，若直接返回结果为**同步任务**（函数），若通过回调函数返回结果为**异步任务**（函数）。

从一小节我们了解了执行栈的概念，理解了JavaScript脚本的执行方式就是通过往执行栈中压入函数。

你们应该知道JavaScript是一种单线程脚本语言，所以代码的执行是需要排队的，如果不对异步任务进行处理，它就会阻塞之后代码的运行，因此JavaScript采用了回调的模式避免等待耗时异步任务，当遇到异步函数时，就会将其放入外部环境如**浏览器环境**，**Node.js运行环境**中的线程中进行处理，待处理完毕后再将运行结果和回调函数放入JavaScript的任务队列中。而这个协调异步任务与同步任务运行顺序的机制被叫做**事件循环**。

> 以浏览器环境为例，有以下线程用于处理异步任务：
>
> 1. 事件循环线程：用于控制事件循环，往JavaScript执行栈压入函数
> 2. 定时器触发线程：处理`setTimeout`和`setInterval`，计时结束后将回调函数加入任务队列
> 3. 异步HTTP请求线程：处理`XMLHttpRequest`，完成请求后将回调函数加入任务队列

根据[HTML标准](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop)，**异步任务**被分为两种，它们是：**宏任务(task)**、**微任务(microTask)**，与此同时任务队列也有两种：**宏任务队列**、**微任务队列**。（在[ECMAScript2015标准](https://www.ecma-international.org/ecma-262/6.0/#sec-jobs-and-job-queues)中，microTask被叫做Job）

**宏任务**可以通过以下方式被创建：

script, setTimeout, setInterval, setImmediate, requestAnimationFrame, I/O, UI Rendering

**微任务**可以通过以下方式被创建：

promise, MutationObserver

>1. 在**浏览器环境**或**Node.js运行环境**中，**最先**被放入宏任务队列的是script宏任务
>2. promise的then回调才是微任务，new Promise(executor)的executor是**立即执行函数**因此为**同步任务**

**事件循环的执行顺序如下：**

1. 从宏任务队列中取出队首任务并执行
2. 队首任务执行完毕，此时执行栈为空
3. **检测微任务队列是否为空**
   1. **若微任务队列非空**，将微任务队列中的任务依次执行，并将执行微任务过程中**新创建的微任务加入微任务队列队尾**，直到所有微任务都执行完毕，再往下执行事件循环
   2. **若微任务队列为空**，往下执行事件循环

4. 进行页面渲染
5. 开始下一轮的事件循环



## 3.习题挑战

从上面两小节，我们已经理解了事件循环的机制，那么下面就让我们来做几道习题来巩固一下知识。

先来一道**比较简单**的习题，试着写出这段程序的输出结果：

```javascript
console.log('script start');

setTimeout(() => {
  console.log('from timer 1');
}, 0);

new Promise((res, rej) => {
  console.log('from promise executor');
  res(100);
}).then(val => {
  console.log(`from promise then callback: ${val}`);
  return val * 2;
}).then(val => {
  console.log(`from promise then callback: ${val}`);
});

console.log('script end');
```

这段程序的输出结果为：（这道题比较简单就不做解释了，如果没法理解的再仔细看一下第二小节）

```
script start
from promise executor
script end
from promise then callback: 100
from promise then callback: 200
from timer 1
```



在前端开发中，相较于promise的回调式异步，人们更加喜欢于以类同步思想组织代码的async/await。在引出下一道题之前，让我们先来看一下ES6中的async-await在事件循环中是如何运作的。

```javascript
console.log('script start');
async function timer() {
  return new Promise((res, rej) => {
    setTimeout(() => {
      res(100)
    }, 0);
  });
}

async function run() {
  console.log('abc');
  const num = await timer();
  console.log(num);
}

run();
console.log('script end')
```

输出顺序为：（为什么100是最后输出的？）

```
script start
abc
script end
100
```

在上面，我们定义了两个async函数，timer为定时器函数，作用为0毫秒后返回数字100。run函数先输出abc，之后await等待timer函数返回数据，之后将timer函数返回的数据输出。虽然在事件循环中没有提到async/await函数的处理顺序，但我们可以将async/await函数转化为等价的promise函数，如下：

```javascript
function run() {
  console.log('abc');
  timer().then(num => {
    console.log(num)
  });
}
```

经过转化之后，我们就能理解为什么`100`是在`script end`之后输出的了，因为`num => console.log(num)`函数处于微任务队列中，它需要等待宏任务`script`结束之后才能执行。



最后我们来一道**比较难**的题（据说出自今日头条前端面试题）：

```javascript
async function async1 () {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2 () {
  console.log('async2');
}

console.log('script start');

setTimeout(function () {
  console.log('setTimeout');
}, 0);

async1();

new Promise(function (resolve) {
  console.log('promise1');
  resolve();
}).then(function () {
  console.log('promise2');
});

console.log('script end');
```

正确的输出顺序如下：

```
script start
async1 start
async2 
promise1
script end
async1 end
promise2
setTimeout
```

其中的易错点是promise1的输出顺序，因为`console.log('promise1')`位于promise的立即执行函数中，因此它也是同步任务。