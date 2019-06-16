---
title: Mindmap 挑战
date: 2019-06-16 11:48:56
tags: 技术
---
最近在V2看到XMind的招聘信息，然后就突发奇想，想试一下自己写一个Mindmap。

为了防止闭门造车，我先在Github上找找可以借鉴的项目，比较有名的是百度脑图Kityminder。Kityminder是由几个部分组成：底层处理SVG的Kity、核心逻辑部分Kityminder-core、外面的Kityminder-editor等。

我花了点时间研究Kity，发现Kityminder里面只用到Kity很小的部分，当初我写Canvas图表前，倒是花了很多功夫写Canvas绘图库；对比之下用SVG的话，的确省事很多。然后Kityminder-core这个项目里，核心的layout逻辑也只是很少的一部分。Kityminder太多不想看的代码。

然后找到Antv的hierarchy，这个repo是只做Mindmap layout的，给出了经典的layout几种变形。我读了一下，其中的基本思路是树遍历。它适合输入数据然后输出图像的模式，不是我想要那种，在交互时动态更新显示的模式。

所以我开始自己写。刚开始时，凭直觉地写了巨大一个Node类，这个类既有model数据，也有view数据，还有很多很多方法。这样写倒是能工作，写layout算法时也很方便。不过代码的确太暴力了，需要分离model和view的数据。

在写Canvas图表时，我是有把源数据和变换后显示用到的数据分离，只是Canvas绘图区域可以一次清理后重新绘制；而SVG还有一个view数据和DOM节点绑定的问题，像@antv/hierarchy，它的demo就是直接用canvas绘制，没有给出和DOM节点结合的方案。我暂时是采用MVC而不是MVVM的思路（不知道该怎样划分View和View model）把数据分离为model的数据和view的数据。Model是树状结构，view也是树状结构；但是DOM节点不是树状结构，每一个节点都是容器元素的子元素。

Model节点和view的节点该如何关联？我开始想了用ID的方案，通过在树中遍历出节点来达到双向映射。但是这方案看是来太慢了（虽然没测过benchmark）。后面想的是，Map可以用Object作为key，可以用Map来存节点的关联关系；如果需要考虑浏览器兼容性时，可以降级为用ID为key。想象一下，两个'A'字母叠在一起，上面的'A'各个顶点都对应着下面的顶点，这就是我需要的数据结构。这个方案的好处是，不需要在暴露给用户的类里带上太多和内部实现相关的数据和方法。比如用户在使用时，有这样的代码：
```javascript
  let map = new Mindmap(document.getElementById('container'));
  let topic = new Topic();
  map.addTopic(topic);

  let subTopic1 = new Topic();
  map.addTopic(subTopic1, topic);
```
里面的Topic类里，只要求实现ITopic接口：
```typscript
interface ITopic {
    parent: ITopic | null
    children: ITopic[]
    content: any
}
```
这样，设计处理的API会简洁很多，用法也很清晰。

就如同上面的例子，我的Mindmap API设计上适合动态添加/删除节点。在内部实现里是，节点有改动时，依次影响的是自身尺寸、自身连同后代节点的总体大小、兄弟节点的位置、父节点的总体大小等。这个过程连续向上，直至根节点。这样的更新逻辑还能再优化，响应更快。

在实现节点渲染时，我使用了SVG的`<foreignObject>`，然后里面渲染普通的DOM节点，如`<textarea>`。我试过把这些节点用普通地挂在一个div下渲染，用CSS Transform, 然后连线用SVG，效果挺好速度也很快；但是看了有些文章，里面提到这种混合不太适合SVG导出，才改为全用SVG，缺点就是Mindmap节点渲染起来慢很多，所以我需要使用上面的按需更新策略。

最后这个Demo还需要加上快捷键，我抽取了VSCode快捷键绑定相关的代码来使用，现实了和XMind一样的快捷键。最后完成的代码看[这个](https://github.com/WaiSiuKei/fin/tree/master/examples/mindmap)。这个还是简单的，接下来我想把它做成Coggle那种可以在节点进行富文本编辑的形式，而不是单行输入 + Image + Marker + URL 这种，不过面对富文本大魔王，还需要更多准备。