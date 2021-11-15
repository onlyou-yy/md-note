# js模块化



## 模块化历史

在很久以前，做项目的时候可能是将全部的 js 逻辑代码，都是放到**同一个文件**进行管理的，这样就导致了，后期代码维护成本高，并且Global被污染，然后命名冲突。后面出现了**简单版本的 namespace** —— 将方法和数据到存到一个对象中，这样减少了 Global 上的变量数，但是极度不安全，应为可以随便访问。再后面使用 **IIFE**  (立即执行函数) 创建局部作用域，在里面编写代码，最后通过 window 暴露，这样就既不会污染 global 也安全，但是代码多了后期维护还是会很麻烦。再在后才就出现了**引用依赖**的方式，这个其实就是在 IIFE 的时候将其他的数据通过 立即执行函数的参数传递进去，这样就实现了功能代码的划分，也是实现现代模块化的基石。

简单版本的 namespace

```js
//foo.js
let obj = {
  msg:"jack",
  foo:function(){
    console.log(this.msg);
  }
}
```

IIFE

```js
(function(){
  let msg = "jack";
  function foo{
    console.log(msg);
  }
  window.foo = {foo}
})()
```

引用依赖

```js
(function(window,$){
  let msg = "jack";
  function foo{
    console.log(msg);
  }
  window.foo = {foo};
  $("body").css("background","red");
})(window,jQuery)
```



## 模块化优点

+ 避免命名冲突（减少命名空间污染）
+ 更好的分离，按需加载
+ 更高复用性
+ 高可维护

通过 `<script />` 引入 js 文件的问题：请求过多，依赖模糊，难以维护



##  模块化规范

### CommonJS

#### 模块的加载

每个文件都可以看做一个模块，在**服务器端**模块的加载是运行时同步加载的（因为着容易出现阻塞的情况），在**浏览器端**模块需要提前编译打包处理（可以使用Browserify）。

#### 模块的基本语法

导出：

```js
module.exports = value;
exports.xxx = value;
//最终暴露的是 exports 对象
```

在 CommonJS 的模块中 默认会 自己导出`module.exports`。

导入

```js
let mod = require("xxx");
//如果自定义模块需要写齐模块的文件路径，如果是第三方模块则字需要写模块名
```



#### 在服务端和浏览器端实现

+ 服务器端实现：因为 nodejs 的模块系统就是基于 CommonJS 的所以可以直接使用
+ 浏览器端实现：需要使用 CommonJS 的浏览器端打包工具 Browserify (除了这个之外还需要使用到 uniq ：可将数组去重，并将按照元素的第一个字符进行排序)



**服务器端**:

有如下目录

```js
-dist
-src
	-module1.js
	-module2.js
	-module3.js
	-app.js
-index.html
-package.json
```

然后代码如下

```js
//module1.js
module.exports = function(){
  console.log("mod1 fun")
}

//module2.js
exports.foo = function(){
  console.log("mod2 foo fun");
}
exports.bar = function(){
  console.log("mod2 bar fun");
}

//module3.js
module.exports = {
  foo: function(){
    console.log("mod3 foo fun");
  }
}
```

导入使用

```js
//app.js
let mod1 = require("./module1");
let mod2 = require("./module2");
let mod3 = require("./module3");

mod1();
mod2.foo();
mod2.bar();
mod3.foo();
```

可以直接运行 app.js `node app.js`

**在浏览器器端**

在 index.html 中引用app.js，然后运行 index.js 会发现 require 报错，因为浏览器端不支持CommonJS 规范，需要使用 Browserify 进行编译。

需要先全局安装 Browserify 然后再局部安装 Browserify 

```shell
npm i Browserify -g
npm i Browserify -D
npm i uniq -S
```

然后使用 Browserify 进行转译

```shell
browserify js/src/app.js -o js/dist/bundle.js
```

将 `js/src` 中的 `app.js` 打包成 `bundle.js` 并且放到 `js/dist`下，并且需要将index.html 中引用的 app.js 改为 应用 boudle.js 



### AMD 规范

AMD（Asynchronous Module Definition）异步模块定义，专门用于浏览器，**模块的加载是异步的**

#### 定义模块

```js
//没有依赖其他模块的
define(function(){
  return 模块
})

//定义有依赖模块
define(['module1','module2'],function(mod1,mod2){
  return 模块
})
```



#### 引入模块

```js
require(["module1","module2"],function(mod1,mod2){
  使用 mod1/mod2
})
```



#### 实现该规范的库

实现了 AMD 模块规范的库是 RequireJS，使用和 jQuery 差不多，都是需要先引入，然后使用。

```
-index.html
-require.js
-main.js
-jquery.js
-angular.js
-src
	-modules
		-dataService.js
		-alerter.js
```

```html
<!-- index.html -->
<script data-mian="./main.js" src="./require.js"></script>
```

main.js 是入口文件，data-mian 用于定义入口文件，不必再自己引入入口文件

