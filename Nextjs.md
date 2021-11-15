# Nextjs



## 什么是客户端渲染？

> 数据由浏览器用过 ajax 动态获得，再通过js将数据填充到 DOM 上最终展示到网页上。



## 什么是服务端渲染？

> 后端先调用数据库，获得数据之后，将数据和页面元素进行拼装，组合成完整的 Html 页面，再直接返回给浏览器，以便用户浏览



## Nextjs 优点

> 1. 搭建轻松，不同在自己配置一大堆的配置，特别是webpack 的配置。
> 2. 自带数据同步，数据再服务端渲染好之后会自动同步到客户端上
> 3. 有丰富的插件，比如 markdom 插件
> 4. 灵活配置



## 项目搭建

### 手动自己搭建

1. 创建项目目录 `react/next/nextDemo`
2. 使用`npm init -y`初始化项目
3. 安装相关库`npm i react react-dom next -S`
4. 在生成的`package.json`中配置脚本命令

```js
"scripts":{
  "dev":"next",
  "build":"next build",
  "start":"next start"
}
```

5. 在 `pages/`下直接创建组件文件即可，nextjs将会自动生成路由配置

```jsx
function Index(){
  return (
  	<div>Hello Nextjs</div>
  )
}
export default Index;
```

6. 运行`npm run start`（第一次运行需要先`npm run build`）



### 使用 create-next-app 脚手架搭建

使用 `create-next-app` 脚手架，可以快速搭建 next 项目

1. 安装 `npm i create-next-app -g`
2. 创建 `npx create-next-app next-demo`



## 路由跳转

在 next 中也是可以使用`标签跳转`和`编程跳转`的，比如`pages/index.js`跳转到`pages/a.js`或者`pages/b.js`下，然后再返回

### 使用标签式路由

pages/index.js

```jsx
import React from "react";
import Link from "next/Link";
const Home = ()=>{
  return (
  	<>
    	<h2>home page</h2>
    	<div><Link href="/a"><a>to page a</a></Link></div>
    	<div><Link href="/b"><a>to page b</a></Link></div>
    </>
  )
}
```

pages/a.js 和 pages/b.js 

```jsx
import Link from "next/Link";
const PageA = ()=>{
  return (
  	<>
    	<h2>page B</h2>
    	<div><Link href="/"><a>to Home</a></Link></div>
    </>
  )
}

import Link from "next/Link";
const PageB = ()=>{
  return (
  	<>
    	<h2>page B</h2>
    	<div><Link href="/"><a>to Home</a></Link></div>
    </>
  )
}
```

**注意**：`<Link>`标签只能有一个子标签`<a>`，而且和 react-router 不同，这里不能使用`to`而要使用`href`;



### 使用编程式路由

可以通过 `next/router`引入 `Router`，或者通过`useRouter`Hooks 获得 router 对象，使用路由的方法 `Router.push`达到路由跳转的效果。

```jsx
import React from "react";
import Link from "next/Link";
import Router from "next/router";
const Home = ()=>{
  return (
  	<>
    	<h2>home page</h2>
    	<div><Link href="/a"><a>to page a</a></Link></div>
    	<div><Link href="/b"><a>to page b</a></Link></div>
    	<div>
      	<button onClick={()=>{Router.push("/a")}}>router to page a</button>
    	</div>
    </>
  )
}
```



### 动态路由跳转

根据路由的参数显示不同的内容，简单来说就是路由的传参，在 Next 中路由的传参只能使用 `query`（?id=1）传递。通过 `next/router`的 `withRouter`处理组件后，组件接收的 `props`就会有一个 `router` 对象，里面存放着路由的相关信息。

pages/index.js

