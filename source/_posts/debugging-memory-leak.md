---
title: JavaScript内存泄漏
date: 2019-06-10 21:11:17
tags: 技术
---
我之前在写的图表应用，写的时候没太注意内存问题，后来在项目里以库的方式引入时，发现会有些内存占用不断升高的问题，就是所谓的内存泄漏了。

一般提到内存泄漏，都会想起 C 的野指针问题，堆里动态分配的内存区域没有释放，在程序运行期间不对消耗可用内存。JavaScript 没有指针，但是也有“无用的”对象占用内存，出现类似的“内存泄漏问题”。我在平时接触到的，会有如下几类：
- 对象赋值到全部变量。使用在`<script>`标签里面未声明的对象，会赋值为全局变量的属性，或者是编程者刻意赋值的。
- 单例对象持有的其他对象的引用。有时程序运行期间需要一些单例，例如一些Register，或者依赖注入容器，在程序结束后，需要注意回收。
- 回调。这里的回调不是指DOM Event Listener, 有些旧文章会提到旧浏览器绑定事件后需要解绑，现代浏览器里已经不需要操心这个；这里说的是`Eventemitter.on(event, cb)`这种事件回调，或者其他以函数作为参数的地方，如果持有了函数的引用，需要注意对称的释放处理。

还有其他一些例子，它们的特点都是与闭包有关，引用在逻辑结束后没用清理。这些问题有时可以通过静态检查查找出来，比如全局变量，有on没有off。静态检查力不能及的时候，需要运行一下代码，通过开发者工具查找问题。

Chrome 的开发者工具里有一项“Memory”, 它可以用于调试对内存区域的问题，它提供的了几项功能里，我用得最多的是Heap Snapshot/堆快照。比方说，一个应用运行时，堆内存占用可能有多有少，但总体占用的均线是水平线，如果有泄漏的情况，往往会是阶梯式上升。

发现出现泄漏时，需要定位泄漏的对象，我习惯在进行UI操作前后分别录一次堆快照，然后使用"Object allocated between Snapshot1 and Snapshot2"这样的filter，看看两次快照之间的差异，列表里的对象会构造函数分类，很多实例也会看到代码位置，code review后通常可以发现上面说的几个问题。