---
layout: post
title: 前端学习笔记之webpack（二）
date: 2018-04-11 16:20:20 +0800
description: Frontend Note for Webpack.
img: what-is-webpack.png
tags: [Frontend, Note, Webpack]
---

# webpack 配置项解析

## 1. 入口及上下文(Entry and Context)

入口即告知 webpack 从何处开始构建 bundle，上下文是包含入口文件的文件夹的绝对路径字符串

### `context`

```javascript
context: path.resolve(__dirname, 'app')
```

上下文的默认值是根目录，但是推荐在配置中传递一个值使得配置与当前工作的文件夹脱离

### `entry`

```
String | Array | Object | Function
```

`注：动态加载的模块不是入口。`

简单的规则：每个 HTML 页面一个入口。<br>
单页面应用程序（SPA）：一个入口。<br>
多页面应用程序（MPA）：多个入口。<br>

如果 Entry 是一个字符或者数组，那么打包出来的文件名为`main`

如果是一个对象，那么对象的`key`则是打包出来的文件名，`value`是入口文件的路径

也支持动态的入口，即通过函数来确定入口文件，函数的返回值必须是`String | Array | Object`

## 2. 输出(Output)

输出配置项告知 webpack 将打包好的文件或静态资源如何输出并且输出到哪

### `chunkFilename`

非入口块文件的名称配置。这些文件通常在运行中根据请求去生成。默认是`[id].js`或者从`output.filename`推断出来的值

### `filename`

输出文件的名称，将会将文件写在`output.path`配置的文件夹下

对于一个单入口应用，该属性可以是一个静态的名字

```javascript
filename: 'bundle.js'
```

当打包多入口应用时，可以通过以下使用方式给每个打包文件命名

```javascript
// 使用入口名
filename: '[name].js'

// 使用内部块的id
filename: '[id].js'

// 使用每次构建不同的hash值(最好与入口名一起使用)
filename: '[name].[hash].js'

// 使用基于每个块内容的hash值
filename: '[chunkhash].js'

// [hash]和[chunkhash]可以使用长度[hash:16]来指定(默认20)
```

也可以使用`js/[name]/bundle.js`来创建文件夹结构

这个配置不会影响到按需加载的块。这些块应当使用`output.chunkFilename`

loaders 创建的文件也不会呗影响。因此可以去尝试具体 loader 的配置

### `hashDigest`

生成 hash 时使用的编码方式，默认是`hex`，支持 Node.js 内`hash.digest`的所有编码方式

### `hashDigestLength`

hash 摘要的前缀长度，默认 20

### `hashFunction`

hash 算法，默认是`md5`，支持 Node.js 内`crypto.createHash`的所有方法

### `hashSalt`

一个可选的加盐值，通过 Node.js`hash.update`更新 hash

### `output.hotUpdateChunkFilename`

自定义热更新块的文件名。

占位符只允许`[id]`和`[hash]`，默认值为

```javascript
// 没有必要修改
hotUpdateChunkFilename: '[id].[hash].hot-update.js'
```

### `path`

输出文件夹的绝对路径

```javascript
path: path.resolve(__dirname, 'dist')
```

### `pathinfo`

告知 webpack 在将所包含模块的信息写入打包出来的文件中，默认值为`false`，并且不应当用在生产环境

### `publicPath`

这个配置项在使用按需加载或者加载外部资源如图片、文件等时特别重要。

该选项指定在浏览器中引用[此输出目录对应的**公开 URL**]。相对的 URL 会被相对于 HTML 页面解析。

相对于服务器的 URL，相对于协议的地址或者绝对地址都可以使用。

举个例子：

```javascript
  path: path.resolve(__dirname, "dist"),
  publicPath: "/assets/"
```

```html
  <script src="/assets/main.js">
```

可选值

```javascript
  publicPath: "https://cdn.example.com/assets/", // CDN (永远是HTTPS)
  publicPath: "//cdn.example.com/assets/", // CDN (相同协议)
  publicPath: "/assets/", // 服务端相对地址
  publicPath: "assets/", // HTML页面相对地址
  publicPath: "../assets/", // HTML页面相对地址
  publicPath: "", // HTML页面相对地址(同一文件夹)
```

### `sourceMapFilename`

仅当`devtool`使用了`SourceMap`时使用，用于定义 source map 如何命名。

`[name]`，`[id]`,`[hash]`,`[chundhash]`均可以使用。

推荐使用`[file]`占位符，即源文件的文件名，因为其他的占位符在生成非块文件时不起作用。

### `sourcePrefix`

改变输出文件的每行前缀。

```javascript
// 没有必要修改
sourcePrefix: '\t'
```

默认情况下使用空字符串。

### `striceModuleExceptionHandling`

告知 webpack 从模块实例缓存中移除一个在`require`时异常的模块。

默认是 false（由于性能原因）

```javascript
// strictModuleExceptionHandling = false
require('module') // error
require('module') // no error

// strictModuleExceptionHandling = true
require('module') // error
require('module') // error
```
