## 源码获取

在看源码的时候首先要做的就是将源代码从github上下载下来。

```shell
git clone https://github.com/vuejs/vue.git
```

需要注意的是，要注意版本，目前vue已经到了3的版本，默认情况下是克隆主干的，所以可以克隆指定分支到本地

```SHELL
git clone --branch dev https://github.com/vuejs/vue.git
```

或者也可以在克隆主干后切换到`dev`分支上`git checkout -b dev origin/dev`

下载之后运行`npm install`安装依赖。之后运行`npm run dev`命令就可以打包生成源码了



## 源码目录结构

```
benchmarks  目录是用来做性能测试的
dist  最终打包出来的结果都放到 dist
examples 官方的例子
flow 类型检查（和ts类似，但是现在没人用了）
packages  一些写好的包（vue源码中包含weex）
scripts  所有打包的脚本都放这里
src 源代码目录
 - compiler 专门用来做模板编译的
 - core vue2代码核心
 - platform 平台
 - server 服务端渲染相关
 - sfc 解析单文件组件的（single file component）
 - shared 就是模块之间的共享属性和方法
```



## 寻找入口文件

在 `package.json` 中的 `scripts`可以看到大多数的命令打包配置都是`scripts/config.js`。所以可以去这个文件中查看打包的配置

