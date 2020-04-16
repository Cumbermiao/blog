---
title: 'React DOM Diff'
date: 2020-03-15 18:49:32
tags: [react,diff]
published: true
hideInList: false
feature: /post-images/react-dom-diff.jpg
isTop: false
---
参考文章：
- [官方： reconciliation](https://zh-hans.reactjs.org/docs/reconciliation.html#the-diffing-algorithm)
- [Deep In React 之详谈 React 16 Diff 策略(二)](https://juejin.im/post/5d3e3231e51d4510926a7c39#comment)
- [深入React fiber架构及源码](https://zhuanlan.zhihu.com/p/57346388)
- [Inside Fiber: in-depth overview of the new reconciliation algorithm in React](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e)
> 首先开头需要明确 dom diff 的目的： 复用节点， 减少 dom 方面的开销。

## Diff 前提策略
1. Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计。
2. 拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构。
3. 对于同一层级的一组子节点，它们可以通过唯一 id 进行区分。

## Diff 过程
> React 将渲染分为两个阶段， render 和 commit 。 在 render 阶段中会构建 workInProgress tree ， 并且会得到一个 effect list， 这个 effect list 就是在 commit 阶段需要处理的节点。

### side effects
> You’ve likely performed data fetching, subscriptions, or manually changing the DOM from React components before. We call these operations “side effects” (or “effects” for short) because they can affect other components and can’t be done during rendering.
在 react 文档中将请求、订阅、操作 DOM 的这些操作叫做 "side effects" , 或者简单称为 "effects" 。
大部分 state 和 props 更新的操作都会导致 effects ，  React 在 fiber 节点中使用 effectTag 字段来记录对应的 effects 操作。

### effectTag 
> 对应上面的的 side effects ， 每个 effect 操作类型在 react 都有对应的变量表示。
```js
// Don't change these two values. They're used by React Dev Tools.
var NoEffect = /*              */0;
var PerformedWork = /*         */1;

// You can change the rest (and add more).
var Placement = /*             */2;
var Update = /*                */4;
var PlacementAndUpdate = /*    */6;
var Deletion = /*              */8;
var ContentReset = /*          */16;
var Callback = /*              */32;
var DidCapture = /*            */64;
var Ref = /*                   */128;
var Snapshot = /*              */256;
var Passive = /*               */512;

// Passive & Update & Callback & Ref & Snapshot
var LifecycleEffectMask = /*   */932;

// Union of all host effects
var HostEffectMask = /*        */1023;

var Incomplete = /*            */1024;
var ShouldCapture = /*         */2048;
```
- NoEffect:：effectTag 初始值， 表示 NoWork。
- PerformedWork：react devtools 使用。
- Placement：插入新的子节点。
- Update：当 props、state发生改变会标记为 update，在执行 commitUpdate 函数时进行属性更新。会调用对应的生命周期函数。
- Deletion：标记将要删除的节点。
- ContentReset：当从文本域节点切换到非文本域或空节点时，打上此标记，将文本内容进行重置，文本域节点包括textarea、option、noscript、string、number和直接在标签中写入的__html。当检测到标记后，执行commitResetTextContent函数将对应节点到text清空。
- Callback：当setState、forceUpdate有callback函数，或者在Commit阶段捕获到错误时，会更新update.callback，并标记Callback，随后检测到标记后会触发commitLifeCycles函数，根据不同到组件类型进行不同的commit。
- DidCapture：针对于懒加载的React.Suspense（SuspenseComponent）组件提供的标志位，DidCapture位置位表示要渲染的组件被挂起，进而先渲染fallback的内容。
- ShouldCapture：标记是否需要将节点挂起，一般捕获边界错误或者超时会置位，随后用于判断是否进行DidCapture。
- Ref：当节点中存在属性ref时，会进行markRef当标记，随后会在commitAllLifeCycles阶段执行commitAttachRef触发相应当ref回调函数。
- Snapshot：在渲染更新之前，当前后当props或state发生变化时，触 getSnapshotBeforeUpdate生命周期钩子。

### effects list
> effects list 是一个链表式的结构， 将所有需要更新的子 fiber 节点连接起来。
在遍历 fiber tree 时采用的时 DFS 遍历算法， 所以正常来说 effects list 中的节点应该也是按照自下而上的方式连接的。
在 fiber 节点中对应的有这么几个字段来维护自己的 effects list ： firstEffect、lastEffect、nextEffect。

### 详细过程解析

#### reconcileChildren
> react diff 从 reconcileChildren 函数开始， 首次渲染调用时, current 为 null ， 此处我们只关心 reconcileChildFibers 函数。
```js
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderExpirationTime: ExpirationTime,
) {
  if (current === null) {
    // If this is a fresh new component that hasn't been rendered yet, we
    // won't update its child set by applying minimal side-effects. Instead,
    // we will add them all to the child before it gets rendered. That means
    // we can optimize this reconciliation pass by not tracking side-effects.
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderExpirationTime,
    );
  } else {
    // If the current child is the same as the work in progress, it means that
    // we haven't yet started any work on these children. Therefore, we use
    // the clone algorithm to create a copy of all the current children.

    // If we had any progressed work already, that is invalid at this point so
    // let's throw it out.
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderExpirationTime,
    );
  }
}
```
#### reconcileChildFibers
> 该函数接受四个参数，返回结果是 Fiber 或者 null。
- returnFiber<Fiber>： diff 开始的容器节点。
- currentFirstChild<Fiber | null>：容器节点的第一个子节点。
- newChild<any>：即将更新的 vdom 节点，可能是 TextNode、ReactElement、数组， 不是 fiber 节点。
- expirationTime<ExpirationTime>：调度任务需要使用的参数。

#### 1. Diff Single TextNode
```js
//1
return (
    <div>
        <div>
            <aaa></aaa>
            <bbb></bbb>
        </div>
    </div>
)

//2
return (
    <div>
        <div>
            文字节点
        </div>
    </div>
)
```
如上代码所示， 从 1 状态转变到 2 状态，对于 1 中来说 aaa bbb 可以为任何类型， 2中变成的 TextNode 节点。 此时 Diff 该如何操作呢？
react 会判断 currentFirstChild 的 tag 是否为 TextNode 类型， 如果 aaa 是 TextNode 类型， 那么会复用该节点， 并且将后面所有的兄弟节点 bbb 打上 delete 标签。
如果 aaa 不是 TextNode 类型， 那么 react 会删除所有的子节点即 aaa bbb 都会被删除， 然后重新创建一个 TextNode 节点。
对应源码如下
```js
function reconcileSingleTextNode(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    textContent: string,
    expirationTime: ExpirationTime,
  ): Fiber {
    // There's no need to check for keys on text nodes since we don't have a
    // way to define them.
    if (currentFirstChild !== null && currentFirstChild.tag === HostText) {
      // We already have an existing node so let's just update it and delete
      // the rest.
      deleteRemainingChildren(returnFiber, currentFirstChild.sibling);
      const existing = useFiber(currentFirstChild, textContent, expirationTime);
      existing.return = returnFiber;
      return existing;
    }
    // The existing first child is not a text node so we need to create one
    // and delete the existing ones.
    deleteRemainingChildren(returnFiber, currentFirstChild);
    const created = createFiberFromText(
      textContent,
      returnFiber.mode,
      expirationTime,
    );
    created.return = returnFiber;
    return created;
  }
  ```

#### 2. Diff Single React Element
与 TextNode 类似， react 首先会判断容器节点的子节点中是否存在 key 与 newChild 相同的节点， 如果有并且 tag 也一样就复用该节点， 并删掉剩余节点。如果 key 一样， 但是 tag 不一样， 那么删掉所有节点， 无法复用。
如果子节点的 key 不一样， 则删掉当前的子节点， 并继续遍历下个 sibling 节点。
此时可能还会出现一个情况， 就是 newChild 和 child 节点都没有 key ， key 都为 null 。 那么此时会走到 tag 判断的地方， 如果两者的 tag 一样， 那么仍然会重用该节点。注释中也说明该情况只有 list 里面的第一个元素会走该流程， 因为如果第一个元素的 tag 不匹配的话， 会删除所有的 sibling 节点。
```js
function reconcileSingleElement(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    element: ReactElement,
    expirationTime: ExpirationTime,
  ): Fiber {
    const key = element.key;
    let child = currentFirstChild;
    while (child !== null) {
      // TODO: If key === null and child.key === null, then this only applies to
      // the first item in the list.
      if (child.key === key) {
        if (
          child.tag === Fragment
            ? element.type === REACT_FRAGMENT_TYPE
            : child.elementType === element.type ||
              // Keep this check inline so it only runs on the false path:
              (__DEV__
                ? isCompatibleFamilyForHotReloading(child, element)
                : false)
        ) {
          deleteRemainingChildren(returnFiber, child.sibling);
          const existing = useFiber(
            child,
            element.type === REACT_FRAGMENT_TYPE
              ? element.props.children
              : element.props,
            expirationTime,
          );
          existing.ref = coerceRef(returnFiber, child, element);
          existing.return = returnFiber;
          if (__DEV__) {
            existing._debugSource = element._source;
            existing._debugOwner = element._owner;
          }
          return existing;
        } else {
          deleteRemainingChildren(returnFiber, child);
          break;
        }
      } else {
        deleteChild(returnFiber, child);
      }
      child = child.sibling;
    }

    if (element.type === REACT_FRAGMENT_TYPE) {
      const created = createFiberFromFragment(
        element.props.children,
        returnFiber.mode,
        expirationTime,
        element.key,
      );
      created.return = returnFiber;
      return created;
    } else {
      const created = createFiberFromElement(
        element,
        returnFiber.mode,
        expirationTime,
      );
      created.ref = coerceRef(returnFiber, currentFirstChild, element);
      created.return = returnFiber;
      return created;
    }
  }
  ```

  #### 3. Diff Array
  对于 newChild 是数组类型来说， 情况相对比较复杂， 源码也比较长， 此处就不放了。下面分成几个情况来解析。

##### updateSlot
updateSlot 接受四个参数， 其中 oldFiber 是 reconcileChildrenArray 传入的 sibling 节点， newChild 是当前遍历的 newChildren 的元素。
函数中首先会判断 newChild 是不是 TextNode 节点， 然后判断 oldFiber 中是否存在 key ， 如果存在 key 那么 return null ，否则进入 updateTextNode 函数。 因为 TextNode 节点不会存在 key。
如果 newChild 是 react element 则会判断 key 是否一致， 一致的时候则进入 update， 只不过此处需要判断 newChild.type 是不是 fragment ， 如果是 fragment 的话 update 传递的是 newChild.props.children。
update 里面会判断是否能重用， 判断的条件与前面解释的过程一样，不能重用的话就会床架你新的节点。
对于 updateSlot 来说只要是无法重用的情况下都会 return null。

##### reconcileChildrenArray
>  Diff Array 都会进入 reconcileChildrenArray 函数， 在该函数中， react 会取出容器节点的第一个节点和 newChild 的第一个元素。 之后会遍历 newChildren 并且每次都会进入 updateSlot 函数中。
如果 updateSlot 返回 null 的话说明当前的 oldFiber 无法重用， newChildren 的遍历会被终止。此时的场景就是可能 sibling 的顺序被移动了， 例如第一个 sibling 被移到了最后一个。 此时 react 使用所有的从当前 sibling 节点时候后面所有节点会生成一个 existingChildren 的 map ， key 为节点的key ， value 则为节点本身。之后会继续之前中断的 newChildren 遍历， 从 existingChildren 查找是否有重用的节点， 具体逻辑在 updateFromMap 中， 基本上就是有没有可复用的节点， 没有就重新创建，否则重用。

总和以上分析， 对于 Diff Array 的情况可以分为以下几种：
1. 新增节点， 前面相同的节点全部复用， 新增的节点全部新创建。
2. 节点顺序不变， 只修改节点属性，全部节点复用。
3. 节点顺序改变， 所有节点复用。 具体逻辑： 前面顺序未变的在第一次遍历时可以全部复用， 在发现 oldFiber 和 newChild key 不一致的情况时， 会生成一个 map ， 在 map 找有没有可以复用的节点，没有则新增。

