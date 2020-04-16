---
title: 记karma进行UI测试
date: 2019-06-19 17:29:03
tags:
---

## Karma
> Karma is not a testing framework, nor an assertion library. Karma just launches an HTTP server, and generates the test runner HTML file you probably already know from your favourite testing framework. So for testing purposes you can use pretty much anything you like. 

Karma 既不是测试框架也不是断言库。

### install
> karma-chrome-launcher 需要配置代理下载无头浏览器
```bash
npm i karma -D
npm i karma-jasmine karma-chrome-launcher jasmine-core -D
```
#### package.json
```
"test": "node ./node_modules/karma/bin/karma start"
```
可以全局安装 `karma-cli` 就能直接使用 `karma`命令了。

### genrate config file
使用命令 `karma init` 生成配置文件。
```

```

## UI测试（backstopjs）

### install 
```npm i backstopjs -g```
会下载 puppeteer

### 基础命令
- ```backstop init``` 生成一个新的实例，包含配置文件。

- ```backstop test``` 测试 scenarios.url 中的页面，生成页面快照并对比 ```paths.btmaps_reference``` 中对应图片。

- ```backstop approve``` 将上一次测试生成的快照作为衡量图片，下次对比时与此快照对比。

