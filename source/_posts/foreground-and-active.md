---
title: foreground-and-active
date: 2019-05-08 20:58:53
tags: 脑洞
---
# Foreground and Active

最近收到一个需求，需要参（抄）考（袭）传统盯盘软件，当用户处于行情页面时，无需点击”输入代码“之类的`<input>`，只要在键盘按下一个键，就有一个股票代码搜索提示弹出来，供用户选中需要查看的股票。

这样的需求分析一下，会发现有些麻烦，前端需要判断当前页面没有弹出的<prompt>/<confirm>，没有正在输入的<input>/<textarea>；换个说法是，前端需要判断当前用户交互的焦点，需要知道用户的确是在盯盘。

HTML Standard 有个章节是讲 Focus 的，并且提供了一个相关的`document.activeElement`，利用这个API，可以知道用户正在交互的元素。但是现在的页面开发不是基于HTMLElement，而是基于某某框架的组件，问题是好像并没有那个框架会提供`app.activeComponent`这样的API。

传统的 Windows GUI 开发中，有两个概念——`foreground`和`active`。传统的窗口叠窗口模式里，很容易区分用户的交互焦点，因为最前面的就是`active`的；而`foreground`/`background`是和线程有关。在浏览器单线程世界里，任务的执行没有`foreground`/`background`之分，都在统一起跑线上，但是UI里，是可以有`foreground`/`background`组件。比如，弹出的对话框，天猫右滑出来的购物车组建，对于整个页面来说，是`foreground`。我想，`z-index`是判断`foreground`/`background`的依据。然后像<input>，一般`z-index`不变，但用户可以通过鼠标点击，实现`focus`/`blur`，从而影响倒页面的活动元素。

目前项目里的代码，只判断了`document.activeElement`这种情况，也就是只考虑了`active`，但是`foreground`是没有处理的，所以往后应该潜在的问题。

今天稍微想了一下（顺便水一篇），对于比较大型的应用来说，需要`FocusTrackingService`。我本来想在`VSCode`源码里找一下方案，