```jsx
import React from "react";
import Link from "next/Link";
import Router from "next/router";
const Home = ()=>{
  function showPage(){
    Router.push("/showRoute?name=b");
    /**
      Router.push({
        pathname:"/showRoute",
        query:{
          name:"b",
        }
      });
    */
  }
  return (
  	<>
    	<h2>home page</h2>
    	<div><Link href="/showRoute?name=a"><a>to showRoute</a></Link></div>
    	<div><Link href={{pathname:"/showRoute",query:{name:"c"}}}><a>to showRoute</a></Link></div>
    	<div><button onClick={showPage}>to showRoute</button></div>
    </>
  )
}
```

pages/showRoute.js

```jsx
import Link from "next/Link";
impott {withRouter} from "next/router";
const showRoute = ({router})=>{
  return (
  	<>
    	<h2>get route value:{router.query.name}</h2>
    	<div><Link href="/"><a>to Home</a></Link></div>
    </>
  )
}
export default withRouter(showRoute);
```

除了上面的这种连接格式进行跳转外，还可以使用`/list/[id]`的格式进行传递，并且是可以传递一到多个参数的。先创建`pages/list/[id].js`和`pages/list/[...id].js`

```jsx
//单个参数
<Link href="/list/[id]" as="/showRoute/1"></Link>
Router.push("/list/[id]","/showRoute/1");

//多个参数
<Link href="/list/[...id]" as="/showRoute/1/2/3"></Link>
Router.push("/list/[...id]","/showRoute/1/2/3");
```

使用`/showRoute/[id]`这种格式的标签路由跳转需要配置 `as`属性使用。接收参数的方式也是一样的使用`router.query.id`

### 路由预加载

为了提升页面的加载速度，可以在进入页面的时候 预加载 可能需要的页面，在 next 中提供了一个 预加载 的功能，将所需的资源提前请求到本地，这样后面再需要用的时候直接从缓存中读取数据即可。

可以在 `<Link>`中使用 `prefetch`属性开启预加载，也可以在`Router.prefetch()`方法实现

```jsx
<Link href="/" prefetch><a>page</a></Link>
```

```jsx
Router.prefetch("/about");
```

**注意**：只在生产环境中有效



### 自定义404和错误页面

可以在`pages/`创建`404.js`页面，这是个静态页面；可以创建`_error.js`错误页面，在 props 中有`statusCode`属性可以知道错误信息。



### 路由的钩子函数

在 next 中有很多的路由钩子函数，这些钩子函数可以帮我们实现一些在路由变化是的处理，使用方法是`Router.events.on("钩子名",fn)`。

+ routeChangeStart：路由变化前执行
+ routeChangeComplete：路由变化结束执行
+ beforeHistoryChange：路由变化时执行
+ routeChangeError：路由发生错误时执行
+ hashChangeStart：哈希模式（#）下路由变化前执行
+ hasChangeComplete：哈希模式（#）下路由变化结束执行

```jsx
import Link from "next/Link";
impott Router,{withRouter} from "next/router";
const showRoute = ({router})=>{
  Router.events.on("routeChangeStart",(...args)=>{
    console.log("1-routeChangeStart",args);
  })
  return (
  	<>
    	<h2>get route value:{router.query.name}</h2>
    	<div><Link href="/"><a>to Home</a></Link></div>
    </>
  ) 
}
export default withRouter(showRoute);
```



## 获取远端数据

在 next 中可以用个几个方法可以在服务端获取数据并且渲染到页面上的，`getInitialProps`和`getServerSideProps`启用的是服务端渲染（每次请求页面都会发送请求请求新数据），`getStaticProps`静态渲染（在build的时候获取到数据，然后渲染生成静态的html）。

**服务端渲染**：访问xxx路由之前，向服务器请求数据，把要回来的数据和HTML加工直接返回前端显示。

**静态化**：访问xxx路由之前，向服务端请求数据，将请求来的数据和HTML加工生成真正的xxx.html文件，在下次访问同一个路由地址的时候，直接返回静态页面，减小服务器的压力，已达到性能优化的目的。

`getServerSideProps(context)`：不静态化，异步async，返回值为一个对象（必须有props属性，且会变成组件的props），因为这样方法是在服务端执行的，所以也可以在这个方法中做读取文件操作

