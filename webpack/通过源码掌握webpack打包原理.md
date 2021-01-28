- [01｜webpack启动过程分析](#01webpack启动过程分析)
  - [开始：从webpack命令行说起(这个过程发生了什么？)](#开始从webpack命令行说起这个过程发生了什么)
  - [分析webpack的入口文件：webpack.js](#分析webpack的入口文件webpackjs)
  - [启动后的结果](#启动后的结果)
- [02｜webpack-cli源码阅读](#02webpack-cli源码阅读)
  - [webpack-cli做的事情](#webpack-cli做的事情)
  - [从NON_COMPLILATION_CMD分析出不需要的编译的命令](#从non_complilation_cmd分析出不需要的编译的命令)
  - [NON_COMPLILATION_CMD的内容](#non_complilation_cmd的内容)
  - [命令行工具包 yargs 介绍](#命令行工具包-yargs-介绍)
  - [webpack-cli 使用args介绍](#webpack-cli-使用args介绍)
  - [webpack-cli 执行结果](#webpack-cli-执行结果)
- [03｜Tapable插件架构与Hooks设计](#03tapable插件架构与hooks设计)
  - [webpack 的本质](#webpack-的本质)
  - [先看一段代码](#先看一段代码)
  - [Tapable 是什么？](#tapable-是什么)
  - [Tapable Hooks 类型](#tapable-hooks-类型)
  - [Tapable 的使用 - new Hook 新建钩子](#tapable-的使用---new-hook-新建钩子)
  - [Tapable 的使用 - 钩子的绑定与执行](#tapable-的使用---钩子的绑定与执行)
  - [Tapable 的使用 - Hook基本用法示例](#tapable-的使用---hook基本用法示例)
  - [Tapable 的使用 - 实际例子演示](#tapable-的使用---实际例子演示)
- [04｜Tapable是如何与webpack进行关联的？](#04tapable是如何与webpack进行关联的)
  - [模拟 Compiler.js](#模拟-compilerjs)
  - [插件 my-plugin.js](#插件-my-pluginjs)
  - [模拟插件执行](#模拟插件执行)
- [05｜webpack流程篇：准备阶段](#05webpack流程篇准备阶段)
  - [webpack的编译按照下面的钩子调用顺序执行](#webpack的编译按照下面的钩子调用顺序执行)
  - [WebpackOptionsApply](#webpackoptionsapply)
- [06｜webpack流程篇：模块构建和chunk生成阶段](#06webpack流程篇模块构建和chunk生成阶段)
  - [Compiler hooks](#compiler-hooks)
  - [Compilation](#compilation)
  - [ModuleFactory](#modulefactory)
  - [Module 支持的模块](#module-支持的模块)
  - [NormalModule](#normalmodule)
  - [Compilation hooks](#compilation-hooks)
  - [Chunk 生成算法](#chunk-生成算法)
- [07｜webpack流程篇：文件生成](#07webpack流程篇文件生成)
- [08｜动手编写一个简易的webpack（上）](#08动手编写一个简易的webpack上)
  - [模块化：增强代码可读性和维护性](#模块化增强代码可读性和维护性)
  - [常见的几种模块化方式](#常见的几种模块化方式)
  - [AST 基础知识](#ast-基础知识)
  - [回顾一下 webpack模块机制](#回顾一下-webpack模块机制)
  - [动手实现一个简易的 webpack](#动手实现一个简易的-webpack)
- [09｜动手编写一个简易的webpack（下）](#09动手编写一个简易的webpack下)

#  01｜webpack启动过程分析
##  开始：从webpack命令行说起(这个过程发生了什么？)
  - 通过 npm scripts 运行webpack
    - 开发环境：npm run dev
    - 生产环境：npm run build
  - 通过 webpack 直接运行
    - webpack entry.js build.js
  - 查找 webpack 入口文件
    - 在命令行运行以上命令后，npm 会让命令行工具进入 node_modules/.bin 目录
      查找是否存在 webpack.sh 或者 webpack.cmd（window） 文件，如果存在，就执行，不存在，则跑出错误
      (在mac上 如果是全局的 则先从 user/local 找)
      ![.bin](pic/Screen%20Shot%202021-01-27%20at%209.28.51%20AM.png)
    - 实际的入口文件是：node_modules/webpack/bind/webpack.js
  - 是webpack 还是 webpack-cli 提供的命令呢？
  - 局部安装的话 如果想在node_modules目录下.bin 有命令 必须在package.json 通过bin这个字段进行指定
  ```json
   "bin": {
      "webpack": "bin/webpack.js", // node_modules/webpack/package.json中的bin指定
      "webpack-cli": "./bin/cli.js" // node_modules/webpack-cli/package.json中的bin指定
  }
  ```
##  分析webpack的入口文件：webpack.js
```javascript
process.exitCode = 0 /* 1. 正常执行返回 */
const runCommand = (command, args) => { /* ... */ } /* 2. 运行某个命令 */
const isInstalled = packageName => { /* ... */ } /* 3. 判断某个包是否安装 */
const runCli = cli => { /* ... */ } /* webpack5 里的源代码 */
const CLIs = [ /* ... */ ] /* 4. webpack 可用的CLI：webpack-cli 和 webpack-command  */
const installedClis = CLIs.filter(cli => cli.installed) /* 5. 判断是否两个cli 是否安装了 */
if (installedClis.length === 0) { /* ... */ } /* 6. 根据安装数量进行处理 */
else if (installedClis.length === 1){ /* ... */ }else { /* ... */ }
```
##  启动后的结果
webpack 最终找到 webpack-cli这个npm包，并且执行CLI

#  02｜webpack-cli源码阅读
##  webpack-cli做的事情
- 引入 yargs，对命令行进行定制
- 分析命令行参数，对各个参数进行转换，组成编译配置项
- 引用webpack，根据配置项编译和构建
##  从NON_COMPLILATION_CMD分析出不需要的编译的命令
webpack-cli 处理不需要经过编译的命令
##  NON_COMPLILATION_CMD的内容
webpack-cli 提供的不需要编译的命令
```javascript
const NON_COMPLILATION_CMD = [
    'init', // 创建一份webpack配置文件
    'migrate', // 进行webpack 版本迁移
    'add', // 往 webpack 配置文件中添加属性
    'remove', // 往 webpack 配置文件中删除属性
    'serve', // 运行 webpack-serve
    'generate-loader', // 生成 webpack loader 代码
    'generate-plugin', // 生成webpack plugin 代码
    'info' // 返回与本地环境相关的一些信息
]
```
##  命令行工具包 yargs 介绍
- 提供命令和分组参数
- 动态生成 help 帮助信息
##  webpack-cli 使用args介绍
参数分组（config/config.js）将命令划分为9类
- config：配置相关参数（文件名称、运行环境等）
- basic：基础参数（entry设置、debug模式、watch监听设置、devtool设置）
- module ：模块参数，给loader设置拓展
- output：输出参数（输出路径、输出文件名称等）
- advanced：高级用法（记录设置、缓存设置、监听频率、bail等）
- resolve：解析参数（alias 和 解析的文件后缀设置）
- optimizing：优化参数
- stats：统计参数
- options：通用参数（帮助命令、版本信息等）
##  webpack-cli 执行结果
- webpack-cli 对配置文件和命令行参数进行转换最终生成配置选项参数 options
- 最终会根据配置参数实例化webpack 对象（创建compile对象），然后执行构建流程 

#  03｜Tapable插件架构与Hooks设计
##  webpack 的本质
webpack中最核心的对象 比如：compiler 和 compilation 它们就是使用了tapable的
然后tapable其实是一个类似于node.js里面的 effect/emit 这种发布订阅的一个模块
webpack 可以将其理解是一种基于事件流的编程范例，一系列的插件运行
它内部的话 也是有一系列各种各样的插件 通过 每个插件的话它会有监听
会监听 compiler 和 compilation上面的一些关键事件节点
##  先看一段代码
源码路径：node_modules/webpack/lib/webpack.js 源码
核心对象 Compiler 和 COmpilation 都继承/使用了 Tapable
```javascript
class Compiler extends Tapable {
    // ...
}
class Compilation extends Tapable {
    // ...
}
```
##  Tapable 是什么？
- Tapable 是一个类似Node.js 的EventEmitter的库，主要是控制钩子函数的发布与订阅，控制着webpack的插件系统
- Tapable库暴露了很多Hooks（钩子）类，为插件提供挂载的钩子
```javascript
const {
	SyncHook,                   // 同步钩子
	SyncBailHook,               // 同步熔断钩子（遇到return 就直接返回）
	SyncWaterfallHook,          // 同步流水钩子（执行结果可以传递给下一个插件）
	SyncLoopHook,               // 同步循环钩子
	AsyncParallelHook,          // 异步并发钩子
	AsyncParallelBailHook,      // 异步并发熔断钩子
	AsyncSeriesHook,            // 异步串行钩子
	AsyncSeriesBailHook,        // 异步串行熔断钩子
	AsyncSeriesWaterfallHook    // 异步串行流水钩子
 } = require('tapable')
``` 
##  Tapable Hooks 类型


| type          | function                                               |
| ------------- | ------------------------------------------------------ |
| Hook          | 所有钩子的后缀                                         |
| Waterfall     | 同步方法，但是它会传值给下一个函数                     |
| Bail          | 熔断：当函数有任何返回值，就会在当前执行函数停止       |
| Loop          | 监听函数返回true表示继续循环，返回undefine表示结束循环 |
| Sync          | 同步方法                                               |
| AsyncParallel | 异步并行钩子                                           |
| AsyncSeries   | 异步串行钩子                                           |

##  Tapable 的使用 - new Hook 新建钩子
- Tapable 暴露出来的都是类方法，new 一个类方法获得我们需要的钩子
- class 接收数组参数options，非必传。类方法会根据传参，接收同样数量的参数。
  
        const hook1 = new SyncHook(['arg1', 'arg2', 'arg3'])

##  Tapable 的使用 - 钩子的绑定与执行
- Tabpable 提供了同步 & 异步绑定钩子的方法，并且它们都有绑定事件和执行事件对应的方法

| Async*                        | Sync*      |
| ----------------------------- | ---------- |
| 绑定：tapAsync/tapPromise/tap | 绑定：tap  |
| 执行：callAsync/promise       | 执行：call |

##  Tapable 的使用 - Hook基本用法示例
```javascript
const hook1 = new SyncHook(['arg1', 'arg2', 'arg3'])

// 绑定事件到 webpack 事件流
hook1.tap('hook', (arg1, arg2, arg3) => console.log(arg1, arg2, arg3)) // 1,2,3

// 执行绑定的事件
hook1.call(1, 2, 3)
```

##  Tapable 的使用 - 实际例子演示
- 定义一个Car方法，在内部hooks上新建钩子
  - 同步钩子：cacelerate（加速 接受一个参数）、brake（刹车）
  - 异步钩子：calculateRoutes（计算/规划路线）
- 使用钩子对应的绑定和执行方法
- caclulateRoutes 使用tapPromise 可以返回一个promise对象
```javascript
const {
    SyncHook,
    AsyncSeriesHook
} = require('tapable')

class Car {
    constructor() {
        this.hooks = {
            cacelerate: new SyncHook(['newspeed']),  // 加速
            brake: new SyncHook(),                   // 刹车
            calculateRoutes: new AsyncSeriesHook(['source', 'target', 'routesList']) // 规划路线
        }
    }
}

const myTesla = new Car()

// 绑定同步钩子
myTesla.hooks.brake.tap('WarningLampPlugin', () => {
    console.log(`You triggered the brake`)
})

// 绑定同步钩子 并传参
myTesla.hooks.cacelerate.tap('LoggerPlugin', newSpeed => {
    console.log(`setup speed ${newSpeed}`)
})

// 绑定一个异步Promise钩子
myTesla.hooks.calculateRoutes.tapPromise('calculateRoutes', (source, target, routesList, callback) => {
    // console.log(`source ${source}`)
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            console.log(`tapPromise to
            source ${source}
            target ${target}
            routesList ${routesList}
            `)
            resolve()
        }, 1000);
    })
})

// 执行/触发钩子
myTesla.hooks.brake.call()
myTesla.hooks.cacelerate.call(100)
console.time('cost')

// 执行/触发异步钩子
myTesla.hooks.calculateRoutes.promise('Async', 'hook', 'demo').then(() => {
    console.timeEnd('cost');
}, err => {
    console.error(err)
    console.timeEnd('cost')
})

```
#  04｜Tapable是如何与webpack进行关联的？
##  模拟 Compiler.js
```javascript
/*
    - 定义一个Car方法，在内部hooks上新建钩子
        - 同步钩子：cacelerate（加速 接受一个参数）、brake（刹车）
        - 异步钩子：calculateRoutes（计算/规划路线）
*/
const {
    SyncHook,
    AsyncSeriesHook
} = require('tapable')

class Compiler {
    constructor() {
        this.hooks = {
            accelerate: new SyncHook(['newspeed']),  // 加速
            brake: new SyncHook(),                   // 刹车
            calculateRoutes: new AsyncSeriesHook(['source', 'target', 'routesList']) // 规划路线
        }
    }
    run() {
        this.accelerate(10)
        this.break()
        this.calculateRoutes('Async', 'hook', 'demo')
    }
    accelerate(speed) {
        this.hooks.accelerate.call(speed)
    }
    break() {
        this.hooks.brake.call()
    }
    calculateRoutes() {
        this.hooks.calculateRoutes.promise(...arguments).then(() => {
            console.log('calc success')
        }, err => {
            console.error(err)
        })
    }

}

module.exports = Compiler
```
##  插件 my-plugin.js
```javascript
class MyPlugins {
    constructor() {

    }
    apply(compiler) {
        /* 
            挂载各类钩子 正常情况下 我们只需要监听其中的几个hook即可 
            webpack里面大大小小有100多个hook 
        */
        compiler.hooks.brake.tap('WarningLampPlugin', () => { console.log(`You triggered the brake`) })
        compiler.hooks.accelerate.tap('LoggerPlugin', newSpeed => { console.log(`setup speed ${newSpeed}`)})
        compiler.hooks.calculateRoutes.tapPromise('calculateRoutes', (source, target, routesList, callback) => {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    console.log(`tapPromise to
                    source ${source}
                    target ${target}
                    routesList ${routesList}`)
                    resolve()
                }, 1000);
            })
        })
    }
}

module.exports = MyPlugins
```
##  模拟插件执行
```javascript
const MyPlugins = require("./MyPlugin");

const MyPlugin = require('./MyPlugin')
const Compiler = require('./Compiler')
const Compilation  = require("./Compilation.js")

const options = {
    plugins: [
        new MyPlugin()
    ]
}

const compiler = new Compiler()

for (const plugin of options.plugins) {
    if (typeof plugin === 'function') {
        plugin.call(compiler, compiler)
    } else {
        plugin.apply(compiler)
    }
}

compiler.run()
```

#  05｜webpack流程篇：准备阶段
##  webpack的编译按照下面的钩子调用顺序执行
1. entry-option：初始化option
2. run：开始编译
3. make：从entry开始递归的分析依赖，对每个依赖模块进行build
4. before-resolve：对模块位置进行解析
5. build-module：开始构建某个模块
6. normal-module-loader：将loader加载完成的modules进行编译，生成AST树
7. program：编译AST，当遇到require等一些调用表达式时，收集依赖
8. seal：所有依赖build完成，开始优化
9. emit：输出到dist目录
![webpack构建钩子调用顺序](./pic/Screen%20Shot%202021-01-27%20at%205.39.38%20PM.png) 
##  WebpackOptionsApply
- 将所有配置options参数转换成webpack内部插件
- 使用默认插件列表
- 例子
  - output.library ->LibraryTemplatePlugin
  - externals -> ExternalsPlugin
  - devtool -> EvaDevtoolModulePlugin,SourceMapDevToolPlugin
  - AMDPlugin,CommonJsPlugin
  - RemoveEmptyChunksPlugin

#  06｜webpack流程篇：模块构建和chunk生成阶段
##  Compiler hooks
- 流程相关：
  - (before-)run
  - (before-/after-)compile
  - make
  - (after-)emit
  - done
- 监听相关：
  - watch-run
  - watch-close
##  Compilation
- Compiler 调用 Compilation 生命周期方法
  - addEntry -> addModuleChain
  - finish（上报模块错误）
  - seal
##  ModuleFactory
- ModuleFactory
  - NormalModuleFactory  比如：module.export = function a() {}
  - ContextModuleFactory 前面有一些路径什么的
  - 两个都继承自ModuleFactory
##  Module 支持的模块
- Module 以下模块都继承了Module
  - NormalModule 普通模块
  - ContextModule 
    - ./src/a
    - ./src/b
  - ExternalModule
    - module.exports = jQuery
  - DelegatedModule
    - manifest
  - MultiModule
    - entry:['a', 'b']
##  NormalModule
- Build
  - 使用 loader-runner 运行loaders
  - 通过 Parser 解析（内部是acron）
  - ParserPlugin 添加依赖
##  Compilation hooks
- 模块相关
  - build-module
  - fail-module
  - succeed-module
- 资源生成相关
  - module-asset
  - chunk-asset
- 优化和seal相关
  - (after-)seal
  - optimize
  - opyimize-modules(-basic/advanced)
  - after-optimize-modules
  - after-optimize-chunks
  - after-optimize-tree
  - optimize-chunk-modules
  - after-optimize-chunk-modules
  - optimize-module/chunk-oerder
  - before-module/chunk-ids
  - (after-)optimize-module/chunk-ids
  - before/after-hash
##  Chunk 生成算法
1. webpack 现将entry 中对象的module 都生成一个新的chunk
2. 编译 module 的依赖列表，将依赖的module也加入到chunk中
3. 如果一个依赖 module 是动态引入的模块，那么就会根据这个module 创建一个新 chunk，继续编译依赖
4. 重复上边的过程，直至得到所有的chunks

#  07｜webpack流程篇：文件生成

#  08｜动手编写一个简易的webpack（上）
##  模块化：增强代码可读性和维护性
- 传统的网页开发转变为Web Apps开发
- 代码复杂度在逐步增高
- 分离的 JS文件/模块，便于后续代码的维护性
- 部署时希望把代码优化成几个HTTP请求
##  常见的几种模块化方式
- ES Module
  ```javascript
  import calc from 'float-number-calc'
  calc.add('0.01', '9.99')
  ```
- CJS Node.js规范（可动态引入）
  ```javascript
  const calc = require('float-number-calc')
  calc.add('0.01', '9.99')
  ```
- AMD 浏览器规范（借鉴CJS）
  ```javascript
  require(['float-number-calc'], function(calc){
    calc.add('0.01', '9.99')
  })
  ``` 
##  AST 基础知识
- 抽象语法书（abstract syntax tree 或者缩写AST）,或者语法树（syntax tree
  是源代码的抽象语法结构的树状表现形式，这里特指编程语言的源代码
  树上每个节点都表示源代码中的一种结构
- 在线[demo](https://esprima.org/demo/parse.html#)
- 前端应用场景
  - 模板引擎 template ： 先转换成字符串 -> AST分析 -> 插入数据 -> 再转换成代码
  - ES6转ES5 或者 TS转JS，比如babel
##  回顾一下 webpack模块机制
- 打包出来是一个IIFE
- modules 是一个数组，每一项是一个模块初始化函数
- __webpack_require 用来加载模块，返回module.exports
- 通过 WEBPACK_REQUIRE_METHOD(0) 启动程序 
##  动手实现一个简易的 webpack
- 可以将ES6 语法转换成ES5 的语法
  - 通过 babylon 生成AST
  - 通过 babel-core 将AST重新生成源码
- 可以分析模块之间的依赖关系
  - 通过 babel-traverse 的 importDeclaration 方法获取依赖属性
- 生成的 JS文件可以在浏览器运行
  
#  09｜动手编写一个简易的webpack（下）
- demo
