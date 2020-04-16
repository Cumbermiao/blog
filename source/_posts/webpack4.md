---
title: 'webpack4 攻略'
date: 2019-03-27 16:15:18
tags: [webpack]
published: false
hideInList: false
feature: 
isTop: false
---

# webpack4 攻略

## webpack4 安装

> `webpack4` 将命令行相关的东西单独拆了出去封装成了 `webpack-cli` ， `webpack xxx` 命令需要依赖 `webpack-cli`。

注意 `node` 版本升级 `>=6.11.5`。  
如果你是从低版本升级过来，需要使用 `npm outdated` 升级老项目的依赖

```
npm i webpack webpack-cli webpack-dev-server -S
```

## webpack 配置项

- mode : `webpack` 打包模式
- context : 基础目录
- entry : 入口文件
- output : 打包文件
- module : 用于配置对不同类型的模块的处理
- resolve : 配置各模块的解析
- plugins : 插件
- optimization : 配置文件的优化
- devServer : `webpack-dev-server` 配置项
- watch : 构建之后是否继续监听文件变化
- watchOptions : 监听文件变化的配置项
- performance : 性能配置项
- externals : 外部扩展配置

### mode

> `webpack4` 引入了零配置的概念，根据 `mode` 在不同的模式下，会有一些默认的配置项，不需要用户配置很多。

- development 默认开启了`NamedChunksPlugin` 和`NamedModulesPlugin`方便调试，提供了更完整的错误信息，更快的重新编译的速度。

```
module.exports = {
+ mode: 'development'
- devtool: 'eval',
- plugins: [
-   new webpack.NamedModulesPlugin(),
-   new webpack.NamedChunksPlugin(),
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]
}
```

- production 默认开启 splitChunks 和 minimize ，包括 tree-shaking ,但是可能会与 babel 冲突，导致失效。

```
module.exports = {
+  mode: 'production',
-  plugins: [
-    new UglifyJsPlugin(/* ... */),
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(),
-    new webpack.NoEmitOnErrorsPlugin()
-  ]
}

```

修改配置文件可以使用 nodemon 检测进行自启动。

### context & entry

> `webpack` 打包的入口文件， 可以根据项目需求进行不同的配置。
> `entry` 的路径是相对于 `context`， `context` 默认是当前目录。

每个 `entry` 里的对象可以理解为一个 `chunk` ， 在打包时 `chunk` 会包含很多模块，如果其中一个模块的文件发生了变化，整个 `chunk` 就会发生变化。
所以不管在开发或者打包时，如果发现时间较长就可以考虑提取`chunk`中的模块。

### output

> `output` 主要用于配置打包文件的路径及文件名。

```
output:{
  filename: [name]_[hash:5]_bundle.js,
  path: path.resolve(__dirname,'./dist')
  publicPath : build?'/webapp/':''
}
```

`publicPath`常使用中有两种场景：

- 第一种是前后端开发分离，但是最终前端打完包之后需要给后端，后端再打包，这时候前端文件相对于服务的路径可能会发生变化，例如访问 `index.html` 后台的路由为 `/webapp/index.html` ，
  那么其他的静态资源路径应该也是 `/webapp/index.js` 这种，这就需要设置 `publicPath` 为 `/webapp` 。

- 第二种是前端的一些静态文件需要放到 cdn 上，那么所有的静态文件的访问就是 cdn+filename 这种形式。

`filename` 的`md5` 分为： `hash`，`chunkhash`，`contenthash` ，主要防止缓存相关的问题。

- `hash` 在每次打包时都会改变。
- `chunhansh` 如果当前`chunk`中有内容修改就会改变。
- `contenthash` 则是针对于当前的文件。

对于图片等静态资源，也会加上 `publicPath` 路径，要注意必须是 `/webapp/`，后面的斜杠不能省略，否则路径就变成了 `/webappfilename` 这种形式。

### module

> `module` 最常用的是配置解析各种类型模块的 `loader`。

