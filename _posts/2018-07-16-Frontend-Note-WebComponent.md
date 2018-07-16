---
layout: post
title: 前端学习笔记之Web Component(一)
date: 2018-7-16 15:58:00 +0800
description: Frontend Note for Web Component.
img: web-component.png
tags: [Frontend, Note, WebComponent]
---

# Web Component

## 1. 什么是 web component

引自 MDN：

> Web Components is a suite of different technologies allowing you to create reusable custom elements — with their functionality encapsulated away from the rest of your code — and utilize them in your web apps.

简单来说，web component 就是封装好具备自己功能的，可复用的自定义元素

### web component 基于以下四个主要说明：

<hr>

#### 1.1 Custom Elements

自定义元素声明允许设计并使用新类型的 DOM 元素

#### 1.2 shadow DOM

shadow DOM 声明定义了如何在 web component 内封装样式及内容

#### 1.3 HTML imports

HTML imports 声明定义了在其他文档中包含和重用 HTML 文档

#### 1.4 HTML Template

HTML template 元素声明定义了如何声明在页面加载时不使用，但是在运行环境下实例化的文档碎片

### 2. 如何使用 web component

```html
  <link rel="import", href="path/to/your/component.html">
  ...
  <my-component></my-component>
```

将定义好的 web component 以该方式引用，而后在页面中直接当成 html 标签书写即可

### 3. 如何定义一个新的 HTML 元素

那么如何定义一个新的 HTML 元素呢？

```javascript
var myComponent = document.registerElement('my-component')
document.body.appendChild(new myComponent())
```

通过`document.registerElement`可以定义一个新的 HTML 元素，并且可以直接使用

#### 3.1 document.registerElement()

`document.registerElement`方法接收 2 个参数

第一个参数为`String`类型，表示自定义元素的标签名，必须包含`-`符，用于区分自定义标签与原生标签

第二个参数表示自定义元素的原型对象，可以自定义 API

```javascript
var myComponentProto = Object.create(HTMLElement.prototype)
myComponentPtoro.hello = function() {
  console.log('hello web component!')
}
document.registerElement('my-component', {
  prototype: myComponentProto
})

var myComponent = document.querySelector('my-component')
myComponent.hello()
```

若是想拓展  某个 HTML 元素或者自定义元素，可以在第二个参数内加上 `extends`属性

```javascript
var myDivProto = Object.create(HTMLElement.prototype)
document.registerElement('my-div', {
  prototype: myDivProto,
  extends: 'div'
})
```

之后使用的话有两种方式

```html
  <my-div></my-div>
  <div is="my-div"></div>
```

### 3.2 回调函数

自定义元素有一些属性，用于指定回调函数，在特定事件发生时触发

```
  createdCallback: 实例生成时触发
  attachedCallback: 实例插入HTML文档时触发
  detachedCallback: 实例从HTML文档移除时触发
  attributeChangedCallback(attrName, oldVal, newVal): 实例属性发生改变时触发(添加，移除，更新)
```

利用回调函数，可以方便的在自定义元素中插入 HTML

```javascript
var myComponentProto = Object.create(HTMLElement.prototype)
myComponentProto.createdCallback = function() {
  this.innerHTML = 'hello <b>web component</b>!'
}
myComponentProto.attachedCallback = function() {
  console.log('attached')
}
document.registerElement('my-component', { prototype: myComponentProto })
```

在页面中使用自定义元素的话则会展示为如下

```html
  <my-component>
    hello <b>web component</b>!
  </my-component>
```

## 4. Shadow DOM

Shadow DOM 指的是，浏览器将模板、样式表、属性、JavaScript 代码等，封装成一个独立的 DOM 元素。外部的设置无法影响到其内部，而内部的设置也不会影响到外部。

Shadow DOM 必须依存于一个现有的 DOM 元素，并且  通过`createShadowRoot`方法创造，然后将其插入该元素

```html
  <div class="box"></div>
  <script>
    var host = document.querySelector('box')
    var root = host.createShadowRoot()
    root.innerHTML = "<p>I'm shadow DOM</p>"
  </script>
```

为 Shadow DOM 加上独立的模板

```html
  <div id="my-component"></div>

  <template>
    <style>
      b {
        color: red;
      }
    </style>

    <div class="outer">
      <p>I'm <b>shadow DOM</b>!</p>
    </div>
  </template>

  <script>
    var root = document.querySelector('#my-component').createShadowRoot()
    var template = document.querySelector('template')
    var importNode = document.importNode(template.content, true)
    root.appendChild(importNode)
  </script>
```

## 5. web component 封装

```html
  <template>
    <style>
      ::content > * {
        color: blue;
      }
      p {
        color: red;
      }
    </style>
    <p>I'm a shadow-element using Shadow DOM!</p>
    <content></content>
  </template>

  <script>
    var importDoc = document.currentScript.ownerDocument
    var myComponentProto = Object.create(HTMLElement.prototype)
    myComponentProto.createdCallback = function() {
      var template = importDoc.querySelector('template')
      var clone = document.importNode(template.content, true)
      var root = this.createShadowRoot()
      root.appendChild(clone)
    }
    document.registerElement('my-component', { prototype: myComponentProto })
  </script>
```