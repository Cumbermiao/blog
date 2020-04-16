---
title: 'new Array 与 map 的问题'
date: 2020-02-13 09:44:52
tags: [js,Array,Sparse Array]
published: true
hideInList: false
feature: /post-images/new-array-yu-map-de-wen-ti.jpg
isTop: false
---

## 起因
今天在写代码时， 想生成一个元素为1~n的数组，当时的操作是这样子的：
```js
let eles = new Array(9)
eles.map((item,idx)=>idx+1)
```
结果 eles 数组并没有如期望的 [1,2...9] ,而仍是 [empty,empty,...] , 随后我就使用 `eles.map((item,idx)=>console.log(idx))` 来查看 map 中的回调是否调用， 结果是没有调用。

## 描述
遇到这个奇怪的问题， 我查看了 MDN 中 map 的描述， 发现其中有这么一段：
> callback is invoked only for indexes of the array which have assigned values (including undefined).    


callback 函数只会在有值的索引上被调用，具体不被调用的情况如下：
- 没有设置索引
- 索引被删除
- 索引上未设值
MDN 上对符合以上条件的数组称为 *稀疏数组*

## 实际结果
> 按照以上的说明，我进行了以下的用例测试。

1. 使用 `new Array` 创建空数组， 结果 callback 未调用。
```js
let eles = new Array(9)
eles.map((item=>console.log(item))
```
2. 使用 `[,,,]` 创建空数组， 结果 callback 未调用。
```js
let eles = [,,,]
eles.map((item=>console.log(item))
```
3. 使用 `[undefined,undefined,undefined]` 创建数组， 结果 callback 调用。
 ```js
let eles = [undefined,undefined,undefined]
eles.map((item=>console.log(item))
```
4. 使用 `[,undefined,undefined]` 创建数组， 结果不为 empty 的元素被调用 callback。
 ```js
let eles = [,undefined,undefined]
eles.map((item=>console.log(item))
//undefined *2
```
5. 使用 `delete` 删除元素, 被删除索引的值变为 empty， 该索引不会调用 callback。
```js
let eles = [1,2,3]
delete eles[0]
eles.map((item=>console.log(item))
//2,3
```
## 结论
> 如同上面描述所说 map 方法对于稀疏数组中元素为 empty 的不会调用 callback。   

与 map 表现一致的的数组方法还有： `every` , `filter`, `flat`, `flatMap`, `forEach`, `reduce`, `reduceRight`, `some`。

会对所有元素调用的数组方法有： `find`, `findIndex`。

对于有需要初始化数组元素可以使用 `fill` 方法， 或者使用 `Array.apply(null,Array(9))` 的方式。