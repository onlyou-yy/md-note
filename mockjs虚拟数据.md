​		在前后端分离的开发模式中，在前端和后台商量好数据结构和接口后，前端就需要一些模拟的接口数据来进行前端页面的开发和前端数据的渲染，这样就不需要等到后台开发完整之后再进行数据的渲染和功能开发了。

获取虚拟数据的方法有很多种

 1.  使用纯mockjs

 2.  使用json-server搭建json服务器,用mockjs提供虚拟数据

 3.  使用fastmock网站

 4.  自己搭建服务器使用mockj提供js数据

     ….

     

## mockjs的安装

CDN

```html
<script src="http://mockjs.com/dist/mock.js"></script>;
```

npm

```shell
npm install mockjs;
```

下载mock.js文件后直接引用



## 使用

**Mock.mock(url, type, data|function( options ))**

function(options)记录用于生成响应数据的函数，options是前端请求的相关数，含有 `url`、`type` 和 `body` 三个属性。当拦截到匹配 `rurl` 的 Ajax 请求时，函数 `function(options)` 将被执行，并把执行结果作为响应数据返回。

参数名 | 参数需求 | 参数描述 | 例子
:---:|:---:|:---:|:---:
url|可选: URL 字符串或 URL 正则|拦截请求的地址|/mock
type|可选|拦截Ajax类型|GET、POST
template|可选: 可以是对象或字符串|生成数据的模板|`{'data|1-10':['mock'] }、'@EMAIL'`

```javascript
Mock.mock('https://www.test.com',{
  "userInfo|4":[{    //生成|num个如下格式名字的数据
    "id|+1":1,  //数字从当前数开始后续依次加一
    "name":"@cname",    //名字为随机中文名字
    "ago|18-28":25,    //年龄为18-28之间的随机数字
    "sex|1":["男","女"],    //性别是数组中的一个，随机的
    "job|1":["web","UI","python","php"]    //工作是数组中的一个
    'income|1500-20000.2':1		//工资最低1500最高20000小数点保留2位
  }]
})
  
//使用ajax发送请求将被mock拦截，所以在network上看不到请求
$.get('https://www.test.com',function(data){
        console.log(JSON.parse(data));
})
```



## 扩展

使用`Mock.Random.extend()`方法可以对mock的数据做扩展，生成符合我们自己需要的数据

```javascript
//extend方法可以为Random对象添加新的随机数据（新方法）
//Mock.Random.extend({
//  方法名:function(){
//    return this.pick(数组)
//  }
//})

Mock.Random.extend({
  //生成状态数据
  status:function(){
  	const arr=['已收货','未收货'，'已付款','未付款','已评价']
    return this.pick(arr)
  }
})
//使用
Mock.Random.status();
Mock.mock({'status':'@status()'})
```



## 在Vue项目中写模拟接口

1. 先在项目中安装依赖`npm i mockjs –-save` 

2. 在创建mock.js文件写接口

   ```javascript
   导入mocksjs模块
   import Mock from 'mockjs';
   
   //获取数据
   Mock.mock('/api/goodslist','get',{
     status:200,
     message:'success',
     'data|5':[
       id:'@increment(1)',
       name:'@cword(2,6)',
      	price:'@natura(2,100)',
       //img:"@image('200x200',@color(),'hello')",//这种方式可能会导致图片访问不到
       img:"@dataImage('200x200')",
     ]
   })
   
   //接收post提交数据
   Mock.mock('/api/postData','post',function(options){
     console.log(options);//{url,type,body}
     //return {name:'@cname'};这样写是用不了mock中的方法的 @cname 会被视为字符串
     return Mock.mock({name:'@cname'})
   })
   
   //使用url正则匹配结构，并获取传递的url变量
   Mock.mock(/\/api\/getGoods\/(\d)+/,'get',function(options){
     console.log(options);
     
     //从url中提取出需要的数据,使用exec
     let reg=/\/api\/getGoods\/(\d+)/
     let res=reg.exec(options.url);
     let id=res[1]-0;
     //使用正则表达式中的命名捕获
     //let reg=/\/apia\/getGoods\/(?<id>\d+)/
     //let res=options.url.match(reg);
     //let id=res.groups.id
     return Mock.mock({
        	id:id,
          name:'@cword(2,6)',
         	price:'@natura(2,100)', 
          img:"@dataImage('200x200')",
        })
      })
   ```

