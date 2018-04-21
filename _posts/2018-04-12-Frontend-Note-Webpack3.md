---
layout: post
title: 前端学习笔记之webpack（三）
date: 2018-04-12 14:30:15 +0800
description: Frontend Note for Webpack.
img: what-is-webpack.png
tags: [Frontend, Note, Webpack]
---

# webpack 配置项解析(二)

## 模块(Module)

这个配置项决定了项目中不同模块应当如何处理。

### `module.noParse`

阻止 webpack 解析给定正则匹配到的任何文件。被忽略的文件不应当有`import`, `require`, `define`等导入机制。

```javascript
  noParse: /jquery|lodash/

  // version >= 3.0.0
  noParse: function(content) {
    return /jquery|lodash/.test(content)
  }
```

### `module.rules`

一个用于当模块创建时匹配请求的`Rule`对象的数组。这些规则可以定义如何创建模块。

### `Rule`

一个 Rule 对象可以分为三部分，条件`Conditions`，结果`Results`和嵌套规则`nested rule`

#### Rule 条件`Rule Conditions`

条件的值有两种：

1.  resource: 被请求文件的绝对路径。根据`resolve`规则解析。
2.  issuer: 发出请求文件的绝对路径。
3.  例如：在`app.js`内使用`import './style.css'`， resource 是`/path/to/style.css`，issuer 是`/path/to/app.js`

Rule 对象中，`test`, `include`，`exclude`和`resource`属性针对 resource，
`issuer`谁能够针对 issuer

#### Rule 结果`Rule results`

该配置仅当条件匹配时生效。

Rule 有两种输出值：

1.  应用的 loaders：一个应用在 resource 的 loader 的数组
2.  解析选项：一个用来创建这个模块的解析器时使用的配置对象

`loader`, `options`, `use`影响 loaders，也兼容`query`和`loaders`属性

`enforce`属性影响 loader 种类。

`parser`属性影响解析器配置。

#### 嵌套规则`Nest rules`

嵌套规则在`rules`和`oneOf`属性下配置。

这些规则用于当条件匹配时取值。

#### `Rule.enforce`

可选值：`"pre" | "post"`

指定 loader 的种类。没有值意味着普通的 loader。

还有一种额外种类的"inlined loader",被应用在 import/require 行内。

1.  所有的 loader 按照`pre, inline, normal, post`排序并使用
2.  所有普通 loader 可以通过前缀`!`来忽略（覆盖）
3.  所有普通和前置 loader 可以通过前缀`-!`来忽略（覆盖）
4.  所有普通，前置和后置 loader 可以通过前缀`!!`来忽略（覆盖）
5.  行内 loader 和`!`前缀是不标准的，所以不应当使用。它们可在由 loader 生成的代码中使用。

#### `Rule.loader`

`Rule.user: [ { loader } ]`的缩写。

#### `Rule.oneOf`

一个 Rules 对象的数组，当 Rule 匹配时只使用第一个匹配规则。

```javascript
  {
    test: /.css$/,
    oneOf: [
      {
        resourceQuery: /inline/,
        use: 'url-loader'
      },
      {
        resourceQuery: /external/,
        use: 'file-loader'
      }
    ]
  }
```

#### `Rule.options`/`Rule.query`

`Rule.use: [ { options } ]`的缩写

#### `Rule.parser`

解析器的配置对象。所有应用的解析器选项都会合并。

解析器将检查这些配置并且由此禁用或者重新配置。大多数默认插件会有如下的值：

1.  设置`false`来禁用解析器
2.  设置`true`或者让其保留`underfined`以启用解析器。

然而，解析器插件可能接受多个布尔值。

#### `Rule.issuer`

这个配置用来将 loader 用于一个特定模块或一组模块的依赖。

#### `Rule.resource`

一个 resource 的匹配条件。可以是使用`Rule.resource`或者`Rule.test`，`Rule.exclude`和`Rule.include`

#### `Rule.resourceQuery`

一个 resource query 的匹配条件。

```javascript
  // 匹配 'import Foo from './foo.css?inline'
  {
    test: /.css$/,
    resourceQuery: /inline/,
    use: 'url-loader'
  }
```

#### `Rule.test`

`Rule.resource.test`的缩写。如果使用了该配置，则不能再使用`Rule.resource`

#### `Rule.exclude`

`Rule.resource.exclude`的缩写。如果使用了该配置，则不能再使用`Rule.resource`

#### `Rule.include`

`Rule.resource.include`的缩写。如果使用了该配置，则不能再使用`Rule.resource`

#### `Rule.use`

一个应用在模块的使用入口(UseEntries)列表。每个入口指定一个使用的 loader。

传入一个字符串(i.e. `use: [ "style-loader" ]`)是 loader 属性的缩写(i.e. `use: [ { loader: "style-loader" } ]`)

Loaders 可以被链式调用，调用顺序为从右至左，即最后一个传入的先调用。

### `Condition`

条件可以是以下任意一种形式：

1.  一个字符串：匹配的输入应当是以给出的字符串开头。如一个文件夹的绝对路径，或者一个文件的绝对路径。
2.  一个正则表达式：用于匹配输入。
3.  一个函数：调用输入值并且必须在匹配时返回 true。
4.  一个条件数组：必须匹配至少一个条件。
5.  一个对象：所有的属性必须匹配。每个属性有一个定义的行为。

`{ test: Condition }`: 条件必须匹配。一般给出一个正则或者一个正则的数组(非强制)。

`{ include: Condition }`: 条件必须匹配。一般给出一个字符串或者一个字符串数组(非强制)。

`{ exclude: Condition }`: 条件必须不匹配。一般给出一个字符串或者一个字符串数组(非强制)。

`{ and: [Condition] }`: 所有条件必须匹配。

`{ or: [Condition] }`: 任意条件必须匹配。

`{ not: [Condition] }`: 所有条件必须不匹配。

例子：

```javascript
  // 匹配 app/styles和 vendor/styles下所有.css结尾文件
  {
    test: /\.css$/,
    include: [
      path.resolve(__dirname, "app/styles"),
      path.resolve(__dirname), "vendor/styles")
    ]
  }
```

### `UseEntry`

必须有一个`loader`属性且值为字符串。

有一个`options`属性是字符串或者对象，是 loader 的配置。

由于兼容性，也有一个`query`属性，是`options`属性的别名。用`options`属性替代即可。

Example:

```javascript
  {
    loader: "css-loader",
    options: {
      module: true
    }
  }
```

注意 webpack 需要根据 resource 和包含配置的 loaders 生成一个唯一的模块标识。这会尝试使用`JSON.stringify`来序列化 options 对象。在 99.9%的情况下是成功的，但是如果你对 resource 使用相同的 loader 却用了不同的配置，并且配置有一些已经序列化的值，那么可能会出问题。

如果配置对象不能被序列化那么会中断。因此你可以使用一个`ident`属性作为唯一标识。
