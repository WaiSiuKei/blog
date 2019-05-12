---
title: 前端开发中的AOP
date: 2019-05-12 22:11:30
tags: 技术
---

Java的Spring框架有两个核心的概念IoC和AOP。我之前写的文中，提到了我在现有项目用了IoC解决循环依赖。而AOP这一个编程范式，再现在的项目中也有使用。使用AOP，可以在不改动原来代码的前提下，对现有类的方法进行增强，从而实现更多的功能。

Java的AOP是通过`@`注解进行的（还有一系列XML配置），类似地，JavaScript里也有[proposal-decorators](https://github.com/tc39/proposal-decorators)，可以使用一模一样的代码写法。JavaScript里的装饰器`@`，可以对类和类方法进行装饰。我目前使用的装饰器语法是legacy的，使用@babel/plugin-proposal-decorators时，需要开启legacy选项，新的proposal比我认知中的功能多了好多东西。

使用decorator-legacy于类时，写法是这样:
```javascript
function foo(target) {
    // blah blah
    return target
}

function bar(...args) {
    return function(target) {
        // blah blah 
        return target
    }
}

@foo
class A{}

@bar(...)
class B{}
```
decorator函数内，可以做一些自定义的处理，比如`autobind-decorator`把类方法都绑定了正确的this；decorator-legacy用于方法是，写法是：
```javascript
function foo(target, key, descriptor) {
    // blah blah blah
    return descriptor
}

function bar(...args) {
    return function(target, key, descriptor) {
        // blah blah blah
        return descriptor
    }
}

class A {
    @foo
    f1() {}

    @bar(...)
    f2() {}
}
```
👆参数里的descriptor就是Property descriptor，返回值可以是参数里的descriptor，也可能是新的descriptor。

在使用AOP时，有一个Aspect切面的概念，对方法进行切面处理时，得到方法前后面，如同切香肠。吐过在方法调用前或者调用后插入目标代码片段，就能在不改动方法原有代码的前提下，实现新的功能。像AspectJS这种规规矩矩的库里，实现AOP就会要求定义before/after的函数。我在实践中，倒没有这么讲究，比如我的代码
```javascript
function foo(target, key, descriptor) {
    let fu = descriptor.value
    return {
        ...descriptor,
        value: function wrapped(...args) {
            // do something before
            let result = fn.call(this, ...args)
            // do something after
            return result
        }
    }
}

function bar(target, key, descriptor) {
    let newKey = ...
    let newDescriptor = ...
    Object.defineProperty(target, newKey, newDescriptor)
    return descriptor
}
```
我的AOP代码基本上始于上面的decorator function，根据业务特点，分别用于方法增强和给类添加新的方法。

