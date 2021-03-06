---
date: 2021-04-03
title: 深入React源码-React的理念
tags:
  - React
describe:  深入源码前，看看facebook团队为何要写React框架，了解它做了什么事情
---

## React理念

从官网中可以看到React的理念：

> 我们认为，React 是用 JavaScript 构建快速响应的大型 Web 应用程序的首选方式。它在 Facebook 和 Instagram 上表现优秀。

可见，关键是实现快速响应。那么制约快速响应的因素是什么呢？

1. 当遇到大计算量的操作或者设备性能不足使页面掉帧，导致卡顿。----CPU瓶颈

2. 发送网络请求后，由于需要等待数据返回才能进一步操作导致不能快速响应。----IO的瓶颈

## React的解决方案

### CPU瓶颈

#### 浏览器相关基础

主流浏览器刷新频率为60Hz，即每（1000ms / 60Hz）16.6ms浏览器刷新一次。

我们知道，JS可以操作DOM，GUI渲染线程与JS线程是互斥的。所以JS脚本执行和浏览器布局、绘制不能同时执行。

在每16.6ms时间内，需要完成如下工作：

```
JS脚本执行 -----  样式布局 ----- 样式绘制
```

当JS执行时间过长，超出了16.6ms，这次刷新就没有时间执行样式布局和样式绘制了。具体表现就是页面掉帧，造成卡顿。

#### 如何解决？

> 以下的前提是开启了concurrent mode，即使用ReactDOM.unstable_createRoot()，使用React.render()使用的legacy模式依旧是同步模式

在浏览器每一帧的时间中，预留一些时间给JS线程，React利用这部分时间更新组件（在源码中，预留的初始时间是5ms）。

当预留的时间不够用时，React将线程控制权交还给浏览器使其有时间渲染UI，React则等待下一帧时间到来继续被中断的工作。

这样浏览器就有剩余时间执行样式布局和样式绘制，减少掉帧的可能性。

所以，解决CPU瓶颈的关键是实现时间切片，而时间切片的关键是：**将同步的更新变为可中断的异步更新**。

### IO的瓶颈

网络延迟是前端开发者无法解决的。如何在网络延迟客观存在的情况下，减少用户对网络延迟的感知？

React给出的答案是将人机交互研究的结果整合到真实的 UI 中。

为此，React实现了Suspense功能及配套的hook——useDeferredValue。

而在源码内部，为了支持这些特性，同样需要将同步的更新变为可中断的异步更新。

## 总结

React为了践行“构建快速响应的大型 Web 应用程序”理念，关键是解决CPU的瓶颈与IO的瓶颈。而落实到实现上，则需要将同步的更新变为**可中断的异步更新**。