+ params：接收`getStaticPaths()`返回的动态路径，方便访问动态数据`/list/xxxx`
+ req：请求对象
+ res：响应对象
+ query：查询字符串

`getStaticPaths()`：会静态化，异步async，用法与`getServerSideProps`一样

```jsx
export default ()=>{
  return (
  	<div>sdf</div>
  )
}

export const getStaticPaths = async ()=>{
  let res = await axios("http:localhost:8000/getList");
  let paths = data.map(item=>(
  	{params:{list:`${item.id}`}}
  ))
  return {
    paths,fallback:false,
  }
}

export const getStaticProps = async ({params:{list}}) =>{
  let res = await axios(`http:localhost:8000/getList/${list}`);
  return {
    props:{res}
  }
}
```

这些方法都只能用在`pages/`下的页面中。

这几个方法都是组件的静态方法，所以可以这样使用

```jsx
import Link from "next/Link";
impott Router,{withRouter} from "next/router";
const showRoute = ({router})=>{
  return (
  	<>
    	<h2>get route value:{router.query.name}</h2>
    	<div><Link href="/"><a>to Home</a></Link></div>
    </>
  ) 
}
//showRoute.getInitialProps = ...
export const getInitialProps = async ()=>{
  return await axios("http:localhost:8000/getList");
}
export default withRouter(showRoute);
```

使用 `withRouter`后，在远端获取到的数据将会被合并到 props 中，所以可以直接在组件的 props 取出数据进行渲染。

**注意：**`getInitialProps ` 什么时候会在浏览器端调用呢？

> 当在单页应用中做页面切换的时候，比如从 Home 页切换到 Product 页，这时候完全和服务器端没关系，只能靠浏览器端自己了，Product页面的 getInitialProps 函数就会在浏览器端被调用，得到的数据用来开启页面的 React 原生生命周期过程。



## 使用 style jsx 编写样式

### 全局样式配置

可以在`pages/`下新建一个`_app.js`文件，这个就是 next 给我们来进行全局事件的管理的，在`_app.js`中的 props 有两个比较特别的属性`Component,pageProps`，`Component`是指当前访问的组件，`pageProps`是当前访问组件的`props`

```jsx
import ".../style.css"
export default ({Component,pageProps}) => {
  <Component {...pageProps}></Component>
}
```



### 局部样式

在 next 中不允许通过 `import “a.css” `（9.3之后得到版本已经支持这个引入）的方式进行样式文件的使用，可以在使用 `style jsx`语法进行样式的编写，但是这个是局部的样式，不在全局范围起效。

