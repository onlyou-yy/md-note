# MongoDB

数据库，是按照数据结构来组织、存储和管理数据的仓库，因为在计算机中程序都是运行在内存中的，因此数据也是会在内存中有效，一旦程序崩溃，或者电脑死机，数据将会丢失，而数据库则可以将数据存储到本地，这样即使计算机死机了，重启之后还是可以找到之前的数据。现在的数据库主要分为关系数据库（RDBMS）和非关系型数据库（NoSQL = not only sql），关系数据库如sql server，mysql，Oricle等，而非关系数据库则有redis，MongoDB（文档数据库）等。

[关系数据库和非关系型数据库的关系](https://www.jianshu.com/p/fd7b422d5f93)

## 关系数据库

关系型数据库的主要特点就是都遵循SQL，许多东西都已经标准化了，这样由多个厂商开发出来的不同数据库产品只要会SQL语言都可以完成数据库的操作，但是也由于已经标准化，数据库的表的设置规则和字段都相对的比较固定，所以就以为这数据库要设计得比较好才能使数据库的可维护性高，一般数据库设计好之后除非必要，否则都不会修改数据库。所以对于一个成型的数据库，如果想要修改数据库表就会相对比较麻烦。

### 关系型数据库的设计规范

>**1、A (Atomicity) 原子性**
>
>原子性很容易理解，也就是说事务里的所有操作要么全部做完，要么都不做，事务成功的条件是事务里的所有操作都成功，只要有一个操作失败，整个事务就失败，需要回滚。
>
>比如银行转账，从A账户转100元至B账户，分为两个步骤：1）从A账户取100元；2）存入100元至B账户。这两步要么一起完成，要么一起不完成，如果只完成第一步，第二步失败，钱会莫名其妙少了100元。
>
>**2、C (Consistency) 一致性**
>
>一致性也比较容易理解，也就是说数据库要一直处于一致的状态，事务的运行不会改变数据库原本的一致性约束。
>
>例如现有完整性约束a+b=10，如果一个事务改变了a，那么必须得改变b，使得事务结束后依然满足a+b=10，否则事务失败。
>
>**3、I (Isolation) 独立性**
>
>所谓的独立性是指并发的事务之间不会互相影响，如果一个事务要访问的数据正在被另外一个事务修改，只要另外一个事务未提交，它所访问的数据就不受未提交事务的影响。
>
>比如现在有个交易是从A账户转100元至B账户，在这个交易还未完成的情况下，如果此时B查询自己的账户，是看不到新增加的100元的。
>
>**4、D (Durability) 持久性**
>
>持久性是指一旦事务提交后，它所做的修改将会永久的保存在数据库上，即使出现宕机也不会丢失。



## 非关系型数据库 MongoDB

MongoDB是为快速开发互联网web应用而设计的数据库系统。MongoDB的设计目标就是极简、灵活、作为web栈的一部分。MongoDB的数据模型是面向文档的，所谓的文档就是一种类似于JSON的结构，可以用来存储各种各样的JSON（BSON）。

**在MongoDB中的三个概念**

+ **数据库（database）**
	+ 数据库是一个仓库，在仓库中可以存放集合
+ **集合（collection）**
	+ 集合类似于数组，在集合中可以存放文档（相当于关系型数据中的表）
+ **文档（document）**
	+ 文档是数据库中的最小单位，存储和操作的都是文档



## 安装MongoDB

在MongoDB中小版本的奇数为开发版，小版本偶数为稳定版(nodejs 已大版本为主)，MongoDB的全版本安装程序[下载地址：](https://www.mongodb.org/dl/win32)`https://www.mongodb.org/dl/win32`。

下载安装之后，配置环境变量，在Path后加上自己的MongoDB的安装路径，如我的是`D:\mongoDB\bin`，然后需要在 C 盘的根目录下创建`data\db`文件夹，这个就是数据库的默认存储路径。如果想想要修改存储路径可以在启动服务的识货指定路径

```shell
mongod --dppath d:\data
```

这样在每次启动服务的时候都要指定。

启动服务完成之后就可以另外开一个 cmd 面板输入`mongo`开始操作mongoDB 数据库了。

比较好用的操作mongoDB 的UI工具软件 [No SQLBooster](https://www.nosqlbooster.com/downloads)

## 基本操作

+ `show dbs`：查看数据库
+ `use 数据库名`：进入到数据，数据不需要手动创建，在创建数据中创建保存了一个文档之后，会自动创建
+ `show collections`：显示数据库中的所有集合

## CURD

### 插入文档

`db.集合名.insert(docs)`：向集合中插入一条或多条文档数据，插入成功后，系统会自动给数据添加一个`_id:ObjectId("xxxxx")`的属性表示是数据的主键。但是如果插入数据的时候指定了`_id`，就会以插入的为主。

+ `db.集合名.insertOne(doc)`：插入一条
+ `db.集合名.insertMany(docs)`：插入多条

```js
db.test.insert({name:"jack",age:18});//插入一条数据
db.test.insert([{name:"jack",age:18},{name:"rocy",age:19}]);//插入多条数据
```

### 查询文档

`db.集合名.find(条件对象,过滤条件)`：查询集合中的数据，如果条件为空则为查找全部，显示指定的字段，默认显示的是全部，并且`_id`是默认显示的。

+ `db.集合名.findOne(条件对象,过滤条件)`：查询符合条件的第一个文档
+ `db.集合名.find(条件对象,过滤条件).pretty()`：格式化显示查询结果
+ `db.集合名.find(条件对象,过滤条件).count()`：得到查询结果数量

```js
//查找名字为 jack 的文档
db.test.find({name:"jack"},{_id:0,name:1});
//查找名字为 jack，age > 18 的文档
db.test.find({name:"jack",age:{$gt:18}});
```

在 mongodb 中对于一些属性值在范围内的条件需要使用操作符来实现，比如上面的`age > 18` 中的大于可以使用`$gt`表示。还有很多其他的查询操作符。

|      操作      | 格式                                   | 范例                                                    |
| :------------: | :------------------------------------- | :------------------------------------------------------ |
|      等于      | `{<key>:<value>`}                      | `db.col.find({"by":"jack"}).pretty()`                   |
|      小于      | `{<key>:{$lt:<value>}}`                | `db.col.find({"likes":{$lt:50}}).pretty()`              |
|   小于或等于   | `{<key>:{$lte:<value>}}`               | `db.col.find({"likes":{$lte:50}}).pretty()`             |
|      大于      | `{<key>:{$gt:<value>}}`                | `db.col.find({"likes":{$gt:50}}).pretty()`              |
|   大于或等于   | `{<key>:{$gte:<value>}}`               | `db.col.find({"likes":{$gte:50}}).pretty()`             |
|     不等于     | `{<key>:{$ne:<value>}}`                | `db.col.find({"likes":{$ne:50}}).pretty()`              |
| 多条件关系 AND | `{key1:value1, key2:value2}`           | `db.col.find({"likes":"50","name":"jack"}).pretty()`    |
| 多条件关系 OR  | `{$or:[{key1:value1}, {key2:value2}]}` | `db.col.find({$or:[{name:"jack"}, {age:18}]}).pretty()` |

还需要注意的是，如果要查询内嵌文档的话，可以使用`一层key.二层key`来表示，如有集合 hobbys

```
[
    {
        hobby:{
            movie:["hero","ET"],
        }
    },
    {
        hobby:{
            movie:["魔兽","泰坦尼克"],
        }
    }
]
```

如果要查询喜欢看的电影有魔兽的，可以这样`db.hobbys.find({"hobby.movie":"魔兽"})`；

### 更新文档

`db.集合名.update(查询条件,更新数据,配置)`：更新集合中符合条件的文档，默认会使用新数据直接替换匹配到的文档，并且只会修改匹配到的第一个，可以修改配置`{multi:true}`进行变更。

+ `db.集合名.updateOne(查询条件,更新数据)`：修改一个
+ `db.集合名.updateOne(查询条件,更新数据)`：修改多个

```js
db.test.update({name:"jack"},{name:"nancy"});//替换
db.test.update({name:"jack"},{$set:{name:"nancy"});//添加或者修改
db.test.update({name:"jack"},{$unset:{age:1});//删除属性
```

比较常用的更新操作符是`$set $unset $push`这几个，还有一些的话可以查看[这里](https://blog.csdn.net/chenzhou123520/article/details/84275529?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)。其实主要的就是针对属性名，数字，数组的操作

**对于属性名的操作**

+ `$set`：修改对应属性名的属性值
+ `$unset`：删除属性
+ `$rename`：对字段进行重命名

**对于数字的操作**

+ `$inc`：对一个数字字段的某个值增加value

**对于数组操作**

+ `$push`：把value追加到field里。注：field只能是数组类型，如果field不存在，会自动插入一个数组类型
+ `$pushAll`：用法同`$push`一样，只是`$pushAll`可以一次追加多个值到一个数组字段内。
+ `$addToSet`：加一个值到数组内，而且只有当这个值在数组中不存在时才增加。
+ `$pop`：删除数组内第一个值：`{$pop:{field:-1}}`、删除数组内最后一个值：`{$pop:{field:-1}}`
+ `$pull`：从数组field内删除一个等于_value的值
+ `$pullAll`：用法同`$pull`一样，可以一次性删除数组内的多个值。

### 删除文档

`db.集合名.remove(条件,justone)`：一条一条地删除符合条件的文档，justone 默认是false，需要注意的是如果集合的全部文档都被删除了，集合也会被删除

+ `db.集合名.deleteOne(条件)`：删除一个
+ `db.集合名.deleteMany(条件)`：删除多个
+ `db.集合名.drop()`：直接删除整个集合
+ `db.dropDatabase()`：删除数据库

```js
db.test.remove({name:"jack"});
```



## mongoDB 中的分页 skip() 和 limit() ,排序 sort()

在 mongoDB 中可以通过`skip()`和`limit()`对查询结果进行分页，skip(n) 跳过前n条数据，limit(m) 只获取条数据。所以根据分页显示公式`(页码 - 1) * 每页显示数量`有

```js
//curP:页码 , n:每页显示数量
db.page.find().skip((curP - 1) * n).limit(n);
```

可以还可以使用`sort()`对查询结果进行排序

```js
db.page.find().sort({age:1});//按年龄升序
db.page.find().sort({age:-1,comein:1});//先按年龄降序，如果年龄一样再按收入升序
```



## 文档间关系

一般文档与文档之间会有三种关系，1对1，1对多，多对多。在 mongoDB 中用内嵌的方式表示这些关系，比如 

1对1：一个女的只能有一个老公

```
{name:'rocy',husband:{name:"jack"}};
```

1对多：一个用户有多个订单

```
{userId:1,orders:[{orderId:1},{orderId:2}]}
```

多对多：一个老师可以有多个学生，一个学生也会有多个老师

```
老师:[
	{id:1,name:"t1"},{id:2,name:"t2"},{id:3,name:"t3"}
]
学生:[
	{id:1,name:"c1",teachers:[1,2]},{id:2,name:"c2",teachers:[1,3]}
]
```



## 聚合

在开发中如果想要对集合进行分组查询或者统计分组后某个字段的数据总和的话，就需要使用到聚合，在 mongodb 中可以使用`db.集合名.aggregate(配置)`使用聚合，比如将书本集合中的数据按作者（by_user）分组，并统计头多少个作者

```js
db.books.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
```

在 aggregate 中有管道的概念（简单来说就是流水线上的工区，数据会依次被各个工区处理），上面的`$group`就是其中之一。

常用的有

- `$project`：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- `$match`：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- `$limit`：用来限制MongoDB聚合管道返回的文档数。
- `$skip`：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- `$unwind`：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- `$group`：将集合中的文档分组，可用于统计结果。
- `$sort`：将输入文档排序后输出。
- `$geoNear`：输出接近某一地理位置的有序文档。

多个管道可以这样写

```js
db.articles.aggregate( [
                        { $match : { score : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
```

这里的意思就是集合中文档的` 70<score<=90`的数据分组后统计数量。



## 备份和恢复

在Mongodb中我们使用mongodump命令来备份MongoDB数据。该命令可以导出所有数据到指定目录中。

**备份/导出**

```shell
mongodump -h 服务地址 -d 数据库名 -o 输出保存的目录
```

**恢复/导入**

```shell
mongorestore -h 服务地址 -d 数据库名 导入的数据库路径或者目录
```



## Mongoose 使 nodejs 操作数据库

Mongoose 是nodejs 的一个库，是对nodejs中原生的`mongodb`模块的封装，使nodejs操作 mongoDB 更加方便。

### **使用mongoose 的好处**

+ 可以为文档创建一个模式结构（约束）Schema
+ 可以对模型中的对象 / 文档进行验证
+ 数据可以通过类型转换转换为对象模型
+ 可以使用中间件来应用业务逻辑挂钩
+ 比node原生的MongoDB简单易用

### **在 mongoose 中的提供的新对象**

+ Schema（模式对象）
	+ schema对象定义约束了数据库中的文档结构
+ Model
	+ model 对象作为集合中的所有文档的表示，相当于MongoDB里面的集合Collections
+ Document
	+ document 表示集合中的将具体文档，相当于集合中的一个具体文档

### 在 nodejs 中使用

#### 安装

```shell
npm i mongoose -S
```

#### 连接数据库

```js
let mongoose = require("mongoose");
mongoose.connect("mongodb://数据库ip地址:端口号/数据库名"，{useMongoClient:true});
//监听连接成功事件
mongoose.connection.once("open",function(){
    console.log("连接成功");
})
//监听连接关闭事件
mongoose.connection.once("close",function(){
    console.log("连接关闭");
})
```

#### 关闭连接

```js
mongoose.disconnect();
```

#### 创建 Schema

Schema，其实就是相当于是sql数据库表中对字段值的约束和结构的接口。

```js
let mongoose = require("mongoose");
let Schema = mongoose.Schema;
//创建学生表
let StuSchema = new Schema({
    name:String,
    age:Number,
    gender:{
        default:"女",
        type:String,
    }
})
```

#### 通过 Schema 创建 Model （集合）

```js
let StuModel = mongoose.model("student",StuSchema);
```

需要注意的是，mongoose会对集合的名字做复数化处理，就是最终得到的学生集合名字会是`students`，如果是`child`可能会变成`children`。

#### Model 中的方法（CURD）

mongoose 中的 Model 相当于MongoDB 中的 collection ，所以CURD 是相对于 Model 来说的。

##### 插入或者新增数据

`Model.create(数据,回调)`：创建一个数据并插入到 model 中，和`db.col.insert()`一样数据可以是单个也可以是多个

```js
StuModel.create([
    {
        name:"jack",
        age:18,
        gender:"男"
    },
    {
        name:"rocy",
        age:18,
    }
],function(err){
    if(!err) console.log("插入成功");
})
```

##### 查询数据

mongoose 的查询和 MongoDB 中的是一样的只不过有一点不同，在最后面多了一个 回调函数，并且如果不传回调函数就不会进行查询。

```js
StuModel.find({name:"jack"},"name jack -_id",{skip:2},function(err,docs){
    if(!err) console.log(docs);
})
StuModel.findOne({name:"jack"},{_id:0,name:1},function(err,doc){
    if(!err) console.log(doc);
})
StuModel.findMany({name:"jack"},function(err,docs){
    if(!err) console.log(docs);
})s
StuModel.findById("xxxxxx",function(err,docs){
    if(!err) console.log(docs);
})
```

##### 修改数据

mongoose 的修改也是和 MongoDB 中的几乎一样。

```js
StuModel.update({name:'jack'},{$set:{age:19}},{multi:false},function(err,docs){
    if(!err) console.log(docs);
})
StuModel.updateOne({name:'jack'},{$set:{age:19}},function(err,doc){
    if(!err) console.log(doc);
})
StuModel.updateMany({name:'jack'},{$set:{age:19}},function(err,docs){
    if(!err) console.log(docs);
})
```

##### 删除数据

删除也是如此

```js
StuModel.remove({name:'jack'},function(err){
    if(!err) console.log("成功");
})
StuModel.deleteOne({name:'jack'},function(err){
    if(!err) console.log("成功");
})
StuModel.deleteMany({name:'jack'},function(err){
    if(!err) console.log("成功");
})

```

##### 其他的API

count：统计数量

```js
StuModel.count({},function(err,count){
    if(!err) console.log(count);
})
```



#### Document 对象API

Model 还可以作为一个独立的对象来创建document，但是实例化出来的数据不会主动插入到集合中。

```js
let doc = new StuModel({
    name:"nancy",
    age:12,
})
```

`doc.save(回调)`：保存Document到集合中

```js
doc.save();
```

并且Model操作集合的方法的回调返回得到的数据都是`Document`实例对象，也可以使用 Model 的大多数方法。如`update,remove`。

```js
StuModel.findOne({name:"jack"},function(err,doc){
    if(!err){
        doc.update($set:{name:"jack2"});
    	doc.age = 20;
        doc.save();
    };
})
```

`doc.toJSON()`：转化为 JSON 对象；

`doc.toObject()`：转化为纯数据对象，对这个对象的操作将不会影响的数据中的文档数据，比如想要 doc 中的name属性`delete doc.name`这样是不起作用的。