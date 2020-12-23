---
title: Module Federation
sort: 8
contributors:
  - sokra
  - chenxsan
  - EugeneHlushko
  - jamesgeorge007
  - ScriptedAlchemy
related:
  - title: 'Webpack 5 Module Federation: A game-changer in JavaScript architecture'
    url: https://medium.com/swlh/webpack-5-module-federation-a-game-changer-to-javascript-architecture-bcdd30e02669
  - title: 'Explanations and Examples'
    url: https://github.com/module-federation/module-federation-examples
  - title: 'Module Federation YouTube Playlist'
    url: https://www.youtube.com/playlist?list=PLWSiF9YHHK-DqsFHGYbeAMwbd9xcZbEWJ
---

## 动机 {#motivation}

多个独立的构建可以组成一个应用程序，这些独立的构建之间不应该存在依赖关系，因此可以单独开发和部署它们。

这通常被称作微前端，但并不仅限于此。

## 底层概念 {#low-level-concepts}

我们区分本地模块和远程模块。本地模块即为普通模块，是当前构建的一部分。远程模块不属于当前构建，并在运行时从所谓的容器加载。

加载远程模块被认为是异步操作。当使用远程模块时，这些异步操作将被放置在远程模块和入口之间的下一个 chunk 的加载操作中。如果没有 chunk 加载操作，就不能使用远程模块。

chunk 的加载操作通常是通过调用 `import()` 实现的，但也支持像 `require.ensure` 或 `require([...])` 之类的旧语法。

容器是由容器入口创建的，该入口暴露了对特定模块的异步访问。暴露的访问分为两个步骤：

1. 加载模块（异步的）
2. 执行模块（同步的）

步骤 1 将在 chunk 加载期间完成。步骤 2 将在与其他（本地和远程）的模块交错执行期间完成。这样一来，执行顺序不受模块从本地转换为远程或从远程转为本地的影响。

容器可以嵌套使用，容器可以使用来自其他容器的模块。容器之间也可以循环依赖。

### 重载（Overriding） {#overriding}

容器能够将选定的本地模块标记为“可重载”。容器的使用者能够提供“重载”，即替换容器中的一个“可重载”的模块。当使用者提供重载模块时，容器的所有模块将使用替换模块而非本地模块。当使用者不提供替换模块时，容器的所有模块将使用本地模块。

容器管理可重载模块的方式为：当使用者已经重写它们后，就不需要下载了。这通常是通过将它们放在单独的 chunk 中来实现的。

另一方面，替换模块的提供者，将只提供异步加载函数。它允许容器仅在需要替换模块时才去加载。提供者管理替换模块的方式为：当容器不请求替换模块时，则无需下载。这通常是通过将它们放在单独的 chunk 中来实现的。

"name" 用于标识容器中可重载的模块。

重载（Overrides）的提供和容器暴露模块类似，它分为两个步骤:

1. 加载（异步）
2. 执行（同步）

W> 当嵌套使用时，向容器提供重载将自动覆盖嵌套容器中具有相同 "name" 的模块。

必须在容器模块加载之前提供重载。在初始 chunk 中使用的重载只能被不使用 Promise 的同步模块重载。一旦执行，就不可再次被重载。

## 高级概念 {#high-level-concepts}

每个构建都充当一个容器，也可将其他构建作为容器。通过这种方式，每个构建都能够通过从对应容器中加载模块来访问其他容器暴露出来的模块。

共享模块是指既可重写的又可作为向嵌套容器提供重写的模块。它们通常指向每个构建中的相同模块，例如相同的库。

packageName 选项允许通过设置包名来查找所需的版本。默认情况下，它会自动推断模块请求，当想禁用自动推断时，请将 requiredVersion 设置为 false 。

## 构建块(Building blocks) {#building-blocks}

### `OverridablesPlugin` (底层 API) {#overridablesplugin-low-level}

这个插件使得特定模块“可重载”。一个本地 API ( `__webpack_override__` ) 允许提供重载。

__webpack.config.js__

```javascript
const OverridablesPlugin = require('webpack/lib/container/OverridablesPlugin');
module.exports = {
  plugins: [
    new OverridablesPlugin([
      {
        // 通过 OverridablesPlugin 定义一个可重载的模块
        test1: './src/test1.js',
      },
    ]),
  ],
};
```

__src/index.js__

```javascript
__webpack_override__({
  // 这里我们重写 test1 模块
  test1: () => 'I will override test1 module under src',
});
```

### `ContainerPlugin` (底层 API) {#containerplugin-low-level}

该插件使用指定的公开模块来创建一个额外的容器入口。它还会在内部使用 OverridablesPlugin，并向容器的使用者暴露 `override` API。

### `ContainerReferencePlugin` (底层 API) {#containerreferenceplugin-low-level}

该插件将特定的引用添加到作为外部资源（externals）的容器中，并允许从这些容器中导入远程模块。它还会调用这些容器的 `override` API 来为它们提供重载。本地的重载（当构建也是一个容器时，通过 `__webpack_override__` 或 `override` API）和指定的重载被提供给所有引用的容器。

### `ModuleFederationPlugin` （高级 API）{#modulefederationplugin-high-level}

该插件组合了 `ContainerPlugin` 和 `ContainerReferencePlugin`。重载（overrides）和可重载（overridables）被合并到指定共享模块的单个列表中。

## 概念目标 {#concept-goals}

- 它既可以暴露，又可以使用 webpack 支持的任何模块类型
- 代码块加载应该并行加载所需的所有内容(web:到服务器的单次往返)
- 从使用者到容器的控制
   - 重写模块是一种单向操作
   - 同级容器不能重写彼此的模块。
