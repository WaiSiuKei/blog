---
title: 观察者模式
date: 2019-05-11 21:39:44
tags: 技术
---

[观察者模式](https://zh.wikipedia.org/wiki/%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F)是前端开发中用得比较多的设计模式，它的基本实现思路是观察者注册、事件通知和事件响应回调。开发实践中，由于Javascript的单线程特性，为了让事件、网络、脚本执行等任务可以高效，VM需要使用EventLoop来实现任务调度，继而各种异步、闭包的概念因此而来。观察者模式的核心，应该就是回调设置。

在处理DOM事件时，需要把给希望处理的事件绑定EventListener，通常的写法会是：
```javascript
// 1
div.onclick = fn
// 2
div.addEventListener('click', fn)
// 3
div.addEventListener('click', {
    handleEvent() {
        // blah blah blah
    }
})
```
设置完EventListener，在事件触发后，内部引擎会逐一调用EventListener，通知处理相应的事件。在Node.js中，也会有类似于浏览器环境的事件处理机制，比如:
```
process.on('uncaughtexception', fn)
```
Node.js很多对象都继承自EventEmitter，比如stream、socket等。

上面说的是，JavaScript原生环境API实现的观察者模式，基本上就是实现EventEmitter接口。类似地，在QT中，有一种signal/slot机制用于事件处理，通过`connect(signal, slot)`这样的代码把signal和slot关联在一起。PhosphorJS实现了一条类似的API，而且提供了几个特别的API：
```typescript
 disconnectBetween(sender: any, receiver: any):void
 disconnectSender(sender: any):void
 disconnectReceiver(receiver: any):void
 disconnectAll(object: any):void
 ```
 在现在类似IDE的layout，处理拖拽组件和布局容器的关系时，👆的方法能快速取消signal和slot的关联关系，而且在组件销毁时，也能方便地进行回收处理。平时开发常用的EventEmitter因为忘记取消回调而发生内存泄漏的人为错误挺多的，实现自己的EventEmitter时，有必要参考PhosphorJS的设计。
