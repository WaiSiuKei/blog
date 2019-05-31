---
title: Monorepo研究
date: 2019-05-30 11:17:09
tags: 技术
---
# Monorepo
共享代码管理有很多种形式，有分仓库分项目的做法、Git Submodules、把全部代码集中在一个代码仓库的方式。后面一种代码组织形式称为Monorepo，它提倡的是，开发团队应将代码放到一个repo里，而不是去维护不同的repo。
假设一种场景，两个模块之间的接口需要修改，如果采用多仓库的形式，需要经过的步骤是：修改被依赖的仓库、发布更新、下游仓库更新依赖进行开发、更新发布等；如果采用monorepo，可以做到开发时同时修改，然后同时发布新版本。
当然优势的地方也有很多，很多文章也有介绍，比如[这个](https://pspdfkit.com/blog/2019/benefits-of-a-monorepo/)。
# Lerna
Monorepo很早就有项目进行实践，比如React, 然后Lerna的流行，出现了更多Monorepo。
Lerna的自我介绍是
> A tool for managing JavaScript projects with multiple packages.

它把各个模块称为package，并且定义了这么代码组织结构，
```
lerna-repo/
  package.json
  packages/
    package-1/
      package.json
    package-2/
      package.json
```
👆的package都放到一个叫做packages的目录里，当然这个目录可以是其他名字，也可以有多个这类目录。然后Lerna可以提供各种方便的功能，比如👇几个：
### bootstrap命令
建立这个的目录结构后，首先运行`lerna bootstrap`命令，安装各个pkg的依赖。与在各个pkg目录手动安装不同，`bootstrap`命令会把公共命令link到同一处，达到节省资源到目标。
### build命令
`lerna bootstrap`命令会对各个pkg进行构建。如果各个pkg之前存在依赖关系，它们之间的关系会形成有向图，Lerna会进行拓扑排序，得到一个打包顺序结果，按这个顺序逐一构建。
### version命令
项目里面的pkg可以采用独立版本号和统一版本号两种版本号管理形式。采用独立版本号时，需要手动对每个包的版本号进行升级管理；采用统一版本号时，`lerna version`命令可以帮助执行版本号更新和release。项目大了还是采用统一版本方便，避免各个版本间潜在的不兼容。
# Typescript Path mapping
刚开始尝试Monorepo时，我参考了一些用了Lerna地项目, 建立类似地目录结构，然后手动到各个目录运行`npm install`，然后用Typescript开发。我的各个模块会有相互依赖，查了一下Typescript文档，找到了模块寻找相关的[文章](https://www.typescriptlang.org/docs/handbook/module-resolution.html)。它提到了Path mapping，可以把指定第三方依赖寻找的目录。
# NPM local paths和NPM link
NPM的[local paths](https://docs.npmjs.com/files/package.json#local-paths)功能提供一种指定本地目录作为依赖的功能，适合本地开发，如果发布为NPM包会有问题，用户电脑上不一定有一样的目录结构。类似的功能是`npm link`, 这个也是**只**适合本地开发，而且在monorepo下要一个一个手动执行。
# Webpack alias
Webpack配置选项里的resolve有个alias属性，可以指定路径的别名，Monorepo里如果有webpack构建的内容时，还需要手动指定这一项。
# Yarn workspace
我在开发自己的monorepo时，Typescript Path mapping基本够用，但是因为定义文件生成工具[dts-bundle-generator](https://github.com/timocov/dts-bundle-generator)在处理依赖项时，如果不是定义在package.json的话，它会把其他模块的内容内联。
解决方案是Yarn workspace，它提供了把各个模块link到上一级依赖目录的功能。按照Node.js模块寻找规则，本级目录下的node_modules不存在的话，Node.js会查找上一级目录里的node_modules目录，一路往上直至用户文件夹。这个symbol link可以在开发时，被依赖模块更新代码后，依赖的模块可以无感地继续开发，不用关注依赖更新问题；Yarn workspace可以自动workspace下的多个目录，比起手动一个一个运行`npm link`会方便很多；各个模块的package.json适合发布到NPM，不会有NPM local paths的问题。Lerna和Yarn workspace可以叠加使用。
# 后记
在搭建自己的monorepo时，试过👆全都方案，最后确定的是：目录按Lerna和Yarn workspace配置、然后运行`lerna bootstrap`安装依赖。
期间还有些开发时遇到的问题：
## WebStorm的坑
WebStorm里面使用的Typescript compiler会有些问题，比如在某个模块里遇到的错误:
> Error:TS6053: File '/Users/wei/fin/packages/linkedlist/node_modules/typescript/lib/lib.dom.d.ts' not found.

解决这个是，在workspace root安装Typescript，即使Yarn不提倡这样做。
> error Running this command will add the dependency to the workspace root rather than the workspace itself, which might not be what you want - if you really meant it, make it explicit by running this command again with the -W flag (or --ignore-workspace-root-check).
info Visit https://yarnpkg.com/en/docs/cli/add for documentation about this command.

## dts-bundle-generator的坑
dts-bundle-generator即使按上面的 Lerna + Yarn workspace 搭建后，还会出现声明内联的情况，试了好久，加了Path mapping倒是可以解决问题，但是不太好，因为在使用Path mapping时，WebStrom的自动import太傻，如果里面有自身目录，同一模块下个两个文件间import，居然不是使用相对路径，而是`pkg-name/src/path/to/file`，这样的形式。
更新一下，给作者提了[Issue](https://github.com/timocov/dts-bundle-generator/issues/91)后，作者给出一个用来处理这种情况的配置项，所以Path mapping可以去掉。