3. 在vue项目中的main.js导入mock.js

   ```js
import './mock.js';
   ```

4. 使用axios或者fetch发送请求，在有mock的情况下请求将被拦截

   ```javascript
   axios({
     method:'get',
     url:'/api/goodslist'
   }).then(res=>{
     console.log(res)
   })
   ```

为了方便管理可以将mock.js才分为多个文件进行管理

mock

 1.  index.js

     ``` javascript
     import './extends.js';
     import './goods.js';
     ```

 2.  extends.js

     ```javascript
     import Mock,{Random} from 'mockjs'
     Random.extend({
       status:function(){
         return arr;
       }
       ...
     })
     ```

 3.  goods.js

     ```javascript
     import Mock from 'mockjs';
     //获取数据
     Mock.mock('/api/goodslist','get',{
       status:200,
       message:'success',
       'data|5':[
         id:'@increment(1)',
         name:'@cword(2,6)',
        	price:'@natura(2,100)',
         //img:"@image('200x200',@color(),'hello')",//这种方式可能会导致图片访问不到
         img:"@dataImage('200x200')",
       ]
     })
     ```

 4.  其他模块文件

     …..

     

     以上我觉得可以使用一个Mock.mock当做是一个拦截器，然后根据请求的地址使用`switch case`做路由的分发

     ```javascript
     import Mock from 'mockjs'
     Mock.mock(/\/api\/(?<api>\w+)\/(?<param>\s+)?/,'get',function(options){
       console.log(options);
       let reg=/\/api\/(?<api>\w+)\/(?<param>\w+)?/
       let res=options.url.match(reg);
       let api=res.groups.api;
       let param=res.groups.param;
       let data=null;
       switch(api){
         case 'goodslist':
           data=Mock.mock({
             ...
           })
           break;
         ....
       }
       retutn data;
     })
           
     Mock.mock(/\/api\/(?<api>\w+)\/(?<param>\s+)?/,'post',function(options){
      	....
     })
     ```

     

# 语法规范

Mock.js 的语法规范包括两部分：

1. 数据模板定义规范（Data Template Definition，DTD）
2. 数据占位符定义规范（Data Placeholder Definition，DPD）

## 数据模板定义规范 DTD（使用规则生成数据）

**数据模板中的每个属性由 3 部分构成：属性名、生成规则、属性值：**

```javascript
// 属性名   name
// 生成规则 rule
// 属性值   value
'name|rule': value
```

**注意：**

- *属性名* 和 *生成规则* 之间用竖线 `|` 分隔。

- *生成规则* 是可选的。

- 生成规则

   

  有 7 种格式：

  1. `'name|min-max': value`
  2. `'name|count': value`
  3. `'name|min-max.dmin-dmax': value`
  4. `'name|min-max.dcount': value`
  5. `'name|count.dmin-dmax': value`
  6. `'name|count.dcount': value`
  7. `'name|+step': value`

- **生成规则 的 含义 需要依赖 属性值的类型 才能确定。**

- *属性值* 中可以含有 `@占位符`。

- *属性值* 还指定了最终值的初始值和类型。

**生成规则和示例：**

### 1. 属性值是字符串 **String**

1. `'name|min-max': string`

   通过重复 `string` 生成一个字符串，重复次数大于等于 `min`，小于等于 `max`。

2. `'name|count': string`

   通过重复 `string` 生成一个字符串，重复次数等于 `count`。

### 2. 属性值是数字 **Number**