对于 `css` 的处理，`style-loader` 使用使用 js 方式将样式使用 `<style>` 标签插入到页面的。浏览器一般情况下是优先请求样式文件，如果使用 js 插入，则需要等待 js 加载完毕，且无法缓存样式文件。
针对于以上问题， 可以使用 `mini-css-extract-plugin` 插件将样式提取成单独文件。但是与此同时要注意 js 文件和 css 文件的打包影响。

`postcss-loader` 用来优化样式代码，要注意`cssnext`包含`autoprefixer`,两者不能同时使用。

```
//.postcssrc.js
module.exports = {
    plugins:[
        require('cssnano'),
        require('postcss-cssnext')
    ]
}
```

`sass-loader`需要注意在 `calc` 中使用变量编译之后变量是不会被替换的，如`calc(100px - $width)`,在打包之后你会发现代码中`$width`并没有被替换成对应的值。  
解决方法：使用 `calc(100px - #{$width})` 的方式。

对于一些自带缓存配置的`loader`，我们可以开启缓存来加快编译构建速度，如`babel-loader`可以配置`cacheDirectory:true`。  
而那些没有缓存配置的我们可以借助于 `cache-loader`。

```
{
  loader: resolve('cache-loader'),
  options: { cacheDirectory: path.join(cache, 'cache-loader-css') },
},
{
  loader: resolve('css-loader'),
  options: {
  importLoaders: 2,
  sourceMap,
  },
}
```

对于性能优化方面配置 `loader` 时, 我们应该尽量指定`include`和`exclude` 的范围。

### resolve

`resolve` 中常用配置为 `alias`，`modules` 和 `extensions`。
`alias` 可以配置路径的别名，如我们可以简化组件的引入:

```
resolve:{
  alias:{
    $component:path.resolve(__dirname,'./src/components')
  }
}
配置了 `$components` 之后，引入组件只需要 `import xxx from '$component'`
```

`modules` 告诉 `webpack` 去哪加载模块默认设置 `["node_modules"]` 。

```
modules:[path.resolve(__dirname, 'src'),path.resolve(__dirname,'node_modules')]
```

扩展如果你使用了绝对路径后，可能就发现 vscode 智能代码导航就失效了，别慌！请在目录下面配置 jsconfig.json 文件解决这个问题，配置和上面对应:

```
{
  "compilerOptions": {
    "baseUrl": ".",
      "paths": {
      src/*": ["./src/*"],
      "components/*": ["./src/components/*"],
      "assets/*": ["./src/assets/*"],
      "pages/*": ["./src/pages/*"]
    }
  },
  "include": ["./src/**/*"]
}
```

`extensions` 可以让我们在引入模块时省略后缀名，在性能方面看我们应该少配置忽略的后缀。

### devServer

`devServer` 配置开发时`webpack-dev-server`启动的服务, 配合 `webpack.HotModuleReplacementPlugin` 进行热更新。
`proxy` 配置开发时接口的访问地址。

```
devServer: {
    host: 'localhost',
    port: 8080,
    hot: true,
    compress: true,
    proxy: {
        '/api': {
            target: 'http://localhost:3000',
            changeOrigin: true,
            pathRewrite: { '^/api': '' },
        },
    },
},
```

### watch & watchOptions

启用 Watch 模式。这意味着在初始构建之后，webpack 将继续监听任何已解析文件的更改。Watch 模式默认关闭。

> webpack-dev-server 和 webpack-dev-middleware 里 Watch 模式默认开启。

```
watchOptions: {
  aggregateTimeout: 300, //构建延时，在这段时间的改动会聚集到一次构建中
  poll: 1000, //轮询的间隔
  ignored: /node_modules/ //忽略的模块
}
```

### performance

```
performance: {
  hints: "warning", // 出现性能问题提醒的级别
  maxEntrypointSize: 250000 ,//入口chunk 最大size
  maxAssetSize: 250000,//单个文件最大size
  //计算性能时的文件
  assetFilter: function(assetFilename) {
    return assetFilename.endsWith('.js');
  }
}
```

### externals

