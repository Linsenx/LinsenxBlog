---
title: JavaScript深入学习(二)--事件与事件委托
date: 2019-11-05 08:20:05
categories: JavaScript深入学习
tags: [JavaScript, Event Bubbling, Event Delegation]
thumbnail: https://image.linsenx.com/blog/2019-11-05-evening-55067.png?imageMogr2/format/webp/blur/1x0/quality/75%7Cimageslim
toc: true
---

这一期，我们来探讨一下 JavaScript DOM 编程中重要的事件机制以及事件委托模式。

文中使用的代码调试工具为： [Ocelot Coding](https://github.com/NightCats/Ocelot-Coding/releases/tag/OcelotCoding)。

<!--more-->

##  事件

事件分为三个阶段：

1. 捕获阶段(Capturing phase)：事件从 window 逐级向下传递到目标的过程
2. 命中阶段(Target Phase)：事件已经到达目标
3. 冒泡阶段(Bubbling Phase)：事件从目标逐级向上传递到 window 的过程

> 下图展示了一个事件传播的完整流程，从中我们可以清晰的看到事件传播的三个阶段(1)、(2)、(3)

<img src="https://image.linsenx.com/blog/2019-11-05-8861DE12-EAD2-47A7-9E91-CF873824AF41.png" height="500" />

## 监听事件

我们已经了解了事件的三个阶段，接下来我们要对事件进行监听，这就需要用到 `addEventListener`。

```javascript
target.addEventListener(type, listener[, options]);
target.addEventListener(type, listener[, useCapture]);
```

`addEventListener`的第三个参数有两种形式：options(Object) 和 useCapture(Boolean)。

* useCapture：Boolean，表示是否监听某个事件的捕获阶段，**默认值为 false**
* options：Object，用于指定 listener 的属性
  * capture：Boolean，同 useCaputre
  * once：Boolean，表示 listener 是否只执行一次
  * passive：Boolean，表示 listener 是否会调用 `preventDeafult()`，如果客户端任调用 `preventDefault()`，抛出一个警告

### Event

当监听的事件被触发时，listener会接收到一个 Event 对象，下面我们来看一下 Event 对象中比较重要的属性

* Event.target：指向**触发事件**的元素
* Event.currentTarget：指向**绑定事件的**元素（即 target ）
* Event.preventDefault()：取消默认动作，**不影响事件的传播**
* Event.stopPropagation()：阻止该事件的传播（包括**事件捕获**和**事件冒泡**）
* Event.stopImmediatePropagation()：阻止该事件的传播，并且**阻止该监听器之后的监听相同事件的其它监听器被调用**

其中比较难以理解的可能是 `stopPropagation` 和 `stopImmediatePropagation` 的区别，下面通过一个小例子来比较它们的不同：

> HTML 结构如下，为A、B、C分别绑定事件，当 C 被点击时，输出如下：

```html
<div id="A">
  A
  <div id="B">
    B
    <div id="C">
      C
    </div>
  </div>
</div>
```

![image-20191105101729646](https://image.linsenx.com/blog/2019-11-05-021732.png)

> 当在 B 元素的监听器中添加 `stopPropagation()` 语句，输出如下：

![image-20191105101758140](https://image.linsenx.com/blog/2019-11-05-030433.png)

可以看出，在事件传播阶段中，位于 B捕获事件 之后的监听器（C命中事件的两个监听器）都没有得到调用，事件停止了传播。

> 之后，改用 `stopImmediatePropagation()`， 输出如下：

![image-20191105102118219](https://image.linsenx.com/blog/2019-11-05-030441.png)

可以看出，**不但 B 之后的事件被阻止**了，**第二个 B捕获 监听器也没有得到调用**（没有输出`B Capture twice`）。

## 事件委托模式

> 试想这么一个场景，有一个动态列表（随时存在列表项的增删），我们如何监听列表项的点击事件？
>
> 最容易想到的方式可能是在增 / 删列表项的时候，注册 / 删除 监听器，但是这个方法存在一些问题：每个监听器的存在都会占用一定的内存，如果大量注册监听器，就会占用大量的内存，而且频繁注册删除监听器可能会在某些浏览器上引起内存泄露。
>
> 由此，我们引入了 **事件委托模式**，这个模式正是利用了我们在前文中详细解释的 **事件机制**中的**事件冒泡**。

### 如何实现事件委托模式

1. 不直接对想要监听的元素添加监听器
2. 对该元素的祖先元素添加监听器
3. 通过 event.target 获得触发事件的元素（即我们想要监听的元素）

### 事件委托模式的小例子

```html
<button id="add">Add Item</button>
<button id="remove">Remove Item</button>
<ul id="container">
</ul>
```

```javascript
const btnAdd = document.getElementById('add')
const btnRemove = document.getElementById('remove')
const container = document.getElementById('container')

btnAdd.addEventListener('click', () => {
  const node = document.createElement('li')
  node.innerHTML = new Date().valueOf()
  container.appendChild(node)
})
btnRemove.addEventListener('click', () => {
  container.removeChild(container.lastChild)
})

container.addEventListener('click', e => {
  const target = e.target
  if (target.tagName === 'LI') {
    console.log('LI被点击', e.target.innerText)
  }
})
```



![image-20191105105848669](https://image.linsenx.com/blog/2019-11-05-030449.png)

