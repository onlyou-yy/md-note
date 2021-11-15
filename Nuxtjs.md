# Nuxtjs



## 为什么要用 Nuxtjs

Nuxtjs 是 vuejs 的一个服务端渲染（SSR：server side render）框架，这种服务端渲染就是对 在服务端将 vue 编译讯渲染后成 html 之后在返回给浏览器，这样做的好处就是**有利于 SEO**。因为使用vue开发的应用基本上所有的 html 的 DOM 元素都是动态生成的，这样也就导致了，当查看源码的时候 html 上就只有几行代码而已，并且都是对 js 文件的引用，而浏览器的爬虫是不会进入到 js 文件中查看具体代码的，因此是不利于做SEO的。



## 哪些应用 / 网站适合使用 SSR。

如果应用 / 网站需要通过浏览器做引流的就需要使用SSR，如新闻、博客、电影、官网等网站或者应用。

在以前也是有SSR 的，并且以前的应用更加广，以前是可以使用 php、J2EE 这些技术做的，但是后来应为 前后端分离开发的趋势越来越大，采用这种方式进行开发的人也越来越多，后面就逐渐少了。



## Nuxt.js是特点（优点）：

- 基于 Vue.js
- 自动代码分层
- 服务端渲染
- 强大的路由功能，支持异步数据
- 静态文件服务
- ES6/ES7 语法支持
- 打包和压缩 JS 和 CSS
- HTML头部标签管理
- 本地开发支持热加载
- 集成ESLint
- 支持各种样式预处理器： SASS、LESS、 Stylus等等



## 环境配置

需要使用 vue/cli 和 nuxtjs 

### 安装 vue/cli

```shell
## vuecli3 安装方式
npm i @vue/cli -g
## vuecli2 安装方式
npm i vue-cli -g
```

安装nuxt就是，有两种方式

1. 使用 npm 直接安装，但是可能需要自己再安装其他必要的包

```shell
npm init nuxt-app <project-name>
```

2. 使用 vuecli 直接创建 nuxt 项目

```shell
vue init nuxt/starter
## 如果是 vuecli3 需要安装一个 @vue/cli-init 才能使用 vuecli2 的命令
```

3. 使用 nuxt 开发的 create-nuxt-app 脚手架

```shell
npx create-nuxt-app <project-name>
# 或者全局安装 create-nuxt-app 后直接 create-nuxt-app <project-name>
```



## nuxt 项目的的目录结构

```
|-- .nuxt                            // Nuxt自动生成，临时的用于编辑的文件，build
|-- assets                           // 用于组织未编译的静态资源入LESS、SASS 或 JavaScript
|-- components                       // 用于自己编写的Vue组件，比如滚动组件，日历组件，分页组件
|-- layouts                          // 布局目录，用于组织应用的布局组件，不可更改。
|-- middleware                       // 用于存放中间件
|-- pages                            // 用于存放写的页面，我们主要的工作区域
|-- plugins                          // 用于存放JavaScript插件的地方
|-- static                           // 用于存放静态资源文件，比如图片
|-- store                            // 用于组织应用的Vuex 状态管理。
|-- .editorconfig                    // 开发工具格式配置
|-- .eslintrc.js                     // ESLint的配置文件，用于检查代码格式
|-- .gitignore                       // 配置git不上传的文件
|-- nuxt.config.json                 // 用于组织Nuxt.js应用的个性化配置，已覆盖默认配置
|-- package-lock.json                // npm自动生成，用于帮助package的统一性设置的，yarn也有相同的操作
|-- package-lock.json                // npm自动生成，用于帮助package的统一性设置的，yarn也有相同的操作
|-- package.json                     // npm包管理配置文件
```



## 关于一些配置项

### 配置开发服务的端口和地址

一般都是在 package.json 中配置 config 项

```
"config":{
	"nuxt":{
		"host":"127.0.0.1",
		"post":"3000"
	}
}
```

这个的意思就是在执行 nuxt 命令的时候加上以上参数 ，如 `npm run dev`对应的是`nuxt`,就是 `nuxt --host 127.0.0.1 –port 3000`



### 关于样式配置

可以在 nuxt.config.js 中配置初始化样式

```js
export default = {
  css:["~assets/css/init.css"] 
}
```

~ ：是根目录的别名

一般设置这个属性是为了添加一些默认的样式，或者是使不同浏览组件的样式差异减小（统一样式），这个也可以使用 github 上现成的库 `normailze.css`(使很多浏览器css效果差异变得最小)