```jsx
import css from "./index.module.css";
function StylePage(){
  let [textColor,setTextColor] = useState("yellow");
  return (
  	<div>hello world</div>
   	<div className="virtual">nextjs is very good</div>
    <div className="useVar">use color variable</div>
    <div className={css.color}>use import css</div>
    <div style={{color:"skyblue"}}>use inline css</div>
    <style jsx>
    	{`
				div{color:blue;}
				.virtual{color:red}
				.useVar{color:${textColor}}
			`}
    </style>
    <style global jsx>
    	{`
				body:{background:#000}
			`}
    </style>
  )
}
```

`<style jsx>`可以通过 `global` 定义里面的样式是放到全局的



## 模块懒加载

当项目越来越大的时候，模块的加载是需要管理的，如果不管理会出现首次打开过慢，页面长时间没有反应一系列问题。这时候可用`Next.js`提供的`LazyLoading`来解决这类问题。让模块和组件只有在用到的时候在进行加载。

### 模块懒加载

可以使用`import()`，实现模块的懒加载，比如使用 `moment`时间处理库

```jsx
function TimeFotmation(){
	const [nowTime,setTime] = useState(Date.now());
  const changeTime = async ()=>{
    const moment = await import("moment");
    setTime(moment.default(Date.now()).format());
  }
  return (
  	<div>
    	<h2>显示事件为：{nowTime}</h2>
      <br></br>
      <button onClick={changeTime}>改变事件格式</button>
    </div>
  )
}
```

使用 `import()`函数可以动态地加载模块，返回的是一个 promise 对象。

### 路由组件懒加载

可以使用 `next/dynamic`的`dynamic`配置`import()`实现路由组件的懒加载

```jsx
import dynamic from "next/dynamic";
const One = dynamic(import("./one"));
```

上面的 `one`组件将会在被使用的时候才会被加载进来`<One/>`



## 自定义 Head

在 next 中，为了让项目有更好 SEO ，一般都是要自定义 head 的内容，标明页面的内容以及关键子等。可以使用`next/head`中的`Head`组件进行定义。

```jsx
import Head from "next/head";
function Header(){
  return (
  	<div>
    	<Head>
      	<title>test page</title>
        <meta charset="utf-8"></meta>
      </Head>
    </div>
  )
}
```

如果需要全局使用的话，可以将 `Head`组件再包装成自定义的组件。然后在需要使用的时候调用一下就好了。



## 使用第三方UI组件库

由于 Next 并不支持`import “a.css”`（9.3之后得到版本已经支持这个引入），所以在 next 项目中使用第三方UI库的时候需要做一下特殊处理。

比如说 使用`antd`UI库，先安装必要的库

```shell
npm i @zeit/next-css antd babel-plugin-import
```

`@zeit/next-css` 是使项目支持`import “a.css”`这种方式，`babel-plugin-import`则是为了使用按需加载功能。

配置`.babelrc`

```json

{
    "presets": [
        "next/babel"
    ],
    "plugins": [
        [
            "import",
            {
                "libraryName": "antd",
                "libraryDirectory":"lib",
                "style": "css"
            }
        ]
    ]
}
```

配置`next.config.js`

```js
const withCss = require('@zeit/next-css')
if(typeof require !==  'undefined'){
    require.extensions['.css']=file=>{}
}
module.exports = withCss({});
```



## api路由

在 next 中不仅有前端页面的路由，还有api路由。api路由是专门用来处理数据的。

在`pages/api/xxx.js`，就可以创建一个api路由文件了，

```js
export default (req,res)=>{
  
}

```

访问：`axios("http://localhost:3000/api/xxx")`

req：

+ req.method 请求方式
+ req.body 经过处理解析成字符串的数据
+ req.query 请求参数
+ req.cookie cookie

res：

+ res.statusCode 设置状态吗
+ res.setHeader 设置响应头
+ res.end() 发送数据
+ res.status(code) 设置状态码，必须是有效的
+ res.json(json) 发送json数据响应
+ res.send(body) 发送http响应，body可以是 string、Object、buffer



### 动态api路由

单个参数：[xxx].js，多个参数：[…xxx].js；接收：`req.query`

优先级：/xxx.js > /[xxx].js > […xxx].js

访问：`axios("http://localhost:3000/api/xxx/123")`；



### 中间件

和 `express`、`koa`一样，在 next 的api路由中是可以使用中间件的

```jsx
app.use((req,res,next)=>{});
```

在 next 中已经提供有很多的中间件了，如 `body-parser`



### 请求转发

为了接请求跨域的问题，如果让后端来处理跨域的话是很不安全的。在 next 中可以使用api路由的请求转发来解决跨域问题。

如我现在前端项目中的地址是 `http://localhost:3000`，要请求的服务端是`http://localhost:9000`。

请求服务端`http://localhost:9000/list`。现在本地创建一个`pages/api/list.js`

```jsx
export default async (req,res)=>{
  let data = await axios("http://localhost:9000/list",{
    method:"POST",
    body:JSON.stringify,
    headers:{
      "Content-Type":"application/json"
    }
  })
  res.setHeader("Content-Type","application/json");
  res.status(200);
  res.json(data);
}
```

