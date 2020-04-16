---
title: webpack入门
date: 2019-03-04 16:15:18
tags:
---

## webpack

### webpack 基础配置

#### webpack4 安装

```
npm i webpack webpack-cli webpack-dev-server -S
```

#### 基础配置

```
module.exports = {
  entry: resolve(__dirname, './src/app.js'),
    output: {
        filename: 'bundle[hash:5].js',
        path: resolve(__dirname, './dist'),
    },
    devServer: {
        host: 'localhost',
        port: 8080,
        hot: true,
        compress: true
    },
    module: {
        rules: [],
    },
    plugins: [],
}
```

#### 升级到 webpack 4

- 新版本将命令行相关的东西单独拆了出去封装成了 `webpack-cli` ， `webpack xxx` 命令需要依赖 `webpack-cli` .
- node 版本升级 `>=6.11.5`
- `npm outdated` 升级老项目的依赖

#### 相关变化

- 新功能 `SideEffects`、`Module Type’s Introduced`、`WebAssembly Support`
- 改动功能

| 废弃                  | 代替                                  | 功能                    |
| --------------------- | ------------------------------------- | ----------------------- |
| UglifyjsWebpackPlugin | optimization.minmize                  | Tree shaking & Minimize |
| CommonsChunkPlugin    | optimization.splitChunks/runtimeChunk | code split              |
| NoEmitOnErrorsPlugin  | noEmitOnErrors                        | 编译错误跳出输出        |

#### 默认配置

> webpack 4 引入了零配置的概念

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

#### 打包相关概念

- bundle : 包，即打包输出的文件
- vendor : 第三方文件， 即我们开始引入的第三方的库
- chunk : 打包提取的独立文件，一般为多页应用公用的文件或者应用中在多个地方使用的文件
- hash ： 每次打包生成的 md5
- chunkhash :
- contenthash : 自身内容变化 hash 才会改变

#### 热更新

> 当路由懒加载的页面较多试，本地热更新会比较慢。
> webpack 4 `使用babel` 的 `plugins` `babel-plugin-dynamic-import-node`。它只做一件事就是：将所有的`import()`转化为`require()`，这样就可以用这个插件将所有异步组件都用同步的方式引入了，并结合 `BABEL_ENV` 这个`bebel`环境变量，让它只作用于开发环境下。将开发环境中所有`import()`转化为`require()`，这种方案解决了之前重复打包的问题，同时对代码的侵入性也很小，你平时写路由的时候只需要按照官方文档路由懒加载的方式就可以了，其它的都交给`babel`来处理，当你不想用这个方案的时候，也只需要将它从`babel` 的 `plugins`中移除就可以了。

```
// script 中添加 BABEL_ENV
"dev": "BABEL_ENV=development webpack-dev-server XXXX"

//bable plugins 中使用 babel-plugin-dynamic-import-node (仅用于开发模式)
{
    test: /\.(js|jsx)$/,
    use: {
        loader: 'babel-loader',
        options: {
            presets: ['@babel/env', '@babel/preset-react'],
            plugins: build?[]:['babel-plugin-dynamic-import-node'],
        },
    },
    exclude: /node_modules/,
},

```

#### Tree shaking

webpack 4 只是增加了 JSON Tree Shaking 和 sideEffects 能让你能更好优化代码。
默认 webpack 是支持 Tree-Shaking 的，但在你的项目中可能会因为 babel 的原因导致它失效。

因为 Tree Shaking 这个功能是基于 ES6 modules 的静态特性检测，来找出未使用的代码，所以如果你使用了 babel 插件的时候，如：babel-preset-env，它默认会将模块打包成 commonjs，这样就会让 Tree Shaking 失效了。

解决方法只要让 babel 不transform modules就可以了(设置 modules:false)
#### 使用 babel

```
npm i @babel/core @babel/preset-env
npm i @babel/polyfill
```

- 配置 .babelrc 或者 babel-loader 的 options

```
{
    "presets": [
        ['env',{
            "modules": false,
            "targets": {
                "browsers": ['> 1%', 'last 2 versions']
            }
        }]
    ]
}

```

