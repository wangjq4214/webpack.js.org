---
title: EvalSourceMapDevToolPlugin
contributors:
  - johnnyreilly
  - simon04
  - kinseyost
  - byzyk
  - madhavarshney
  - koke
  - jamesgeorge007
  - anshumanv
  - EugeneHlushko
related:
  - title: Building Eval Source Maps
    url: https://survivejs.com/webpack/building/source-maps/#sourcemapdevtoolplugin-and-evalsourcemapdevtoolplugin
---

本插件可以为 source map 的生成提供更好更细粒度的控制。[`devtool`](/configuration/devtool/) 中的某些配置会自动使用它。

``` js
new webpack.EvalSourceMapDevToolPlugin(options);
```


## 选项 {#options}

支持以下选项：

- `test` (`string|RegExp|array`): 默认值为 `.js` 和 `.css`，为与给定的模块扩展名相匹配的模块，添加 source map。
- `include` (`string|RegExp|array`): 为与给定路径相匹配的模块，添加 source map。
- `exclude` (`string|RegExp|array`): 排除一些与给定值相匹配的模块，不会为它们生成映射关系。
- `append` (`string`): 将给定的值添加到源代码中，通常是 `#sourceMappingURL` 注释，`[url]` 在 source map 文件中将会被替换成 url，值为 `false` 表示不添加。
- `moduleFilenameTemplate` (`string`): 查看 [`output.devtoolModuleFilenameTemplate`](/configuration/output/#outputdevtoolmodulefilenametemplate)。
- `module` (`boolean`): 默认值为 `true`，表示是否为 loaders 添加 source map。
- `columns` (`boolean`): 默认值为 `true`，表示是否使用列映射(column mapping)。
- `protocol` (`string`): 默认协议名为 `webpack-internal://`，允许用户重新定义协议。

T> 将 `module` 或者 `columns` 设置为 `false` 将产生较不精确的源映射，但会明显提高编译性能。

T> 如果要在 [开发模式](/configuration/mode/#mode-development) 下对此插件使用自定义配置，请确保禁用默认配置。 即设置 `devtool: false`。 

## 示例 {#examples}

以下示例演示了此插件的一些常见用例。

### 基本用例 {#basic-use-case}

可以使用以下代码替换配置选项 `devtool: eval-source-map`，并使用等效的自定义插件配置：

```js
module.exports = {
  // ...
  devtool: false,
  plugins: [
    new webpack.EvalSourceMapDevToolPlugin({})
  ]
};
```

### 排除 Vendor(第三方库) Maps {#exclude-vendor-maps}

下面的代码将排除 `vendor.js` 包中任何模块 source map 的生成：

``` js
new webpack.EvalSourceMapDevToolPlugin({
  exclude: ['vendor.js']
});
```
