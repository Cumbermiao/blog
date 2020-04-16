---
title: '前端监控'
date: 2020-03-30 09:25:18
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
- [如何进行前端监控](http://www.alloyteam.com/2020/01/14184/)
## 性能监控
- FCP ： 监听文档中第一个元素被渲染的时间。
## js 错误监控
- 当JavaScript运行时错误（包括语法错误）发生时，window会触发一个ErrorEvent接口的error事件，并执行window.onerror()。
- 当一项资源（如`<img>`或`<script>`）加载失败，加载资源的元素会触发一个Event接口的error事件，并执行该元素上的onerror()处理函数。这些error事件不会向上冒泡到window，不过（至少在Firefox中）能被单一的`window.addEventListener`捕获。
## http 请求监控
