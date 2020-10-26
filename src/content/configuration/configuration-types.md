---
title: Configuration Types
sort: 3
contributors:
  - sokra
  - skipjack
  - kbariotis
  - simon04
  - fadysamirsadek
  - byzyk
  - EugeneHlushko
  - dhurlburtusa
  - anshumanv
---

除了导出单个配置外，还有一些能满足更多需求的使用方式。


## 导出函数 {#exporting-a-function}

你可能会遇到需要区分[开发](/guides/development)环境和[生产](/guides/production)环境的情况。你（至少）有两种选择：

其中一种选择是由 webpack 配置导出一个函数而非对象，这个函数包含两个参数：

- 参数一是环境（environment）。请查阅 [environment 选项文档](/api/cli/#environment-options)了解更多。
- 参数二是传递给 webpack 的配置项，其中包含 [`output-filename`](/api/cli/#output-options) 和 [`optimize-minimize`](/api/cli/#optimize-options) 等。

```diff
-module.exports = {
+module.exports = function(env, argv) {
+  return {
+    mode: env.production ? 'production' : 'development',
+    devtool: env.production ? 'source-map' : 'eval',
     plugins: [
       new TerserPlugin({
         terserOptions: {
+          compress: argv['optimize-minimize'] // only if -p or --optimize-minimize were passed
         }
       })
     ]
+  };
};
```


## 导出 Promise {#exporting-a-promise}

当需要异步加载配置变量时，webpack 将执行函数并导出一个配置文件，同时返回一个 Promise。

T> 支持使用 `Promise.all([/* Your promises */])` 导出多个 Promise。

```js
module.exports = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({
        entry: './app.js',
        /* ... */
      });
    }, 5000);
  });
};
```

W> 只有通过 webpack 命令行工具返回的 `Promise` 才生效。[`webpack()`](/api/node/#webpack) 只接受对象。


## 导出多种配置 {#exporting-multiple-configurations}

除了导出单个配置对象/函数，你可能也会需要导出多种配置（webpack 3.1.0 起支持）。当运行 webpack 时，所有配置项都会构建。比如，对于多 [targets](/configuration/output/#outputlibrarytarget)（如 AMD 和 CommonJS）[构建 library](/guides/author-libraries) 时会非常有用。

```js
module.exports = [{
  output: {
    filename: './dist-amd.js',
    libraryTarget: 'amd'
  },
  name: 'amd',
  entry: './app.js',
  mode: 'production',
}, {
  output: {
    filename: './dist-commonjs.js',
    libraryTarget: 'commonjs'
  },
  name: 'commonjs',
  entry: './app.js',
  mode: 'production',
}];
```

T> 如果你只传了一个 [`--config-name`](/api/cli/#configuration-options) 名字标识，webpack 将只会构建指定的配置项。
