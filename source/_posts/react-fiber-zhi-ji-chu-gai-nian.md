---
title: 'React Fiber 基础'
date: 2020-02-16 15:30:17
tags: [js, react, fiber]
published: true
hideInList: false
feature: /post-images/react-fiber-zhi-ji-chu-gai-nian.jpg
isTop: false
---
## 前言
> React 在 15.x 版本对其核心的 reconciler 的算法进行了重构， 由此产生了 Fiber。 Fiber 解决了 React 现在存在的性能问题，使得 React 能更好的处理 state 更新和 UI 更新。

![fiber前后](https://user-gold-cdn.xitu.io/2019/10/21/16deecd21336ca41?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

之前版本的 React reconcilation 对于更新任务无法中断， 如果更新任务长时间占用主线程则会发生卡顿，无法响应用户交互。  
React 16 对 reconcilation 进行了重构， 将 DOM 更新拆分成一个个小任务， 且每个任务有各自的优先级， 高优先级的任务可以中断低优先级的任务。  

**Fiber 的核心思想就是协程以及合作调度模式， 其 fiberNode 的数据结构以及使用链表模拟函数调用栈(call stack) 都是为了实现合作调度。**

## Fiber
> Fiber 原意为协程， 协程的特点在于允许执行被挂起与被恢复，协程本身是没有并发或者并行能力的（需要配合线程），它只是一种控制流程的让出机制。
> 我们知道 CPU 的最小的分配单位为进程，最小的调度单位为线程， JS engine 是基于单线程的。以 chrome 来说， 一个 tab 页会有一个 renderer process 来负责这个 tab 页的事物处理。其中 js 线程 和 UI 渲染的线程是互斥的， 两者只能有一个同时执行。正常来说我们无法介入 renderer process 的主线程的调度，而通过协程我们则可以做到。

```js
const tasks = []
function * run() {
  let task

  while (task = tasks.shift()) {
    // 🔴 判断是否有高优先级事件需要处理, 有的话让出控制权
    if (hasHighPriorityEvent()) {
      yield
    }

    // 处理完高优先级事件后，恢复函数调用栈，继续执行...
    execute(task)
  }
}
```

## 合作调度模式
> 把渲染更新过程拆分成多个子任务，每次只做一小部分，做完看是否还有剩余时间，如果有继续下一个任务；如果没有，挂起当前任务，将时间控制权交给主线程，等主线程不忙的时候在继续执行。 

合作式调度主要就是用来分配任务的，当有更新任务来的时候，不会马上去做 Diff 操作，而是先把当前的更新送入一个 Update Queue 中，然后交给 Scheduler 去处理，Scheduler 会根据当前主线程的使用情况去处理这次 Update。为了实现这种特性，使用了requestIdelCallbackAPI。

## requestIdelCallback
> requestIdelCallback 是浏览器提供的一个 api ， 使用其包裹的函数会在浏览器每帧的空余时间执行。

![renderer process](https://note.youdao.com/yws/api/personal/file/9981C9BF2B7741CAB26BB59AE94F097F?method=download&shareKey=d59ea04f63e30e62343b0688891c0785)

## FiberNode
```js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  this.tag = tag; //定义fiber的类型。它在reconcile算法中用于确定需要完成的工作。如前所述，工作取决于React元素的类型，函数createFiberFromTypeAndProps将React元素映射到相应的fiber节点类型。在我们的应用程序中，ClickCounter组件的属性标记是1，表示ClassComponent，而span元素的属性标记是5，表示Host Component。
  this.key = key; //具有一组children的唯一标识符，可帮助React确定哪些项已更改，已添加或从列表中删除。它与此处描述的React的“list and key”功能有关。
  this.elementType = null; //调试过程中发现该值与 type 一样， 暂不知道具体作用。
  this.type = null; //定义与此fiber关联的功能或类。对于类组件，它指向构造函数；对于DOM元素，它指定HTML tag。可以使用这个字段来理解fiber节点与哪个元素相关。
  this.stateNode = null; //保存对组件的类实例，DOM节点或与fiber节点关联的其他React元素类型的引用。

  // Fiber
  this.return = null; //父节点的 fiberNode
  this.child = null; //子节点的 fiberNode
  this.sibling = null; //兄弟节点的 fiberNode
  this.index = 0;

  this.ref = null; //{current}

  this.pendingProps = pendingProps; //已从React元素中的新数据更新，并且需要应用于子组件或DOM元素的props
  this.memoizedProps = null; //在前一次渲染期间用于创建输出的props
  this.updateQueue = null; //用于状态更新，回调函数，DOM更新的队列
  this.memoizedState = null; //于创建输出的fiber状态。处理更新时，它会反映当前在屏幕上呈现的状态。
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  this.expirationTime = NoWork;
  this.childExpirationTime = NoWork;

  this.alternate = null; //current 与 alternate 相互引用
}
```
![fiber 链表](https://user-gold-cdn.xitu.io/2019/10/21/16deecc6db5530be?imageslim)

![fiber 迭代顺序](https://user-gold-cdn.xitu.io/2019/10/21/16deecca7850a24d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**使用链表模拟函数调用栈更为可控，fiber 节点的处理可以随时中断和恢复， 对于处理过程中发生异常的节点， 我们可以根据 return 回溯打印出完整的'节点栈'。**