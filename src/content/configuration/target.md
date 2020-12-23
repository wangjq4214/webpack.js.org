---
title: 构建目标(Targets)
sort: 13
contributors:
  - juangl
  - sokra
  - skipjack
  - SpaceK33z
  - pastelsky
  - tbroadley
  - byzyk
  - EugeneHlushko
  - smelukov
---

webpack 能够为多种环境或 _target_ 构建编译。想要理解什么是 `target` 的详细信息，
请阅读 [target 概念页面](/concepts/targets/)。

## `target` {#target}

`string` `[string]` `false`

告知 webpack 为目标(target)指定一个环境。默认值为 `"browserslist"`，如果没有找到 browserslist 的配置，则默认为 `"web"`


### `string` {#string}

通过 [`WebpackOptionsApply`](https://github.com/webpack/webpack/blob/master/lib/WebpackOptionsApply.js)，
可以支持以下字符串值：

选项                | 描述
--------------------- | -----------------------
`async-node`          | 编译为类 Node.js 环境可用（使用 fs 和 vm 异步加载分块）
`electron-main`       |  编译为 [Electron](https://electronjs.org/) 主进程。
`electron-renderer`   | 编译为 [Electron](https://electronjs.org/) 渲染进程，使用 `JsonpTemplatePlugin`, 
`FunctionModulePlugin` 来为浏览器环境提供目标，使用 `NodeTargetPlugin` 和 `ExternalsPlugin`
为 CommonJS 和 Electron 内置模块提供目标。
`electron-preload`    | 编译为 [Electron](https://electronjs.org/) 渲染进程，
使用 `NodeTemplatePlugin` 且 `asyncChunkLoading` 设置为 `true` ，`FunctionModulePlugin` 来为浏览器提供目标，使用 `NodeTargetPlugin` 和 `ExternalsPlugin` 为 CommonJS 和 Electron 内置模块提供目标。
`node`                | 编译为类 Node.js 环境可用（使用 Node.js `require` 加载 chunks）
`node-webkit`         | 编译为 Webkit 可用，并且使用 jsonp 去加载分块。支持 Node.js 内置模块和 [`nw.gui`](http://docs.nwjs.io/en/latest/) 
导入（实验性质）
`nwjs[[X].Y]`         | 等价于 `node-webkit`
`web`                 | 编译为类浏览器环境里可用 __（默认）__
`webworker`           | 编译成一个 WebWorker
`esX`                 | 编译为指定版本的 ECMAScript。例如，es5，es2020
`browserslist`        | 从 browserslist-config 中推断出平台和 ES 特性 __（如果 browserslist 可用，其值则为默认）__

例如，当 `target` 设置为 `"electron-main"`，webpack 引入多个 electron 特定的变量。

可指定 `node` 或者 `electron` 的版本。上表中使用 `[[X].Y]` 表示。

__webpack.config.js__

```js
module.exports = {
  // ...
  target: 'node12.18'
};
```

它有助于确定可能用于生成运行时代码的 ES 特性（所有的 chunk 和模块都被运行时代码所包裹）

#### `browserslist`

如果一个项目有 browserslist 配置，那么 webpack 将会使用它：

- 确定可用于运行时代码的 ES 特性。
- 推断环境（例如：`last 2 node versions` 等价于 `target: node`，并会进行一些 [`output.environment`](/configuration/output/#outputenvironment) 设置).

支持的 browserslist 值：

- `browserslist` - 使用自动解析的 browserslist 配置和环境（从最近的 `package.json` 或 `BROWSERSLIST` 环境变量中获取，具体请查阅 [browserslist 文档](https://github.com/browserslist/browserslist#queries)）
- `browserslist:modern` - 使用自动解析的 browserslist 配置中的 `modern` 环境
- `browserslist:last 2 versions` - 使用显式 browserslist 查询（配置将被忽略）
- `browserslist:/path/to/config` - 明确指定 browserslist 配置路径
- `browserslist:/path/to/config:modern` - 明确指定 browserslist 的配置路径和环境

### `[string]` {#string}

当传递多个目标时，将使用共同的特性子集：

__webpack.config.js__

```js
module.exports = {
  // ...
  target: ['web', 'es5']
};
```

webpack 将生成 web 平台的运行时代码，并且只使用 ES5 相关的特性。

目前并不是所有的 target 都可以进行混合。

__webpack.config.js__

```js
module.exports = {
  // ...
  target: ['web', 'node']
};
```

此时会导致错误。webpack 暂时不支持 universal 的 target。

### `false` {#false}

如果上述列表中的预设 target 都不符合你的需求，你可以将 `target` 设置为 `false`，这将告诉 webpack 不使用任何插件。

__webpack.config.js__

```js
module.exports = {
  // ...
  target: false
};
```

或者可以使用你想要指定的插件

__webpack.config.js__

```js
const webpack = require('webpack');

module.exports = {
  // ...
  target: false,
  plugins: [
    new webpack.JsonpTemplatePlugin(options.output),
    new webpack.LoaderTargetPlugin('web')
  ]
};
```

当没有提供 target 或 [environment](/configuration/output/#outputenvironment) 特性的信息时，将默认使用 ES2015。
