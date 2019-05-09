---
title: 循环依赖和IoC
date: 2019-05-09 14:18:07
tags: 技术
---

目前项目里，因为前期的代码没有分层，底层和上层的代码相互import，出现了循环依赖，最终导致构建失败。

最好的解决方案当然时严格执行分层。TSLint里的`import-patterns`配置项，能控制一个文件只能从何处引入其他模块：
```json
{
    "import-patterns": [
	true,
	{
		"target": "**/vs/base/common/**",
		"restrictions": [
			"vs/nls",
			"**/vs/base/common/**"
		]
	},
}
```
比如👆摘录自VSCode的配置，它约束了基础模块目录下文件不能从目录外import。

但是考虑项目基本成形，而且缺乏各种单元测试集成测试，实在没有信心对现成代码大规模重构。所以我想的办法是，把被相互依赖的实例，挂到一个全局对象，类似早期前端开发的做法：
```javascript
let ioc = {}
ioc.serviceA = {}
ioc.serviceB = {}
```
这样能解决文件级别的相互依赖，但是不能解决代码逻辑级别的依赖，比如两个类的方法会相互调用实例：
```javascript
let ioc = {}

class ServiceA {
    foo() {
        ioc.serviceB.bar()
    }
}

class ServiceB {
    bar() {
        ioc.serviceA.foo()
    }
}
ioc.serviceA = new ServiceA()
ioc.serviceB = new ServiceB()
```
可见，在挂载到依赖注入容器时，如果挂载的是实例，就会有👆的问题。InversifyJS是使用实例来注册依赖的
```typescript
const myContainer = new Container();
myContainer.bind<Warrior>(TYPES.Warrior).to(Ninja);
myContainer.bind<Weapon>(TYPES.Weapon).to(Katana);
```
所以这个库也碰到[循环依赖](https://github.com/inversify/InversifyJS/blob/master/wiki/circular_dependencies.md)的问题。

如果在注册依赖时，注册的时构造函数，而不是实例，在获取依赖时再去初始化，应该就能解决循环依赖的问题。而且InversifyJS也好，VSCode也好，他们注册或注入依赖都受限于TypeScript写法，不太简洁。如果单纯考虑在JavaScript使用，依赖注册可以做成把构造函数赋值给依赖容器的某个属性名，依赖注入就是返回实例，如果没有初始化的，要初始化实例。刚开始时我用了getter/setter，其实使用Proxy机制，可以做到属性拦截，让依赖注入用起来更简洁，最后得到的依赖注入服务的代码如下：
```javascript
// module InistanceService
class InistanceService {
  constructor() {
    this.serviceCtors = {};
    this.services = {};
  }

  get(name) {
    let serv = this.services[name];
    let Ctor = this.serviceCtors[name];
    if (!serv && !Ctor) {
      console.warn('[ioc]: Could not resolve ', name); 
      return null;
    }

    if (serv) return serv;
    let instance = new Ctor();
    this.services[name] = instance;
    return instance;
  }
}

const instance = new InistanceService();

const handler = {
  get(target, name) {
    if (target[name]) return target[name];
    return target.get(name);
  },
  set(obj, prop, value) {
    obj.set(prop, value);
    return true;
  }
};

export const ioc = new Proxy(instance, handler);
```
