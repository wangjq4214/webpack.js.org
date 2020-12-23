---
title: 生产环境
sort: 17
contributors:
  - henriquea
  - rajagopal4890
  - makuzaverite
  - markerikson
  - simon04
  - kisnows
  - chrisVillanueva
  - swapnilmishra
  - bring2dip
  - redian
  - skipjack
  - xgqfrms
  - kelset
  - xgirma
  - mehrdaad
  - SevenOutman
  - AnayaDesign
  - wizardofhogwarts
  - aholzner
  - EugeneHlushko
  - snitin315
---

在本指南中，我们将深入一些最佳实践和工具，将站点或应用程序构建到生产环境中。

T> 以下示例来源于 [tree shaking](/guides/tree-shaking) 和 [开发环境](/guides/development)。在继续之前，请确保你已经熟悉这些指南中所介绍的概念/配置。

## 配置 {#setup}

_development(开发环境)_ 和 _production(生产环境)_ 这两个环境下的构建目标存在着巨大差异。在_开发环境_中，我们需要：强大的 source map 和一个有着 live reloading(实时重新加载) 或 hot module replacement(热模块替换) 能力的 localhost server。而_生产环境_目标则转移至其他方面，关注点在于压缩 bundle、更轻量的 source map、资源优化等，通过这些优化方式改善加载时间。由于要遵循逻辑分离，我们通常建议为每个环境编写__彼此独立的 webpack 配置__。

