---
title: 'Promise A+ 规范翻译及个人总结'
date: 2020-03-11 14:23:15
tags: [promise]
published: true
hideInList: false
feature: /post-images/promise-a-gui-fan-fan-yi.jpg
isTop: false
---
翻译自[原文链接](https://promisesaplus.com/)， 本文格式基本参照原文， 对于带有标号的术语，请参考对应标号的解释。
 英语水平所限， 看不懂的可以阅读原文。

# 正文
> 一个针对实现者，实现健全的，可操作的 JavaScript promise 的开放的规范。

一个 *promise* 代表了一个异步操作的最终的结果。与 promise 互动的主要方法是通过它的 `then` 方法， `then` 方法用来注册接收 promise 的终值或者为何该 promise 为什么无法完成的 reason 的回调。

此规范详细说明了 `then` 方法的行为，提供一个所有符合 Promises/A+ 规范实现的 promise 都可以依赖的可互操作的基础。 因此， 此规范需要考虑到非常稳定。虽然 Promises/A+ 组织可能会偶尔修改此规范， 对其进行一些向后兼容的小修改以解决新发现的问题，只有在进过仔细考虑、讨论和测试之后我们才会集成大型的或向后不兼容的变更。

从历史上看，Promises / A +阐明了早期Promises / A提案的行为条款，将其扩展为涵盖实际行为，并省略了未指定或有问题的部分。

最后， Promises/A+ 规范的核心不涉及如何创建、完成或者拒绝 promises ， 而是聚焦于提供可互操作的`then`方法。未来在配套规范中的工作可能会涉及这些主题。

## 1. 术语

1.1.    "promise" 是一个带有符合本规范中`then` 方法的行为的对象或者函数。
1.2.    "thenable" 是一个定义 `then` 方法的对象或者函数。
1.3.    "value" 是任意合法的 JavaScript 值， 包含 `undefined`, `thenable`,`promise`。
1.4.    "exception" 是通过 `throw` 声明语句抛出来的值。
1.5.    "reason" 是表明为何一个 promise 被拒绝的值。

## 2.  要求

### 2.1 Promise 状态

一个 promise 必须处于以下三个状态之中： pending， fulfilled， rejected 。

2.1.1.  当处于 pending 状态
        2.1.1.1.    promise 可以转变为 fulfilled 或者 rejected 状态。

2.1.2.  当处于 fulfilled 状态
        2.1.2.1.    promise 无法转变成其他状态。
        2.1.2.2.    promise 必须要有一个无法改变的 value。

2.1.3.  当处于 rejected 状态
        2.1.3.1.    promise 无法转变成其他状态。
        2.1.3.2.    promise 必须要有一个无法改变的 value。

在此处， 无法改变意味着不可变， 类似 `===` 全等， 但不意味深层次的不变。（译者： 可以理解为指针不变即可。）

### 2.2 `then` 方法

一个 promise 必须提供一个 `then` 方法去获取它当前或者最终的 value 或 reason 。
promise 的`then` 方法接收两个参数：
```js
promise.then(onFulfilled, onRejected)
```

2.2.1.  `onFulfilled` 和 `onRejected` 参数都是可选的
        2.2.1.1.     如果 `onFulfilled` 不是函数， 它将会被忽略。
        2.2.1.2.     如果 `onRejected` 不是函数， 它将会被忽略。

2.2.2.  如果 `onFulfilled` 是函数
        2.2.2.1.    在 `promise` 处于`fulfilled`状态之后， 它将被调用， 其第一个参数为 `promise` 的 value。
        2.2.2.2.    在 `promise` 处于`fulfilled`状态之前， 它无法被调用。
        2.2.2.3.    它只能被调用一次。

2.2.3.  如果 `onRejected` 是函数
         2.2.3.1.    在 `promise` 处于`rejected`状态之后， 它将被调用， 其第一个参数为 `promise` 的 reason 。
        2.2.3.2.    在 `promise` 处于`rejected`状态之前， 它无法被调用。
        2.2.3.3.    它只能被调用一次。

2.2.4.  `onFulfilled` 和 `onRejected` 函数只有当 `execution context stack ` (执行栈?执行上下文?) 只包含 `platform code `[3.1] 。

2.2.5.  `onFulfilled` 和 `onRejected`  必须以函数的方式调用(例如， 没有 `this`)[3.2]

2.2.6.  `then` 可能在同一个 promise 中调用多次
        2.2.6.1.    当 promise 处于 `fulfilled` 状态， 所有各自的`onFulfilled`回调必须按照它们的`then` 的原始顺序执行。
        2.2.6.2.    当 promise 处于 `rejected` 状态,  所有各自的`onRejected`回调必须按照它们的`then`的原始顺序执行。

2.2.7.  `then` 需要返回一个 promise [3.3]
```js
promise2 = promise1.then(onFulfilled, onRejected);
```

 2.2.7.1.    如果 `onFulfilled` 或者`onRejected` 之一返回一个 value `x`,  执行 promise resolution procedure `[[Resolve]](promise2, x)`。
 2.2.7.2.    如果 `onFulfilled` 或者`onRejected` 之一抛出一个 exception `e` , `promise2` 必须被 rejected 并使用 `e` 作为 reason。
2.2.7.3.    如果 `onFulfilled` 不是一个函数， 并且 `promise1` 处于 `fulfilled` 状态， `promise2` 必须被置为 `fulfilled` 状态， 并且使用 `promise1` 的 `value` 作为 `promise2` 的 `value`。
2.2.7.4.    如果 `onRejected`不是一个函数， 并且 `promise1` 处于 `rejected` 状态， `promise2` 必须被置为 `rejected` 状态， 并且使用 `promise1` 的 `reason` 作为 `promise2` 的 `reason`。

### 2.3 The Promise Resolution Procedure (Promise 解决程序)
The promise resolution procedure 是一个抽象操作， 接受一个 promise 和 一个 value 作为输入，表示为 `[[Resolve]](promise, x)` 。 如果 `x` 是一个 thenable , 假设 `x` 的行为至少在某种程度上与 promise 类似，它将会尝试使用 `x` 的状态作为 `promise` 的状态。
否则， 它将会使用 value `x` 将 `promise` 置为 `fulfilled` 状态。

只要它们暴露一个基于 Promises/A+规范实现的 `then` 方法， 对于 thenables 的处理就允许 promise 互操作的实现。 它也允许 Promises/A+ 同化使用 reasonable `then` 方式的不合格的实现。

执行 `[[Resolve]](promise, x)` 会执行以下步骤：

2.3.1.  如果 `promise` 和 `x` 指向同一个对象， `promise` 使用 `TypeError` 作为 reason， 并置为 `rejected` 状态。 (new Promise 里面必须调用 resolve 或者 reject， 如何将 promise 和 x 指向同一个对象？)

2.3.2.  如果 `x` 是一个 promise， 采用`x`的状态[3.4]。
        2.3.2.1.    如果 `x` 处于 pending 状态， `promise` 必须处于 pending
        状态直到 `x` 状态改变。
         2.3.2.2.    如果 `x` 处于 fulfilled 状态, 使用 `x` 的 value 将`promise` 置为 fulfilled 状态。
        2.3.2.3.     如果 `x` 处于 rejected 状态, 使用 `x` 的 reason 将`promise` 置为 rejected 状态。

2.3.3.  如果 `x` 是一个对象或者函数
        2.3.3.1.    让 `then` 变成 `x.then`. [3.5]
        2.3.3.2.    如果查找 `x.then` 属性造成抛出异常 `e`, 将 promise 置为`rejected` 状态并使用 `e` 作为 reason。
        2.3.3.3.    如果 `then` 是一个函数， 会调用该函数，并使用使用 `x`作为 this 。该函数有两参数， 第一个参数为 `resolvePromise`, 第二个参数为`rejectPromise`
                    2.3.3.3.1.  如果 `resolvePromise` 使用 value `y` 调用， 执行 `[[Resolve]](promise, y)`。
                    2.3.3.3.2.  如果 `rejectPromise ` 使用 reason `r` 调用， 使用 `r` 将 promise 状态置为 `rejected`。
                    2.3.3.3.3.  如果 `resolvePromise` 和`rejectPromise`都被调用， 或者多次调用了相同参数， 第一次调用会执行， 其他的会被忽略。
                    2.3.3.3.4.  如果执行的 then 抛出异常 `e`
                                    2.3.3.3.4.1.    如果  `resolvePromise` 或`rejectPromise`被调用， 忽略它。
                                    2.3.3.3.4.2.    使用 `e` 将 promise 置为 `rejected` 状态。
        2.3.3.4.    如果 then  不是函数， 使用 `x` 将 promise 置为 `fulfilled` 状态。
 
 2.3.4.     如果`x`不是对象或者函数， 使用 `x` 将 promise 置为 `fulfilled` 状态。


 如果 promise resolved 了一个 thenable 对象， 该 thenable 对象参与到了循环的 thenable 链， 例如 `[[Resolve]](promise, thenable)` 的递归性质最终会导致 `[[Resolve]](promise, thenable)` 被再次调用， 最终会导致无限递归。我们鼓励检测这样的递归并且使用一个带有信息的`TypeError` reason 将 promise 置为`rejected`状态， 但此实现不是必须的。

 ### 3. Notes

 3.1.    此处的 “platform code”代表引擎、环境和 promise 实现的代码。实际上， 这个要求确保在事件循环中轮到 then 调用之后，`onFulfilled` 和 `onRejected` 能在最新的执行栈下异步执行。这个可以通过使用宏任务方法例如 `setTimeout` `setImmediate` 或者 微任务方法例如`MutationObserver` `process.nextTick` 实现。由于 promise 实现被认为是“platform code”, 它自己有可能会包含一个任务调度队列或者 “trampoline” , 在这个队列里面调用处理程序。

 3.2.   就是说， 在严格模式下 `this` 在里面会指向 undefined , 在宽松模式下， 它将指向全局对象。

 3.3.   如果实现满足了所有要求， 实现会允许出现 `promise2===promise1` 的情况。 每个实现应该表明是否会产生`promise2 === promise1`的现象， 以及在什么情况下会出现该情况。

 3.4.   通常来说， 只有当 `x` 来自当前的实现时才能知道 `x` 是一个真的 promise 。此条款允许特定实现方式的使用可以采用已知符合的 promise 状态。(译者： 使用 x 的状态来代替外层 promise 的状态)

 3.5.   程序首先存储对 `x.then` 的应用， 然后测试该引用， 然后调用该引用， 防止多次访问 `x.then` 属性。 此类预防措施对于确保访问者属性的一致性非常重要，因为访问者属性的值在两次检索之间可能会发生变化。

 3.6.   实现中不应该对 thenable 链的深度设置任意限制， 认为当超过该限制时递归就是无限的。 只有真正的循环才会导致 TypeError 。如果遇到多个不同的 thenables 组成的链， 递归永远是正确的行为。


 ## 总结
 > 需要重点注意的是 [[Resolve]](promise,x) 对于 x 不同类型执行不同流程。
 -  Promises/A+ 规范的核心在于提供 then 方法。
 -  promise 是一个带有符合规范的`then`方法的对象或者函数。
 -  promise 有三个状态: `pending`, `fulfilled`,`rejected` ， 状态一旦改变就无法更改。
 -  promise.then(onFulfilled,onRejected) 中 onFulfilled，onRejected 如果不是函数会被忽略。
 -  then 方法返回一个 promise。
 -  promise2 = promise1.then(onFulfilled, onRejected) 中根据 onFulfilled，onRejected 返回的 value ， reason 更改 promise2 的状态并使用对应的 value， reason 作为返回值。
 -  [[Resolve]](promise,x) 
    - 如果 x 是一个 promise ， 使用 x 的状态代替 promise 的状态；
    - 如果 x 不是一个对象或者函数， 使用 x 将 promise 置为 fullfiled 状态。
    -  如果 x 是一个对象或者函数， 
        - 如果 x.then 不是函数， 使用 x 将 promise 置为 fulfilled 状态。
        -  尝试查找 x.then， 如果查找报错使用该错误 rejected， 
        -  如果 x.then 是一个函数，会使用 x 作为 this 调用 x.then ，
            - x.then(resolvePromise, rejectPromise), x.then 接受所示两个参数， 如果调用 resolvePromise(y) 则执行 [[Resolve]](promise,y) ；如果调用 rejectPromise(r), 使用 reason r 将 promise 状态置为 rejected 。

