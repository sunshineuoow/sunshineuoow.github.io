---
layout: post
title: 前端学习笔记之Event Loop
date: 2018-04-18 16:49:30 +0800
description: Frontend Note for Event Loop.
img: js.png
tags: [Frontend, Note, Event Loop]
---

# whatwg中的Event loop

## 定义

为了协调事件，用户交互，脚本，渲染，网络等，UA(user agent)必须使用以下描述的事件循环。

存在两种事件循环：针对于浏览上下文以及针对worker

浏览上下文时间循环永远至少有一个浏览上下文。如果该事件循环的浏览上下文都消失了，那么事件循环也消失了。浏览上下文总是有一个事件循环来协调其活动。

一个事件循环有一个或者多个**任务队列**。任务队列指的是一个有序的任务列表。任务则是响应如下工作的算法：

**Events(事件)**
  
&nbsp;&nbsp;&nbsp;&nbsp;通常由专门的任务来完成在特定的事件目标对象(EventTarget object)派发一个任务对象

&nbsp;&nbsp;&nbsp;&nbsp;`注：不是所有的事件都通过任务队列派发，许多事件都在其他任务中。`

**Parsing(解析)**

&nbsp;&nbsp;&nbsp;&nbsp;标记一个或多个字节，然后处理所有的标记结果，通常是HTML解析器的一项任务。

**Callbacks(回调)**

&nbsp;&nbsp;&nbsp;&nbsp;调用回调函数通常由一个特定的任务完成。

**Reacting to DOM manipulation(响应DOM操作)**

&nbsp;&nbsp;&nbsp;&nbsp;某些元素拥有响应DOM操作而触发的任务，例如，当这个元素被插入文档中时。

每个浏览上下文时间循环中的任务都与一个Document关联:
1. 如果任务在元素的上下文中排列，那么该文档就是这个元素的节点文档(node document);如果
2. 如果任务在浏览上下文的上下文中排列，那么该文档是任务排列时浏览上下文的活动文档
3. 如果任务在脚本中排列，那么该文档是由脚本的设置对象指定的响应文档(responsible document)

任务针对特定的事件循环：处理与Document或者worker关联的任务的事件循环

当UA排列一个任务时，必须将给定任务添加到任务队列相关的事件循环中。

每个任务都被定义为来自某个特定的任务源。所有的来自同一个特定任务源并且指定到统一事件循环中的任务(例如，由Document定时器产生的回调，Document上鼠标移动触发的事件，解析文档而排列的任务)必须始终添加到相同的任务队列，但是不同任务源的任务可能被放在不同的任务队列。

    Example
    某个UA可能有一个鼠标和键盘事件的任务队列，还有另一个处理其他事件的任务队列。UA可以花相对于其他事件三倍的时间来处理键盘和鼠标事件，并且保持接口响应但是不会使得其他任务队列没有任务，并且不会处理错任何一个任务源事件的顺序。

每个事件循环都有一个当前运行的任务。最初这个任务是null，用于处理重入(reentrancy)。每个事件循环还有一个执行microtask的检查点标识(初始值为false)。用于防止microtask检查点算法的重复调用。

## 处理模型

一个事件循环在其存在期间必须持续运行以下步骤：
  1. 让oldestTask成为事件循环任务队列中最后的任务，在浏览上下文事件循环中，忽略相关联文档没有完全激活的任务(如果有的话)。UA可以选择任意任务队列。如果没有任务选择，跳到下面的microtasks步骤。
  
  2. 将事件循环当前执行任务设置为oldestTask。
  
  3. 执行oldestTask。
  
  4. 将事件循环当前执行任务设置为null。
  
  5. 从oldestTask的任务队列中移除它。
  
  6. Microtasks: 执行microtask的检查点。
  
  7. 更新渲染(如果是一个浏览上下文的事件循环)

  8. 如果是一个worker的事件循环，但是事件循环的任务队列中没有任务并且`WorkerGlobalScope`对象的关闭标识(closing flag)是true的话，销毁这个事件循环，中断以上步骤，恢复运行在Web worker中描述的worker步骤

