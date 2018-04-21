---
layout: post
title: 前端学习笔记之webpack（四）
date: 2018-04-12 18:30:15 +0800
description: Frontend Note for Webpack.
img: what-is-webpack.png
tags: [Frontend, Note, Webpack]
---

# webpack 配置项解析(三)

## Resolve

这个配置设置如何解析模块。webpack 提供了可靠的默认值，但是可以改变解析的细节。

### `resolve`

配置如何解析模块。例如，当使用`import "lodash"`在 ES2015 中，`resolve`配置可以改变 webpack 寻找`"lodash"`的位置。

#### `resolve.alias`

创建`import`或`require`别名使得模块引入更加简单。例如，给`src/`目录下的一些常用模块起别名：

```javascript
  // webpack.config.js
  alias: {
    Utilities: path.resolve(__dirname, 'src/utilities/'),
    Templates: path.resolve(__dirname, 'src/templates/')
  }

  // app.js (没有别名)
  import Utility from '../../utilities/utility'

  // app.js (使用别名)
  import Utility from 'Utilities/utility'
```

可以在对象的 key 后面添加一个`$`以表示精准匹配。

```javascript
// webpack.config.js
alias: {
  xyz$: path.resolve(__dirname, 'path/to/file.js')
}

// app.js
import Test1 from 'xyz' // 导入path/to/file.js文件
import Test2 from 'xyz/file.js' // 导入'xyz/file.js'
```

下方表格列出了所有的情况：

|              alias               |            import "xyz"             |       import "xyz/file.js"        |
| :------------------------------: | :---------------------------------: | :-------------------------------: |
|                {}                |   /abc/node_modules/xyz/index.js    |   /abc/node_modules/xyz/file.js   |
| { xyz: "/abs/path/to/file.js" }  |        /abs/path/to/file.js         |               error               |
| { xyz$: "/abs/path/to/file.js" } |        /abs/path/to/file.js         |   /abc/node_modules/xyz/file.js   |
|     { xyz: "./dir/file.js" }     |          /abc/dir/file.js           |               error               |
|    { xyz$: "./dir/file.js" }     |          /abc/dir/file.js           |   /abc/node_modules/xyz/file.js   |
|       { xyz: "/some/dir "}       |         /some/dir/index.js          |        /some/dir/index.js         |
|      { xyz$: "/some/dir" }       |         /some/dir/index.js          |   /abc/node_modules/xyz/file.js   |
|         { xyz: "./dir" }         |          /abc/dir/index.js          |         /abc/dir/file.js          |
|         { xyz: "modu" }          |   /abc/node_modules/modu/index.js   |  /abc/node_modules/modu/file.js   |
|         { xyz$: "modu" }         |   /abc/node_modules/modu/index.js   |   /abc/node_modules/xyz/file.js   |
|   { xyz: "modu/some/file.js" }   | /abc/node_modules/modu/some/file.js |               error               |
|       { xyz: "modu/dir" }        | /abc/node_modules/modu/dir/index.js |   /abc/node_modules/dir/file.js   |
|        { xyz: "xyz/dir" }        | /abc/node_modules/xyz/dir/index.js  | /abc/node_modules/xyz/dir/file.js |
|       { xyz$: "xyz/dir" }        | /abc/node_modules/xyz/dir/index.js  |   /abc/node_modules/xyz/file.js   |

`abc/node_modules`也可能在`/node_modules`中解析

#### `resolve.aliasFields`

指定一个字段，例如`browser`，根据指定规范解析。

```javascript
// 默认值
aliasFields: ['browser']
```

#### `resolve.cacheWithContext`(version >= 3.1.0)

如果启用了不安全缓存，请在缓存键(cache key)中包含`request.context`。`enhanced-resolve`模块将该选项考虑在内。从 3.1.0 版本开始，配置了 resolve 或 resolveLoader 插件时，解析缓存中的上下文会被忽略，用于解决性能衰退。

#### `resolve.descriptionFiles`

用于描述的 JSON 文件，默认值为`descriptionFiles: ["package.json"]`

#### `resolve.enforceExtension`

如果值为`true`，将不允许无拓展名文件。因此当启用时，只有`require('./foo.js')`生效。

默认值为`false`。

#### `resolve.enforceModuleExtension`

对模块是否需要使用拓展(例如 loaders)，默认值为`false`

### `resolve.extensions`

自动解析特定的拓展。

默认值是`extensions: [".js", ".json"]`

允许用户当引用时不需要书写拓展名。使用此选项将覆盖默认的数组，意味着 webpack 将不再使用默认拓展名去解析模块。想要正确解析使用拓展名导入的模块(`import SomeFile from "./somefile.ext"`)，数组中必须包含`"*"`

#### `resolve.mainFields`

