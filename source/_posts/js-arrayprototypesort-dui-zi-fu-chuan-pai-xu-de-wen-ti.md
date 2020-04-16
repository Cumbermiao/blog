---
title: 'JS Array.prototype.sort 对字符串排序的问题'
date: 2020-02-22 11:44:27
tags: [Array,js,sort,string]
published: true
hideInList: false
feature: /post-images/js-arrayprototypesort-dui-zi-fu-chuan-pai-xu-de-wen-ti.jpg
isTop: false
---
## 起因
今天刷 leetcode 中遇到一个场景需要对字符数组进行排序， 书写是按照之前常用的写法 `arr.sort((a,b)=>a-b)` , 但是其排序结果跟未排序之前一样。

## 描述
**sort 语法**：`arr.sort([compareFunction])`。
`compareFunction` 参数可选，其作用是按照指定的某种顺序进行排列。如果省略， 元素按照==转换为的字符串的各个字符的 Unicode 位点进行排序==。
其中 `compareFunction`还接受两个参数`firstEle`表示第一个用于比较的元素， `secondEle`表示第二个比较的元素。
**sort 返回值** : 原地排序后的数组。  

## 实际结果
> 从上述描述中可知，`sort`如果不指定 `compareFunction` 比较函数， 本就会将元素转成字符串并比较 Unicode 编码大小。
按照上面的结论，解决起因中的问题直接去掉 `compareFunction` 即可, 实际结果也是对的。
```js
['b','a','f','c'].sort();
//['a','b','c','f']
```
再分析下我其中代码的逻辑, 在 `(a,b)=>a-b` 中我期望按照升序的方式排序， 由于 a,b 是字符，所以我期望 `a-b` 计算两者的 Unicode 编码，但实际上结果是 `NaN`。
这实际上是与比较操作符的规则弄混了， 对于比较操作符如果两个操作符是字符串时，比较的是其 Unicode 编码， 对于 `-` 操作符如果有一个操作符是字符串， 会先调用 `Number()` 方法将其转成数字类型进行比较， 而对于非数字的字符串`Number()`转换后的结果都是 `NaN`。

## 结论
`sort` 方法如果不提供 `compareFunction` 参数，默认会将元素转成字符串，比较其 Unicode 编码大小。
`arr.sort((a,b)=>a-b)` 在比较字符串时， 根据 `-`操作符的规则， 会先调用 `Number()`, 对于无法转换成数字的字符结果为 `NaN`, 最终排序后数组不会改变。想要获得预期结果，可以利**比较操作符**的规则。
```js
arr.sort((a,b)=>{
    if(a>b)return 1
    return -1
})
```
`-` 操作符的转换规则：
- 如果有一个操作数是 NaN，则结果是 NaN； 
- 如果是 Infinity 减 Infinity，或者 -Infinity 减-Infinity ，则结果是 NaN； 
- 如果是 Infinity 减-Infinity，则结果是 Infinity； 
- 如果是-Infinity 减 Infinity，则结果是-Infinity； 
- 如果有一个操作数是字符串、布尔值、 null 或 undefined，则先在后台调用 Number()函数将其转换为数值，然后再根据前面的规则执行减法计算。如果转换的结果是 NaN，则减法的结果就是 NaN； 
- 如果有一个操作数是对象，则调用对象的 valueOf()方法以取得表示该对象的数值。如果得到的值是 NaN，则减法的结果就是 NaN。如果对象没有 valueOf()方法，则调用其 toString()方法并将得到的字符串转换为数值。 
