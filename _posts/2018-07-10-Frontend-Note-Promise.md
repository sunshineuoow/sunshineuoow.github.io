---
layout: post
title: 前端学习笔记之Promise规范
date: 2018-7-10 20:30:21 +0800
description: Frontend Note for Promise.
img: js.png
tags: [Frontend, Note, Promise]
---

# Promise/A+规范

一个Promise表示着一个异步操作的最终结果。与Promise的最主要交互方式是通过它的`then`方法，用于注册回调去接收一个promise的最终的值或者promise不能够完成的理由。

这个规范具体描述了`then`方法的行为，给所有符合Promise/A+规范的promise实现提供了一个交互的基础，使得它们变得可靠。因此，这个规范十分稳定。即使Promise/A+组织可能偶尔做一些次要且向下兼容的规范改动来适应新发现的边缘案例，我们将只在严密的考虑，讨论和测试后整合大的或者不向下兼容的改动。

历史上，Promise/A+澄清过早期的Promise/A提案，将其拓展以覆盖实际行为和未确认以及有问题的部分。

最后，这个核心的Promise/A+声明并不提供如何创建，履行，或者拒绝promise，仅仅专注于提供一个可交互的`then`方法。

## 1. 术语

1.1 `promise`是一个具有其行为满足这份规范的`then`方法的对象或者函数

1.2 `thenable`是一个定义`then`方法的对象或者函数

1.3 `value`是一个任意合法的JavaScript值(包含`undefined`，`thenable`或者`promise`)

1.4 `exception`是一个通过`throw`声明抛出的值

1.5 `reason`是一个表明promise为什么被拒绝的值

## 2. 要求

### 2.1 Promise状态

一个promise必须处于下列三种状态中的一种：pending，fulfilled，rejected

#### 2.1.1 pending
1. promise可能转换为fulfilled或者rejected状态

#### 2.1.2 fulfilled
1. promise不能转换为另一种状态
2. promise必须拥有一个`value`，并且不可以改变

#### 2.1.3 rejected
1. promise不能转换为另一种状态
2. promise必须有一个`reason`，并且不能改变

这里"不能改变"指的是不变的标识符（如`===`），但是不意味着深层次的不变性

### 2.2 `then`方法

promise必须提供一个`then`方法去获取它当前或者最终的`value`或者`reason`

promise的`then`方法接收两个参数
```javascript
promise.then(onFulfilled, onRejected)
```

#### 2.2.1 `onFulfilled`和`onRejectd`都是可选参数

- 1.如果`onFulfilled`不是一个函数，必须被忽略

- 2.如果`onRejectd`不是一个函数，必须被忽略

- 3.如果`onFulfilled`是一个函数
  - 3.1 必须在`promise`履行后调用，并且`promise`的`value`是它的第一个参数
  - 3.2 在`promise`处于`fulfilled`前不能被调用
  - 3.3 调用次数不得多于一次

- 4.如果`onRejectd`是一个函数
  - 4.1 必须在`promise`履行后调用，并且`promise`的`reason`是它的第一个参数
  - 4.2 在`promise`处于`rejected`前不能被调用
  - 4.3 调用次数不得多于一次

- 5.`onFulfilled`和`onRejected`必须直到执行上下文栈只包含平台代码(platform code)时才可以调用[3.1]

- 6.`onFulfilled`和`onRejected`必须和函数一样调用(例如，没有`this`值)[3.2]

- 7.`then`可以在同一个promise内被多次调用
  - 7.1 如果/当`promise`已经处于`fulfilled`，所有各自的`onFulfilled`回调必须按照当初调用`then`的顺序执行
  - 7.2 如果/当`promise`已经处于`onRejected`，所有各自的`onRejected`回调必须按照当初调用`then`的顺序执行

- 8.`then`必须返回一个promise[3.3]
```javascript
promise2 = promise1.then(onFulfilled, onRejected);
```
  - 8.1 如果`onFulfilled`或者`onRejected`返回一个值`x`，运行promise解决程序`[[Resolve]](promise2, x)`
  - 8.2 如果`onFulfilled`或者`onRejected`抛出一个异常`e`，`promise2`必须以`e`为理由拒绝
  - 8.3 如果`onFulfilled`不是一个函数并且`promise1`处于fulfilled，`promise2`必须以相同值履行
  - 8.4 如果`onRejected`不是一个函数并且`promise1`被拒绝了，那么`promise2`应当以相同理由拒绝

