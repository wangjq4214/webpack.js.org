---
title: less-loader
source: https://raw.githubusercontent.com/webpack-contrib/less-loader/master/README.md
edit: https://github.com/webpack-contrib/less-loader/edit/master/README.md
repo: https://github.com/webpack-contrib/less-loader
---


[![npm][npm]][npm-url]
[![node][node]][node-url]
[![deps][deps]][deps-url]
[![tests][tests]][tests-url]
[![cover][cover]][cover-url]
[![chat][chat]][chat-url]
[![size][size]][size-url]



webpack 将 Less 编译为 CSS 的 loader。

## 快速开始 {#getting-started}

首先，你需要先安装 `less` 和 `less-loader`：

```console
$ npm install less less-loader --save-dev
```

然后将该 loader 添加到 `webpack` 的配置中去，例如：

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        loader: "less-loader", // 将 Less 文件编译为 CSS 文件
      },
    ],
  },
};
```

接着使用你习惯的方式运行 `webpack`。

## 可选项 {#options}

|                  名称                   |         类型         |         默认值          | 描述                                      |
| :-------------------------------------: | :------------------: | :----------------------: | :----------------------------------------------- |
|    **[`lessOptions`](#lessoptions)**    | `{Object\|Function}` | `{ relativeUrls: true }` | Less 的可选项。                                |
|     **[`additionalData`](#additionaldata)**     | `{String\|Function}` |       `undefined`        | 在入口文件起始或末尾添加 Less 代码。  |
|      **[`sourceMap`](#sourcemap)**      |     `{Boolean}`      |    `compiler.devtool`    | 是否生成 source map。       |
| **[`implementation`](#implementation)** |      `{Object}`      |          `less`          | 配置 Less 使用的实现库                |

### `lessOptions` {#lessoptions}

类型: `Object|Function`
默认值: `{ relativeUrls: true }`

通过 `lessOptions` 属性，你可以设置 [loader options](/configuration/module/#ruleoptions--rulequery) 中的任意特定的选项值给 `less-loader`。所有可用的选项值请参看 [Less 命令行可选参数文档](http://lesscss.org/usage/#command-line-usage-options)。由于我们是通过编程的方式将这些选项传递给 Less，因此您需要将破折号（dash-case）转换为驼峰值（camelCase）后传递它们。

#### `Object` {#object}

使用对象（Object）的形式传递 options 给 Less。

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          {
            loader: "style-loader",
          },
          {
            loader: "css-loader",
          },
          {
            loader: "less-loader",
            options: {
              lessOptions: {
                strictMath: true,
              },
            },
          },
        ],
      },
    ],
  },
};
```

#### `Function` {#function}

允许根据 loader 的 context 来设置 options，再传递给 Less。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          "style-loader",
          "css-loader",
          {
            loader: "less-loader",
            options: {
              lessOptions: (loaderContext) => {
                // 更多可用的属性见 https://webpack.js.org/api/loaders/
                const { resourcePath, rootContext } = loaderContext;
                const relativePath = path.relative(rootContext, resourcePath);

                if (relativePath === "styles/foo.less") {
                  return {
                    paths: ["absolute/path/c", "absolute/path/d"],
                  };
                }

                return {
                  paths: ["absolute/path/a", "absolute/path/b"],
                };
              },
            },
          },
        ],
      },
    ],
  },
};
```

### `additionalData` {#additionaldata}

类型: `String|Function`
默认值: `undefined`

在实际入口文件的起始位置添加 `Less` 代码。
这种情况下，`less-loader` 只会**追加**并不会覆盖文件内容。

当你的 Less 变量依赖环境变量时这个属性将非常有用：

> ℹ 由于你注入了代码，因此它将破坏入口文件的源映射关系。通常有比这更简单的解决方案，例如多个 Less 入口文件。

#### `String` {#string}

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          "style-loader",
          "css-loader",
          {
            loader: "less-loader",
            options: {
              additionalData: `@env: ${process.env.NODE_ENV};`,
            },
          },
        ],
      },
    ],
  },
};
```

#### `Function` {#function}