- 概念适用于独立于环境
   - 可用于 web、Node.js 等
- 共享中的相对和绝对请求
   - 会一直提供，即使不使用
   - 会将相对路径解析到 `config.context` 
   - 默认不会使用 `requiredVersion` 
- 共享中的模块请求
   - 只在使用时提供
   - 会匹配构建中所有使用的相等模块请求
   - 将提供所有匹配模块
   - 将从图中这个位置的 package.json 提取 `requiredVersion` 
   - 当你有嵌套的 node_modules 时，可以提供和使用多个不同的版本
- 共享中尾部带有 `/` 的模块请求将匹配所有具有这个前缀的模块请求

## 用例 {#use-cases}

### 每个页面单独构建 {#separate-builds-per-page}

单页应用的每个页面都是在单独的构建中从容器暴露出来的。主体应用程序(application shell)也是独立构建，会将所有页面作为远程模块来引用。通过这种方式，可以单独部署每个页面。在更新路由或添加新路由时部署主体应用程序。主体应用程序将常用库定义为共享模块，以避免在页面构建中出现重复。

### 将组件库作为容器 {#components-library-as-container}

许多应用程序共享一个通用的组件库，可以将其构建成暴露所有组件的容器。每个应用程序使用来自组件库容器的组件。可以单独部署对组件库的更改，而不需要重新部署所有应用程序。应用程序自动使用组件库的最新版本。

## 动态远程容器 {#dynamic-remote-containers}

该容器接口支持 `get` 和 `init` 方法。
`init` 是一个兼容 `async` 的方法，调用时，只含有一个参数：共享作用域对象(shared scope object)。此对象在远程容器中用作共享作用域，并由 host 提供的模块填充。
可以利用它在运行时动态地将远程容器连接到 host 容器。

__init.js__

```javascript
(async () => {
  // 初始化共享作用域（shared scope）用提供的已知此构建和所有远程的模块填充它
  await __webpack_init_sharing__('default');
  const container = window.someContainer; // 或从其他地方获取容器
  // 初始化容器 它可能提供共享模块
  await container.init(__webpack_share_scopes__.default);
  const module = await container.get('./module');
})();
```

容器尝试提供共享模块，但是如果共享模块已经被使用，则会发出警告，并忽略所提供的共享模块。容器仍能将其作为降级模块。

你可以通过动态加载的方式，提供一个共享模块的不同版本，从而实现 A/B 测试。

T> 在尝试动态连接远程容器之前，确保已加载容器。

例子：

__init.js__

```javascript
function loadComponent(scope, module) {
  return async () => {
    // 初始化共享作用域（shared scope）用提供的已知此构建和所有远程的模块填充它
    await __webpack_init_sharing__('default');
    const container = window[scope]; // 或从其他地方获取容器
    // 初始化容器 它可能提供共享模块
    await container.init(__webpack_share_scopes__.default);
    const factory = await window[scope].get(module);
    const Module = factory();
    return Module;
  };
}

loadComponent('abtests', 'test123');
```

[查看完整实现](https://github.com/module-federation/module-federation-examples/tree/master/advanced-api/dynamic-remotes)

## 故障排除 {#troubleshooting}

__`Uncaught Error: Shared module is not available for eager consumption`__

应用程序正急切地执行一个作为全局主机运行的应用程序。有如下选项可供选择:

你可以在模块联邦的高级 API 中将依赖设置为即时依赖，此 API 不会将模块放在异步 chunk 中，而是同步地提供它们。这使得我们在初始块中可以直接使用这些共享模块。但是要注意，由于所有提供的和降级模块是要异步下载的，因此，建议只在应用程序的某个地方提供它，例如 shell。

我们强烈建议使用异步边界(asynchronous boundary)。它将把初始化代码分割成更大的块，以避免任何额外的开销，以提高总体性能。

例如，你的入口看起来是这样的：

__index.js__

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
ReactDOM.render(<App />, document.getElementById('root'));
```

让我们创建 bootstrap.js 文件，并将入口文件的内容放到里面，然后将 bootstrap 引入到入口文件中:

__index.js__

```diff
+ import('./bootstrap');
- import React from 'react';
- import ReactDOM from 'react-dom';
- import App from './App';
- ReactDOM.render(<App />, document.getElementById('root'));
```

__bootstrap.js__

```diff
+ import React from 'react';
+ import ReactDOM from 'react-dom';
+ import App from './App';
+ ReactDOM.render(<App />, document.getElementById('root'));
```

这种方法有效，但存在局限性或缺点。

通过 `ModuleFederationPlugin` 将依赖的 `eager` 属性设置为 `true` 

__webpack.config.js__

```javascript
// ...
new ModuleFederationPlugin({
  shared: {
    ...deps,
    react: {
      eager: true,
    }
  }
});
```

__`Uncaught Error: Module "./Button" does not exist in container.`__

错误提示中可能不会显示 `"./Button"`，但是信息看起来差不多。这个问题通常会出现在将 webpack beta.16 升级到 webpack beta.17 中。

在 ModuleFederationPlugin 里，更改 exposes:

```diff
new ModuleFederationPlugin({
  exposes: {
-   'Button': './src/Button'
+   './Button':'./src/Button'
  }
});
```

__`Uncaught TypeError: fn is not a function`__

此处错误可能是丢失了远程容器，请确保在使用前添加它。
如果已为试图使用远程服务器的容器加载了容器，但仍然看到此错误，则需将主机容器的远程容器文件也添加到 HTML 中。
