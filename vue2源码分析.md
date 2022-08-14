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