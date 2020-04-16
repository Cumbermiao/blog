---
title: 'JS 中闭包的应用'
date: 2020-02-13 12:22:20
tags: [js,闭包]
published: true
hideInList: false
feature: /post-images/js-zhong-bi-bao-de-ying-yong.jpg
isTop: false
---
> 闭包是 JS 中非常常用的手段， 本篇文章根据闭包的特点来总结其日常应用。

## 闭包的概念
> 闭包是词法作用域和书写代码所产生的自然结果， 其表现形式在于函数在其定义以外的词法环境执行，但是函数仍然能够访问其定义的词法环境。 简单来说，外层函数执行之后返回内部函数， 内部函数在其它词法环境执行时仍然能够访问外层函数中所定义的变量。

就如上概念所说， 闭包最主要的特点就是能够访问外层函数的变量， 下面就根据该特点来分析其应用。

## 闭包的应用


<!-- 
TODO: 模块化
### 模块化的前身
> js 早期是没有模块化的概念， 而使用闭包可以解决全局变量污染的问题。

```js
var  a = ()
``` -->

### 请求的并发控制
> 我们知道， 浏览器的 http 请求是有并发限制的， 过多的 http 请求会占用过多的浏览器资源， 所以并发控制是常用的场景。

实现一个  maxFetch 函数,  使用 fetch 请求， 不考虑 fetch 的具体实现。
```js
  function generateMaxFetch(max=4){
    let urls = []
    function parcelFetch(){
      if(urls.length&&max>0){
        let url = urls.shift()
        max--;
        mockFetch(url).then(res=>{
          console.log(res)
          max++;
          parcelFetch()
        })
        .catch(err=>{
          throw new Error(err)
        })
      }
    }
    return function(url){
      urls.push(url)
      parcelFetch()
    }
  }
  const maxFetch = generateMaxFetch(3)
  maxFetch('/111')
  maxFetch('/222')
  maxFetch('/333')
  maxFetch('/444')
  maxFetch('/555')
  maxFetch('/666')
  maxFetch('/777')
```

### 单例模式
> 在一些业务场景中需要保证全局只有一个实例对象， 如登录框等， 借助闭包很容易实现。
假设 artDialog 有 init 方法用来初始化弹框， hide 方法隐藏登录框， show 方法显示弹框, 实现一个全局唯一的登录框。
```js
const loginDialog = (function(){
    let instance = null;
    return function(){
        if(instance==null)instance = artDialog.init();
        instance.show()
        return  instance.hide
    }   
})()

let hideLogin = loginDialog();
hideLogin();
```

### debounce & throttle
> 对于用户可能频繁触发的事件，我们可以使用防抖或者节流的方式来优化性能。
实现节流和防抖， 注意要保持用户最后一次触发事件的结果的一致性。
```js
const debounce = function(fn,delay){
    let timer = null;
    
    return function(){
        if(timer)clearTimeout(timer)
        timer = setTimeout(()=>{
            fn.apply(null,arguments)
        },delay)
    }
}

const throttle = function (fn, duration) {
      var canRun = true;
      var timer = null;
      var lastFn = null;

      const wrap = function(fn,args){
        if(canRun){
          canRun = false
          fn.apply(this,args&&[...args])
          timer = setTimeout(()=>{
            canRun = true;
            timer = null;
            lastFn&&wrap(lastFn)
            lastFn = null
          },duration)
        }else{
          lastFn = fn.bind(this,...args)
        }
      }
      return function () {
        const args = arguments
        wrap(fn,args)
      }
    }
```
### 柯里化函数
> 柯里化函数允许函数的参数分多次传递， 其中也使用了闭包。

#### 闭包的错误使用
在之前实现柯里化函数错误使用了闭包， 具体实现如下。
```js
//考虑参数中带有占位符 _
    const _ = { type: '@@/placeholder' }
    const isPlaceholder = arg => Object.prototype.toString.call(arg) === '[object Object]' && arg === _;
    const hasPlaceholder = function (args) {
      return Array.prototype.some.call(args, item => isPlaceholder(item))
    }
    const paramsIsReady = (params, expectedLen) => {
      // console.log(params.length >= expectedLen)
      // console.log(hasPlaceholder(params))
      return params.length >= expectedLen && !hasPlaceholder(params)
    }
    const insertParam = (params, insert) => {
      if (!hasPlaceholder(params)) return params.concat(insert)
      const res = params.slice(0)
      let idx = 0, param = Array.isArray(insert) ? insert.shift() : insert
      while (idx < res.length) {
        if (!isPlaceholder(res[idx])) {
          idx++
          continue
        }
        res[idx]=param
        param = Array.isArray(insert) ? insert.shift() : undefined
        idx++
      }
      if(Array.isArray(insert)&&insert.length){res = res.concat(insert)}
      return res
    }
    function curry2(fn) {
      let params = new Array(fn.length).fill(_)
      return function recursive(...args) {
        params = insertParam(params,args)
        // console.log(paramsIsReady(params,fn.length))
        if (paramsIsReady(params,fn.length)) {
          return fn.apply(null, params)
        } else {
          return recursive
        }
      }
    }
```
以上代码问题在于 curry2 中的 params ,  上面写法的问题在于同一个经过 curry2 处理过后的函数都能访问到同一个 params， 具体查看下面的测试用例。
```js
const sum = (x,y,z)=>return x+y+z
const curSum = curry2(sum)
let a = curSum(1,2)
let b = curSum(2,3)
console.log(a(3))
console.log(b(4))
```
如以上代码所示， 其中 a,b 访问的都是 curSum 中的 params ， 所以最终的输出结果必然与期望不一致。 

#### 修复错误闭包问题
> 要解决以上问题也是很简单， 只要将 fn 函数所需要的的参数都放入到返回的 recursive 中即可。 

```js
function curry2(fn) {
      return function recursive(...args) {
        if (paramsIsReady(args, fn.length)) {
          return fn.apply(null, args)
        } else {
          return function(...restArgs){
            let params = insertParam(args,restArgs)
            return recursive(...params)
          }
        }
      }
    }
```

## 总结
> 闭包作为 js 中重要手段， 其使用也是千变万化， 以上只是简单总结了几个常用的场景， 后面遇到其他闭包的场景也会持续更新。