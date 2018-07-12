---
layout: post
title: 前端学习笔记之Promise实现
date: 2018-7-11 17:25:00 +0800
description: Frontend Note for Promise.
img: promise.jpg
tags: [Frontend, Note, Promise]
---

# 自己实现一个 Promise

[Promises/A+ 规范原文](https://promisesaplus.com/)

[Promises/A+ 规范翻译](https://sunshineuoow.github.io/Frontend-Note-Promise/)

## 1. 初始化

- Promise 初始化时接收一个 executor 参数。
- executor 参数带有`resolve`和`reject`两个参数，用于修改 promise 的状态
- Promise 有 3 个状态，`pending`，`fulfilled`和`rejected`

```javascript
  //  定义三种状态常量
  const PENDING = 'pending'
  const FULFILLED = 'fulfilled'
  const REJECTED = 'rejected'

  class Promise {
    constructor (executor) {
      this.state = PENDING // 初始化时为pending
      this.value = undefined // 初始化值
      this.reason = undefined // 初始化错误
      this.fulfilledQueue = [] // 成功回调队列
      this.rejectedQueue = [] // 失败回调队列

      const resolve = value => {
        if (this.state === PENDING) {
          this.state = FULFILLED
          this.value = value
          this.fulfilledQueue.forEach(cb => cb())
        }
      }

      const reject = reason => {
        if (this.state === PENDING) {
          this.state = REJECTED
          this.reason = reason
          this.rejectedQueue.forEach(cb => cb())
        }
      }

      // 处理 new Promise((resolve, reject) => { throw new Error() })
      try {
        executor(resolve, reject)
      } catch (e) {
        reject(e)
      }
    }
  }
```

## 2. `then`方法

- `then`方法接受两个可选参数`onFulfilled`和`onRejected`
- 两个参数均为函数，如果不是函数，那么必须要被忽略
- 如果调用`then`方法时，promise 已经处于 fulfilled，那么新的 promise 将用相同值 fulfill
- 同理，如果调用`then`方法时，promise 已经处于 rejected，那么新的 promise 将以相同理由 reject

初步想法是 Promise 实例本身维护一个成功回调和失败回调队列，然后调用 then 时将回调加入队列中，并且返回 this，最后根据状态依次执行回调，达到链式调用的目的。后发现若执行成功回调中拋错无法  很好定位至某个点后执行后续的失败回调，因此舍弃(捂脸)。

那么`then`方法应当返回一个新的 Promise 实例，并且当上一个 promise 处于 pending 时，将新实例的 resolve 和 reject 方法注入上一个 promise 的回调队列中。

```javascript
class Promise {
  constructor(executor) {
    // 相同代码省略
  }

  then(onFulfilled, onRejected) {
    // 先进行一次判断，即onFulfilled和onRejected是否是函数
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val
    onRejected =
      typeof onRejected === 'function'
        ? onRejected
        : err => {
            throw err
          }

    const promise = new Promise((resolve, reject) => {
      // 对promise进行状态判断
      if (this.state === FULFILLED) {
        const value = onFulfilled(this.value)
        resolvePromise(promise, value, resolve, reject)
      } else if (this.state === REJECTED) {
        const reason = onRejected(this.reason)
        resolvePromise(promise, value, resolve, reject)
      } else if (this.state === PENDING) {
        // 也可以写成else
        this.fulfilledQueue.push(() => {
          const value = onFulfilled(this.value)
          resolve(value)
        })
        this.rejectedQueue.push(() => {
          const reason = onRejected(this.reason)
          resolvePromise(promise, value, resolve, reject)
        })
      }
    })

    return promise
  }
}
```

## 3. 完成`Promise Resolution Procedure`，即`resolvePromise`方法

规范中说明，如果`onFulfilled`或者`onRejected`返回一个值`x`，那么需要运行 promise 解析程序

- 1、如果 x 是一个 promise，则采用 x 的状态
- 2、如果 x 是一个对象或者函数，并且有`then`属性：
  - 2.1、如果`then`是个函数，那么以 x 为`this`调用
  - 2.2、如果`then`不是一个函数，以 x 为值 fulfill promise
- 3、如果 x 不是一个函数或者对象，以 x 为值 fulfill promise

```javascript
const resolvePromise = (promise, x, resolve, reject) => {
  // 避免x === promise时，自身等待自身结果，陷入死循环
  if (x === promise) {
    return reject(new TypeError('Chaining cycle detected for promise'))
  }

  // 避免多次调用
  let called = false
  if (x instanceof Promise) {
    // 如果x是一个promise实例
    if (x.state === PENDING) {
      x.then(y => resolvePromise(promise, y, resolve, reject), e => reject(e))
    } else {
      x.then(resolve, reject)
    }
  } else if (x !== null && (typeof x === 'function' || typeof x === 'object')) {
    // 如果x是一个对象或者函数
    try {
      const then = x.then

      if (typeof then === 'function') {
        then.call(
          x,
          y => {
            if (called) return
            called = true
            resolvePromise(promise, y, resolve, reject)
          },
          e => {
            if (called) return
            called = true
            reject(e)
          }
        )
      } else {
        resolve(x)
      }
    } catch (e) {
      if (called) return
      called = true
      reject(e)
    }
  } else {
    resolve(x)
  }
}
```

## 4. 完善`then`函数

- 1、`then`内还缺少保证执行栈只存在平台代码的操作
- 2、`then`内缺少对 onFulfilled 和 onRejected 的抛错处理

```javascript
class Promise {
  constructor(executor) {
    // 相同代码省略
  }

  then(onFulfilled, onRejected) {
    // 先进行一次判断，即onFulfilled和onRejected是否是函数
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val
    onRejected =
      typeof onRejected === 'function'
        ? onRejected
        : err => {
            throw err
          }

    const promise = new Promise((resolve, reject) => {
      const fulfilledCb = () => {
        setTimeout(() => {
          try {
            const value = onFulfilled(this.value)
            resolvePromise(promise, value, resolve, reject)
          } catch (e) {
            reject(e)
          }
        }, 0)
      }
      const rejectedCb = () => {
        setTimeout(() => {
          try {
            const reason = onRejected(this.reason)
            resolvePromise(promise, value, resolve, reject)
          } catch (e) {
            reject(e)
          }
        })
      }

      // 对promise进行状态判断
      if (this.state === FULFILLED) {
        fulfilledCb()
      } else if (this.state === REJECTED) {
        rejectedCb()
      } else if (this.state === PENDING) {
        // 也可以写成else
        this.fulfilledQueue.push(fulfilledCb)
        this.rejectedQueue.push(rejectedCb)
      }
    })

    return promise
  }
}
```

### 5. 完整的Promise代码

[仓库地址](https://github.com/sunshineuoow/promise)

```javascript
'use strict'

const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

const resolvePromise = (promise, x, resolve, reject) => {
  if (x === promise) throw new TypeError('Chaining cycle detected for promise')

  let called = false
  if (x instanceof Promise) {
    if (x.state === PENDING) {
      x.then(y => resolvePromise(promise, y, resolve, reject), err => reject(err))
    } else {
      x.then(resolve, reject)
    }
  } else if (x !== null && (typeof x === 'function' || typeof x === 'object')) {
    try {
      const then = x.then

      if (typeof then === 'function') {
        then.call(x, y => {
          if (called) return
          called = true
          resolvePromise(promise, y, resolve, reject) 
        }, err => {
          if (called) return
          called = true
          reject(err)
        })
      } else {
        resolve(x)
      }

    } catch (e) {
      if (called) return
      called = true
      reject(e)
    }
  } else {
    resolve(x)
  }
}

class Promise {

  constructor(executor) {
    this.state = PENDING
    this.value = undefined
    this.reason = undefined
    this.fulfilledQueue = []
    this.rejectedQueue = []

    const resolve = value => {
      setTimeout(() =>{
        if (this.state === PENDING) {
          this.state = FULFILLED
          this.value = value
          this.fulfilledQueue.forEach(cb => cb())
        }
      }, 0)
    }

    const reject = reason => {
      setTimeout(() => {
        if (this.state === PENDING) {
          this.state = REJECTED
          this.reason = reason
          this.rejectedQueue.forEach(cb => cb())
        }
      }, 0)
    }
    
    try {
      executor(resolve.bind(this), reject.bind(this))
    } catch(e) {
      reject(e)
    }
  }

  static resolve(value) {
    return new Promise((resolve, reject) => {
      resolve(value)
    })
  }

  static reject(reason) {
    return new Promise((resolve, reject) =>{
      reject(reason)
    })
  }

  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err }
    const promise = new Promise((resolve, reject) => {
      const fulfillCb = () => {
        try {
          const value = onFulfilled(this.value)
          resolvePromise(promise, value, resolve, reject)
        } catch (e) {
          reject(e)
        }
      }
      const rejectCb = () => {
        try {
          const reason = onRejected(this.reason)
          resolvePromise(promise, reason, resolve, reject)
        } catch (e) {
          reject(e)
        }
      }
      if (this.state === FULFILLED) {
        setTimeout(fulfillCb, 0)
      } else if (this.state === REJECTED) {
        setTimeout(rejectCb, 0)
      } else if (this.state === PENDING) {
        this.fulfilledQueue.push(fulfillCb)
        this.rejectedQueue.push(rejectCb)
      }
    })
    return promise
  }

  catch(onRejected) {
    this.then(null, onRejected)
  }
}

// 用于进行promises aplus test
Promise.defer = Promise.deferred = function () {
  let dfd = {}
  dfd.promise = new Promise((resolve,reject)=>{
    dfd.resolve = resolve
    dfd.reject = reject
  })
  return dfd
}

module.exports = Promise
```


## 6. TODO List

1.  增加`all`,`race`方法
2.  增加`x`是 Promise 实例判断后会有部分测试用例不通过，删除或者在构造器中的`resolve`和`reject`函数内加一层 setTimeout即可解决，待研究其原因
