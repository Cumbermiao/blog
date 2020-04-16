---
title: 'React Hooks'
date: 2020-03-16 23:41:13
tags: [react,hooks]
published: true
hideInList: false
feature: /post-images/react-hooks.jpg
isTop: false
---
参考文章：
- [React Hooks 原理](https://github.com/brickspert/blog/issues/26)
- [react hooks 实现原理](https://juejin.im/post/5c4c43f5e51d45518c681a69)


## hooks api 类型
> Hooks 主要分为以下三种：
- State hooks： 可以让函数式组件使用 state
- Effect hooks： 可以让函数式组件使用生命周期和 side effect， 如 useEffect 可以让组件具备 life-cycles。
- Custom hooks： 自定义 hooks
  

## 数据结构

### FiberNode
```js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;// 可能是 FiberRootNode、dom、FiberNode

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null; //在使用 hooks 是此处指向的是 Hook 对象
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.effectTag = NoEffect;
  this.nextEffect = null;
  this.firstEffect = null;
  this.lastEffect = null;

  this.expirationTime = NoWork;
  this.childExpirationTime = NoWork;
  this.alternate = null;
```

### Hook
```js
export type Hook = {
  memoizedState: any, //存储 state 变量的地方
  baseState: any, //基于此状态进行更新
  baseUpdate: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null, //存放  hooks 执行的回调
  next: Hook | null,
};
```

### Effect
```js
type Effect = {
  tag: HookEffectTag, //当前 hook 作用的生命周期
  create: () => (() => void) | void,
  destroy: (() => void) | void,
  deps: Array<mixed> | null,
  next: Effect,
};
```

### Update
```js
type Update<S, A> = {
  expirationTime: ExpirationTime,
  suspenseConfig: null | SuspenseConfig,
  action: A,
  eagerReducer: ((S, A) => S) | null,
  eagerState: S | null,
  next: Update<S, A> | null,

  priority?: ReactPriorityLevel,
};
```