每个事件循环都有一个**microtask队列**。**microtask**最初在microtask队列中排列而不是在task队列。有两种microtask: **单独回调微任务(solitary callback microtask)**和**复合微任务(compound microtask)**

`注:本说明只包含单回调微任务。`

当算法要求microtask排列时，它必须被添加到相关事件循环的microtask队列;这个microtask的任务源是微任务任务源(microtask task source)。

`注:如果在microtask的初始执行过程中，microtask也可能被移动到常规的任务队列，将转动事件循环。这种情况下，微任务任务源将是使用的任务源。通常，microtask的任务源是不相关的。`

当一个UA区执行microtask的检查点时，如果检查点标识为false，那么UA必须运行以下步骤:
  1. 将检查点flag设置为true
  
  2. 当事件循环的microtask队列不为空时:
    <br><br>
    2.1 将oldestMicrotask设置为事件循环的microtask队列中最老的microtask
    <br><br>
    2.2 将事件循环当前执行任务设置为oldestMicrotask
    <br><br>
    2.3 执行oldestMicrotask
    <br><br>
      `注:这可能设置脚本回调，最终在运行脚本步骤之后调用清理，可能导致再次执行微任务检查点算法，这就是为什么我们使用执行微任务检查点标识来避免重入`
    <br><br>
    2.4 将事件循环当前执行任务设置为null
    <br><br>
    2.5 将oldestMicrotask从microtask队列中移除
    <br>

  3. 对于每个响应的事件循环是该事件循环的环境设置的对象，通知该环境设置对象上被拒绝的promise

  4. 清除索引数据库事务

  5. 将检查点标识设置为false

当一个组合微任务运行时，UA必须运行一系列步骤来执行组合微任务的子任务，步骤如下：
  1. 让父任务是事件循环当前执行任务

  2. 让子任务变为由执行一系列给定步骤的新任务。这个microtask的任务源是microtask task source。这是一个组合微任务的子任务。

  3. 将事件循环当前执行任务设置为子任务。

  4. 执行子任务。

  5. 将事件循环当前执行任务设置为父任务。


当并行运行的算法要等待稳定状态时，UA必须运行以下步骤对microtask排列，然后停止执行(如下列步骤所述，当microtask执行时算法将恢复运行):
  1. 运行算法的同步部分

  2. 如果可以，按照算法步骤中的描述，恢复并行算法的运行。

当一个算法要推动事件循环知道符合条件目标时，UA必须执行以下步骤:
  1. 将事件循环当前任务设置为该任务<br>
    `注:这个可能是一个单回调微任务，或者是组合微任务的子任务，或者是常规任务。不可能是组合微任务。`

  2. 将任务的任务源设置为该任务任务源

  3. 将JS执行上下文栈拷贝给旧栈(old stack)

  4. 清空JS执行上下文栈

  5. 运行microtask检查点

  6. 停止任务，允许任何调用它的算法恢复，但是并行继续这些步骤

  7. 等待符合的条件目标出现

  8. 排列一个任务继续这些步骤，使用任务源的任务源。在新任务运行后继续执行这些步骤。

  9. 用旧栈替换JS执行上下文栈

  10. 返回调用者

## 通用任务源

本规范和其他规范中大多数不相关的功能使用以下任务源。

**DOM manipulation task source(DOM操作任务源)**

&nbsp;&nbsp;&nbsp;&nbsp;
这个任务源用于响应DOM操作的功能，例如将一个元素插入文档时以非阻塞的形式运行。

**user interaction task source(用户交互任务源)**

&nbsp;&nbsp;&nbsp;&nbsp;
这个任务源用于响应用户交互，例如键盘或者鼠标输入。

&nbsp;&nbsp;&nbsp;&nbsp;
响应用户输入(如鼠标点击事件)的事件必须通过用户交互任务源进行排序后发送。

**networking task source(网络任务源)**

&nbsp;&nbsp;&nbsp;&nbsp;
这个任务源用于响应网络请求时触发的功能。

**history traversal task source(历史遍历任务源)**

&nbsp;&nbsp;&nbsp;&nbsp;
这个任务源用于排列调用`history.back()`和相似API。