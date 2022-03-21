## 为什么要使用Vite

以往在开发前端项目的时候，为了提高效率（不想做麻烦的配置工作）创建项目往往都是使用框架自带的脚手架工具，比如创建 vue 项目，使用 vue-cli；创建 react 项目，使用 create-react-app 。这些工具随时对于自己项目来说是很方便，但是却没有泛用性，也就是 vue-cli 创建不了 react 项目，create-react-app 创建不了 vue 项目。

vue-cli、create-react-app 都是基于 webpack 开发，只是帮我们创建好项目结构，还有一些常用的配置而已。

webpack 最让人难以忍受的问题是多如牛毛的配置，以及版匹配问题，还有就是对大型项目的的编译时间，HMR 的时间会很长。而 vite 就很好的解决了这些问题。

**Vite 的特点**：

- 一个开发服务器，它基于 [原生 ES 模块](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 提供了 [丰富的内建功能](https://vitejs.cn/guide/features.html)，如速度快到惊人的 [模块热更新（HMR）](https://vitejs.cn/guide/features.html#hot-module-replacement)。
- 一套构建指令，它使用 [Rollup](https://rollupjs.org/) 打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

Vite 意在提供开箱即用的配置，同时它的 [插件 API](https://vitejs.cn/guide/api-plugin.html) 和 [JavaScript API](https://vitejs.cn/guide/api-javascript.html) 带来了高度的可扩展性，并有完整的类型支持。

Vite 和 webpack 一样是一个工程化工具，而不是专属 Vue 的 Cli ，所以 Vite 也可以用来创建 react 等项目。



## Vite 的基本使用

创建 Vite 项目

```shell
npm init vite@latest
# yarn create vite
```

创建完成后安装依赖

```shell
npm i
```

然后运行项目，会发现项目服务会被迅速开启

```shell
npm run dev
```