当引用一个 npm 包时，例如`import * as D3 from "D3"`，这个选项决定在`package.json`中使用哪个字段导入模块。默认值随着`target``配置项的不同而改变。

1.  `target`为`webworker`, `web`或者未指定时：`mainFields: ["browser", "module", "main"]`
2.  其他的`target`值: `mainFields: ["module", "main"]`

#### `resolve.mainFiles`

当解析文件夹时使用的文件名。默认值`mainFiles: ["index"]`

#### `resolve.modules`

告知 webpack 解析模块时在哪个文件夹搜索。

绝对和相对路径都可以使用，但是他们的表现会有些不同。

相对路径会类似于 Node 搜索`node_modules`，会搜索当前文件夹及上级目录。(如`./node_modules`, `../node_modules`至根目录)

绝对路径将只搜索给定的文件夹。

默认值为`modules: ["node_modules"]`

如果想添加一个文件夹并且优于`node_modules/`搜索:

```javascript
modules: [path.resolve(__dirname, 'src'), 'node_modules']
```

#### `resolve.unsafeCache`

启用激进但是不安全的模块。传递`true`将缓存所有文件。默认`unsafeCache: true`

可以传递一个正则或者一个正则的数组，用于仅仅缓存特定模块。

改变缓存路径在少数情况下可能会导致失败。

#### `resolve.plugins`

一系列需要使用的解析插件。类似于

```javascript
plugins: [new DirectoryNamedWebpackPlugin()]
```

#### `resolve.symlinks`

是否解析 symlinks 到它们接的位置(symlinked location)。默认值`true`

当允许时，symlinked 的资源将被解析到真实路径，而不是 symlinked location。注意这可能导致当使用 symlink 工具包(如`npm link`)时，模块解析错误。

#### `resolve.cachePredicate`

一个决定是否缓存请求的函数。函数参数为一个带着`path`和`request`属性的对象。

默认值为`cachePredicate: function() { return true }`

### `resolveLoader`

这个配置项属性与`resolve`的配置项相同，但是仅仅用来解析 webpack 的 loader 包

```javascript
  // 默认值
  {
    modules: [ 'node_modules' ],
    extensions: [ '.js' , '.json' ],
    mainFields: [ 'loader', 'main' ]
  }
```

#### `resolveLoader.moduleExtensions`

当解析 loaders 是使用的拓展名。自从 2.0.0 版本以来，我们强烈推荐使用全名，例如`example-loader`。然而，如果你真的想省略`-loader`，可以使用这个属性这么做：

```javascript
moduleExtensions: ['-loader']
```

## Externals

`externals`配置项提供了从输出的 bundle 中排除依赖的方法。然而，创建出来的 bundle 依赖于当前用户环境的依赖。此功能最适用于库的开发者(library developers)，然而也有许多应用场景。

### `externals`

阻止打包某些引用的依赖，并且在运行时将这些外在的依赖取回。

例如，使用 CDN 引入 jQuery 而非打包它:

```html
  <script src="https://code.jquery.com/jquery-3.1.0.js"></script>
```

```javascript
// webpack.config.js
externals: {
  jquery: 'jQuery'
}

// app.js
import $ from 'jquery'
```

这就剔除了哪些不需要改动的模块。

具有外部依赖的 bundle 可以在各种模块上下文中使用，例如 CommonJS, AMD, global 和 ES2015 模块。

外部库可以是以下任意一种形式：

1.  **root**: 通过一个全局变量访问(例如 script 标签)
2.  commonjs: 是一个 CommonJS 模块
3.  commonjs2: 和 CommonJS 模块相似但是导出是`module.exports.default`
4.  amd: 和`commonjs`相似但是使用 AMD 模块系统

接受以下句法:

#### string

见上述示例。属性名`jquery`表明`import $ from 'jquery'`导入的`jquery`模块将被排除。为了替代这个模块，属性值`jQuery`将从全局变量`jQuery`中取回。换句话说，当提供一个字符串时，它将被当做`root`(在上方和下方定义)

#### array

```javascript
externals: {
  subtract: ['./math', 'subtract']
}
```

转化为一个父子结构，`./main`是父模块并且你的 bundle 只引用`subtract`变量下的子集

#### object

```javascript
  externals: {
    react: 'react'
  }

  externals: {
    lodash: {
      commonjs: "lodash",
      amd: "lodash",
      root: "_"
    }
  }

  externals: {
    subtract: {
      root: ["math", "subtract"]
    }
  }
```

这个语法用于描述外部库所有可能的获取方式。

`lodash`这里可以通过`lodash`在 AMD 和 CommonJS 模块系统下访问，但是在全局变量中以`_`访问。

`subtarct`通过全局的`math`对象下属性`subtract`访问。

#### function

定义你自己的函数来控制你想要将哪些库从 webpack 中隔离出来可能十分有用。

例如，webpack-node-externals 将所有模块从`node_modules`中排除并且提供一些配置项去生成一个白名单。

它基本上归结于此

```javascript
externals: [
  function(context, request, callback) {
    if (/^yourregex$/.test(request)) {
      return callback(null, 'commonjs ' + request)
    }
    callback()
  }
]
```

`'commonjs ' + request`定义了需要排除的模块的类型

#### regex

每个被给定正则匹配到的依赖都会从输出的 bundles 中移除。

```javascript
externals: /^(jquery|\$)$/i
```

这个例子中任何命名为`jQuery`(不区分大小写)或者`$`将会被排除
