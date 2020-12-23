---
title: 外部扩展(externals)
sort: 15
contributors:
  - sokra
  - skipjack
  - pksjce
  - fadysamirsadek
  - byzyk
  - zefman
  - Mistyyyy
  - jamesgeorge007
  - tanhauhau
  - snitin315
  - beejunk
  - EugeneHlushko
  - chenxsan
  - pranshuchittora
  - kinetifex
---

`externals` 配置选项提供了「从输出的 bundle 中排除依赖」的方法。相反，所创建的 bundle 依赖于那些存在于用户环境(consumer's environment)中的依赖。此功能通常对 __library 开发人员__来说是最有用的，然而也会有各种各样的应用程序用到它。


## `externals` {#externals}

`string` `[string]` `object` `function`  `RegExp`

__防止__将某些 `import` 的包(package)__打包__到 bundle 中，而是在运行时(runtime)再去从外部获取这些_扩展依赖(external dependencies)_。

例如，从 CDN 引入 [jQuery](https://jquery.com/)，而不是把它打包：

__index.html__

``` html
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous">
</script>
```

__webpack.config.js__

```javascript
module.exports = {
  //...
  externals: {
    jquery: 'jQuery'
  }
};
```

这样就剥离了那些不需要改动的依赖模块，换句话，下面展示的代码还可以正常运行：

```javascript
import $ from 'jquery';

$('.my-element').animate(/* ... */);
```

具有外部依赖(external dependency)的 bundle 可以在各种模块上下文(module context)中使用，例如 [CommonJS, AMD, 全局变量和 ES2015 模块](/concepts/modules)。外部 library 可能是以下任何一种形式：

- __root__：可以通过一个全局变量访问 library（例如，通过 script 标签）。
- __commonjs__：可以将 library 作为一个 CommonJS 模块访问。
- __commonjs2__：和上面的类似，但导出的是 `module.exports.default`.
- __amd__：类似于 `commonjs`，但使用 AMD 模块系统。

可以接受以下语法……


### 字符串 {#string}

请查看上面的例子。属性名称是 `jquery`，表示应该排除 `import $ from 'jquery'` 中的 `jquery` 模块。为了替换这个模块，`jQuery` 的值将被用来检索一个全局的 `jQuery` 变量。换句话说，当设置为一个字符串时，它将被视为`全局的`（定义在上面和下面）。

另一方面，如果你想将一个符合 CommonJS 模块化规则的类库外部化，你可以提供外联类库的类型以及类库的名称。

如果你想将 `fs-extra` 从输出的 bundle 中剔除并在运行时中引入它，你可以如下定义：

```javascript
module.exports = {
  // ...
  externals: {
    'fs-extra': 'commonjs2 fs-extra',
  }
};
```

这样的做法会让任何依赖的模块都不变，正如以下所示的代码：

```javascript
import fs from 'fs-extra';
```

会将代码编译成：

```javascript
const fs = require('fs-extra');
```

### `[string]` {#string}

```javascript
module.exports = {
  //...
  externals: {
    subtract: ['./math', 'subtract']
  }
};
```

`subtract: ['./math', 'subtract']` 转换为父子结构，其中 `./math` 是父模块，而 bundle 只引用 `subtract` 变量下的子集。该例子会编译成 `require('./math').subtract;`

### 对象 {#object}

W> 一个形如  `{ root, amd, commonjs, ... }` 的对象仅允许用于  [`libraryTarget: 'umd'`](/configuration/output/#outputlibrarytarget) 这样的配置.它不被允许 用于其它的 library targets 配置值.

```javascript
module.exports = {
  //...
  externals : {
    react: 'react'
  },

  // 或者

  externals : {
    lodash : {
      commonjs: 'lodash',
      amd: 'lodash',
      root: '_' // 指向全局变量
    }
  },

  // 或者

  externals : {
    subtract : {
      root: ['math', 'subtract']
    }
  }
};
```

此语法用于描述外部 library 所有可用的访问方式。这里 `lodash` 这个外部 library 可以在 AMD 和 CommonJS 模块系统中通过 `lodash` 访问，但在全局变量形式下用 `_` 访问。`subtract` 可以通过全局 `math` 对象下的属性 `subtract` 访问（例如 `window['math']['subtract']`）。


### 函数 {#function}

`function ({ context, request }, callback)`

对于 webpack 外部化，通过定义函数来控制行为，可能会很有帮助。例如，[webpack-node-externals](https://www.npmjs.com/package/webpack-node-externals) 能够排除 `node_modules` 目录中所有模块，还提供一些选项，比如白名单 package(whitelist package)。

函数接收两个入参：

- `ctx` (`object`)：包含文件详情的对象。
  - `ctx.context` (`string`): 包含引用的文件目录。
  - `ctc.request` (`string`): 被请求引入的路径。
- `callback` (`function (err, result, type)`): 用于指明模块如何被外部化的回调函数

回调函数接收三个入参：

- `err` (`Error`): 被用于表明在外部外引用的时候是否会产生错误。如果有错误，这将会是唯一被用到的参数。
- `result` (`string` `[string]` `object`): 描述外部化的模块。可以接受形如 `${type} ${path}` 格式的字符串，或者其它标准化外部化模块格式，([`string`](#string), [`[string]`](#string-1)，或 [`object`](#object))。
- `type` (`string`): 可选的参数，用于指明模块的类型（如果它没在 `result` 参数中被指明）。

作为例子，要外部化所有匹配一个正则表达式的引入，你可以像下面那样处理：

__webpack.config.js__

```javascript
module.exports = {
  //...
  externals: [
    function({ context, request }, callback) {
      if (/^yourregex$/.test(request)){
        // 使用 request 路径，将一个 commonjs 模块外部化
        return callback(null, 'commonjs ' + request);
      }

      // 继续下一步且不外部化引用
      callback();
    }
  ]
};
```

其它例子使用不同的模块格式：

__webpack.config.js__

```javascript
module.exports = {
  externals: [
    function(ctx, callback) {
      // 该外部化的模块，是一个 `commonjs2` 的模块，且放在 `@scope/library` 目录中
      callback(null, '@scope/library', 'commonjs2');
    }
  ]
};
```

__webpack.config.js__

```javascript
module.exports = {
  externals: [
    function(ctx, callback) {
      // 该外部化模块是一个全局变量叫作 `nameOfGlobal`.
      callback(null, 'nameOfGlobal');
    }
  ]
};
```

__webpack.config.js__

```javascript
module.exports = {
  externals: [
    function(ctx, callback) {
      // 该外部化模块是一个在`@scope/library`模块里的命名导出（named export）。
      callback(null, ['@scope/library', 'namedexport'], 'commonjs');
    }
  ]
};
```

__webpack.config.js__

```javascript
module.exports = {
  externals: [
    function(ctx, callback) {
      // 外部化模块是一个 UMD 模块
      callback(null, {
        root: 'componentsGlobal',
        commonjs: '@scope/components',
        commonjs2: '@scope/components',
        amd: 'components'
      });
    }
  ]
};
```

### RegExp {#regexp}

匹配给定正则表达式的每个依赖，都将从输出 bundle 中排除。

__webpack.config.js__

```javascript
module.exports = {
  //...
  externals: /^(jquery|\$)$/i
};
```

这个示例中，所有名为 `jQuery` 的依赖（忽略大小写），或者 `$`，都会被外部化。

### 混用语法 {#combining-syntaxes}

有时你需要混用上面介绍的语法。这可以像以下这样操作：

__webpack.config.js__

```javascript
module.exports = {
  //...
  externals: [
    {
      // 字符串
      react: 'react',
      // 对象
      lodash : {
        commonjs: 'lodash',
        amd: 'lodash',
        root: '_' // indicates global variable
      },
      // 字符串数组
      subtract: ['./math', 'subtract']
    },
    // 函数
    function({ context, request }, callback) {
      if (/^yourregex$/.test(request)){
        return callback(null, 'commonjs ' + request);
      }
      callback();
    },
    // 正则表达式
    /^(jquery|\$)$/i
  ]
};
```

W> [Default type](/configuration/externals/#externalstype) will be used if you specify `externals` without a type e.g. `externals: { react: 'react' }` instead of `externals: { react: 'commonjs-module react' }`.

关于如何使用此 externals 配置的更多信息，请参考 [如何编写 library](/guides/author-libraries)。


## `externalsType` {#externalstype}

`string = 'var'`

指定 externals 的默认类型。当 external 被设置为 `amd`，`umd`，`system` 以及 `jsonp` 时，**[`output.libraryTarget`](/configuration/output/#outputlibrarytarget)** 的值也应相同。例如，你只能在 `amd` 库中使用 `amd` 的 externals。

支持的类型如下：

- `'var'`
- `'module'`
- `'assign'`
- `'this'`
- `'window'`
- `'self'`
- `'global'`
- `'commonjs'`
- `'commonjs-module'`
- `'amd'`
- `'amd-require'`
- `'umd'`
- `'umd2'`
- `'jsonp'`
- `'system'`
- `'promise'` - 与 `'var'` 相同，但会 awaits 结果（适用于 async 模块）
- `'import'` - 使用 `import()` 加载原生的 EcmaScript 模块（适用于 async 模块）
- `'script'` - 使用 HTML 的 `<script>` 元素加载 script，用于暴露预定义的全局变量

__webpack.config.js__

```javascript
module.exports = {
  //...
  externalsType: 'promise'
};
```

### `script` {#script}

External script can be loaded from any URL when [`externalsType`](#externalstype) is set to `'script'`. The `<script>` tag would be removed after the script has been loaded.

#### Syntax {#syntax}

```javascript
module.exports = {
  externals: {
    packageName: ['http://example.com/script.js', 'global', 'property', 'property'] // properties are optional
  }
};
```

如果你不打算定义任何熟悉，你可以使用简写形式：

```javascript
module.exports = {
  externals: {
    packageName: 'global@http://example.com/script.js' // no properties here
  }
};
```

T> [`output.publicPath`](/configuration/output/#outputpublicpath) 不会被添加到提供的 URL 中。

#### 示例 {#example}

从 CDN 加载 `lodash`：

__webpack.config.js__

```js
module.exports = {
  // ...
  externalsType: 'script',
  externals: {
    lodash: ['https://cdn.jsdelivr.net/npm/lodash@4.17.19/lodash.min.js', '_'],
  }
};
```

然后，代码中使用方式如下：

```js
import _ from 'lodash';
console.log(_.head([1, 2, 3]));
```

下面示例是针对上面示例新增了属性配置：

```js
module.exports = {
  // ...
  externalsType: 'script',
  externals: {
    lodash: ['https://cdn.jsdelivr.net/npm/lodash@4.17.19/lodash.min.js', '_', 'head'],
  }
};
```

当你 `import 'loadsh'` 时，局部变量 `head` 和全局变量 `window._` 都会被暴露：

```js
import head from 'lodash';
console.log(head([1, 2, 3])); // logs 1 here
console.log(window._.head(['a', 'b'])); // logs a here
```

T> 当加载带有 HTML `<script>` 标签的代码时，webpack 的 runtime 将试图寻找一个已经存在的 `<script>` 标签，此标签需与 `src` 的属性相匹配，或者具有特定的 `data-webpack` 属性。对于 chunk 加载来说，`data-webpack` 属性的值为 `'[output.uniqueName]:chunk-[chunkId]'`，而 external 脚本的值为 `'[output.uniqueName]:[global]'`。

T> 像 `output.chunkLoadTimeout`，`output.crossOriginLoading` 以及 `output.scriptType` 等选项也会对这种方式加载的 external 脚本产生影响。

## `externalsPresets` {#externals-presets}

`object`

为特定的 target 启用 externals 的 preset。

W> 在早期的 webpack 版本中，通过使用 [`target`](/configuration/target/) 实现以下功能。



选项   | 描述                                      | 输入类型
----------- | ------------------------------------------------ | ----------
`electron`     | 将 main 和预加载上下文中常见的 electron 内置模块视为 external 模块（如 `electron`，`ipc` 或 `shell`），使用时通过 `require()` 加载。                     | boolean
`electronMain`     | 将 main 上下文中的 electron 内置模块视为 external 模块（如 `app`，`ipc-main` 或 `shell`），使用时通过 `require()` 加载。                     | boolean
`electronPreload`     | 将预加载上下文的 electron 内置模块视为 external 模块（如 `web-frame`，`ipc-renderer` 或 `shell`），使用时通过 `require()` 加载。                     | boolean
`electronRenderer`     | 将 renderer 上下文的 electron 内置模块视为 external 模块（如 `web-frame`、`ipc-renderer` 或 `shell`），使用时通过 `require()` 加载。                     | boolean
`node`     | 将 node.js 的内置模块视为 external 模块（如 `fs`，`path` 或 `vm`），使用时通过 `require()` 加载。| boolean
`nwjs`    | 将 `NW.js` 遗留的 `nw.gui` 模块视为 external 模块，使用时通过 `require()` 加载。| boolean
`web`     | 将 `http(s)://...` 以及 `std:...` 视为 external 模块，使用时通过 `import` 加载。__（注意，这将改变执行顺序，因为 external 代码会在该块中的其他代码执行前被执行）__。| boolean
`webAsync`     | 将 `http(s)://...` 以及 `std:...` 的引用视为 external 模块，使用时通过 `async import()` 加载。__（注意，此 external 类型为 `async` 模块，它对执行会产生各种副作用）__。| boolean


__示例__

使用 `node` 的 preset 不会构建内置模块，而会将其视为 external 模块，使用时通过 `require()` 加载。

__webpack.config.js__

```js
module.exports = {
  // ...
  externalsPresets:{
    node: true
  }
};
```
