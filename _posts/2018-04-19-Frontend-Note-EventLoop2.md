---
layout: post
title: 前端学习笔记之Event Loop(二)
date: 2018-04-18 18:34:21 +0800
description: Frontend Note for Event Loop.
img: js.png
tags: [Frontend, Note, Event Loop]
---

# Event Loop

JavaScript 是一门单线程的语言，意味着 JS 无法进行多线程编程。但是  在浏览器运行脚本时，很可能  产生一些高耗时的任务，如 Ajax 请求。

假设这些任务是  同步的，那么 js 的  运行就将被阻塞，直到这些任务结束才能正常运行。因此我们通过异步的方式来处理  这些高耗时任务，以达到浏览器非阻塞运行任务的目的。

## 任务队列 

JS 中任务执行过程如下：

1.  所有的同步任务在主线程上执行，形成一个执行栈(execution context stack)
2.  主线程之外，还存在一个任务队列(task queue)
3.  异步任务有了结果后，往任务队列中添加一个事件
4.  执行栈中  所有同步任务执行完毕，系统会读取任务队列，依次提取对应的异步任务进入  执行栈执行
5.  主线程不断重复以上 4 步

下图为主线程和任务队列的示意图。

![](../assets/img/task_queue.png)

## 回调函数

任务队列是一个事件的队列，也可以理解成异步任务的消息队列。

当异步任务完成时，就在任务队列中添加一个  任务，表示  该异步任务可以进入执行栈  了。

而所谓回调  函数，即指的是异步任务推进任务队列中的函数。主线程从任务队列中提取任务并执行，其实就是  执行了异步任务对应  的回调函数。

任务队列是一个先进先出的数据结构，排在前面的任务会先被主线程推进执行栈执行。

主线程读取任务队列的过程基本是自动的，只要执行栈空了，就会立即从任务队列中获取任务。

## Event Loop

主线程从任务队列中读取任务的过程是一直持续的，因此被称为 Event Loop(事件循环)。

事件循环的运行机制如下图

![](../assets/img/event_loop.png)

主线程  运行时产生堆(heap)和  栈(stack)，栈中代码会调用各种 WebAPI(DOM 操作，Ajax 请求，计时器)，而后  在其完成时将其注入任务队列中。一旦主线程任务执行完毕，就会从任务队列中依次取出任务并执行其  回调函数。

```javascript
console.log('script start')
setTimeout(function() {
  console.log('timeout')
}, 0)
console.log('script end')

// script start
// script end
// timeout
```

通常会在代码中看到如下书写方式。是为了保证定时器中函数在主线程任务结束后执行。

其  具体过程如下

1.  打印 script start
2.  将计时器任务添加到计时器线程进行计时， 完成后将其  添加到任务队列中
3.  打印 script end
4.  主线程任务执行完毕，从  任务队列中拉取任务，执行回调，打印 timeout

## microtask 和 macrotask

先看一道题

```javascript
console.log('script start')

setTimeout(function() {
  console.log('timeout')
}, 0)

Promise.resolve()
  .then(function() {
    console.log('promise 1')
  })
  .then(function() {
    console.log('promise 2')
  })

console.log('script end')
```

这段代码的打印结果为
      
    script start
    script end
    promise 1
    promise 2
    timeout

为什么会是如下结果呢，这就涉及到microtask和macrotask的区别了

首先看一下哪些属于microtask和macrotask

### microtask
 * process.nextTick
 * promise
 * Object.observe

### macrotask
  * setTimeout
  * setInterval
  * setImmediate
  * I/O

每个event loop中都存在一个microtask和task的任务队列，而在每个事件循环的周期中，都会把所有的Microtask队列内的任务全部执行完毕。

因此对于上述题目，执行顺序如下

1. 打印`script start`
2. 将计时器任务添加到计时器线程，完成后添加到macrotask队列
3. 将promise任务添加到microtask队列
4. 打印`script end`
5. 依次将microtask队列中的任务取出执行直至清空microtask队列，打印`promise 1`和`promise2`
6. 将macrotask队列中任务取出最后一个执行，打印`timeout`


## setTimeout和setInterval

setTimeout和setInterval本质上都是先将任务提交给计时器线程计时，而后计时结束后将任务添加到任务队列中。

因此，其实具体的执行时间取决于主线程任务的执行时间。

```javascript
  setTimeout(function() {
    console.log('timeout')    
  }, 1000)
  // something costs 10s
```
正常期待是1秒后执行打印timeout，但是由于主线程任务花费了10秒，所以实际打印会在10秒之后。

因此``setInterval``会出现一个任务跳过的问题。
```javascript
  setInterval(function() {
    // something casts 3s
  }, 2000)
```
该代码标识每隔2秒钟将一个任务推送到任务队列中。
但是实际上却不会按照每隔2秒执行一次的方式去进行，而是持续不断地执行(执行完毕后任务队列中总是有一个setInterval任务)。

由于函数执行事件需要3秒，而每隔两秒`setInterval`就会向任务队列中增加一个任务，那么这样会不会导致任务越来越多呢(不清除计时器的情况下)。

实际上，`setInterval`中存在一个机制，如果任务队列中已经存在该任务，则不会向任务队列中推送。因此避免了上述任务无限叠加问题。
