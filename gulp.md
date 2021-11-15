# gulp

gulp 和 webpack 、browserify 、 grunt 一样，也是自动化构建工具，和 webpack 以及 grunt 相比，gulp 更加简单易用，不用配置一大堆的东西就可以实现文件的打包、压缩、合并等操作。gulp 是是一个基于流和任务的工具，gulp拥有独立的处理文件数据的内存，在工作的时候会将需要的文件数据读取到内存中，再对这些文件数据做处理操作，最后已数据流的形式输出到指定的地方。

## gulp 中一些常用的API

+ `gulp.src(globs)`：读取/输出（Emits）符合所提供的匹配模式（glob）或者匹配模式的数组（array of globs）的文件。
	+ `./a/b/index.html`：找到a/b/index.html
	+ `./a/b/*.html`：找到a/b下的所有 .html 文件
	+ `./a/**`：找到 a 下所有文件，不含子目录
	+ `./a/**/*`：找到 a 下所有文件，含子目录
	+ `./a/**/*.html`：找到 a 下所有 .html 文件，含子目录
+ `gulp.dest(path)`：能被 pipe 进来，并且将会写文件。并且重新输出（emits）所有数据，因此你可以将它 pipe 到多个文件夹。如果某文件夹不存在，将会自动创建它。
+ `gulp.task(name,fn)`：创建一个gulp任务
+ `gulp.watch(glob [, opts], tasks) | gulp.watch(glob [, opts, cb]) `：监视文件，并且可以在文件发生改动时候做一些事情。它总会返回一个 EventEmitter 来发射（emit） `change` 事件。

## gulp 中一些常用的插件