虽然，以上我们将_生产环境_和_开发环境_做了略微区分，但是，请注意，我们还是会遵循不重复原则(Don't repeat yourself - DRY)，保留一个 "common(通用)" 配置。为了将这些配置合并在一起，我们将使用一个名为 [`webpack-merge`](https://github.com/survivejs/webpack-merge) 的工具。此工具会引用 "common" 配置，因此我们不必再在环境特定(environment-specific)的配置中编写重复代码。

我们先从安装 `webpack-merge` 开始，并将之前指南中已经成型的那些代码进行分离：

``` bash
npm install --save-dev webpack-merge
```

__project__

``` diff
  webpack-demo
  |- package.json
- |- webpack.config.js
+ |- webpack.common.js
+ |- webpack.dev.js
+ |- webpack.prod.js
  |- /dist
  |- /src
    |- index.js
    |- math.js
  |- /node_modules
```

__webpack.common.js__

``` diff
+ const path = require('path');
+ const { CleanWebpackPlugin } = require('clean-webpack-plugin');
+ const HtmlWebpackPlugin = require('html-webpack-plugin');
+
+ module.exports = {
+   entry: {
+     app: './src/index.js',
+   },
+   plugins: [
+     // 对于 CleanWebpackPlugin 的 v2 versions 以下版本，使用 new CleanWebpackPlugin(['dist/*'])
+     new CleanWebpackPlugin(),
+     new HtmlWebpackPlugin({
+       title: 'Production',
+     }),
+   ],
+   output: {
+     filename: '[name].bundle.js',
+     path: path.resolve(__dirname, 'dist'),
+   },
+ };
```

__webpack.dev.js__

``` diff
+ const { merge } = require('webpack-merge');
+ const common = require('./webpack.common.js');
+
+ module.exports = merge(common, {
+   mode: 'development',
+   devtool: 'inline-source-map',
+   devServer: {
+     contentBase: './dist',
+   },
+ });
```

__webpack.prod.js__

``` diff
+ const { merge } = require('webpack-merge');
+ const common = require('./webpack.common.js');
+
+ module.exports = merge(common, {
+   mode: 'production',
+ });
```

现在，在 `webpack.common.js` 中，我们设置了 `entry` 和 `output` 配置，并且在其中引入这两个环境公用的全部插件。在 `webpack.dev.js` 中，我们将 `mode` 设置为 `development`，并且为此环境添加了推荐的 `devtool`（强大的 source map）和简单的 `devServer` 配置。最后，在 `webpack.prod.js` 中，我们将 `mode` 设置为 `production`，其中会引入之前在 [tree shaking](/guides/tree-shaking) 指南中介绍过的 `TerserPlugin`。

注意，在环境特定的配置中使用 `merge()` 功能，可以很方便地引用 `webpack.dev.js` 和 `webpack.prod.js` 中公用的 common 配置。`webpack-merge` 工具提供了各种 merge(合并) 高级功能，但是在我们的用例中，无需用到这些功能。

## NPM Scripts {#npm-scripts}

现在，我们把 `scripts` 重新指向到新配置。让 `npm start` script 中 `webpack-dev-server`, 使用 `webpack.dev.js`, 而让 `npm run build` script 使用 `webpack.prod.js`:

__package.json__

``` diff
  {
    "name": "development",
    "version": "1.0.0",
    "description": "",
    "main": "src/index.js",
    "scripts": {
-     "start": "webpack serve --open",
+     "start": "webpack serve --open --config webpack.dev.js",
-     "build": "webpack"
+     "build": "webpack --config webpack.prod.js"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "clean-webpack-plugin": "^0.1.17",
      "css-loader": "^0.28.4",
      "csv-loader": "^2.1.1",
      "express": "^4.15.3",
      "file-loader": "^0.11.2",
      "html-webpack-plugin": "^2.29.0",
      "style-loader": "^0.18.2",
      "webpack": "^4.30.0",
      "webpack-dev-middleware": "^1.12.0",
      "webpack-dev-server": "^2.9.1",
      "webpack-merge": "^4.1.0",
      "xml-loader": "^1.2.1"
    }
  }
```

随便运行下这些脚本，然后查看输出结果的变化，然后我们会继续添加一些_生产环境_配置。

## 指定 mode {#specify-the-mode}

许多 library 通过与 `process.env.NODE_ENV` 环境变量关联，以决定 library 中应该引用哪些内容。例如，当`process.env.NODE_ENV` 没有被设置为 `'production'` 时，某些 library 为了使调试变得容易，可能会添加额外的 log(日志记录) 和 test(测试) 功能。并且，在使用 `process.env.NODE_ENV === 'production'` 时，一些 library 可能针对具体用户的环境，删除或添加一些重要代码，以进行代码执行方面的优化。从 webpack v4 开始, 指定 [`mode`](/configuration/mode/) 会自动地配置 [`DefinePlugin`](/plugins/define-plugin)：

__webpack.prod.js__

``` diff
  const { merge } = require('webpack-merge');
  const common = require('./webpack.common.js');

  module.exports = merge(common, {
    mode: 'production',
  });
```

T> 技术上讲，`NODE_ENV` 是一个由 Node.js 暴露给执行脚本的系统环境变量。通常用于决定在开发环境与生产环境(dev-vs-prod)下，server tools(服务期工具)、build scripts(构建脚本) 和 client-side libraries(客户端库) 的行为。然而，与预期相反，在构建脚本 `webpack.config.js` 中`process.env.NODE_ENV` 并没有被设置为 `"production"`，请查看 [#2537](https://github.com/webpack/webpack/issues/2537)。因此，在 webpack 配置文件中，`process.env.NODE_ENV === 'production' ? '[name].[contenthash].bundle.js' : '[name].bundle.js'` 这样的条件语句，无法按照预期运行。

如果你正在使用像 [`react`](https://react.docchina.org/) 这样的 library，那么在添加此 DefinePlugin 插件后，你应该看到 bundle 大小显著下降。还要注意，任何位于 `/src` 的本地代码都可以关联到 process.env.NODE_ENV 环境变量，所以以下检查也是有效的：

__src/index.js__

```diff
  import { cube } from './math.js';
+
+ if (process.env.NODE_ENV !== 'production') {
+   console.log('Looks like we are in development mode!');
+ }

  function component() {
    const element = document.createElement('pre');

    element.innerHTML = [
      'Hello webpack!',
      '5 cubed is equal to ' + cube(5)
    ].join('\n\n');

    return element;
  }

  document.body.appendChild(component());
```

## 压缩(Minification) {#minification}

webpack v4+ will minify your code by default in [`production mode`](/configuration/mode/#mode-production).

注意，虽然生产环境下默认使用 [`TerserPlugin`](/plugins/terser-webpack-plugin) ，并且也是代码压缩方面比较好的选择，但是还有一些其他可选择项。以下有几个同样很受欢迎的插件：

- [`ClosureWebpackPlugin`](https://github.com/webpack-contrib/closure-webpack-plugin)

如果决定尝试一些其他压缩插件，只要确保新插件也会按照 [tree shake](/guides/tree-shaking) 指南中所陈述的具有删除未引用代码(dead code)的能力，并将它作为 [`optimization.minimizer`](/configuration/optimization/#optimization-minimizer)。

## 源码映射(Source Mapping) {#source-mapping}

我们鼓励你在生产环境中启用 source map，因为它们对 debug(调试源码) 和运行 benchmark tests(基准测试) 很有帮助。虽然有着如此强大的功能，然而还是应该针对生产环境用途，选择一个可以快速构建的推荐配置（更多选项请查看 [`devtool`](/configuration/devtool)）。对于本指南，我们将在_生产环境_中使用 `source-map` 选项，而不是我们在_开发环境_中用到的 `inline-source-map`：

__webpack.prod.js__

``` diff
  const { merge } = require('webpack-merge');
  const common = require('./webpack.common.js');

  module.exports = merge(common, {
    mode: 'production',
+   devtool: 'source-map',
  });
```

T> 避免在生产中使用 `inline-***` 和 `eval-***`，因为它们会增加 bundle 体积大小，并降低整体性能。

## 压缩 CSS {#minimize-css}

将生产环境下的 CSS 进行压缩会非常重要，请查看 [在生产环境下压缩](/plugins/mini-css-extract-plugin/#minimizing-for-production) 章节。

## CLI 替代选项 {#cli-alternatives}

以上所述也可以通过命令行实现。例如，`--optimize-minimize` 标记将在幕后引用 `TerserPlugin`。和以上描述的 `DefinePlugin` 实例相同，`--define process.env.NODE_ENV="'production'"` 也会做同样的事情。而且，`webpack -p` 将自动地配置上述这两个标记，从而调用需要引入的插件。

虽然这种简写方式很好，但通常我们建议只使用配置方式，因为在这两种方式中，配置方式能够更准确地理解现在正在做的事情。配置方式还为可以让你更加细微地控制这两个插件中的其他选项。