### 2.3 Promise解决程序

**promise解决程序**是一个抽象的操作，以promise和值为输入，我们将其表示为`[[Resolve]](promise, x)`。如果`x`是一个`thenable`，它尝试去使得`promise`采用`x`的状态，在这个假设下`x`表现的有些像一个promise。并且，它使用值`x`履行了`promise`。

这个`thenables`的对待允许promise实现去进行交互，只要它们暴露了向下兼容的Promises/A+`then`方法。它也允许Promises/A+实现通过合理的`then`方法兼容不合规范的实现。

运行`[[Resolve]](promise, x)`，需要进行如下步骤：

- 1.如果`promise`和`x`指向相同的对象，用一个`TypeError`作为理由拒绝`promise`

- 2.如果`x`是一个promise，采用它的状态[3.4]
  - 2.1 如果`x`正在pending，`promise`必须保持pending直到`x`变为`fulfilled`或者`rejected`
  - 2.2 如果/当`x`处于fulfilled，用相同值履行`promise`
  - 2.3 如果/当`x`被拒绝，以相同理由拒绝`promise`

- 3.另外，如果`x`是一个对象或者函数
  - 3.1 让`then`变为`x.then`[3.5]
  - 3.2 如果检索属性`x.then`的结果在一个抛出的异常`e`内，以`e`为理由拒绝`promise`
  - 3.3 如果`then`是一个函数，以`x`作为`this`调用它，第一个参数为`resolvePromise`，并且第二个参数为`rejectPromise`
    - 3.3.1 如果/当`resolvePromise`以值`y`调用，运行`[[Resolve]](promise, y)`
    - 3.3.2 如果/当`rejectPromise`以理由`r`调用，以理由`r`拒绝`promise`
    - 3.3.3 如果`resolvePromise`和`rejectPromise`都被调用了，或者对同一参数多次调用，第一次调用有优先权，之后的调用被忽略
    - 3.3.4 如果调用`then`抛出异常`e`
      - 3.3.4.1 如果`resolvePromise`或者`rejectPromise`被调用，忽略
      - 3.3.4.1 除此之外，以`e`为理由拒绝`promise`
  - 3.4 如果`then`不是一个函数，以`x`履行`promise`
- 4.如果`x`不是一个函数或者对象，以`x`履行`promise`

如果一个promise使用一个在环形`thenable`链中的`thenable`解决，例如`[[Resolve]](promise, thenable)`的递归最终导致`[[Resolve]](promise, thenable)`再次被调用，遵循上面的算法将导致无限递归。实现被鼓励，但是不要求去检测这种递归并且以一个有信息的`TypeError`为理由拒绝`promise`。[3.6]

## 3. 注意项

1. 这里'platform code'指的是引擎，环境和promise实现代码。在实践中，这个要求确保`onFulfilled`和`onRejected`异步执行，在调用`then`的事件循环后，并且在一个新的栈中。这可以通过一个'macro-task'机制例如`setTimeout`或者`setImmediate`实现，或者一个'micro-task'机制如`MutationObserver`或者`process.nextTick`实现。既然promise实现被认为是平台代码，它自己可能包含一个任务调度队列或者'trampoline'来调用处理函数。

2. 严格模式下，在函数中`this`将会变为`undefined`。在非严格模式下，将会变为一个全局对象。

3. 实现可能允许`promise2 === promise1`，提供的实现满足所有要求。每个实现应当提供文档说明是否会产生`promise2 === promise1`并且在何种情形。

4. 通常，这种情况出现在`x`是一个来自于当前实现的真promise。这个条约允许使用特定实现手段来采用已知符合规范的promises状态

5. 这个程序首先存储一个`x.then`引用，然后测试这个引用，并且之后调用该引用，避免多次获取`x.then`属性。这些注意事项十分重要，用于确保属性值可能在检索中改变的属性的一致性。

6. 实现不应当限制`thenable`链的深度，并且假设超出该限制递归将是无限的。只有真的环才会导致一个`TypeError`，如果遇见一个不同thenables的无限链，永远递归是正确的做法。