```js
const builds = {}
-
function genConfig(){}
-
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

可以看出最终打包的配置是根据命令传入的环境变量`TARGET`在`builds`中找到对应的配置的，而在运行`npm run dev`的之后传入的`TARGET`就为`web-full-dev`。他们的含义如下

```
dev 开发   
prod 生产
web-runtime 运行时，无法解析new Vue传入的template
web-full 含有runtime和模板解析
web-compiler 只有模板解析
cjs commonjs规范
esm es6模块规范
browser 浏览器规范可以通过script引入使用
umd 支持global amd + cjs
```

那么在`builds`中找到`web-full-dev`可以看到，打包的入口是`web/entry-runtime-with-compiler.js`。但是需要注意的是，这个并不是真正的入口，因为这个路径还需要经过`resolve`方法进行解析得到`src/platforms/web/entry-runtime-with-compiler.js`

> 对比`src/platforms/web/entry-runtime-with-compiler.js`和`src/platforms/web/entry-runtime.js`会发现他们之间的却别其实就是重写了`$mount`方法

可以看到在`entry-runtime-with-compiler.js`中的Vue是来自于`runtime/index.js`中的，所以要看`runtime/index.js`中的Vue，如此往复就能找到vue的定义了。

+ `entry-runtime-with-compiler.js` 重写 `mount`方法
+ => `runtime/index.js`  所谓的运行时会提供一些dom操作的api，如属性操作，元素操作，提供一些组件和指令
+ => `core/index.js` 有`initGlobalAPI`初始化全局api
+ => `/instance/index` Vue的构造函数

**总体流程**

![1660977617405](vue2源码分析/1660977617405.png)



知道大致每个文件都干了些什么之后就可以看一下其中的方法是怎么实现的了，主要的方式有两种

+ 如果已经知道了核心的流程你那么可以单独打开源码去看
+ 如果不知道流程 可以通过一些测试用例来debugger，查看其中的流程和实现

在用第二种方法的时候要注意啊，默认的配置中是不回打包出`sourcemap`文件的，也就是断点看看不了源码。所以我们需要开启已`sourcemap`打包设置。有两种方式开启，一种是到`scripts/config.js`的`genConfig`方法的`output`中加上`sourcemap:true`；一种是直接在`package.json`文件的`scripts`脚本命令中添加一个`--sourcemap` 参数.

```shell
"dev": "rollup -w -c scripts/config.js --sourcemap --environment TARGET:web-full-dev",
```

之后就可以运行`npm run dev`就会重新运行生成文件，从实例中开始调试了。



## 关于Vue的响应式数据

通过断点可以知道，在实例化Vue（`core/instance/index.js`）的时候会传入 data 属性，实例化Vue的时候会调用`this._init()`方法来进行初始化，而这个`_init`是通过`initMixin`注入的，在其中又会调用`initState(vm)`来进行响应式数据的初始化，而在其中又使用了`observer()`来对对象的每个属性进行观察，创建成为`Observer`（在这里面用`defineReactive`来劫持数据）。

`core/instance/index.js -> this._init()|initMixin() -> initState(vm) -> observer() ->   new Observer(val) -> this.walk(value) -> defineReactive()（首次一定会执行这个方法，因为根的数据只会是一个对象）`

**那么对Vue响应式数据的理解**

可以监控一个数据的修改和获取操作，针对对象格式会给每个对象的属性使用`Object.defineProperty`进行劫持

在源码层面来说就是在初始化数据的时候，使用`observe`方法来为数据创建`Observer`对象，在实例化这个对象的时候会调用`defineReactive`方法来劫持对象中的每个属性，递归的为每个对象中的属性增加`getter setter`。

`defineReactive`方法内部对所有属性进行重写，这会导致性能问题，所以我们在使用Vue的时候如果数据的层级过高就需要考虑优化，如果数据不是响应式的就不要放到data中，在属性取值的时候尽量避免多次取值，比如在循环中`this.num++;`，如果有些对象是放到data中，但是不是响应式的数据的时候可以考虑使用`Object.freeze`来冻结对象。



## 关于Vue对数组数据的观察

在Vue中通过索引的方式来改变数据`this.arr[2]='jack'`，页面不会做出响应，因为在Vue中并没有对数组进行数据劫持，因为这样做的性价比不高，在开发中很少会有需要使用索引来修改数据的情况，比如说`this.arr[122]`这种情况很少出现，还有一个原因就是当数组数据很长的时候（比如`length = 122`的空内容数组），如果对每一想都进行数据劫持的话那么性能消耗是很高的。

所以在Vue中不会对数组中的每项做数据劫持，而是通过重写数据的变异方法（函数劫持）来实现响应式数据。所以通过索引修改数组数据，修改长度都是无法进行监控的

`core/instance/index.js -> this._init()|initMixin() -> initState(vm) -> observer() ->   new Observer(val) ->  protoAugment(value, arrayMethods) `

`protoAugment(value, arrayMethods)`方法对传入的数组数据进行原型链（`__proto__`）修改，后续调用的方法都是重写后的方法，同时也会对数组中的每个对象也再次进行代理，实现对数组中的对象进行代理。



## 关于Vue中的依赖收集

所谓的依赖收集（观察者模式）被观察者指的是数据（dep），观察者（watcher，有3种：渲染watcher、计算属性watcher、用户watcher），一个watcher中可能会对应多个数据 watcher 中还需要保存dep（重新渲染的时候可以让属性重新记录watcher）计算属性也会用到

watcher 和 dep 是一个多对多的关系，一个dep对应多个watcher，一个watcher有多个dep，默认渲染的时候会进行依赖收集（会触发get方法），数据更新了就找到属性对应的watcher去触发更新

`platforms/web/runtime/index.js -> $mount() -> mountComponent() -> new Watcher() -> this.get() -> this.getter()`

`this.getter()`就是调用`vm._update(vm._render())`，此时会触发数据的 getter 方法，这时候就会做依赖收集的工作

![img](vue2源码分析/fow.34669a8f.png)



## 关于Vue模板模板编译原理

当向Vue中传递 template 属性，我们需要将这个template编译成render函数

`template -> ast语法树 -> 对语法树进行标记（标记的是静态节点）-> 将ast语法树生成render函数`

最终每次渲染可以调用render函数返回对应的虚拟节点（递归是先子后父），之后调用`_update`就可以将虚拟DOM生成为真实DOM

`src/platforms/web/entry-runtime-with-compiler.js -> $mount() -> compileToFunctions() -> `



## 关于Vue生命周期钩子如何实现

就是内部利用一个发布订阅模式，将用户写的钩子维护成一个数组，后续一次调用callHook，主要靠的是 mergeOptions 方法将根的 options 和子组件的 options 合并在一起

`instance/index.js -> _init() -> mergeOptions()`