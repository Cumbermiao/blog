---
title: 浏览器的进程与线程
date: 2019-11-12 16:03:47
tags:
---

<!-- UNDONE: 参考文章中有些地方矛盾， html parse 和 style calc 是否同步执行？ main thread 的详细过程？  是否存在 paint 步骤还是直接调用 compositor thread -->
[inside browser 系列](https://developers.google.com/web/updates/2018/09/inside-browser-part2)

[Frame 解析](https://aerotwist.com/blog/the-anatomy-of-a-frame/)

[js 主线程](https://medium.com/@francesco_rizzi/javascript-main-thread-dissected-43c85fce7e23), 在阅读此文章前需要阅读 [event loop](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5) ,了解下相关概念。

[可以部分参考](https://segmentfault.com/a/1190000012925872)

[页面绘制流程](https://frarizzi.science/journal/web-engineering/browser-rendering-queue-in-depth)


## 进程与线程
> 牢记一个进程可以由多个线程组成。
进程是 cpu 资源分配的最小单位，可以拥有内存和独立运行的最小单位。 可以理解为一个工厂。

线程是 cpu 调度的最小单位， 即给工厂打工的工人。 cpu 可以调度工人给工厂工作，  且同一个工厂的工厂在同一块区域，即共享资源。

## 浏览器的多进程
> 浏览器是基于多进程的， 每个 tab 页或者插件都是一个单独的进程， 除此之外还有主进程、 GPU 进程等，此处只解释主要的几个进程。

- browser 进程： 该进程是浏览器的主进程，处理除了 tab 页之外的功能， 如：地址栏、标签、前进后退、网络资源管理等。

- renderer 进程: 默认每个 tab 页面一个进程， 负责该 tab 页所有展示的内容。

- GPU 进程： 用于处理 GPU 任务。

- plugin 进程： 处理插件相关的任务， 如 flash 

- extension 进程： 处理谷歌插件的任务。

chrome 浏览器对每个 tab 页都申请一个单独的 render process ， 好处在于某个 tab 的 render process 崩溃时不会影响其他的 tab 页， 甚至 chrome 现在对每个 iframe 都申请了一个 render process。
坏处就在于 tab 页一多， 那么浏览器就会占据较大的内存资源。

browser process 拥有 UI thread 用于绘制 button 等元素， network thread 用于请求资源， storage thread 控制文件读写权限。

## 简单分析一个简单请求的过程
从浏览器地址栏输入 url ， 浏览器请求获取数据并且展示在页面上， 这个过程调用了哪些线程？

首先当你输入 url 的时候， UI thread 会判断这个是一个 搜索问题还是一个 url ， 如果是搜索问题则请求搜索引擎否则请求 url。

确认之后，  UI thread 会发起一个网络请求， 此时 network thread 会使用对应的协议建立请求。此时， netowrk thread 可能会收到 301 中状态码， 此时会告诉 UI thread 资源被重定向， UI thread 重新发起请求。

network thread 得到响应之后会根据响应数据的类型进行不同的处理， 如果响应的数据是 html 文件， network thread 会将 html 数据传递给 renderer process 。如果是 zip 文件这种就会传递数据给下载管理器。如果不符合浏览器的安全策略，如发生了跨域此时就会报错，不需要调用 renderer process。

如果 network thread 认为浏览器需要跳转至对应的页面， 它会告诉 UI thread 数据已经准备好， UI thread 会找一个 renderer process 并传递数据。
在 network thread 请求数据的同时， UI thread 就已经开始请求 renderer process 了， 在 network thread 告诉 UI thread 数据已经获取时， renderer process 已经待命了。

由于 UI thread 是属于 browser process 的， 与 renderer process 通信需要通过 IPC ， 使用 stream 的方式传递 html 数据。 一旦 browser process 接收到 renderer process 已经 commit 的确认之后结束导航阶段， 开始文档加载阶段。
此时，地址栏会更新，展示出新页面的网页信息。history tab 会更新，可通过返回键返回导航来的页面，为了让关闭 tab 或者窗口后便于恢复，这些信息会存放在硬盘中。。

当 renderer process 渲染页面结束之后， 它会通过 IPC 通知 browser process 结束loading，此处也是 onload 事件发生的地方。

至此，整个流程结束。

## renderer process
> renderer process 负责对应 tab 页的渲染， 一般会包含 a main thread, a raster thread, a compositor thread, 如果使用了 web worker 或者 service worker ， 还会有 worker threads .

![threads in renderer](https://note.youdao.com/yws/api/personal/file/9981C9BF2B7741CAB26BB59AE94F097F?method=download&shareKey=d59ea04f63e30e62343b0688891c0785)

上图展示了一帧的步骤， 有的步骤不是每帧都会执行。


### renderer process 处理过程
1. renderer process 在接收 html 数据时， main thread 将 html 字符串解析成 DOM。

2. 如果在解析 html 元素时遇到了 img、link 标签时，preload scanner 会在 html 解析器解析到 tokens 时， 就告诉 browser process 的 network thread 请求对应资源。

3. 如果 html 解析器遇到了 script 标签， main thread 会停止解析 html ， 去加载解析并执行 js 代码。 因为 js 可能会改变 DOM 结构，如果 js 改变了 DOM 结构，那么就需要在 js 之后重新构成 DOM 结构。对于不改变 DOM 结构的 js 我们可以使用 defer 或者 async 属性。

4. main thread 解析 css ， 计算每个 node 节点的样式。

5. 通过遍历 DOM 和每个元素的样式计算， 会得出 DOM 树和 css 树。

6. 使用 DOM 树和 css 树生成 layout 树， layout 树与 DOM 树类似，但是不包含 display:none 这种不展示的元素。

7. 即使知道了不同元素的位置及样式信息，我们还需要知道不同元素的绘制先后顺序才能正确绘制出整个页面。在绘制阶段，主线程会遍历布局树以创建绘制记录。绘制记录可以看做是记录各元素绘制先后顺序的笔记。

8. 在获取了所有需要的信息之后， 浏览器使用合成技术来绘制每帧。 
合成技术会将一个页面分割成不同的层， 并对每层进行光栅化， 最终在 compositor thread 中重新组合。
在分层时， main thread 会遍历 layout 树创建 layer 树(即 update layer tree)。 添加了 `will-change` CSS 属性的元素，会被看做单独的一层。

一旦 layer 树被创建，渲染顺序被确定，主线程会把这些信息通知给 compositor thread， compositor thread 会栅格化每一层。有的层的可以达到整个页面的大小，因此，合成器线程将它们分成多个磁贴，并将每个磁贴发送到 raster threads，raster threads 会栅格化每一个磁贴并存储在 GPU 显存中。

一旦磁贴被光栅化，合成器线程会收集称为绘制四边形的磁贴信息以创建合成帧。

compositor thread 随后会通过 IPC 消息传递给 browser process ，由于浏览器的 UI 改变或者其它插件的渲染进程也可以添加合成帧，这些合成帧会被传递给 GPU 用以展示在屏幕上，如果滚动发生，合成器线程会创建另一个合成帧发送给 GPU。

合成器的优点在于，其工作无关主线程，合成器线程不需要等待样式计算或者 JS 执行，这就是为什么合成器相关的动画 最流畅，如果某个动画涉及到布局或者绘制的调整，就会涉及到主线程的重新计算，自然会慢很多。




![main thread](https://note.youdao.com/yws/api/personal/file/4C8C97130D784A1987586082011C5353?method=download&shareKey=5e3499aeca6993c1c53826196026c818)