有一些第三方或者必要的文件会放在一个公共的地址（cdn）上，使用直接使用 `script` 标签引入即可，构建时就需要将其排除在外。

```
externals: {
  jquery: 'jQuery'
}
```

### plugins

#### 常用 plugin

- html-webpack-plugin
- webpack.DllPlugin & webpack.DllReferencePlugin
- webpack.namedChunksPlugin & webpack.HashedModuleIdsPlugin 
- webpack.DefinePlugin
- HappyPack
- optimize-css-assets-webpack-plugin
- mini-css-extract-plugin / extract-text-webpack-plugin
- uglifyjs-webpack-plugin / terser-webpack-plugin
- clean-webpack-plugin
- webpack-bundle-analyzer

* progress-bar-webpack-plugin
* speed-measure-webpack-plugin
* webpack-build-notifier

## webpack 优化

#### 区分 dev & prod 配置

- publicPath
- baseUrl

```
new webpack.DefinePlugin({
  mode: JSON.stringfy(MODE)
}),
```

#### 指定 loader & plugin 的 exclude、include 范围

#### 通过 `alias` 指定模块路径

#### 多线程

- `loader` 多线程 ： `HappyPack`
- `plugin` 多线程 ： `UglifyWebpackPlugin`、`TerserPlugin`、

#### 第三方包关系映射

使用 `DllPlugin` 生成包映射， 开发环境使用 `DllReferencePlugin` ，不需要重复编译第三方包

#### minimize & tree shaking

- 使用支持 `esModule` 的第三方
- 关闭不常用的配置，指定压缩范围

```
new UglifyWebpackPlugin({
    parallel: true,
    cache: true,
    uglifyOptions: {
        compress: {
            warnings: false,
            drop_console: true,
            pure_funcs: ['console.log'],
            booleans: false,
            collapse_vars: false,
            comparisons: false,
            hoist_funs: false,
            hoist_props: false,
            hoist_vars: false,
            if_return: false,
            inline: false,
            join_vars: false,
            keep_infinity: true,
            loops: false,
            negate_iife: false,
            properties: false,
            reduce_funcs: false,
            reduce_vars: false,
            sequences: false,
            side_effects: false,
            switches: false,
            top_retain: false,
            toplevel: false,
            typeofs: false,
            unused: false,

            // 除非声明了正在使用生产版本的react-devtools，
            // 否则关闭所有类型的压缩。
            conditionals: true,
            dead_code: true,
            evaluate: true,
        },
        mangle: true,
    },
    sourceMap: true,
    chunkFilter: (chunk)=>{
        if(chunk.name==='vendor'){
             return false
        }
        return true
    }
}),
```

#### 提取的 css 文件使用 contenthash

#### runtime 环境代码提取

#### 异步组件提取

#### 使用 namedChunksPlugin / HashedModuleIdsPlugin 固定 chunkId

#### 合理的拆包策略

- chunk-initial 基础类库
- chunk-common 公共方法/组件
- chunk-ui UI 库
- chunk-others 低频使用的组件
- main 业务代码

<!-- ### inline-manifest-webpack-plugin -->





### 参考链接

- [手摸手，带你用合理的姿势使用 webpack4 ]("https://juejin.im/post/5b5d6d6f6fb9a04fea58aabc")
- [A Field Guide for Better Build Performance ]("https://slack.engineeringkeep-webpack-fast-a-field-guide-for-better-build-performance-f56a5995e8f1")
  [中文版](中文版 "https://github.com/xitu/gold-miner/blob/master/TODO/keep-webpack-fast-a-field-guide-for-better-build-performance.md")
- [webpack-dev-server]("https://survivejs.com/webpack/developing/webpack-dev-server/")
- [Webpack 4 进阶]("https://zhuanlan.zhihu.com/p/35407642")
- [webpack4.0 打包优化策略]("https://juejin.im/post/5abbc2ca5188257ddb0fae9b")
- [webpack-and-spa-guide]("https://github.com/wallstreetcn/webpack-and-spa-guide")