##### Sync

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          "style-loader",
          "css-loader",
          {
            loader: "less-loader",
            options: {
              additionalData: (content, loaderContext) => {
                // 更多可用的属性见 https://webpack.js.org/api/loaders/
                const { resourcePath, rootContext } = loaderContext;
                const relativePath = path.relative(rootContext, resourcePath);

                if (relativePath === "styles/foo.less") {
                  return "@value: 100px;" + content;
                }

                return "@value: 200px;" + content;
              },
            },
          },
        ],
      },
    ],
  },
};
```

##### Async

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          "style-loader",
          "css-loader",
          {
            loader: "less-loader",
            options: {
              additionalData: async (content, loaderContext) => {
                // More information about available properties https://webpack.js.org/api/loaders/
                const { resourcePath, rootContext } = loaderContext;
                const relativePath = path.relative(rootContext, resourcePath);

                if (relativePath === "styles/foo.less") {
                  return "@value: 100px;" + content;
                }

                return "@value: 200px;" + content;
              },
            },
          },
        ],
      },
    ],
  },
};
```

### `sourceMap` {#sourcemap}

类型: `Boolean`
默认值: 取决于 `compiler.devtool` 的值

默认生成的 source map 取决于 `compiler.devtool` 的值。除了值等于 `eval` 和 `false` 外，其他值都能生成 source map

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/i,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              sourceMap: true,
            },
          },
          {
            loader: "less-loader",
            options: {
              sourceMap: true,
            },
          },
        ],
      },
    ],
  },
};
```

### `webpackImporter` {#webpackimporter}

类型：`Boolean`
默认值：`true`

启用/禁用 webpack 默认的 importer。

在某些情况下，这样做可以提高性能，但是请慎用，因为可能会使得 aliases 和以 `~` 开头的 `@import` 规则失效。

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/i,
        use: [
          "style-loader",
          "css-loader",
          {
            loader: "less-loader",
            options: {
              webpackImporter: false,
            },
          },
        ],
      },
    ],
  },
};
```

## 示例 {#examples}

### 常规用法 {#normal-usage}

将 `less-loader`、[`css-loader`](/loaders/css-loader/) 和 [`style-loader`](/loaders/style-loader/) 串联起来使用可立即将所有样式应用于 DOM。

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          {
            loader: "style-loader", // 从 JS 中创建样式节点
          },
          {
            loader: "css-loader", // 转化 CSS 为 CommonJS
          },
          {
            loader: "less-loader", // 编译 Less 为 CSS
          },
        ],
      },
    ],
  },
};
```

不幸的是，Less 并没有将所有选项 1 对 1 映射为 camelCase（驼峰值）。如有疑问，请[检查执行文件]（https://github.com/less/less.js/blob/3.x/bin/lessc）并搜索破折号选项。

### Source maps {#source-maps}

为了生成 CSS 的 source map, 你需要在 loader 的可选项中设置 `sourceMap` 属性。如果没设置的话 loader 将会继承你 webpack 中为生成 source map 设置的属性值 `devtool`。

**webpack.config.js**

```javascript
module.exports = {
  devtool: "source-map", // 任何类似于 "source-map" 的  devtool 值都可以
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          "style-loader",
          {
            loader: "css-loader",
            options: {
              sourceMap: true,
            },
          },
          {
            loader: "less-loader",
            options: {
              sourceMap: true,
            },
          },
        ],
      },
    ],
  },
};
```

如果你想在 Chrome 中修改原始的 Less 文件，可以参考[这篇不错的博客](https://medium.com/@toolmantim/getting-started-with-css-sourcemaps-and-in-browser-sass-editing-b4daab987fb0)。这篇博客虽然写的 Sass，但也适合于 Less。

### 生产环境 {#in-production}

在生产环境中推荐使用 [MiniCssExtractPlugin](/plugins/mini-css-extract-plugin/) 来提取样式表到专门的文件中，这样你的样式就不需要依赖 JavaScript。

### 导入 {#imports}

从 `less-loader` v4 版本起，你有两种解析器可用，Less 内置解析器和 webpack 解析器。默认情况使用 webpack 解析器。

#### webpack 解析器 {#webpack-resolver}

webpack 提供了一种 [解析文件的高级机制](/configuration/resolve/)。`less-loader` 作为 Less 的插件，该插件将所有的查询结果传递给 webpack 解析器，因此你可以从 `node_modules` 中导入 Less 模块，只需要在它们前面加上 `~` 符号告诉 webpack 从 [`modules`](/configuration/resolve/#resolvemodules) 中去查找。

```css
@import "~bootstrap/less/bootstrap";
```

在其前面加上 `〜` 很关键，因为 `〜/` 会解析到根目录。webpack 需要区分 `bootstrap` 和 `〜bootstrap`，因为 CSS 和 Less 文件没有用于导入相对路径文件的特殊语法。写 `@import“ file”` 等同于 `@import“ ./file”;`

#### Less 解析器 {#less-resolver}

如果指定 `paths` 选项，将从指定的 `paths` 中搜索模块，这是 Less 的默认行为。`paths` 应该是具有绝对路径的数组。

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/,
        use: [
          {
            loader: "style-loader",
          },
          {
            loader: "css-loader",
          },
          {
            loader: "less-loader",
            options: {
              lessOptions: {
                paths: [path.resolve(__dirname, "node_modules")],
              },
            },
          },
        ],
      },
    ],
  },
};
```

