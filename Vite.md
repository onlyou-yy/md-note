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
npm run dev
```

**vite基本指令**

```json
{
  'scripts':[
    'dev':'vite --open',
    'build':'vite build',
    'preview':'view preview --open',
  ]
}
```

**vite 基本配置**

vite 的配置文件是在根目录下的 `vite.config.js`，基础配置如下

```js
import vue from "@vitejs/plugin-vue";
import {path} from "path";
import {defineConfig} from "vite";
//defineConfig 可以提供配置选项的代码提示
export default defineConfig({
  base:'./',//基础路径，默认是 /。
  resolve:{
    alias:{//配置别名
      '@':path.resolve(__dirname,'src'),
      'comps':path.resolve(__dirname,'src/components')
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

> mode 一般为development、production 或者自定定义的其他名字，在运行`npm run dev`时使用.env.development 中的变量，在运行`npm run build`时使用的是 .env.production 中的变量，如果要使用自己定义的模式的变量需要使用`--mode 模式名`来指定，如`npm run dev --mode test`

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

那么只有在`test`模式(`npm run dev --mode test`)下访问`import.meta.env.MY_ENV` 就为`my2`



## vite 配置vue项目

### 配置使用 mock

需要使用到 `vite-plugin-mock`插件，还有`mockjs`

```shell
npm i vite-plugin-mock -D
npm i mockjs -S
```

在配置文件中注册插件

```js
import { viteMockServe } from 'vite-plugin-mock'
export default {
  plugins: [viteMockServe({mockPath:'./mock'})],
}
```

创建mock数据文件`mock/user.js`

```js
export default [
  {
    url:'/api/getUsers',
    method:'post',
    response:({body}) => {
      console.log('body>>>>>>',body);
      return {
        code:0,
        message:'ok',
        data:['user1','user2'],
      }
    }
  }
]
```

使用，在组件中请求时，mock就会拦截到请求并返回数据

```js
fetch('/api/getUsers').then(res=>res.json()).then(res=>{
  console.log(res);
})
```



### 配置使用vuex，vue-router

安装

```shell
npm i vuex@next vue-router@4 -S
```

创建 `router/index.js`路由文件

```js
import {createRouter,createWebHashHistory} from 'vue-router'
const router = createRouter({
  history:createWebHistory(),
 	routes:[
    {path:'/',component:()=>import('view/home.vue')}
  ]
})
export default router
```

创建 `store/index.js`store文件

```js
import { createStore } from 'vuex'
// 创建一个新的 store 实例
const store = createStore({
  state () {
    return {
      count: 0
    }
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
export default store;
```

注册需要在入口的`main.js`中注册到vue 中

```js
import {createApp} from 'vue'
import App from './App.js'
import router from './router';
import store from './store';
createApp(App).use(router).use(store).mount('#app');
```



### 配置使用 scss

只需要安装`sass`就可以使用 scss 了，但是要记得在`main.js`中将样式引用一下

```shell
npm i sass -D
```



### 配置 CDN 引入

需要使用到`vite-plugin-cdn-import`

安装

```shell
npm i vite-plugin-cdn-import -D
```

配置

```js
import importToCDN,{autoComplete} from 'vite-plugin-cdn-import';

//------------
plugins:[
  importToCDN({
    module:[
      autoComplete('react'),
      autoComplete('react-dom')
    ]
  })
]
```



其他插件可以在 [vite插件社区](https://github.com/vitejs/awesome-vite#plugins) 中查找



## Vite 常用配置

```js
import vue from "@vitejs/plugin-vue";
import importToCDN,{autoComplete} from 'vite-plugin-cdn-import';
import viteImagemin from 'vite-plugin-imagemin'
import compress from 'vite-plugin-compress'
import {path} from "path";
import {defineConfig} from "vite";
//defineConfig 可以提供配置选项的代码提示
export default defineConfig({
  base:'./',//基础路径，默认是 /。
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
    //注册 vue 配置插件,
    vue(),
    //使用 gzip 压缩
    compress(),
    //配置通过 CDN 引入包
    importToCDN({
      modules:[
        //这里的是通过 cdn 引入element-plus，因为它依赖 vue，所以需要先引入 vue。而且也是第一次编译的时候也是需要下载下来使用的。
        //使用的话还是需要在main.js中进行引入并注册的,但是就回不被打包进去了
        //import ElementPlus from 'element-plus';
        //createApp(App).use(ElementPlug)
        {
          name:'vue',
          var:'Vue',
          path:'https://unpkg.com/vue@next',
        },
        {
          name:'element-plus',
          var:'ElementPlus',
          path:'https://unpkg.com/element-plus',
          css:'https://unpkg.com/element-plus/dist/index.css'
        },
      ]
    }),
    //图片压缩配置
    viteImagemin({
        gifsicle: {
          optimizationLevel: 7,
          interlaced: false,
        },
        optipng: {
          optimizationLevel: 7,
        },
        mozjpeg: {
          quality: 20,
        },
        pngquant: {
          quality: [0.8, 0.9],
          speed: 4,
        },
        svgo: {
          plugins: [
            {
              name: 'removeViewBox',
            },
            {
              name: 'removeEmptyAttrs',
              active: false,
            },
          ],
        },
      }),
  ],
  server:{//开发服务器设置
    proxy:{//服务代理
      '^/api/.*': {//跨域代理
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
        // proxy 是 'http-proxy' 的实例
        //configure: (proxy, options) => {}
      }
    }
  },
  build:{//打包配置，在生产环境才会使用
    minify:'terser',//启用terser优化选项，默认是 esBuild
    terserOptions:{
      drop_console:true,//移除console输出
      drop_debugger:true,//移除 debugger 关键字
    },
    rollupOptions:{
      output:{//打包输出文件夹的具体位置，是在dist下的
        chunkFileNames:'js/[name]-[hash].js',
        entryFileNames:'js/[name]-[hash].js',
        assetFileNames:'[ext]/[name]-[hash].[ext]'
      }
    }
  }
})
```





## vite基本原理

在开发的时候 vite 运行`npm run dev`比在 webpack 运行`npm run dev`进行开发环境开发的时候会发现 vite 几乎是秒开，因为 webpack 是直接将整个项目进行打包然后再放到开发者服务器中，所以当项目比较大的时候运行时就会很慢；而 vite 并不会将整个项目进行打包，而是仅仅是开启一个开发者服务器，当访问到哪页面的时候再将该页面使用到的包、资源等进行打包编译返回，另外，现在浏览器已经原生支持ESModule规范，所以在开发环境中是不需要打包编译的，但是为了能顺利获取到请求资源会对`import from`导入的路径进行重写。

### vite 的基本实现

建立基本的目录结构，index.js 是vite服务器

```
|-src
|---App.vue
|---main.js
|-index.html
|-vite.js
```

**App.vue** (单文件组件：SFC，single file component)

```vue
<script setup>
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
	<h2>hello vite</h2>
  <img alt="Vue logo" src="./assets/logo.png" />
  <HelloWorld msg="Hello Vue 3 + Vite" />
</template>

<style>
  h2{color:red}
</style>
```

**main.js**

```js
import {createApp,h} from "vue";
import App from './App.vue';
createApp(App).mount('#app');
```

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

**vite.js**

```js
const Koa = require('koa');
const app = new Koa();
const fs = require("fs");
const compilerSFC = require('@vue/compiler-sfc');//如果不存在可以用npm安装一下
const compilerDOM = require('@vue/compiler-dom');
app.use(async ctx => {
  const {url,query} = ctx.request;
  // koa-static 可以
  if(url === '/'){//请求首页,返回 index.html
    ctx.type = 'text/html';
    ctx.body = fs.readFileSync('./src/index.html','utf8');
  }else if(url.endsWith('.js')){//请求main.js
    //拼接文件服务器地址
    const p = path.join(__direname,url);
    ctx.type = 'application/javascript';
    //读取文件内容，并将文件中的裸模块地址替换掉，因为浏览器加载js代码中的 import xx from 'xx'; 的时候并不认识这个模块，它只认识地址，所以我们要将模块替换为地址，让import时也发起请求获取内容，然后再在服务器进行处理
    ctx.body = rewriteImport(fs.readFileSync(p,'utf8'));
  }else if(url.startsWith('/@modules/')){//处理替换后的第三方模块
    //获取裸模块名称
    const moduleName = url.replace('/@modules/','');
    //获取裸模块在 node_modules 中的路径
    const prefix = path.join(__dirname,'../node_modules',moduleName);
    //从 package.json 的 module 字段获取该模块打包编译后的 模块地址
    const module = require(prefix + '/package.json').module;
   	const filePath = path.join(prefix,module);
    const ret = fs.readFileSync(filePath,'utf8');
    ctx.body = rewriteImport(ret);
  }else if(url.indexOf('.vue') > -1){//处理vue 文件，将vue文件处理成js
    // 读取 vue 文件，解析为 js，css，html,使用 @vue/compiler-sfc 来完成
    let filePath = path.join(__dirname,url.split('?')[0]);
    let ret = compilerSFC.parse(fs.readFileSync(filePath,'utf8'));
    if(!query.type){
      //脚本部分内容
    	let scriptContent = ret.discriptor.script.content;
      //替换掉默认导出为一个常量，方便后续修改
      const script = scriptContent.replace("export default","const __script = ");
      ctx.type = "application/javascript";
      ctx.body = `
				//将 <script> 中的 import 也替换一下 
				${rewriteImport(script)}
				//解析 tpl,这里用到的经过编译转换后得到的 render 函数
				import { render as __render } from '${url}?type=template'
				// render 方法最终将会被 vue 调用
				__script.render = __render
				// 导入解析后的 style
				import '${url}?type=style'
				export default __script
			`
    }else if(query.type === 'template'){
      //template部分内容
      let tpl = ret.discriptor.template.content;
      //通过 @vue/compiler-dom 来将 template 进行编译成 render 函数
      const render = compilerDOM.compiler(tpl,{mode:'module'}).code;
      ctx.type = 'application/javascript';
      ctx.body = rewriteImport(render);
    }else if(query.type === 'style'){
      //style部分内容,sfc 解析出来的对象里面有 styles 是一个数组里面有每一个样式
      let styles = ret.discriptor.styles;
      let css = styles.reduce((pre,next)=>pre+next,'');
      ctx.type = 'text/css';
      ctx.body = `
        const style = document.createElement('style');
        style.setAttribute('type','text/css');
        style.innerHTML = \`${css}\`;
        document.head.appendChild(style);
			`;
    }
  }
})

/**
* 裸模块（第三方模块）地址重写，import xx from 'xx' => import xx from '/@modules/xx'
*/
function rewriteImport(content){
  // 在Vite中是使用 es-module-lexer 进行import的语法解析的
  // magic-string 可以将一个字符串当成对象来进行处理，方便处理字符串
  return content.replace(/ from ['"](.*)['"]/g,function(s,s1){
    if(s1.startsWith('/') || s1.startsWith('./') || s1.startsWith('../')){
      return s;
    }else{
      return `from '/@modules/${s2}'`
    }
  })
}

app.listen('3000',()=>{
  console.log("vite server runing in \n http://localhost:3000");
})
```

[Vite工作原理和手写实现「基本完结」](https://www.bilibili.com/video/BV1dh411S7Vz?p=7&vd_source=41ed998ac767425fb616fd9071ce9682)
