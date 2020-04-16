---
title: webpack优化
date: 2019-03-21 17:12:23
tags:
---

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
#### alias

#### 多线程

- loader 多线程 ： HappyPack
- plugin 多线程 ： UglifyWebpackPlugin、TerserPlugin、

#### 第三方包关系映射

使用 DllPlugin 生成包映射， 开发环境使用 DllReferencePlugin ，不需要重复编译第三方包

#### minimize & tree shaking

- 使用支持 esModule 的第三方
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

### runtime 环境代码提取


#### 合理的拆包策略

- chunk-initial 基础类库
- chunk-common 公共方法/组件
- chunk-ui UI 库
- chunk-others 低频使用的组件
- main 业务代码

<!-- ### inline-manifest-webpack-plugin -->

### nodemon 修改配置文件自动重启


### 参考链接

- [手摸手，带你用合理的姿势使用 webpack4 ]("https://juejin.im/post/5b5d6d6f6fb9a04fea58aabc")
- [A Field Guide for Better Build Performance ]("https://slack.engineeringkeep-webpack-fast-a-field-guide-for-better-build-performance-f56a5995e8f1")
  [中文版](中文版 "https://github.com/xitu/gold-miner/blob/master/TODO/keep-webpack-fast-a-field-guide-for-better-build-performance.md")
- [webpack-dev-server]("https://survivejs.com/webpack/developing/webpack-dev-server/")
- [Webpack 4 进阶]("https://zhuanlan.zhihu.com/p/35407642")
- [webpack4.0 打包优化策略]("https://juejin.im/post/5abbc2ca5188257ddb0fae9b")
- [webpack-and-spa-guide]("https://github.com/wallstreetcn/webpack-and-spa-guide")
