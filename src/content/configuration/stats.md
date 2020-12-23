---
title: stats 对象
sort: 18
contributors:
  - SpaceK33z
  - sallar
  - jungomi
  - ldrick
  - jasonblanchard
  - byzyk
  - renjithspace
  - Raiondesu
  - EugeneHlushko
  - grgur
  - anshumanv
  - pixel-ray
  - snitin315
  - u01jmg3
  - grrizzly
---

`object` `string`

`stats` 选项让你更精确地控制 bundle 信息该怎么显示。 如果你不希望使用 `quiet` 或 `noInfo` 这样的不显示信息，而是又不想得到全部的信息，只是想要获取某部分 bundle 的信息，使用 stats 选项是比较好的折衷方式。

T> 对于 webpack-dev-server，这个属性要放在 [`devServer` 配置对象](/configuration/dev-server/#devserverstats-).

W> 在使用 Node.js API 时，此选项无效。

__webpack.js.org__

```js
module.exports = {
  //...
  stats: 'errors-only'
};
```

## Stats Presets {#stats-presets}

webpack 有一些特定的预设选项给统计信息输出：


| 预设              | 可选值 | 描述                                                    |
| ------------------- | ----------- | -------------------------------------------------------------- |
| `'errors-only'`     | _none_      | 只在发生错误时输出                                                |
| `'errors-warnings'` | _none_      | 只在发生错误或有新的编译时输出                                      |
| `'minimal'`         | _none_      | 只在发生错误或新的编译开始时输出                                    |
| `'none'`            | `false`     | 没有输出                                                        |
| `'normal'`          | `true`      | 标准输出                                                        |
| `'verbose'`         | _none_      | 全部输出                                                        |
| `'detailed'`        | _none_      | 全部输出除了 `chunkModules` 和 `chunkRootModules`                |

## Stats 选项 {#stats-options}

你可以在统计输出里指定你想看到的信息。

T> 所有在统计信息配置里的选项都是可选的。

### `stats.all` {#statsall}

当统计信息配置没被定义，则该值是一个回退值。它的优先级比本地的 webpack 默认值高。

```javascript
module.exports = {
  //...
  stats: {
    all: undefined
  }
};
```

### `stats.assets` {#statsassets}

`boolean = true`

告知 `stats` 是否展示资源信息。将 `stats.assets` 设置成 `false` 会禁用.

```javascript
module.exports = {
  //...
  stats: {
    assets: false
  }
};
```

### `stats.assetsSort` {#statsassetssort}

`string = 'id'`

告知 `stats` 基于给定的字段对资源进行排序。所有的 [排序字段](#sorting-fields)都被允许作为 `stats.assetsSort`的值。使用 `!` 作为值的前缀以反转基于给定字段的排序结果。

```javascript
module.exports = {
  //...
  stats: {
    assetsSort: '!size'
  }
};
```

### `stats.builtAt` {#statsbuiltat}

`boolean = true`

告知 `stats` 是否添加构建日期与时间信息。将 `stats.builtAt` 设置成 `false` 来隐藏.

```javascript
module.exports = {
  //...
  stats: {
    builtAt: false
  }
};
```

### `stats.moduleAssets` {#statsmoduleassets}

`boolean = true`

告知 `stats` 是否添加模块内的资源信息。将 `stats.moduleAssets` 设置成 `false` 以隐藏。

```javascript
module.exports = {
  //...
  stats: {
    moduleAssets: false
  }
};
```

### `stats.cached` {#statscached}

`boolean = true`

告知 `stats` 是否添加关于缓存模块的信息 (并非被构建的模块)。

```javascript
module.exports = {
  //...
  stats: {
    cached: false
  }
};
```

### `stats.assetsSpace` {#statsassetsspace}

`number = 15`

告诉 `stats` 应该显示多少个 asset 项目（将以组的方式折叠，以适应这个空间）。

```javascript
module.exports = {
  //...
  stats: {
    assetsSpace: 15
  }
};
```

### `stats.modulesSpace` {#statsmodulesspace}

`number = 15`

告诉 `stats` 应该显示多少个模块项目（将以组的方式折叠，以适应这个空间）。

```javascript
module.exports = {
  //...
  stats: {
    modulesSpace: 15
  }
};
```

### `stats.chunkModulesSpace` {#statschunkmodulesspace}

`number = 10`

告诉 `stats` 显示多少个 chunk 模块项目（将以组的方式折叠，以适应这个空间）。

```javascript
module.exports = {
  //...
  stats: {
    chunkModulesSpace: 15
  }
};
```

### `stats.nestedModulesSpace` {#statsnestedmodulesspace}

`number = 10`

告诉 `stats` 应该显示多少个嵌套模块的项目（将以组的方式折叠，以适应这个空间）。

```javascript
module.exports = {
  //...
  stats: {
    nestedModulesSpace: 15
  }
};
```

### `stats.cachedModules` {#statscachedmodules}

`boolean = true`

告诉 `stats` 是否要缓存（非内置）模块的信息。

```javascript
module.exports = {
  //...
  stats: {
    cachedModules: false
  }
};
```

### `stats.runtimeModules` {#statsruntimemodules}

`boolean = true`

告诉 `stats` 是否要添加运行时模块的信息。

```javascript
module.exports = {
  //...
  stats: {
    runtimeModules: false
  }
};
```

### `stats.dependentModules` {#statsdependentmodules}

`boolean`

告诉 `stats` 是否要展示该 chunk 依赖的其他模块的 chunk 模块。

```javascript
module.exports = {
  //...
  stats: {
    dependentModules: false
  }
};
```

### `stats.groupAssetsByChunk` {#statsgroupassetsbychunk}

`boolean`

告诉 `stats` 是否按照 asset 与 chunk 的关系进行分组。

```javascript
module.exports = {
  //...
  stats: {
    groupAssetsByChunk: false
  }
};
```

### `stats.groupAssetsByEmitStatus` {#statsgroupassetsbyemitstatus}

`boolean`

告诉 `stats` 是否按照 asset 的状态进行分组（emitted，对比 emit 或缓存）.

```javascript
module.exports = {
  //...
  stats: {
    groupAssetsByEmitStatus: false
  }
};
```

### `stats.groupAssetsByInfo` {#statsgroupassetsbyinfo}

`boolean`

告诉 `stats` 是否按照 asset 信息对 asset 进行分组（immutable，development。hotModuleReplacement 等）。

```javascript
module.exports = {
  //...
  stats: {
    groupAssetsByInfo: false
  }
};
```

### `stats.groupModulesByAttributes` {#statsgroupmodulesbyattributes}

`boolean`

告诉 `stats` 是否按模块的属性进行分组（errors，warnings，assets，optional，orphan 或者 dependent）。

```javascript
module.exports = {
  //...
  stats: {
    groupModulesByAttributes: false
  }
};
```

### `stats.cachedAssets` {#statscachedassets}

`boolean = true`

告知 `stats` 是否添加关于缓存资源的信息。 将 `stats.cachedAssets` 设置成 `false` 会告知 `stats` 仅展示被生成的文件 (并非被构建的模块)。

```javascript
module.exports = {
  //...
  stats: {
    cachedAssets: false
  }
};
```

### `stats.children` {#statschildren}

`boolean = true`

告知 `stats` 是否添加关于子模块的信息。

```javascript
module.exports = {
  //...
  stats: {
    children: false
  }
};
```

### `stats.chunks` {#statschunks}

`boolean = true`

告知 `stats` 是否添加关于 chunk 的信息。 将 `stats.chunks` 设置为 `false` 会引发更少的输出。

```javascript
module.exports = {
  //...
  stats: {
    chunks: false
  }
};
```

### `stats.chunkGroups` {#statschunkgroups}

`boolean = true`

告知 `stats` 是否添加关于 `namedChunkGroups` 的信息。

```javascript
module.exports = {
  //...
  stats: {
    chunkGroups: false
  }
};
```

### `stats.chunkModules` {#statschunkmodules}

`boolean = true`

告知 `stats` 是否添加关于已构建模块和关于 chunk 的信息。

```javascript
module.exports = {
  //...
  stats: {
    chunkModules: false
  }
};
```

### `stats.chunkOrigins` {#statschunkorigins}

`boolean = true`

告知 `stats` 是不添加关于 chunks 的来源和 chunk 合并的信息。

```javascript
module.exports = {
  //...
  stats: {
    chunkOrigins: false
  }
};
```

### `stats.chunksSort` {#statschunkssort}

`string = 'id'`

告知 `stats` 基于给定的字段给 chunks 排序。所有 [排序字段](#sorting-fields) 都被允许用于作为 `stats.chunksSort` 的值。使用  `!` 作为值里的前缀用以将基于给定字段排序的结果反转。

```javascript
module.exports = {
  //...
  stats: {
    chunksSort: 'name'
  }
};
```

### `stats.context` {#statscontext}

`string = '../src/'`

设置上下文目录用以将文件请求信息变短。

```javascript
module.exports = {
  //...
  stats: {
    context: '../src/components/'
  }
};
```

### `stats.colors` {#statscolors}

`boolean = false` `object`

告知 `stats` 是否输出不同的颜色。

```javascript
module.exports = {
  //...
  stats: {
    colors: true
  }
};
```

它也可用通过命令行的参数实现：

```bash
webpack-cli --colors
```

你可以通过使用 [ANSI escape sequences](https://en.wikipedia.org/wiki/ANSI_escape_code) 指定你自己的命令行终端颜色。

```js
module.exports = {
  //...
  colors: {
    green: '\u001b[32m',
  },
};
```

### `stats.depth` {#statsdepth}

`boolean = false`

告知 `stats` 是否展示每个模块与入口文件的距离。

```javascript
module.exports = {
  //...
  stats: {
    depth: true
  }
};
```

### `stats.entrypoints` {#statsentrypoints}

`boolean = true` `string = 'auto'`

告知 `stats` 是否展示入口文件与对应的文件 bundles。

```javascript
module.exports = {
  //...
  stats: {
    entrypoints: false
  }
};
```

当 `stats.entrypoints` 被设置为 `'auto'` 时，webpack 将自动决定是否在 stats 输出中展示入口信息。

### `stats.env` {#statsenv}

`boolean = false`

告知 `stats` 是否展示 `--env` 信息.

```javascript
module.exports = {
  //...
  stats: {
    env: true
  }
};
```

### `stats.orphanModules` {#statsorphanmodules}

`boolean = false`

告知 `stats` 是否隐藏 `孤儿(orphan)` 模块. 一个模块属于 `孤儿(orphan)` 如果它不被包含在任何一个 chunk里。孤儿模块默认在 `stats` 中会被隐藏。

```javascript
module.exports = {
  //...
  stats: {
    orphanModules: true
  }
};
```

### `stats.errors` {#statserrors}

`boolean = true`

告知 `stats` 是否展示错误。

```javascript
module.exports = {
  //...
  stats: {
    errors: false
  }
};
```

### `stats.errorDetails` {#statserrordetails}

`boolean = true`

告知 `stats` 是否添加错误的详情。

```javascript
module.exports = {
  //...
  stats: {
    errorDetails: false
  }
};
```

### `stats.errorStack` {#statserrorstack}

`boolean = true`

告知 `stats` 是否展示错位的栈追踪信息。

```javascript
module.exports = {
  //...
  stats: {
    errorStack: false
  }
};
```

### `stats.excludeAssets` {#statsexcludeassets}

`array = []: string | RegExp | function (assetName) => boolean` `string` `RegExp` `function (assetName) => boolean`

告知 `stats` 排除掉匹配的资源信息。这个可以通过设置一个 `字符串`, 一个 `正则表达式`, 一个 `函数` 取得资源的名字作为入参且返回一个 `布尔值`。 `stats.excludeAssets` 可以是一个包括上面任意一类型值的 `数组` 。

```javascript
module.exports = {
  //...
  stats: {
    excludeAssets: [
      'filter',
      /filter/,
      (assetName) => assetName.contains('moduleA')
    ]
  }
};
```

### `stats.excludeModules` {#statsexcludemodules}

`array = []: string | RegExp | function (assetName) => boolean` `string` `RegExp` `function (assetName) => boolean` `boolean: false`

告知 `stats` 排除掉匹配的资源信息。这个可以通过设置一个 `字符串`, 一个 `正则表达式`, 一个 `函数` 取得资源的名字作为入参且返回一个 `布尔值`。 `stats.excludeModules` 可以是一个包括上面任意一类型值的 `数组` 。`stats.excludeModules` 会与 `stats.exclude` 的配置值[进行合并](https://github.com/webpack/webpack/blob/master/lib/Stats.js#L215)。

```javascript
module.exports = {
  //...
  stats: {
    excludeModules: [
      'filter',
      /filter/,
      (moduleSource) => true
    ]
  }
};
```

将 `stats.excludeModules` 设置为 `false` 会禁用以上的排除行为。

```javascript
module.exports = {
  //...
  stats: {
    excludeModules: false
  }
};
```

### `stats.exclude` {#statsexclude}

详参 [`stats.excludeModules`](#statsexcludemodules).

### `stats.hash` {#statshash}

`boolean = true`

告知 `stats` 是否添加关于编译哈希值的信息。

```javascript
module.exports = {
  //...
  stats: {
    hash: false
  }
};
```

### `stats.logging` {#statslogging}

`string = 'info': 'none' | 'error' | 'warn' | 'info' | 'log' | 'verbose'` `boolean`

告知 `stats` 是否添加日志输出。

- `'none'`, `false` - 禁用日志
- `'error'` - 仅显示错误
- `'warn'` - 仅显示错误与告警
- `'info'` - 显示错误，告警与信息
- `'log'`, `true` - 显示错误，告警与信息，日志，组别，清理。折叠组别会在折叠状态中被显示 。
- `'verbose'` - 输出所有日志除了调试与追踪。 折叠组别会在扩展状态中被显示 。

```javascript
module.exports = {
  //...
  stats: {
    logging: 'verbose'
  }
};
```

### `stats.loggingDebug` {#statsloggingdebug}

`array = []: string | RegExp | function (name) => boolean` `string` `RegExp` `function (name) => boolean`

告知 `stats` 去包括特定的日志工具调试信息比如插件或加载器的日志工具。当 [`stats.logging`](#statslogging) 被设置为 `false`, `stats.loggingDebug` 配置会被忽略。

```javascript
module.exports = {
  //...
  stats: {
    loggingDebug: [
      'MyPlugin',
      /MyPlugin/,
      /webpack/, // To get core logging
      (name) => name.contains('MyPlugin')
    ]
  }
};
```

### `stats.loggingTrace` {#statsloggingtrace}

`boolean = true`

启用错误，告警与追踪的日志输出中的堆栈追踪。将 `stats.loggingTrace` 设置为 `false` 隐藏追踪。


```javascript
module.exports = {
  //...
  stats: {
    loggingTrace: false
  }
};
```

### `stats.modules` {#statsmodules}

`boolean = true`

告知 `stats` 是否添加关于构建模块的信息。

```javascript
module.exports = {
  //...
  stats: {
    modules: false
  }
};
```

### `stats.modulesSort` {#statsmodulessort}

`string = 'id'`

告知 `stats` 基于给定的字段对资源进行排序。所有的 [排序字段](#sorting-fields)都被允许作为 `stats.modulesSort`的值。使用 `!` 作为值的前缀以反转基于给定字段的排序结果。

```javascript
module.exports = {
  //...
  stats: {
    modulesSort: 'size'
  }
};
```

### `stats.moduleTrace` {#statsmoduletrace}

`boolean = true`

告知 `stats` 展示依赖和告警/错误的来源。`stats.moduleTrace` 从 webpack 2.5.0 起可用。

```javascript
module.exports = {
  //...
  stats: {
    moduleTrace: false
  }
};
```

### `stats.outputPath` {#statsoutputpath}

`boolean = true`

告知 `stats` 展示 `outputPath`.

```javascript
module.exports = {
  //...
  stats: {
    outputPath: false
  }
};
```

### `stats.performance` {#statsperformance}

`boolean = true`

告知 `stats` 当文件大小超过 [`performance.maxAssetSize`](/configuration/performance/#performancemaxassetsize)配置值时，展示性能提性。

```javascript
module.exports = {
  //...
  stats: {
    performance: false
  }
};
```

### `stats.preset` {#statspreset}

`string` `boolean: false`

为展示的信息类型设置 [预设值](/configuration/stats/#stats-presets)。这对[扩展统计信息行为](/configuration/stats/#extending-stats-behaviours)非常有用。

```javascript
module.exports = {
  //...
  stats: {
    preset: 'minimal'
  }
};
```

将 `stats.preset` 的值设置为`false` 告知 webpack 使用 `'none'` [统计信息预设值](/configuration/stats/#stats-presets)。

### `stats.providedExports` {#statsprovidedexports}

`boolean = false`

告知 `stats` 去展示模块的导出。

```javascript
module.exports = {
  //...
  stats: {
    providedExports: true
  }
};
```

### `stats.errorsCount` {#statserrorscount}

`boolean = true`

添加展示 errors 个数。

```javascript
module.exports = {
  //...
  stats: {
    errorsCount: false
  }
};
```

### `stats.warningsCount` {#statswarningscount}

`boolean = true`

添加展示 warnings 个数。

```javascript
module.exports = {
  //...
  stats: {
    warningsCount: false
  }
};
```

### `stats.publicPath` {#statspublicpath}

`boolean = true`

告知 `stats` 展示 `publicPath`。

```javascript
module.exports = {
  //...
  stats: {
    publicPath: false
  }
};
```

### `stats.reasons` {#statsreasons}

`boolean = true`

告知 `stats` 添加关于模块被引用的原因信息。

```javascript
module.exports = {
  //...
  stats: {
    reasons: false
  }
};
```

### `stats.relatedAssets`  {#statsrelatedassets}

`boolean = false`

告诉 `stats` 是否需添加与其他 assets 相关的信息（例如 assets 的 SourceMaps）。

```javascript
module.exports = {
  //...
  stats: {
    relatedAssets: true
  }
};
```

### `stats.source`  {#statssource}

`boolean = false`

告知 `stats` 去添加模块的源码。

```javascript
module.exports = {
  //...
  stats: {
    source: true
  }
};
```

### `stats.timings` {#statstimings}

`boolean = true`

告知 `stats` 添加时间信息。

```javascript
module.exports = {
  //...
  stats: {
    timings: false
  }
};
```

### `stats.usedExports` {#statsusedexports}

`boolean = false`

告知 `stats` 是否展示模块用了哪些导出。

```javascript
module.exports = {
  //...
  stats: {
    usedExports: true
  }
};
```

### `stats.version` {#statsversion}

`boolean = true`

告知 `stats` 添加关于 webpack 版本的信息。

```javascript
module.exports = {
  //...
  stats: {
    version: false
  }
};
```

### `stats.chunkGroupAuxiliary` {#statschunkgroupauxiliary}

`boolean = true`

在 chunk 组中展示辅助 asset。

```javascript
module.exports = {
  //...
  stats: {
    chunkGroupAuxiliary: false
  }
};
```

### `stats.chunkGroupChildren` {#statschunkgroupchildren}

`boolean = true`

显示 chunk 组的子 chunk。（例如，预置（prefetched），预加载（preloaded）的 chunk 和 asset)。

```javascript
module.exports = {
  //...
  stats: {
    chunkGroupChildren: false
  }
};
```

### `stats.chunkGroupMaxAssets` {#statschunkgroupmaxassets}

`number`

chunk 组中的 asset 数上限。

```javascript
module.exports = {
  //...
  stats: {
    chunkGroupMaxAssets: 5
  }
};
```

### `stats.warnings` {#statswarnings}

`boolean = true`

告知 `stats` 添加告警。

```javascript
module.exports = {
  //...
  stats: {
    warnings: false
  }
};
```

### `stats.warningsFilter` {#statswarningsfilter}

`array = []: string | RegExp | function (warning) => boolean` `string` `RegExp` `function (warning) => boolean`

告知 `stats` 排除掉匹配的告警信息。这个可以通过设置一个 `字符串`, 一个 `正则表达式`, 一个 `函数` 取得资源的名字作为入参且返回一个 `布尔值`。 `stats.warningsFilter` 可以是一个包括上面任意一类型值的 `数组` 。

```javascript
module.exports = {
  //...
  stats: {
    warningsFilter: [
      'filter',
      /filter/,
      (warning) => true
    ]
  }
};
```


W> `stats.warningsFilter` 已被弃用，请改用 [`ignoreWarnings`](/configuration/other-options/#ignorewarnings)。

### `stats.chunkRelations` {#statschunkrelations}

`boolean = false`

告知 `stats` 展示 chunk 的父chunk，孩子chunk和兄弟chunk。

### 字段排序 {#sorting-fields}

对于 `assetsSort`, `chunksSort` 和 `modulesSort` 它们有几个可用的字段用于排序：

- `'id'` 是元素（指资源，chunk或模块，下同）的 id;
- `'name'` - 一个元素的名字，在导引的时候被分配；
- `'size'` - 一个元素的大小，单位字节（bytes）;
- `'chunks'` - 元素来源于哪些 chunks (例如，一个 chunk 有多个子 chunks， - 子 chunks 会被基于主 chunk 组合到一起);
- `'errors'` - 元素组错误的数量;
- `'warnings'` - 元素中告警的数量;
- `'failed'` - 元素是被编译失败;
- `'cacheable'` - 元素是否被缓存;
- `'built'` - 资源是否被构建;
- `'prefetched'` - 资源是否被预拉取;
- `'optional'` - 资源是否可选;
- `'identifier'` - 元素的标识符;
- `'index'` - 元素加工指针;
- `'index2'`
- `'profile'`
- `'issuer'` - 发起者(issuer)的标识符;
- `'issuerId'` - 发起者(issuer)的id;
- `'issuerName'` - 发起者(issuer)的名字;
- `'issuerPath'` - 一个完整的发起者(issuer)对象。基于这个字段排序没有现实的需要;

### 扩展统计信息行为 {#extending-stats-behaviours}

如果你想使用其中一个预定义的行为，例如 `'minimal'`，但仍想重载一个或更多的规则：请指定想要设置的 `stats.preset` 同时在后面添加自定义或额外的规则。

__webpack.config.js__

```javascript
module.exports = {
  //..
  stats: {
    preset: 'minimal',
    moduleTrace: true,
    errorDetails: true
  }
};
```
