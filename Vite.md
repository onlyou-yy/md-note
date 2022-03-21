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

**创建 Vite 项目**

```shell
npm init vite@latest
# yarn create vite
```

**创建完成后安装依赖**

```shell
npm i
```

**然后运行项目，会发现项目服务会被迅速开启**

```shell
vite dev
```

**vite 基本配置**

vite 的配置文件是在根目录下的 `vite.config.js`，基础配置如下

```js
import vue from "@vitejs/plugin-vue";
import {path} from "path";
import {defineConfig} from "vite";
//defineConfig 可以提供配置选项的代码提示
export default defineConfig({
  resolve:{
    alias:{//配置别名
      '@':path.resolve(__dirname,'src'),
      'comps':path.resolve(__dirname,'src/components'),
      'utils':path.resolve(__dirname,'src/utils'),
      'store':path.resolve(__dirname,'src/store'),
      'routers':path.resolve(__dirname,'src/routers'),
      'styles':path.resolve(__dirname,'src/styles'),
    }
  },
  plugins:[
    vue(),//注册 vue 配置插件
  ]
})
```

**环境变量的使用**

如果需要根据环境进行配置的话可以把配置作为一个函数进行导出，这个函数接收一个对象作为参数,在开发环境下 command 的值为 serve，在生产环境下为 build

```js
export defult ({command,mode})=>{
  if(command === 'serve'){
    return {}
  }else{
    return {}
  }
}
```

Vite 在一个特殊的 `import.meta.env`对象上暴露环境变量

- `import.meta.env.MODE`: {string} 应用运行的[模式](https://vitejs.cn/guide/env-and-mode.html#modes)。
- `import.meta.env.BASE_URL`: {string} 部署应用时的基本 URL。他由[`base` 配置项](https://vitejs.cn/config/#base)决定。
- `import.meta.env.PROD`: {boolean} 应用是否运行在生产环境。
- `import.meta.env.DEV`: {boolean} 应用是否运行在开发环境 (永远与 `import.meta.env.PROD`相反)。

有时候我们还需要自定义环境变量并使用它，这时候可以在**环境目录**（可以在vite.config.js 中配置 envDir 来指定）下定义环境变量文件。

```js
.env                # 所有情况下都会加载
.env.local          # 所有情况下都会加载，但会被 git 忽略
.env.[mode]         # 只在指定模式下加载
.env.[mode].local   # 只在指定模式下加载，但会被 git 忽略
```

比如我有如下环境变量

```js
//.env
MY_ENV=my1
VITE_ENV_TEST=test1

//.env.test
MY_ENV=my2
VITE_ENV_TEST=test2
```

> 只有以 `VITE_` 为前缀的变量才会暴露给经过 vite 处理的代码

那么只有在`test`模式(`vite dev --mode test`)下访问`import.meta.env.MY_ENV` 就为`my2`



## vite 配置vue项目

