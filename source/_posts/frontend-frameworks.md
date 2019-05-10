---
title: 前端框架存在的意义
date: 2019-05-10 16:45:02
tags:
---

我曾两次被问到同一个问题：前端框架存在的意义/前端框架解决的是什么问题。当时回答得都不太好，现在再组织一下语言写一下。

作为个人项目和我公司及其外包项目的主程，Vanilla.js的信仰者，人肉优化的实践者，我一向坚持自己进行DOM操作。我给公司项目写的图表表格，基本都是零外部依赖的代码。这些特殊应用，有性能追求，框架的效率绝对比不上手工优化的代码。

不过，我专注的场景，应该只是前端1%方向，剩下的99%，都是些拉数显示、CRUD的页面。这样的场景下，框架的确很流行。各类框架诞生后，已经把jQuery这样底层的库给淘汰。伴随框架而来的MVC、MVP、MVVM概念，的确是前端开发的新武器。各类框架基本上都会要求定义与显示相关数据，和用于生成DOM结构的模版，和事件处理。然后框架们基本都能按数据变化，自动执行模版渲染，将数据同步到页面。另一方面，用户在页面交互产生的事件，都能自动或者半自动反映到状态数据来。可以看出，前端开发工作中比较多的问题，是数据和页面显示的同步。其实不难理解，如果不使用任何辅助的库的话，需要更新一个显示数据，需要先在DOM🌲找到对应的节点再进行修改；再者在表单中，需要获知数据值的变化，也需要监听事件。这也就是当你jQuery这类辅助DOM操作的库得以流行的原因。

<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/1754_RC01/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"angular.js","geo":"US","time":"2010-04-10 2019-05-10"},{"keyword":"backbone.js","geo":"US","time":"2010-04-10 2019-05-10"},{"keyword":"react.js","geo":"US","time":"2010-04-10 2019-05-10"},{"keyword":"vue.js","geo":"US","time":"2010-04-10 2019-05-10"}],"category":0,"property":""}, {"exploreQuery":"date=2010-04-10%202019-05-10&geo=US&q=angular.js,backbone.js,react.js,vue.js","guestPath":"https://trends.google.com:443/trends/embed/"}); </script>

这样