```js
(function(){
  requirejs.config({
    //baseUrl:"js/src/modules",//模块基本路径,写了这个下面就没有必要写 ./modules/ 了
    paths:{//模块映射路径,路径不可以加 js 后缀
      dataService:"./modules/dataService",
      alerter:"./modules/alerter",
      jquery:"./jquery"
      angular:"./angular"
    },
    shim:{
    	angular:{
        exports:"angular" //将./angular.js 暴露为angular对象
      }
  	}
  })
  
  requirejs(["alerter"],function(alerter){
    alerter.show();
  })
})();
```

dataService.js

```js
define(function(){
  let msg = "dataService";
  function getMsg(){
    return msg;
  }
  return {getMsg};
})
```

alerter.js

```js
define(["dataService","jquert"],function(dataService,$){
  let myMsg = "alerter";
  function show(){
    console.log(myMsg,dataService.getMsg());
  }
  $("body").css("background","red")
  return {show};
})
```

需要**注意**的是，当引入第三方库的时候需要确定这个库支不支持 AMD 规范，并且导出库的名字是什么。比如 jQuery 虽然是支持 AMD 规范的，但是导出的名字却是 jquery 。angular不支持 AMD 规范，但是可以认为得将 它暴露出来，使得它变成一个对象。



### CMD 规范

CMD 规范也是专门用于浏览器，模块的加载是异步的，模块使用时才会加载执行。实现这个规范的库是 SeaJS

#### 定义模块

```js
//定义没有依赖的模块
define(function(require,exports,module){
  exports.xxx = value;
  module.exports = value;
})

//定义有依赖的模块
define(function(require,exports,module){
  //引入依赖模块（同步）
  var mod2 = require("./module2");
  //引入依赖模块（异步）
  require.async("./module3",function(m3){
    
  })
  //暴露模块
  exports.xxx = value;
})
```

#### 引入模块

```js
define(function(require){
  var mod1 = require("./module1");
  var mod2 = require("./module2");
  mod1.show();
  mod2.show();
})
```

在 index.html 中引用 SeaJs

```html
<script type="text/javascript" src="./sea.js"></script>
<script>
	seajs.use("./main.js");
</script>
```



### ES6 模块化规范

ES6 中天生支持模块化，为了解决模块规范不统一而生，但是浏览中，有些是不支持 ES6 语法的，所以需要使用工具（babel）进行转译 ES6 为 ES5，这样还不够，因为在ES6 中可以使用两种方式导入包 import / require，浏览器还是不能够识别 require 语法所以需要使用 Browserify 将转译好的 ES5 再转译一遍。

```
-index.html
-.babelrc
-main.js
-src
	-modules
		-module1.js
		-module2.js
```



安装 babel，Browserify 

```shell
npm i babel-cli browserify -g
npm i babel-preset-es2015 -D
## preset 将es6转es5的所有插件打包
```

配置 .babelrc

```json
{
  "presets":["es2015"]
}
```



#### 导出模块语法

```js
export value
export default value 
```



#### 导入模块语法

```js
import value, {value2} from "./module";
const value = import("./module");
```



导出模块：module.js

```js
//分别导出
export function foo(){
  console.log("foo");
}
export function bar(){
  consoel.log("bar");
}
export let arr = [1,2,3];

//统一导出
function fun1(){
  console.log("fun1")
}
function fun2(){
  console.log("fun2");
}
export const fun = {
	fun1,fun2
}

//默认导出，一个模块中只能有一个
const msg1 = "msg1";
const msg2 = "msg2";
export default {
  msg1,msg2
}
```

导入模块：main.js

```js
//单个，非默认导出的模块
import {fun} from "./module";
//多个导入,并定义别名,非默认导出模块
import {foo, bar as showBar} from "./module";
//导入默认模块
import msg from "./module";

fun.fun1();
fun.fun2();
foo();
showBar();
console.log(msg.msg1,msg.msg2);
```



#### 引用方式

使用 babel 将 es6 打包为 es5 

```shell
babel js/src -d js/lib
```

js/src 下的js文件将被转化成 es5 规范的代码，名字不会更改，被保存在 js/lib 中

使用 Browserify 编译 js 

```shell
browserify js/lib/main.js -o js/lib/bundle.js
```

使用时，可以直接将 bundle.js 导入到 index.html

```html
<script src="./lib/bundle.js"></script>
```



#### 在浏览器中直接使用

要在浏览器中直接使用 ES6 的模块化系统 需要将设置`<script type='module'>`，但是大多数浏览器还不支持。n

```html
<script type='module'>
	import './m.js';//这样是导入整个文件
	import k,{aaa as a,bbb} form './m.js';
  
 	//也可以导入在线文件
  import 'https://code.jquery.com/jquery-3.3.1.js';
  //也可以使用import()函数按需要导入
  if(k){
    import('./a');
  }
</script>
```