### 关于覆盖 webpack 部分配置

在nuxt.config.js里是可以对webpack的基本配置进行覆盖的，比如现在我们要配置一个url-loader来进行小图片的64位打包。就可以在nuxt.config.js的build选项里进行配置。

```js
build: {
    loaders:[
      {
        test:/\.(png|jpe?g|gif|svg)$/,
        loader:"url-loader",
        query:{
          limit:10000,
          name:'img/[name].[hash].[ext]'
        }
      }
    ],
}
```



## 路由使用

### 路由和参数传递

在 nuxt 中无需自己亲自配置和书写路由配置。nuxt 会根据 pages 目录下的文件夹自行创建路由配置，可以在 .nuxt/router.js 中查看。如 `pages/news/index.vue`，将会被编译成下面这样，并且默认使用的首路由懒加载

```js
{
  name:"news",//表示的是 index.vue
  path:"/news",
  component:"pages/news/index.vue"
}
```

如果有多个路由文件，如 news 下有 news.vue，one.vue，那么对应的路由配置应该是

```js
{
  name:"news",
  path:"/news",
  component:"pages/news/index.vue"
},
{
  name:"news-one",
  path:"/news/one",
  component:"pages/news/one.vue"
}
//需要注意的是 根路由 的表示是
{
  name:"index",
  path:"/",
  component:"pages/index.vue"
}
```





创建 pages/about/index.vue 文件

```vue
<template>
    <div>
        <h3>this is about</h3>
        <nuxt-link to="/">home</nuxt-link>
    </div>
</template>
```

创建 pages/news/index.vue

```vue
<template>
    <div>
        <h3>this is news</h3>
        <h4>newsID:{{$route.params.newsId}}</h4>
        <nuxt-link to="/">home</nuxt-link>
    </div>
</template>
```

那么在导航页 pages/index.vue

```vue
<template>
  <div class="container">
    <ul>
      <li><nuxt-link to="/">home</nuxt-link></li>
      <li><nuxt-link :to="{name:'about'}">about</nuxt-link></li>
      <li><nuxt-link :to="{name:'news',params:{newsId:33}}">news</nuxt-link></li>
    </ul>
  </div>
</template>
```

其实在 nuxt 的路由和 vue-router 都是一样的，不同的是他们的组件名和路由层级结构的区别，也就是在固定的路由生成规则中，应该怎么样写路由的名字，让 nuxt 能够识别并且匹配。



### 动态路由和参数校验

#### 动态路由

如果我们需要在点解某天新闻的时候跳转到指定的新闻的详情，并且使用路由跳转（使用编程式跳转就没有必要了）的时候就需要设置动态路由了。在 nuxt 中将会吧文件夹下的 以 `_` 开头的 vue文件编译成路由参数，如 有`pages/news/index.vue`和`pages/news/_id.vue`文件，将会被解析成

```js
{
  name:"news-id",
  path:"/news/:id?",
  component:"pages/news/index.vue"
}
```

那么 pages/news/_id.vue

```vue
<template>
    <div>
        <h3>this is news detail</h3>
        <h4>news detail:{{$route.params.id}}</h4>
        <nuxt-link to="/">home</nuxt-link>
    </div>
</template>
```

在 pages/news/index.vue

```vue
<template>
    <div>
        <h3>this is news</h3>
        <h4>newsID:{{$route.params.newsId}}</h4>
        <ul>
         	<li><nuxt-link to="/news/1">detail 1</nuxt-link></li> 
          <li><nuxt-link :to="{name:'news-id',params:{id:2}}">detail 2</nuxt-link></li> 
          <li><nuxt-link to="/news/3">detail 3</nuxt-link></li> 
  			</ul>
      	<nuxt-link to="/">home</nuxt-link>
    </div>
</template>
```



#### 参数校验

nuxt 提供了一个路由参数验证的函数 `validate(routeParams)`，接收路由参数对象，在 pages/news/_id.vue

```js
export default {
  validate({params}){
    return /^\d+$/.test(params.id);
  }
}
```

如果返回的是 true 则正常显示页面，如果是 false 则显示 nuxt 准备的错误页面。



### 嵌套路由

