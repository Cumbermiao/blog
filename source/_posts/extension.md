---
title: chrome extension
date: 2019-02-26 13:51:13
tags:
---

## Chrome extension 开发

#### 参考资料
-  {% link chrome 官网 https://developer.chrome.com %}
-  {% link chrome 非官方中文翻译 https://crxdoc-zh.appspot.com/extensions/ %}

### 注意点
##### 1. content_scripts 中 css 文件会直接插入到页面，要注意否则影响到所有页面原来的样式。
##### 2. options 页面 input 框中显示 url 形式的字符串 ，在最后会默认加一个 / ，如果是空字符串 '' 是也会显示 / 所以在保存输入的 url 时要去掉最后的 / 。