1. `'name|+1': number`

   属性值自动加 1，初始值为 `number`。

2. `'name|min-max': number`

   生成一个大于等于 `min`、小于等于 `max` 的整数，属性值 `number` 只是用来确定类型。

3. `'name|min-max.dmin-dmax': number`

   生成一个浮点数，整数部分大于等于 `min`、小于等于 `max`，小数部分保留 `dmin` 到 `dmax` 位。

```javascript
Mock.mock({
    'number1|1-100.1-10': 1,
    'number2|123.1-10': 1,
    'number3|123.3': 1,
    'number4|123.10': 1.123
})
// =>
{
    "number1": 12.92,
    "number2": 123.51,
    "number3": 123.777,
    "number4": 123.1231091814
}
```

### 3. 属性值是布尔型 **Boolean**

1. `'name|1': boolean`

   随机生成一个布尔值，值为 true 的概率是 1/2，值为 false 的概率同样是 1/2。

2. `'name|min-max': value`

   随机生成一个布尔值，值为 `value` 的概率是 `min / (min + max)`，值为 `!value` 的概率是 `max / (min + max)`。

### 4. 属性值是对象 **Object**

1. `'name|count': object`

   从属性值 `object` 中随机选取 `count` 个属性。

2. `'name|min-max': object`

   从属性值 `object` 中随机选取 `min` 到 `max` 个属性。

### 5. 属性值是数组 **Array**

1. `'name|1': array`

   从属性值 `array` 中随机选取 1 个元素，作为最终值。

2. `'name|+1': array`

   从属性值 `array` 中顺序选取 1 个元素，作为最终值。

3. `'name|min-max': array`

   通过重复属性值 `array` 生成一个新数组，重复次数大于等于 `min`，小于等于 `max`。

4. `'name|count': array`

   通过重复属性值 `array` 生成一个新数组，重复次数为 `count`。

### 6. 属性值是函数 **Function**

1. `'name': function`

   执行函数 `function`，取其返回值作为最终的属性值，函数的上下文为属性 `'name'` 所在的对象。

### 7. 属性值是正则表达式 **RegExp**

1. `'name': regexp`

   根据正则表达式 `regexp` 反向生成可以匹配它的字符串。用于生成自定义格式的字符串。

   ```javascript
   Mock.mock({
       'regexp1': /[a-z][A-Z][0-9]/,
       'regexp2': /\w\W\s\S\d\D/,
       'regexp3': /\d{5,10}/
   })
   // =>
   {
       "regexp1": "pJ7",
       "regexp2": "F)\fp1G",
       "regexp3": "561659409"
   }
   ```



## 数据占位符定义规范 DPD（使用Random生成数据）

*占位符* 只是在属性值字符串中占个位置，并不出现在最终的属性值中。

*占位符* 的格式为：

```javascript
@占位符
@占位符(参数 [, 参数])
```

**注意：**

1. 用 `@` 来标识其后的字符串是 *占位符*。
2. *占位符* 引用的是 `Mock.Random` 中的方法。
3. 通过 `Mock.Random.extend()` 来扩展自定义占位符。
4. *占位符* 也可以引用 *数据模板* 中的属性。
5. *占位符* 会优先引用 *数据模板* 中的属性。
6. *占位符* 支持 *相对路径* 和 *绝对路径*。

```javascript
Mock.mock({
    name: {
        first: '@FIRST',
        middle: '@FIRST',
        last: '@LAST',
        full: '@first @middle @last'
    }
})
// =>
{
    "name": {
        "first": "Charles",
        "middle": "Brenda",
        "last": "Lopez",
        "full": "Charles Brenda Lopez"
    }
}
```

# 参考

[mock.js](https://github.com/nuysoft/Mock/wiki/Syntax-Specification)

[正确开启Mockjs的三种姿势：入门参考（一）](https://www.cnblogs.com/soyxiaobi/p/9846057.html)