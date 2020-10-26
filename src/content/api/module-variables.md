---
title: Module Variables
group: Modules
sort: 8
contributors:
  - skipjack
  - sokra
  - ahmehri
  - tbroadley
  - byzyk
  - EugeneHlushko
  - wizardofhogwarts
  - anikethsaha
  - chenxsan

related:
  - title: CommonJS
    url: https://en.wikipedia.org/wiki/CommonJS
  - title: Asynchronous Module Definition
    url: https://en.wikipedia.org/wiki/Asynchronous_module_definition
---

本章节涵盖了使用 webpack 编译的代码中所有的 __变量__。模块将通过 `module` 和其他变量，来访问编译过程中的某些数据。


### `module.loaded` (NodeJS) {#moduleloaded-nodejs}

`false` 表示该模块正在执行， `true` 表示同步执行已经完成。


### `module.hot` (webpack-specific) {#modulehot-webpack-specific}

表示 [模块热替换(Hot Module Replacement)](/concepts/hot-module-replacement) 是否启用，并给进程提供一个接口。详细说明请查看 [模块热替换 API 页面](/api/hot-module-replacement)。


### `module.id` (CommonJS) {#moduleid-commonjs}

当前模块的 ID。

``` javascript
module.id === require.resolve('./file.js');
```


### `module.exports` (CommonJS) {#moduleexports-commonjs}

调用者通过 `require` 对模块进行调用时返回的值（默认为一个新对象）。

``` javascript
module.exports = function doSomething() {
  // Do something...
};
```

W> 无法在异步函数中访问该变量。


### `exports` (CommonJS) {#exports-commonjs}

该变量默认值为 `module.exports`（即一个对象）。 如果 `module.exports` 被重写的话， `exports` 不再会被导出。

``` javascript
exports.someValue = 42;
exports.anObject = {
  x: 123
};
exports.aFunction = function doSomething() {
  // Do something
};
```


### `global` (NodeJS) {#global-nodejs}

见 [node.js global](https://nodejs.org/api/globals.html#globals_global).

出于兼容性考虑，webpack 默认填充了 `global` 变量。


### `__dirname` (NodeJS) {#__dirname-nodejs}

取决于 `node.__dirname` 配置选项：

- `false`: 未定义
- `mock`: 等同于 `'/'`
- `true`: [node.js __dirname](https://nodejs.org/api/globals.html#globals_dirname)

如果在一个被 Parser 解析的表达式内部使用，则配置选项会被当作 `true` 处理。

### `import.meta.url` {#import-meta-url}

返回模块以 `fike:` 开头绝对路径的 URL。

__src/index.js__

```javascript
console.log(import.meta.url); // 输出结果类似于 `file:///path/to/your/project/src/index.js`
```

### `import.meta.webpack` {#import-meta-webpack}

返回 webpack 的版本

__src/index.js__

```javascript
console.log(import.meta.webpack); // output `5` for webpack 5
```

### `__filename` (NodeJS) {#__filename-nodejs}

取决于 `node.__filename` 配置选项：

- `false`: 未定义
- `mock`: 等同于 `'/index.js'`
- `true`: [node.js __filename](https://nodejs.org/api/globals.html#globals_filename)

如果在一个被 Parser 解析的表达式内部使用，则配置选项会被当作 `true` 处理。


### `__resourceQuery` (webpack 特有变量) {#__resourcequery-webpack-specific}

当前模块的资源查询(resource query) 。如果进行了如下的 `reqiure` 调用，那么查询字符串(query string)在`file.js` 中可访问。

``` javascript
require('file.js?test');
```

__file.js__

``` javascript
__resourceQuery === '?test';
```


### `__webpack_public_path__` (webpack 特有变量) {#__webpack_public_path__-webpack-specific}

等同于 `output.publicPath` 配置选项。


### `__webpack_require__` (webpack 特有变量) {#__webpack_require__-webpack-specific}

原始 require 函数。这个表达式不会被解析器解析为依赖。


### `__webpack_chunk_load__` (webpack 特有变量) {#__webpack_chunk_load__-webpack-specific}

内部 chunk 载入函数，有两个输入参数：

- `chunkId` 需要载入的 chunk id。
- `callback(require)` chunk 载入后调用的回调函数。


### `__webpack_modules__` (webpack 特有变量) {#__webpack_modules__-webpack-specific}

访问所有模块的内部对象。


### `__webpack_hash__` (webpack 特有变量) {#__webpack_hash__-webpack-specific}

这个变量只有在启用 `HotModuleReplacementPlugin` 或者 `ExtendedAPIPlugin` 时才生效。这个变量提供对编译过程中(compilation)的 hash 信息的获取。


### `__non_webpack_require__` (webpack 特有变量) {#__non_webpack_require__-webpack-specific}

生成一个不会被 webpack 解析的 `require` 函数。配合全局可以获取到的 require 函数，可以完成一些酷炫操作。


### `__webpack_exports_info__` (webpack 特有变量) {#__webpack_exports_info__-webpack-specific}

在模块中, `__webpack_exports_info__` 可以被获取到，以便导出模块用以自我检查：

- `__webpack_exports_info__` 总是 `true`

- 当导出模块未被使用时 `__webpack_exports_info__.<exportName>.used` 为 `false`, 否则是 `true`

- `__webpack_exports_info__.<exportName>.useInfo` 是

    - `false` 当导出模块未被使用
    - `true` 当导出模块被使用
    - `null` 当导出模块的使用情况取决于运行时的条件
    - `undefined` 当没有可用信息时

- `__webpack_exports_info__.<exportName>.provideInfo` 是

    - `false` 当导出模块没有被提供
    - `true` 当导出模块被提供
    - `null` 当导出模块的提供情况取决于运行时的条件
    - `undefined` 当没有可用信息时

- 可以从嵌套的 exports 中得到相关信息: 例如 `__webpack_exports_info__.<exportName>.<exportName>.<exportName>.used`

### `DEBUG`  (webpack 特有变量) {#debug--webpack-specific}

等同于 `debug` 配置选项。