入口文件中添加 babel-polyfill ，保证其首先加载

```
entry:['babel-polyfill','app.js']
```

### postcss loader

> postcss-loader cssnao autoprefixer cssnext(包含 autoprefixer，不能同时安装)

### optimize-css-assets-webpack-plugin

> 优化，压缩 css 代码，默认使用 cssnano ， 如果使用了 postcss-loader 可以配置 postcssrc.js 使用 cssnano，也会自动压缩。

```
new OptimizeCssAssetsPlugin({
    assetNameRegExp: /\.css$/g,
    cssProcessor: require('cssnano'),
    cssProcessorPluginOptions: {
        preset: ['default', { discardComments: { removeAll: true } }],
    },
    canPrint: true,
})
```

#### webpack-bundle-analyzer

> 分析打包的目录结构

```
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

plugins:[
    new BundleAnalyzerPlugin()
]
```

### mini-css-extract-plugin ? extract-text-webpack-plugin

> webpack 4 使用 mini-css-extract-plugin 将样式提取成单独的 css 文件，而且据说处理速度比 extract-text-webpack-plugin 快。
> 可以 filename 使用 contenthash ，这样在打包时 css 文件没有变化时不会重新打包。

```
new MiniCssExtractPlugin({
    filename: '[name].css',
    chunkFilename: '[id].css',
})
```

### speed-measure-webpack-plugin

> 测 loader 和 plugin 的处理速度

```
const smp = new SpeedMeasurePlugin();
module.exports= smp.wrap(merge(defaultConf, build ? prodConf : devConf))
```

### clean-webpack-plugin

> 清除打包目录文件

### uglifyjs-webpack-plugin & terser-webpack-plugin

> 压缩 js 文件，webpack4 打包时设置 mode 为 production 会自动压缩 ， 该插件还有其他配置，如可以去除 console，开启多线程提高打包速度等。
> 可以在 optimization 中配置压缩优化的插件， 官方文档中说默认的使用的是 terser-webpack-plugin

虽然该插件有其他功能，但是在实际使用中该插件的**处理较长**，压缩的大小稍微小一点。

```
optimization: {
        minimize: true,
        minimizer: [
            new UglifyWebpackPlugin({
                parallel: true,
                uglifyOptions: {
                    compress: {
                        warnings: false,
                        drop_console: true,
                        pure_funcs: ['console.log'],
                    },
                },
                sourceMap: true,
            }),
            // new TerserPlugin({
            //     cache: true,
            //     parallel: true,
            //     sourceMap: true, // Must be set to true if using source-maps in production
            //     terserOptions: {
            //         // https://github.com/webpack-contrib/terser-webpack-plugin#terseroptions
            //     },
            // }),
        ],
    },
```

### webpack-build-notifier

> 打包结束时气泡通知

### progress-bar-webpack-plugin

> 显示当前的 webpack 进度，与 progress 类似，样式好看一点

### DllPlugin & DllReferencePlugin

> DllPlugin 用于生成第三方包的 manifest.json ， 提取编译第三方的 js 而不需要每次启动 webpack 时都要重新编译 ，manifest 用于映射引入包和编译的 js 位置。 配置 DllReferencePlugin 可以在编译时通过 manifest 的映射直接找到包，加快 webpack 的编译速度。

在 npm run dev 之前先要 npm run dll 生成第三方的编译文件和 manifest.json 映射， DllPlugin 的 entry 只接受数组，所以 dev 和 dll 的 entry 要分开配置。

- 注意 context 需要配置，否则会包 data must be a string or a buffer 的错误

```
//dll.config.js
module.exports = {
    plugins:[
        new webpack.DllPlugin({
            path: path.resolve(__dirname,'../manifest.json')
        })
    ]
}

//dev.config.js
plugins:[
    new webpack.DllReferencePlugin({
        context:__dirname,
        path: path.resolve(__dirname,'../manifest.json)
    })
]

// webpack.config.js
let entry = process.env.NODE_ENV === 'dll'?{
    vendor:['react','redux','react-redux'],
}:{
    main: './src/app.js'
    vendor:['react','redux','react-redux'],
}
```

### generate-asset-webpack-plugin
