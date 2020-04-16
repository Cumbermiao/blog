---
title: 记一次webpack拆包
date: 2019-09-11 16:03:47
tags:
---

最近部门对买家vo管理系统准备使用 react + webpack 进行工程化管理和重构， 本次开发的新功能账期便直接上了， 主要用了 react + react-router-dom + antd 。
在最终打包的时候遇到了一个问题： 使用 splitChunk 将 node_modules 中的三方包全部拆分到一个 vendor chunk 里面体积较大， 达到了 1.34m 。
分析了打包结构之后， 发现 antd 占了很大比重， 想到以后开发其他模块也是基于 antd 于是就决定将 antd 单独拆分出来， 其他的 react 等三方库还是放在同一个chunk里。

以下是 splitChunk 配置
```js
splitChunks: {
  cacheGroups: {
    vendor: {
        test: /node_modules/,
        chunks: 'initial',
        name: (module)=>{
            const packageName = module.context.match(/[\\/]node_modules[\\/](.*?)([\\/]|$)/)[1];
            if(packageName.match(/(ant\-design|antd)/))return 'common-ui'
            return 'common'
        }
    },
  }
}
```
上面的配置会将 ant-design 和 ant 的模块拆分到 common-ui 里面， 其中 css 也需要单独提取出来，需要注意打包的文件名
```js
new MiniCssExtractPlugin({
  filename: '[name]/[name].css',
  chunkFilename: '[name]/[name].css'
})
```

最终拆分的目录结构如下：
common        common.js common.css
common-ui     common-ui.js common-ui.css
credit        credit.js credit.css

由于对 antd 的主题进行了定制， 最终打出来的样式文件 credit.css 好像包含了 antd 中的样式， 后面还需要对其进行优化。

随着各个模块的不断加入， 开发时候的热更新肯定会越来越慢， 到时候需要使用 dll 进行优化， 对于打包速度到时候看情况， 对于一些三方模块可以使用开发时候的缓存，不再重新打包减少打包时长。

## 后记
以上打包只是固定拆分了 antd 库， 其中在实践过程中遇到了一些问题。

由于需要定制主题， 所以使用 ant.less 改变了主题颜色， 如下：
```less
//ant.less

@import "~antd/dist/antd.less";

@primary-color: #1470CC; // 全局主色
@link-color: #1470CC; // 链接色
@success-color: #00b300; // 成功色
@warning-color: #FF7733; // 警告色
@error-color: #E64545; // 错误色
@font-size-base: 14px; // 主字号
@heading-color:#222; // 标题色
@text-color: #555; // 主文本色
@text-color-secondary:#888; // 次文本色
@disabled-color : #b3b3b3; // 失效色
@border-radius-base: 6px; // 组件/浮层圆角
@border-color-base: #ced3d9; // 边框色
@box-shadow-base: none; // 浮层阴影
```

在多页中引入 ant.less 和 各自的样式
```js
//page1
import 'ant.less';
import 'index.scss';

//page2
import 'ant.less';
```
不知道是由于 scss 的原因， 使用以上写法打包之后 page1.css 和 page2.css 中还是包含了 ant 的样式， 虽然 common-ui 也提取了 ant 的样式。

最终还是改了写法， ant 中只定义覆盖主题的变量， 在 js 中引入 antd.less， 如下:
```js
import 'antd/dist/antd.less';
import 'ant.less';
import 'index.scss';

//page2
import 'antd/dist/antd.less';
import 'ant.less';
```

以上只是对 antd 提取， 在多页中还有通用组件， 如果组件较多也需要进行提取。