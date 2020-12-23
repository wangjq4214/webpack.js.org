---
title: Node
sort: 17
contributors:
  - sokra
  - skipjack
  - oneforwonder
  - Rob--W
  - byzyk
  - EugeneHlushko
  - anikethsaha
  - chenxsan
---

这些选项可以配置是否 polyfill 或 mock 某些 [Node.js 全局变量](https://nodejs.org/docs/latest/api/globals.html)。

此功能由 webpack 内部的 [`NodeStuffPlugin`](https://github.com/webpack/webpack/blob/master/lib/NodeStuffPlugin.js) 插件提供。

W> 从 webpack 5 开始，你只能在 `node` 选项下配置 `global`、`__filename` 或 `__dirname`。如果需要在 webpack 5 下的 Node.js 中填充 `fs`，请查阅 [resolve.fallback](/configuration/resolve/#resolvefallback) 获取相关帮助。

## `node` {#node}

`boolean: false` `object`

__webpack.config.js__

```javascript
module.exports = {
  //...
  node: {
    global: false,
    __filename: false,
    __dirname: false,
  }
};
```

从 webpack 3.0.0 开始，`node` 选项可能被设置为 `false`，以完全关闭 `NodeStuffPlugin` 插件。

## `node.global` {#nodeglobal}

`boolean`

T> 如果你正在使用一个需要全局变量的模块，请使用 `ProvidePlugin` 替代 `global`。

关于此对象的准确行为，请查看[Node.js 文档](https://nodejs.org/api/globals.html#globals_global)。

选项：

- `true`: 提供 polyfill.
- `false`: 不提供任何 polyfill。代码可能会出现 `ReferenceError` 的崩溃。

## `node.__filename` {#node__filename}

`boolean` `string: 'mock' | 'eval-only'`

选项：

- `true`: __输入__ 文件的文件名，是相对于 [`context` 选项](/configuration/entry-context/#context)。
- `false`: webpack 不会更改 `__filename` 的代码。在 Node.js 环境中运行时，__输出__ 文件的文件名。
- `'mock'`: value 填充为 `'index.js'`.


## `node.__dirname` {#node__dirname}

`boolean` `string: 'mock' | 'eval-only'`

选项：

- `true`: __输入__ 文件的目录名，是相对于 [`context` 选项](/configuration/entry-context/#context)。
- `false`: webpack 不会更改 `__dirname` 的代码，这意味着你有常规 Node.js 中的 `__dirname` 的行为。在 Node.js 环境中运行时，__输出__ 文件的目录名。
- `'mock'`: value 填充为 `'/'`。
- `'eval-only'`
