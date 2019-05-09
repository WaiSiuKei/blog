---
title: Disposable 和 GC
date: 2019-05-09 09:44:42
tags: 技术
---
Disposable Object这个概念源自C#，它会实现IDisposable接口，从而实现一种释放Unmanaged Resources的机制。在C#里，Managed resources基本是指垃圾回收器管理的内存，Unmanaged Resources是指打开的文件、打开的网络链接等。

JavaScript也是一门带有GC的语言，一般来说不太需要自行管理内存。但是由于异步编程是JavaScript开发的基本技术，和异步相关的Callback、闭包往往是前端应用内存泄漏的原因。被不当使用的Callback、闭包，对于垃圾管理器来说，实属Unmanaged Resources。对于这些需要被“回收”的内容，其实可以借用C#的IDisposable的概念，管理资源释放问题。

IDisposable接口定义了dispose()方法，如下：
```typescript
interface IDisposable {
    dispose(): void
}
```
往往在Disposable Object需要被销毁前，手工调用dispose()方法。

在VSCode源码和PhosphorJS里面，IDisposable到处可见，有点不同的是，PhosphorJS的IDisposable会有disposed属性。VSCode源码里，和'Unmanaged Resources'，相关的对象/方法，都会返回IDisposable，用于释放资源。比如EventEmitter的事件监听回调设置、DOM元素的事件绑定等。

实现IDisposable接口然后显式调用dispose()，这样虽然可行，但如果有C++的RAII机制，在对象从栈中销毁时，自动调用dispose()，会减少很多人为的错误。JavaScript的Class没有类似析构函数的概念，目前有一个相关的提案[proposal-weakrefs](https://github.com/tc39/proposal-weakrefs/), 它提到了Finalization，可惜也是需要直接或者间接调用，并不像RAII那么方便。