关于动态路由的创建安方法可以在[官网](https://www.nuxtjs.cn/guide/routing)查看，也是根据目录结构来生成对应点路由配置。不过在 nuxt 中可以使用 `<nuxt-child/>`，他的作用和 `<router-view>`相似。



**路由动画**

在 vue-router 中可以设置路由的过度动画，在nuxt中也一样可以设置。添加**全局的过度动画**，在`assets/css/normailize.css`中

```css
.page-enter-active,.page-leave-active{
  transition:opacity  2s;
}
.page-enter,.page-leave-active{
  opacity:0;
}
```

需要注意的是，只有在使用 `<nuxt-link>`进行跳转的时候才会生效，使用`<a>`是不会生效的。

如果要为某一特定的路由组件添加过度动画的话可以自定义一下过度

```css
.test-enter-active,.test-leave-active{
  transition:all  2s;
}
.test-enter,.test-leave-active{
  opacity:0;
  font-size:24px;
}
```

比如 为 pages/news/index.vue 添加

```js
export default {
  transition:"test",
}
```



## 默认模板和默认布局

### 默认模板

more模板可以将组件添加的默认的 HTML 中也就是相当于将组件插入到网页中。如果需要使用默认模板可以在根目录中创建 app.html 文件，这个就是 nuxt 默认模板文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    {{HEAD}}
</head>
<body>
    <h2>my nuxt demo</h2>
    {{APP}}
</body>
</html>
```

nuxt.config.js 中已经创建好了 head 数据了，执行使用模板语法引入即可，同理也可以将组件引入。

### 默认布局

使用默认布局也是可以达到和默认模板相同的效果的，只需要在 layouts/default.vue 中定义就好

```vue
<template>
  <div>
    <h2>my nuxt demo</h2>
    <Nuxt />
  </div>
</template>
```



## 设置错误页面和个性meta标签

### 设置错误页面

可以在 layouts 目录下创建 error.vue 进行错误页面的自定义

```vue
<template>
  <div>
      <h2 v-if="error.statusCode==404">404页面不存在</h2>
      <h2 v-else>500服务器错误</h2>
      <ul>
          <li><nuxt-link to="/">HOME</nuxt-link></li>
      </ul>
  </div>
</template>

<script>
export default {
  props:['error'],
}
</script>
```

代码用v-if进行判断错误类型，需要注意的是这个错误是你需要在`<script>`里进行声明的，如果不声明程序是找不到error.statusCode的。



### 单独设置个性meta标签

可以在 nuxt.config.js 中设置 head 中的内容，这个是全局设置的。在 nuxt 中提供了一个 head 方法我们去自定义 html 的 head 数据，并且在每个组件中都可以设置，最终在指定页上显示指定的head，如在 pages/news/index.vue 下

```js
export default {
  data(){
    return {
      title: "hello jack"
    }
  },
  head(){
    return {
      title: this.title,
      meta: [
        {hid:"description",name:"news",content:"this is news page"}
      ]
    }
  }
}
```

hid 相当于一个唯一标识，如果有重复会被覆盖。如果没有就会新增。



## asyncData 异步数据加载

asyncData 方法是nuxt 提供给我们的在组件被初始化前去加载准备数据的方法，但需要**注意**的是由于`asyncData`方法是在组件 **初始化** 前被调用的，所以在方法内是没有办法通过 `this` 来引用组件的实例对象。

asyncData(context,callback) 接收两个参数，第一个是当前主键的执行上下文（包括，请求参数，请求对象，响应对象等），第二个参数相当于是 data 方法和Object.assign 的结合，可以将数据导出到 data 中。但是 asyncData 的返回值应该要是一个 对象，这个对象将会被合并到 data中去

```js
export default {
  data(){
    return {
      title:"hello",
    }
  }
  asyncData(){
    let data = null;				//返回 {title:"world",name:"haha"}
    axios.get("http://localhost:3000/getData").then(res=>{
      data = res.data;
    })
    return data;
  }//之后就可以在模板中 直接使用 title 和 name
}
```



## 静态资源文件和生产静态HTML

一些图片在项目开发时可用，但是打包后就不可用了，这可能是因为自己运行html文件中的原因，nuxt 需要运行在服务端开可以动态的路由请求到响应的资源，本地的话是不可以的。

一般静态资源文件都是放在 /static 下的。在引入的时候可以 使用 `~` 代替根目录，这样方便查找

```css
<div><img src="~static/logo.png" class="diss" /></div>
 .diss{
    width: 300px;
    height: 100px;
    background-image: url('~static/logo.png')
  }
```





[Vue现有项目改造为Nuxt项目](https://segmentfault.com/a/1190000019909396)