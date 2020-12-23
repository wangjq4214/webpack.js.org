---
title: 入口和上下文(entry and context)
sort: 4
contributors:
  - sokra
  - skipjack
  - tarang9211
  - byzyk
  - madhavarshney
  - EugeneHlushko
  - smelukov
  - anshumanv
  - snitin315
---

入口对象是用于 webpack 查找开始构建 bundle 的地方。上下文是入口文件所处的目录的绝对路径的字符串。


## `context` {#context}

`string`

基础目录，__绝对路径__，用于从配置中解析入口点(entry point)和 加载器(loader)。

``` js
const path = require('path');

module.exports = {
  //...
  context: path.resolve(__dirname, 'app')
};
```

默认使用当前目录，但是推荐在配置中传入一个值。这使得你的配置独立于 CWD(current working directory, 当前工作目录)。

---


## `entry` {#entry}

`string` `[string]` `object = { <key> string | [string] | object = { import string | [string], dependOn string | [string], filename string }}` `(function() => string | [string] | object = { <key> string | [string] } | object = { import string | [string], dependOn string | [string], filename string })`

开始应用程序打包过程的一个或多个起点。如果传入数组，则会处理所有条目。

动态加载的模块 __不是__ 入口起点。

简单规则：每个 HTML 页面都有一个入口起点。单页应用(SPA)：一个入口起点，多页应用(MPA)：多个入口起点。

```js
module.exports = {
  //...
  entry: {
    home: './home.js',
    about: './about.js',
    contact: './contact.js'
  }
};
```


### Naming {#naming}

如果传入一个字符串或字符串数组，chunk 会被命名为 `main`。如果传入一个对象，则每个属性的键(key)会是 chunk 的名称，该属性的值描述了 chunk 的入口点。

### Entry descriptor {#entry-descriptor}

如果传入一个对象，对象的属性的值可以是一个字符串、字符串数组或者一个描述符(descriptor):

```js
module.exports = {
  //...
  entry: {
    home: './home.js',
    shared: ['react', 'react-dom', 'redux', 'react-redux'],
    catalog: {
      import: './catalog.js',
      filename: 'pages/catalog.js',
      dependOn:'shared'
    },
    personal: {
      import: './personal.js',
      filename: 'pages/personal.js',
      dependOn: 'shared',
      chunkLoading: 'jsonp',
    }
  }
};
```

描述符语法可以用来传入额外的选项给入口。


### Output filename {#output-filename}

默认情况下，入口 chunk 的输出文件名是从 [`output.filename`](/configuration/output/#outputfilename) 中提取出来的，但你可以为特定的入口指定一个自定义的输出文件名。

```js
module.exports = {
  //...
  entry: {
    app: './app.js',
    home: { import: './contact.js', filename: 'pages/[name][ext]' },
    about: { import: './about.js', filename: 'pages/[name][ext]' }
  }
};
```

描述符语法在这里被用来将 `filename`—选项传递给指定的入口点。


### Dependencies {#dependencies}

默认情况下，每个入口 chunk 保存了全部其用的模块(modules)。使用 `dependOn` 选项你可以与另一个入口 chunk 共享模块:

```js
module.exports = {
  //...
  entry: {
    app: { import: './app.js', dependOn: 'react-vendors' },
    'react-vendors': ['react', 'react-dom', 'prop-types']
  }
};
```

`app` 这个 chunk 就不会包含 `react-vendors` 拥有的模块了.

`dependOn` 选项的也可以为字符串数组：

```js
module.exports = {
  //...
  entry: {
    moment: { import: 'moment-mini', runtime: 'runtime' },
    reactvendors: { import: ['react', 'react-dom'], runtime: 'runtime' },
    testapp: {
      import: './wwwroot/component/TestApp.tsx',
      dependOn: ['reactvendors', 'moment'],
    },
  },
};
```

此外，你还可以使用数组为每个入口指定多个文件：

```js
module.exports = {
  //...
  entry: {
    app: { import: ['./app.js', './app2.js'], dependOn: 'react-vendors' },
    'react-vendors': ['react', 'react-dom', 'prop-types']
  }
};
```

### Dynamic entry {#dynamic-entry}

如果传入一个函数，那么它将会在每次 [make](/api/compiler-hooks/#make) 事件中被调用。

> 要注意的是，make 事件在 webpack 启动和每当 [监听文件变化](/configuration/watch/) 时都会触发。

```js
module.exports = {
  //...
  entry: () => './demo'
};
```

或者

```js
module.exports = {
  //...
  entry: () => new Promise((resolve) => resolve(['./demo', './demo2']))
};
```

例如，你可以使用动态入口来从外部来源（远程服务器，文件系统内容或者数据库）获取真正的入口：

__webpack.config.js__

``` js
module.exports = {
  entry() {
    return fetchPathsFromSomeExternalSource(); // 返回一个会被用像 ['src/main-layout.js', 'src/admin-layout.js'] 的东西 resolve 的 promise
  }
};
```

当和 [`output.library`](/configuration/output/#outputlibrary) 选项结合：如果传入的是一个数组，只有数组的最后一个条目会被导出。
