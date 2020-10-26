---
title: 公共路径
sort: 23
contributors:
  - rafaelrinaldi
  - chrisVillanueva
  - gonzoyumo
---

`publicPath` 配置选项在各种场景中都非常有用。你可以通过它来指定应用程序中所有资源的基础路径。


## 示例 {#use-cases}

下面提供一些用于实际应用程序的示例，通过这些示例，此功能显得极其简单。实质上，发送到 `output.path` 目录的每个文件，都将从 `output.publicPath` 位置引用。这也包括（通过 [代码分离](/guides/code-splitting/) 创建的）子 chunk 和作为依赖图一部分的所有其他资源（例如 image, font 等）。

### 基于环境设置 {#environment-based}

在开发环境中，我们通常有一个 `assets/` 文件夹，它与索引页面位于同一级别。这没太大问题，但是，如果我们将所有静态资源托管至 CDN，然后想在生产环境中使用呢？

想要解决这个问题，可以直接使用一个有着悠久历史的 environment variable(环境变量)。假设我们有一个变量 `ASSET_PATH`：

``` js
import webpack from 'webpack';

// 尝试使用环境变量，否则使用根路径
const ASSET_PATH = process.env.ASSET_PATH || '/';

export default {
  output: {
    publicPath: ASSET_PATH,
  },

  plugins: [
    // 这可以帮助我们在代码中安全地使用环境变量
    new webpack.DefinePlugin({
      'process.env.ASSET_PATH': JSON.stringify(ASSET_PATH),
    }),
  ],
};
```

### 在运行时设置 {#on-the-fly}

另一个可能出现的情况是，需要在运行时设置 `publicPath`。webpack 暴露了一个名为 `__webpack_public_path__` 的全局变量。所以在应用程序的 entry point 中，可以直接如下设置：

```js
__webpack_public_path__ = process.env.ASSET_PATH;
```

这些内容就是你所需要的。由于我们已经在配置中使用了 `DefinePlugin`，
`process.env.ASSET_PATH` 将始终都被定义，
因此我们可以安全地使用。

W> 注意，如果在 entry 文件中使用 ES2015 module import，则会在 import 之后进行 `__webpack_public_path__` 赋值。在这种情况下，你必须将 public path 赋值移至一个专用模块中，然后将它的 import 语句放置到 entry.js 最上面：

```js
// entry.js
import './public-path';
import './app';
```
