---
title: AltGr和Dead Key
date: 2019-05-25 16:15:20
tags: 技术
---
最近项目里加了个和键盘控制有关的功能，因为之前没做过这一块，我要未(xiang)雨(de)绸(tai)缪(duo)研究一下快捷键功能的实现。参考的实现是VSCode的代码，因为在做图表时剪裁了部分代码，真有需求来时糊弄一下也能满足，但是自己还不太明白这一块，所以还得从原理研究。

VSCode处理不同键盘布局时，通过一个叫做native-keymap的模块，获取每个物理的Virtual Key Name，还有和各种装饰键组合时的输出字符：
```
  { key_code: 'VKEY_OEM_6',
    value: ']',
    withShift: '}',
    withAltGr: '',
    withShiftAltGr: '',
    valueIsDeadKey: false,
    withShiftIsDeadKey: false,
    withAltGrIsDeadKey: false,
    withShiftAltGrIsDeadKey: false
  },
```
这里，value/withShift的意思可以猜到，但是AltGr和DeadKey还是第一次听说。

查了一下维基，倒是增长了见闻。键盘结构中和文字输入有关的是Character key，Modifier key和Dead keys。Character key基本是ACSII里面的有意义的文字，这个可以很好理解。

Modifier key可以和Character key组合输出，比如
- C → c（小写——第 1 级）
- Shift+C → C（大写）
- AltGr+C → ©（版权符号）
​- AltGr+Shift+C → ¢（美分符号）

被划分Modifier key的是：
- Shift
- Ctrl
- Alt(⌥ Option)
- AltGr
- ⊞ Win
- ⌘ Command
- Fn

这里面唯独AltGr这个键没见过。现在国内用的键盘布局，是美式键盘布局，就是Win7下切换到英文输入的那个；还有其他很多各种布局，特别是在欧洲。美式键盘（不同于美式国际键盘）是没有AltGr这个键的。欧洲地区的键盘布局都会有这个键，用来输入一些比较少见的字母：
```
¡ ² ³ ¤ € ¼ ½ ¾ ‘ ’ ¥ ×
 ä å é ® þ ü ú í ó ö « »
  á ß ð           ø ¶ ´ ¬
   æ   ©     ñ µ ç   ¿
```
具体可以看[维基关于AltGr的介绍](https://en.wikipedia.org/wiki/AltGr_key)。

Dead Key这个也和欧洲语言有关，比如法文字母à这种衍生拉丁字母，上面的`代表变音的符号，需要用Dead Key来输入。具体可以看[维基关于Dead Key的介绍](https://en.wikipedia.org/wiki/Dead_key)。

考虑到我写的软件只面向国内用户，最多最多再放放Github，这AltGr和Dead Key就不深入研究了。