+ `gulp-concat`：合并文件 js/css
+ `gulp-uglify`：压缩js文件，但是不能压缩es6语法
+ `gulp-rename`：文件重命名
+ `gulp-less`：编译less
+ `gulp-sass`：编译sass，需要注意的是sass还需要依赖一个`node-sass`，安装`npm i node-sass --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/`
+ `gulp-clean-css | gulp-cssmin`：压缩css
+ `gulp-autoprefixer`：自动添加浏览器前缀
+ `gulp-htmlmin`：压缩html文件
+ `gulp-livereload`：实时自动编译刷新
+ `gulp-babel`：将js文件中的es6语法转化成es5的语法，[使用方法](https://www.cnblogs.com/jiaoshou/p/12189356.html)
	+ gulp 3 一般用 gulp-babel@7，gulp 4 一般用 gulp-babel@8，并且除了这 gulp-babel 之外，还需要下载`@babel/core @babel/preset-env`
+ `gulp-load-plugins`：将 `gulp` 的插件集合在一起。
+ `del`：用于删除目录
+ `gulp-webserver|gulp-connect`：开启本地服务
+ `gulp-file-include`：html文件组件导入

## gulp 3

## 安装 3.9.1

```shell
npm i gulp@3.9.1 -g
npm i gulp@3.9.1 -D
```

## 使用

先创建一个项目目录

```
--gulp_test
	--dist
	--src
		--js
			--test
				--1.js
			--2.js
			--3.js
		--less
		--css
		--index.html
	--gulpfile.js
```

### 对 js 进行处理

现在有个需求时要将js中所有的 js 文件都合并到一个文件中，并压缩重命名，转化es6语法，并输出到 dist 目录下，这是就可以使用`gulp-concat`插件。

先安装 `gulp-concat`：`npm i gulp-concat gulp-rename gulp-uglify gulp-babel@7 @babel/core @babel/preset-env -D`；

```js
var gulp = require("gulp");
var concat = require("gulp-concat");
var rename = require("gulp-rename");
var uglify = require("gulp-uglify");
var babel = require("gulp-babel");
//注册转换js文件的任务
gulp.task("js",function(){
    return gulp.src("src/js/**/*.js")  //找到js目录下所有目录的 js 文件
    	.pipe(concat('build.js')) // 在内存中合并文件，并命名问build.js
    	.pipe(gulp.dest("dist/js/")) // 将合并好后的文件输出到 dist/js/ 中
    	.pipe(babel({presets:["es2015"]})) //处理es6语法，如果使用gulp-babel@7配置是{presets:["es2015"]}，如果是gulp-babel@8 则是 {presets:["@babel/env"]}
    	.pipe(uglify()) //压缩文件
    	.pipe(rename({suffix:'.min'})) //重命名
    	.pipe(gulp.dest("dist/js/")) //继续输出文件到 dist/js/
})

//导出默认任务
gulp.task("default",[]);
```

写好之后可以使用`gulp 任务名`命令来执行任务。`gulp js`

### 对 css 进行处理

现在有需求要将 `.less`文件编译成 css 之后，并添加浏览器前缀，和 css 目录下的所有 css 文件进行合并。这里需要使用到`gulp-clean-css gulp-less gulp-autoprefixer`

安装：`npm i gulp-clean-css gulp-less gulp-autoprefixer -D `

```js
var gulp = require("gulp");
var concat = require("gulp-concat");
var rename = require("gulp-rename");
var less = require("gulp-less");
var cleanCss = require("gulp-clean-css");
var autoprefixer  = require("gulp-autoprefixer ");

//注册转换less文件的任务
gulp.task("less",function(){
    return gulp.src("src/less/*.less")  //找到less目录下所有目录的 less 文件
    	.pipe(less()) // 使用less 插件将 less 转化成 css
    	.pipe(gulp.dest("src/css/")) // 将转化后的文件输出到 src/css/ 中
})

//注册转换 css 文件的任务
gulp.task("css",function(){
    return gulp.src("src/css/*.css")  //找到 css 目录下所有目录的 css 文件
    	.pipe(concat("build.css")) // 将所有 css 文件合并成一个文件 build.css
    	.pipe(autoprefixer({browsers:["last 2 versions","firefox < 20","IOS < 7"]}))//设置自动添加浏览器前缀，规则也可以在 package.json 中添加 "browserslist":["last 2 versions","firefox < 20","IOS < 7"]
    	.pepe(rename({suffix:".min"})) //重命名
    	.pipe(cleanCss({compatibility:"ie8"})) // 将文件进行压缩，并兼容到ie8
    	.pipe(gulp.dest("dist/css/")) // 将转化后的文件输出到 dist/css/ 中
})

//导出默认任务
gulp.task("default",["less","css"]);
```

先执行任务`gulp less`，然后再执行`gulp css`。其实也可以直接使用`gulp`去一次性执行多个任务。

**需要注意的是**：gulp 的任务可以是异步的也可以使用同步的，这取决于 gulp.task 的回调函数是否有 `return`，如果有就是异步的，如果没有就是同步的。

**任务依赖**

上面在使用`gulp`命令开启多个任务的时候可以会遇到一个问题，就是当 less 任务还是没有执行完成的时候，可以就执行了css任务，这样我们可能得不到正确的结果，这时候可以使用 任务的第二个参数——任务依赖

```js
gulp.task("css",["less"],function(){
    return gulp.src("src/css/*.css")  //找到 css 目录下所有目录的 css 文件
    	.pipe(concat("build.css")) // 将所有 css 文件合并成一个文件 build.css
    	.pepe(rename({suffix:".min"})) //重命名
    	.pipe(cleanCss({compatibility:"ie8"})) // 将文件进行压缩，并兼容到ie8
    	.pipe(gulp.dest("dist/css/")) // 将转化后的文件输出到 dist/css/ 中
})
```

这样就会先等 less 任务执行完之后才会这些 css 任务。其实也可以将相关的任务改成同步执行的任务，不过这样可能等待的时间会比较就。

### 处理 html 文件

可以使用 `gulp-htmlmin` 对 html 文件进行压缩

```js
var gulp = require("gulp");
var htmlmin = require("gulp-htmlmin");

//注册转换html文件的任务
gulp.task("html",function(){
    return gulp.src("src/index.html")  //找到 src 目录下的 index.html 文件
    	.pipe(htmlmin({
        	collapseWhitespace:true,//去除空格和换行
        	minifyCss:true,//压缩内嵌式css
        	minifyJS:true,//压缩内嵌js
    	})) // 使用 htmlmin 插件将 html压缩
    	.pipe(gulp.dest("dist/")) // 将转化后的文件输出到 dist/ 中
})

//导出默认任务
gulp.task("default",["html"]);
```

需要注意的是，导出到dist下的html文件中对其他 js ，css 文件的引用要正确。

### 处理 html 组件化文件

使用组件化开发会提高我们的开发效率，写更少的代码做更多的事，使用`gulp-file-include`插件可以将html代码片段插入到 html 文件的指定位置中，并且还可以传递参数，这个就类似模板的作用。我们可以将重复的 html 代码抽离出来形成一个 模板，这样就可以重复使用。

安装：`npm i gulp-file-include -D`

```js
var gulp = require("gulp");
var htmlmin = require("gulp-htmlmin");
var fileInclude = require("gulp-file-include");
//注册转换html文件的任务
gulp.task("html",function(){
    return gulp.src("src/index.html")  //找到 src 目录下的 index.html 文件
    	.pipe(fileInclude({
        	prefix:"@@",//自定义标识符
        	basepath:"./src/components",//基准目录，组件所在目录
    	})) //根据配置导入html片段
    	.pipe(htmlmin({collapseWhitespace:true})) // 使用 htmlmin 插件将 html压缩
    	.pipe(gulp.dest("dist/")) // 将转化后的文件输出到 dist/ 中
})
```

之后再需要导入 html 片段的地方使用`@@include("路径",参数)`即可导入，路径是相对于`basepath`的，所，参数是对象`{key:value}`在片段中可以用`@@key`接收。

将一个代码抽离成 head.html ,header.html，footer.html，script.html。如果多个页面只有内容不一样还还可以只将内容区域抽取出来，之后根据需求填充。

```html
<!-- head.html -->
<div>
    这里可以放 title ，link ,meta等
    <title>@@title</title>
</div>

<!-- header.html -->
<div>
	<span class="@@color">这里可以放 nav search 等</span>
</div>

<!-- footer.html -->
<div>这里可以放友情链接，帮助，网站信息等</div>

<!-- script.html -->
<script>
	console.log("hello world");
</script>
```

然后有主页 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    @@include("./head.html",{title:"index"})
</head>
<body>
    @@include("./header.html",{color:"red"})
    <div>
        内容
    </div>
    @@include("./footer.html")
    @@include("./script.html")
</body>
</html>
```



### 处理 视频 、 图片、第三方库

对于 视频 、 图片、第三方库 的处理，只需要将图片移动到指定的目录即可。

```js
var gulp = require("gulp");

//注册转换 image 文件的任务
gulp.task("image",function(){
    return gulp.src("src/img/**")
    	.pipe(gulp.dest("dist/img"))
})
//注册转换 image 文件的任务
gulp.task("video",function(){
    return gulp.src("src/videos/**")
    	.pipe(gulp.dest("dist/videos"))
})
//注册转换 image 文件的任务
gulp.task("lib",function(){
    return gulp.src("src/lib/**")
    	.pipe(gulp.dest("dist/lib"))
})

//导出默认任务
gulp.task("default",["html"]);
```

### 删除目录任务

gulp 在打包的时候并不会请求目标目录，而是直接将文件放过去，这就导致如果修改了原文件并且在处理的时候没有名的话，新打包出来的文件目录中还会有就文件目录。可以使用`del`插件在打包之前先将目标目录清空。

```js
var gulp = require("gulp");
var del = require("del");

gulp.task("del",function(){
    return del["./dist/"];
})

//注册转换 image 文件的任务
gulp.task("image",["del"],function(){
    return gulp.src("src/img/**")
    	.pipe(gulp.dest("dist/img"))
})

//导出默认任务
gulp.task("default",["image"]);
//在 gulp 4 中可以
//module.exports.default = gulp.series(
//	del,
//    gulp.parallel(任务1,任务2,....)
//)
```



### 半自动实时刷新

可以使用`gulp-livereload`插件配合`gulp.watch()`方法配置一个监视任务，当文件有变动的时候自动执行任务。

```js
var gulp = require("gulp");
var htmlmin = require("gulp-htmlmin");
var livereload = require("gulp-livereload");

//注册转换html文件的任务
gulp.task("html",function(){
    return gulp.src("src/index.html")  //找到 src 目录下的 index.html 文件
    	.pipe(htmlmin({collapseWhitespace:true})) // 使用 htmlmin 插件将 html压缩
    	.pipe(gulp.dest("dist/")) // 将转化后的文件输出到 dist/ 中
    	// 实时刷新 ---------每个被监视的任务都需要添加
    	.pipe(livereload()) 
})

// 创建半自动监视任务
gulp.task("watch",["default"],function(){
    // 开启监听
    livereload.listen();
    // 确认监听的目标以及绑定的相应任务
    gulp.watch("src/js/**/*.js",["js"]);
    gulp.watch(["src/less/*.less","src/css/*.css"],["css"]);
})

//导出默认任务
gulp.task("default",["html"]);
```

之后可以使用`gulp watch`执行任务。

### 全自动实时刷新

半自动实时刷新在编译完成之后，还是需要自己手动去刷新页面才能看到最新的效果，并且访问页面的方式还是本地文件的方式。全自动实时刷新就是在本地搭建一个服务器，在文件发送改变的时候会自动编译并且会自动刷新页面。

需要安装`gulp-connect`

```js
var gulp = require("gulp");
var htmlmin = require("gulp-htmlmin");
var livereload = require("gulp-livereload");
var connect = require("gulp-connect");

//注册转换html文件的任务
gulp.task("html",function(){
    return gulp.src("src/index.html")  //找到 src 目录下的 index.html 文件
    	.pipe(htmlmin({collapseWhitespace:true})) // 使用 htmlmin 插件将 html压缩
    	.pipe(gulp.dest("dist/")) // 将转化后的文件输出到 dist/ 中
    	// 实时刷新 ---------每个被监视的任务都需要添加
    	.pipe(livereload()) 
    	// 实时刷新 作用和 livereload 一样。
    	.pipe(connect.reload()) 
})

// 创建全自动监视任务
gulp.task("server",["default"],function(){
    //配置服务器选项
    connect.server({
        root:"dist/",//服务根目录
        livereload:true,//实时刷新
        port:3000,//端口号
    })
    // 确认监听的目标以及绑定的相应任务
    gulp.watch("src/js/**/*.js",["js"]);
    gulp.watch(["src/less/*.less","src/css/*.css"],["css"]);
})

//导出默认任务
gulp.task("default",["html"]);
```

也可以使用`gulp-webserver`开启服务

```js
gulp.task("server",["default"],function(){
    return gulp.src("./dist")
    .pipe(webserver({
        host:"www.test.com",//如果想要使用自定义域名的话可以在 hosts 中定义 127.0.0.1   www.test.com
        livereload:true,//实时刷新
        port:3000,//端口号
        open:"./dist/index.html",
        proxies:[
            {
                source:"/api1",//代理标识符 ，如访问 getData 可以写成 /api/getData
                target:"www.test1.com/",//代理地址
            },
            {
                source:"/api2",
                target:"www.test2.com/",
            },
        ],// 用于解决跨域，如果没有带了不要设置
    }))
    // 确认监听的目标以及绑定的相应任务
    gulp.watch("src/js/**/*.js",["js"]);
    gulp.watch(["src/less/*.less","src/css/*.css"],["css"]);
})
```



### 插件打包集合

上面每个 gulp 插件都需要单独使用`require`进行引入，这样比较麻烦，可以使用`gulp-load-plugins`将全部的`gulp`插件集合在一起，默认的名字就是`gulp`后的名字，多个名字用小驼峰命名法命名。

```js
var gulp = require("gulp");
var $ = require("gulp-load-plugins")();
//注册转换html文件的任务
gulp.task("html",function(){
    return gulp.src("src/index.html")  //找到 src 目录下的 index.html 文件
    	.pipe($.htmlmin({collapseWhitespace:true})) // 使用 htmlmin 插件将 html压缩
    	.pipe(gulp.dest("dist/")) // 将转化后的文件输出到 dist/ 中
})
```



## gulp 4 中的变化

**新增API**

+ `gulp.series(任务1，任务2.....)`：逐个执行多个任务，返回的是一个函数
+ `gulp.parallel(任务1，任务2.....)`：并发执行多个任务，返回的是一个函数

在 gulp 4 中同步任务和异步任务不再通过`gulp.task`的回调函数中是否`return`决定，现在定义同步任务或异步任务通过`series` 和`parallel`实现，`return`则表示任务的结束。

**定义任务的变化**

在 gulp 4 中定义任务可以不使用`gulp.task`定义，可以直接定义一个函数来代表任务，需要导出后才可用

```js
var gulp = require("gulp");
var cssmin = require("gulp-cssmin");

var cssHandle = function(){
    return 
    	gulp.src("./src/css/*.css")
    	.pipe(cssmin())
    	.pipe(gulp.dest("dist/css/"))
}
module.exports.cssHandle = cssHandle;
```

执行任务`gulp cssHandle`

**导出默认任务**

可以使用`series`或者`parallel`将任务集合在一起导出，然后执行`gulp default`

```js
module.exports.default = gulp.parallel(任务函数1,任务函数2,.....);
```

一般默认的任务顺序应该是 `清理文件 -> 并发任务 -> 开启服务 -> 开启监控`

```js
module.exports.default = gulp.series(
	清理文件任务,
    并发任务(gulp.parallel),
    开启服务,
    开启监控
);
```