### 插件 {#plugins}

想要使用 [插件](http://lesscss.org/usage/#plugins)，只需要简单设置下 `plugins` 选项即可，
具体配置如下：

```js
// webpack.config.js
const CleanCSSPlugin = require('less-plugin-clean-css');

module.exports = {
  ...
    {
      loader: 'less-loader',
      options: {
        lessOptions: {
          plugins: [
            new CleanCSSPlugin({ advanced: true }),
          ],
        },
      },
    },
  ...
};
```

> ℹ️ 使用 `less.webpackLoaderContext` 属性可以访问自定义插件中的 [loader context](/api/loaders/#the-loader-context)。

```js
module.exports = {
  install: function (less, pluginManager, functions) {
    functions.add("pi", function () {
      // Loader context is available in `less.webpackLoaderContext`

      return Math.PI;
    });
  },
};
```

### 提取样式 {#extracting-style-sheets}

通过 webpack 打包 CSS 有很多好处，比如给引用图片和字体文件路径添加 hash, 在开发环境可以 [模块热替换](/concepts/hot-module-replacement/)。另一方面，在生产环境，根据 JS 来控制应用样式表不是一个好的方式，可能会导致延迟渲染，甚至可能会出现[闪烁现象](https://en.wikipedia.org/wiki/Flash_of_unstyled_content)。因此，在你最终的生产环境中将它们拆分成单独的文件来存放通常是比较好的选择。

有两种从 bundle 中提取样式表的方式：

- [`extract-loader`](https://github.com/peerigon/extract-loader) （简单，但得专门指定 `css-loader` 的 output）
- [MiniCssExtractPlugin](/plugins/mini-css-extract-plugin/) （较复杂，但适用于所有的场景）

### CSS modules 陷阱 {#css-modules-gotcha}

Less 和 [CSS modules](https://github.com/css-modules/css-modules) 有一个已知的问题，关于 `url（...）` 语句中的相对文件路径，[看下这个问题的解释](https://github.com/webpack-contrib/less-loader/issues/109#issuecomment-253797335)。

## 贡献 {#contributing}

如果你还没有看过我们的贡献者指南请先花点时间看一下。

[CONTRIBUTING](https://github.com/webpack-contrib/less-loader/blob/master/.github/CONTRIBUTING.md)

## 许可 {#license}

[MIT](https://github.com/webpack-contrib/less-loader/blob/master/LICENSE)

[npm]: https://img.shields.io/npm/v/less-loader.svg
[npm-url]: https://npmjs.com/package/less-loader
[node]: https://img.shields.io/node/v/less-loader.svg
[node-url]: https://nodejs.org/
[deps]: https://david-dm.org/webpack-contrib/less-loader.svg
[deps-url]: https://david-dm.org/webpack-contrib/less-loader
[tests]: https://github.com/webpack-contrib/less-loader/workflows/less-loader/badge.svg
[tests-url]: https://github.com/webpack-contrib/less-loader/actions
[cover]: https://codecov.io/gh/webpack-contrib/less-loader/branch/master/graph/badge.svg
[cover-url]: https://codecov.io/gh/webpack-contrib/less-loader
[chat]: https://img.shields.io/badge/gitter-webpack%2Fwebpack-brightgreen.svg
[chat-url]: https://gitter.im/webpack/webpack
[size]: https://packagephobia.now.sh/badge?p=less-loader
[size-url]: https://packagephobia.now.sh/result?p=less-loader
