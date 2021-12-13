## RESTful

RESTful 其实是一种接口的设计风格，全称是*表现层状态转移*，是一种面向资源编程的思想，同一接口可以根据不同的请求方式作出不同的操作。



## 为什么不直接使用本地 mockjs

1. 本地 mockjs 会拦截 ajax 请求，也就是说对于其他方式的请求是拦截不了的，如`fetch`。
2. 本地 mockjs 可以拦截请求导致在调试面板中无法检测到请求的发送，也无法进行网速限制的调试
3. 由于数据是直接在本地生成的，所以数据是不能在持久化保存的，这样的话就会导致有一些功能接口无法实现。（当然，一些在线的mock数据的网站如`easy-mock`也是无法做到数据的持久化的）



## json-server

json-server是一个基于 nodejs 的 express 框架满足 RESTful 规范的服务端框架，支持多种请求`GET POST PATCH PUT DELETE`等方式，并且可以实现数据的持久化，相当于是一个自带文档数据库的服务器；json-server既可以当作是一个服务命令来使用，也可以最为一个npm包来使用。

### 简单使用

先全局安装`json-server`

```shell
npm i json-server -g
```

然年后创建一个服务资源文件夹 server ,并创建一个 db.json 或 db.js 文件。

```shell
mkdir server && cd json-server
```

> `json-server`可以直接把一个`json`文件托管成一个具备全`RESTful`风格的`API`,并支持跨域、`jsonp`、路由订制、数据快照保存等功能的 web 服务器。

```json
{
    "course": [
        {
          "id": 1000,
          "course_name": "马连白米且",
          "autor": "袁明",
          "college": "金并即总变史",
          "category_Id": 2
        },
        {
          "id": 1001,
          "course_name": "公拉农题队始果动",
          "autor": "高丽",
          "college": "先了队叫及便",
          "category_Id": 2
        }
     ]
}
```

然后就可以启动服务器了

```js
json-server --watch --port 5300 db.js
```

> 还可以通过`--config`来指定配置文件，默认文件是`json-server.json`；
>
> ```json
> {
>   "port": 53000,//端口
>   "watch": true,//是否开启文件监视
>   "static": "./public",//静态目录
>   "read-only": false,//是否开启限制cookie访问
>   "no-cors": false,//是否不允许跨域
>   "no-gzip": false,//是否不开启gzip压缩
>   "routes": "route.json" //路由文件
> }
> ```
>
> `--routes`指定路由配置
>
> ```json
> {
>   "/api/*": "/$1",    //   /api/course   <==>  /course
>   "/:resource/:id/show": "/:resource/:id",
>   "/posts/:category": "/posts?category=:category",
>   "/articles\\?id=:id": "/posts/:id"
> }
> ```

访问数据直接将服务文件 db.js 返回的对象数据当成是文档来访问即可，如要`course`的数据只需

```txt
http://localhos:5300/course
```



这样简单的服务器就搭建好了。不过还有一些其他的使用技巧和功能需要了解。如查询的参数，以及post，put，patch，delete等。

### 对于查询 get

#### 过滤查询

查询数据，可以额外提供

```http
GET /posts?title=json-server&author=typicode
GET /posts?id=1&id=2

# 可以用 . 访问更深层的属性。
GET /comments?author.name=typicode
# 可以用 / 查询符合id值的项。查询到的是一个对象
GET /comments/001  
# 相当于 查询到的是一个对象数组
GET /comments?id=001 
```

还可以使用一些判断条件作为过滤查询的辅助。

```http
GET /posts?views_gte=10&views_lte=20
```

可以用的拼接条件为：

- `_gte` : 大于等于
- `_lte` : 小于等于
- `_ne` : 不等于
- `_like` : 包含

```http
GET /posts?id_ne=1
GET /posts?id_lte=100
GET /posts?title_like=server
```

#### 分页查询

默认后台处理分页参数为： `_page` 第几页， `_limit`一页多少条。

```
GET /posts?_page=7
GET /posts?_page=7&_limit=20
```

> 默认一页10条。

后台会返回总条数，总条数的数据在响应头:`X-Total-Count`中。

#### 排序

- 参数： `_sort`设定排序的字段
- 参数： `_order`设定排序的方式（默认升序）

```http
GET /posts?_sort=views&_order=asc
GET /posts/1/comments?_sort=votes&_order=asc
```

支持多个字段排序：

```http
GET /posts?_sort=user,views&_order=desc,asc
```

#### 任意切片数据

```http
GET /posts?_start=20&_end=30
GET /posts/1/comments?_start=20&_end=30
GET /posts/1/comments?_start=20&_limit=10
```

#### 全文检索

可以通过`q`参数进行全文检索，例如：`GET /posts?q=internet`

#### 实体关联

**关联子实体**

包含children的对象, 添加`_embed`

```http
GET /posts?_embed=comments
GET /posts/1?_embed=comments
```

**关联父实体**

包含 parent 的对象, 添加`_expand`

```http
GET /comments?_expand=post
GET /comments/1?_expand=post
```



### 对于添加数据 post

在给 course 添加一个数据时应该给`http://localhost:5300/course`发送 post 请求

```js
axios.post('http://localhost:5300/course',{
    data:{course_name:'jack'}}).then(res=>{
    console.log(res)
})
```

如果body没有设置，json-server仍然会给/data1数组添加一个新对象，并赋予一个默认的id值，新对象有且仅有id这一个属性。如果用户只设置了id以外的属性，json-server也会给新对象生成一个随机的id值。



### 对于修改数据 put 和 patch

put 是直接替换整一项数据，而 patch 则是修改指定的属性。如果要替换的数据不存在的话就会新增。需要注意的是需要在url里面指明 id `http://localhost:5300/course/1002`

```js
axios.put('http://localhost:5300/course/1002',{
    data:{name:'jack'}}).then(res=>{
    console.log(res)
})
```



### 对于删除数据 delete

```js
axios.delete('http://localhost:5300/course/1002').then(res=>{
    console.log(res)
})
```



### 作为npm包使用

`json-server`本身就是依赖express开发而来，可以进行深度定制。细节就不展开，具体详情请参考[官网](https://github.com/typicode/json-server)。

- 自定义路由

```js
const jsonServer = require('json-server')
const server = jsonServer.create()
const router = jsonServer.router('db.json')
const middlewares = jsonServer.defaults({
    static: "./public",//静态目录
    readOnly: false,//是否开启限制cookie访问
    noCors: false,//是否不允许跨域
})

server.use(middlewares)

server.use(jsonServer.bodyParser)
//自定义用户校验
server.use((req, res, next) => {
 if (isAuthorized(req)) { // 验证方法
   next() 
 } else {
   res.sendStatus(401)
 }
})
//其他请求拦截
server.use((req, res, next) => {
  if (req.method === 'POST') {
    req.body.createdAt = Date.now()
  }
  next()
})

//自定义路由
server.get('/echo', (req, res) => {
  res.jsonp(req.query)
})

// 路由映射，必须要在 server.use(router) 之前设置
server.use(jsonServer.rewriter({
  '/api/*': '/$1',
  '/blog/:resource/:id/show': '/:resource/:id'
}))
server.use(router)
server.listen(3000, () => {
  console.log('JSON Server is running')
})
```

- 自定义输出内容

```js
router.render = (req, res) => {
  res.jsonp({
    body: res.locals.data
  })
}
```

+ 把整个路由挂载到另外一个地址上

```js
server.use('/data', router);
```

