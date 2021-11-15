# Promise 基础

## Promise 中的状态和结果

Promise 实例中有个 **[PromiseState]** 保存着当前 Promise 实例的状态，初始状态是 pending ，成功为 resolved / fullfilled ，失败为 rejected ，并且只有可能由 pending 到  resolved / fullfilled  或者 rejected

Promise 实例中还有个 **[PromiseResult]** 保存着当前 Promise  实例异步任务的结果，但是这属性只有 resolve 和 reject 方法可以修改



## Promise 中的静态方法

+ **Promise.resolve(val)**：快速得到有个 成功 / 失败 的promise对象，val为promise结果
	+ val不为promise：返回的结果是成功的promise
	+ val 为promise ：返回的结果根据 val 而定，如果 val 结果是成功的pomise则返回的是成功promise，如果 val 结果是失败的pomise则返回的是成功promise。
+ **Promise.reject(val)**：无论 val 为什么类型的对象，得到的都是一个失败的promise对象
+ **Promise.all(promises)**：promises 为 promise 实例数组，可以同时执行多个promise，返回的是一个新的promise对象，promise 的状态由 promises 决定，如果 promises 中全部的 promise 都是成功的，则为成功，如果有一个是失败的则为失败。并且 [PromiseResult] 和`Promise.all(promises).then(res=>{})`的 res 也是一个数组，存储的时候成功的 promise 的结果
+ **Promise.race(promises)**：promises 为 promise 实例数组，可以同时执行多个promise，返回的是一个新的 promise 对象，状态和结果由 promises 中第一个改变状态的 promise 决定。



## Promise 特点

Promise 是 ES6 提出的一个行api，用于处理异步调用的导致的回调地狱情况，promise 有一些比较重要的特点

1. 定义 / 声明 Promise 的时候是一个同步的操作,在声明的时候就会被执行，当使用 .then 的时候是异步的微任务 

```js
let p = new Promise((res,rej) => {
  console.log('promise start');
  res('hello');
})

console.log('start');
let res = p.then(res => {
  console.log(res);
})
console.log('end');
```

这里的输出的会是 

```shell
promise start
start
end
hello
```

2. promise 的初始状态是 pending ，成功为 resolved / fullfilled ，失败为 rejected ，并且只有可能由 pending 到  resolved / fullfilled  或者 rejected，**当状态发生改变后将不再变化**。

```js
let p = new Promise((res,rej) => {
  console.log('rej start');
  rej('hello');
  console.log('res start');
  res('world');
})
p.then(res => {console.log(res)}).catch(err => {console.log(err)})
```

输出的将会是

```shell
rej start
res start
hello
```

3. **改变 Promise 的状态的方法**：resolve 回调，reject 回调，throw 抛出错误，代码出错
4. **then 可以注册两个回调**：成功 resolve 和失败 rejected 的，如果失败的没有写则默认为 `reason=>{throw reason}`。
5. **then 回调执行的条件**：当 promise 的状态改变了，那么所有的 .then() 都会被执行，但是如果 promise 的状态为 pending 则 .then() 全都不会执行
6. **状态的改变与then的执行顺序**：.then() 方法是用于注册回调函数的，也是同步执行的，但是 then 注册的回调函数是异步执行的。在实例化一个 promise 的时候，当 promise 中的是同步任务，并且调用 resolve / reject 话，就是先改变 promise 的状态，然后当使用 then 方法绑定回调函数的时候回调函数就会被立即执行；当 promise 中的代码是异步任务，并且在异步任务中调用 resolve / reject 的话，就会先执行 then 方法绑点回调函数，然后在异步任务结束后调用 resolve / reject 的时候会先修改 promise 的状态，最后执行回调函数。
7. **then 回调函数的结果**：回调函数 then 执行的结果是一个 新的 promise 对象，它的状态和结果由回调函数中的返回值决定

```js
let p = new Promise((res,rej)=>{
  res('ok');
})
let result = p.then(res=>{
  // 如果代码出错 / throw ==> 状态：rejected 结果:错误
  // throw("错误")
  // 如果返回的是普通的数据 ==> 状态：resolved 结果:33
  // return 33
  // 如果返回的是promise ==> 状态：promise状态 结果:promise结果
  // return Promise.resolve(55)
},reason=>{
  console.log(reason);               
})
```

8. **中断 then 的链式回调方法**：返回一个 pending 状态的回调 `return new Promise(()=>{});`



## util.promisify 方法

在 nodejs 中因为历史原因，nodejs 中的很多 api 都是采用回调的形式处理异步的，后面 nodejs 提供了一个 `util.promisify()` 方法将原本回调函数是错误优先的函数（(err,data)=>{},错误结果为第一参数）进行 promise 转化。如

```js
const fs = require("fs");
const uitl = require("util")
const myReadFile = util.promisify(fs.readFile);

myReadFile('./data.txt').then(res=>{
  console.log(res.toString());
}).catch(err=>{
  console.log(err);
})
```



# 自定义 Promise API

## 1. 分析

+ Promise 是一个类（构造函数）。
+ Promise 有两个私有实例属性 状态：PromiseState，结果：PromiseValue
+ 这个类在实例化应该传入一个执行器 excutor 。
+ 这个执行器是一个接受两个参数 resolve 和 reject ，而且都是回调函数
+ Promise 上应该有两 实例方法 then 和 catch ，并且接受一到两个回调函数，用于注册成功 / 失败回调函数
+ Promise 上应该还有几个静态的方法，Promise.resolve()，Promise.reject()，Promise.all()，Promise.rece()。用于快速处理或者生产 promise

```js
(function(){
  /**定义构造函数*/
  function Promise(excutor){}
  /**
  定义实例方法 then 指定成功和失败的回调函数
  返回一个行的promise对象
  */
  Promise.prototype.then = function(onResolved,onRejected){}
  /**
  定义实例方法 catch 指定失败的回调函数
  返回一个行的promise对象
  */
  Promise.prototype.catch = function(onRejected){}
  /**
  定义实例方法 finally 无论结果为 resolve 还是 reject都执行回调
  */
  Promise.prototype.finally = function(callback){}
  /**
  定义静态方法 resolve 
  返回一个新的成功的promise对象
  */
  Promise.resolve = function (value){}
  /**
  定义静态方法 reject 
  返回一个新的失败的promise对象
  */
  Promise.reject = function (reason){}
  /**
  定义静态方法 all 同时管理多个 promise 实例
  @param promises promise实例对象数组
  返回一个状态由 promises 的全部状态决定的新promise对象
  */
  Promise.all = function (promises){}
  /**
  定义静态方法 race 同时管理多个 promise 实例
  @param promises promise实例对象数组
  返回一个状态由 promises 中第一个完成的promise状态决定的新promise对象，结果是第一个完成的 promise 的结果
  */
  Promise.race = function (promises){}
})(window)
```



## 2.实现

### **Promise属性**

每个promise实例中都会有自己的状态和结果，并且会存储通过 then 绑定的回调函数

```js
function Promise(excutor){
  const _this = this;
  _this.status = 'pending';//存储实例的状态，初始状态为 pending
  _this.data = undefined;//存储实例的结果，初始状态为 undefined
  _this.callbacks = [];//保存通过 then 方法绑定的回调函数 结构为 {onResolved(value){},onRejected(reason){}}
}
```

### **Promise构造器定义**

在原 Promise 中实例化的时候应该是`new Promise((resolve,reject)=>{})`，在实例化的时候 执行器中的代码会同步执行，在决定状态的时候才会调用 resolve / reject，而这两个 回调是由 Promise 提供的，excutor 是使用者提供的。

构造器的代码为

```js
function Promise(excutor){
  const _this = this;
  /**存储实例的状态，初始状态为 pending*/
  _this.status = 'pending';
  /**存储实例的结果，初始状态为 undefined*/
  _this.data = undefined;
  /**保存通过 then 方法绑定的回调函数 结构为 {onResolved(value){},onRejected(reason){}}*/
  _this.callbacks = [];
  
  /**交给使用者的成功回调*/
  function resolve(value){
    // 如果状态已经改变了则不能继续改变
    if(_this.status !== 'pending') return;
    //修改状态 --> resolved
    _this.status = 'resolved';
    //保存数据
    _this.data = value;
    //如果有待执行的回调就执行 onResolved 回调，并且是异步执行的
    if(_this.callbacks.length > 0){
      _this.callbacks.forEach(item => {
        // 模仿微任务队列延迟执行
        setTimeout(()=>{
          item.onResolved(value);
        })
      })
    }
  }
  /**交给使用者的失败回调*/
  function reject(reason){
    if(_this.status !== 'pending') return;
    _this.status = 'rejected';
    _this.data = value;
    if(_this.callbacks.length > 0){
      _this.callbacks.forEach(item => {
        setTimeout(()=>{
          item.onRejected(reason);
        })
      })
    }
  }
  
  /**执行 excutor执行器 如果有代码错误则执行reject*/
  try{
    excutor(resolve,reject);
  }catch(err){
    reject(err)
  }
}
```

这里的**关键点**是

1. 执行器excutor是外接传入的，但是接受的两个回调函数是 promise 提供的。
2. excutor 是立即执行的，promise会把成功 和 失败的操作方法传递出去，所以 resolve,reject 是promise提供的。
3. 如果excutor 的代码有错误就应该直接 执行 失败的 reject
4. 状态一但发生改变就不能再改变
5. resolve / reject 一定是异步执行的

```
我（promise），同伴（使用者）
某天，同伴来到我的店里面（实例化）让我准备些道具之后方便通信，然后给了我一个袋子（excutor），我看了看袋子有没有问题（执行excutor），如果有我把两个遥控器（resolve,reject）
```

> **注意**：
>
> 上面模仿微任务队列的时候使用的是 setTimeout 方法在测试的时候，如果也使用到了定时器或者其他的宏任务的话，输出或者执行的顺序会达不到 真正的promise的效果，所以可以使用 node 里面的 `process.nextTick()` 方法模拟，但是只能在node环境中使用。
>
> ```js
> process.nextTick(function () {
>    handle(onRejected);
> });
> ```
>
> 又或者使用 `MutationObserver` 来实现
>
> ```js
> var observer = new MutationObserver(function () {
>    handle(onRejected);
> });
> var targetNode = document.createElement("div");
> targetNode.setAttribute("data-h","2")
> var observerOptions = {
>   childList: true,  // 观察目标子节点的变化，是否有添加或者删除
>   attributes: true, // 观察属性变动
>   subtree: true     // 观察后代节点，默认为 false
> }
> observer.observe(targetNode, observerOptions);
> ```
> 
> 当属性发送改变的时候就会执行回调，这个回调是微任务。所以可以使用`targetNode.setAttribute("data-h","3")`



### then 绑定成功 / 失败的回调函数

then 方法可以绑定成功和失败的回调函数，并且当状态已经改变了 、不为 pending 的时候会根据状态执行相应的回调函数，并且是异步调用的。返回的是一个 promise 对象

```js
/**
定义实例方法 then 指定成功和失败的回调函数
返回一个行的promise对象
*/
Promise.prototype.then = function(onResolved,onRejected){
  const _this = this;
  //返回一个promise对象
 	return new Promise((resolve,reject)=>{
    if(_this.status === 'pending'){
    //状态为 pending 时，加入回调队列，在执行队列的时候也需要修改返回的 promise 的状态和结果
      _this.callbacks.push({
        onResolvedCallback(value){
          try{
            const result = onRejected(_this.data);
            if(result instanceof Prmoise){
              result.then(
                value=>resolve(value),
                reason=>reject(reason)
              )
            }else{
              resolve(result)
            }
          }catch(error){
            reject(error)
          }
        },
        onRejectedCallback(reason){
          try{
            const result = onRejected(_this.data);
            if(result instanceof Prmoise){
              result.then(
                value=>resolve(value),
                reason=>reject(reason)
              )
            }else{
              resolve(result)
            }
          }catch(error){
            reject(error)
          }
        },
      })
    }else if(_this.status === 'resolved'){
      //状态为 resolved 时，异步调用成功回调
      setTimeout(()=>{
        try{
          const result = onResolved(_this.data);
          if(result instanceof Prmoise){
            //如果回调函数返回的是promise，return的promise结果就是这个promise的结果
            result.then(
              value=>resolve(value),
              reason=>reject(reason)
            )
            //也可以这样写 ：result.then(resolve,reject)
          }else{
            //如果回调函数返回的不是promise，return的promise就是成功，value就是promise的结果
            resolve(result)
          }
        }catch(error){
          //如果代码抛出异常，return 的 promise就是失败的，reason 就是 error
          reject(error)
        }
      })
    }else{
      //状态为 rejected 时，异步调用是失败回调
      setTimeout(()=>{
        try{
          const result = onRejected(_this.data);
          if(result instanceof Prmoise){
            result.then(
              value=>resolve(value),
              reason=>reject(reason)
            )
          }else{
            resolve(result)
          }
        }catch(error){
          reject(error)
        }
      })
    }
  })
}
```

抽离公共代码整合后可以是这样的

```js
Promise.prototype.then = function(onResolved,onRejected){
  const _this = this;
  //返回一个promise对象
 	return new Promise((resolve,reject)=>{
    //设置默认的回调 ，实现异常穿透的关键
    onResolved = typeof onResolved === 'function' ? onResolved : value => value
    //实现异常穿透的关键
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason}
    /**
    调用指定的回调函数处理。根据执行结果改变return的pormise状态
    1.如果回调函数返回的是promise，return的promise结果就是这个promise的结果
    2.如果回调函数返回的不是promise，return的promise就是成功，value就是promise的结果
    3.如果代码抛出异常，return 的 promise就是失败的，reason 就是 error
    */
    function handle(callback){
      try{
        const result = callback(_this.data);
        if(result instanceof Prmoise){
          result.then(
            value=>resolve(value),
            reason=>reject(reason)
          )
        }else{
          resolve(result)
        }
      }catch(error){
        reject(error)
      }
    }
    
    if(_this.status === 'pending'){
    //状态为 pending 时，加入回调队列，在执行队列的时候也需要修改返回的 promise 的状态和结果
      _this.callbacks.push({
        onResolvedCallback(value){
          handle(onResolved)
        },
        onRejectedCallback(reason){
          handle(onRejected)
        },
      })
    }else if(_this.status === 'resolved'){
      //状态为 resolved 时，异步调用成功回调
      setTimeout(()=>{
        handle(onResolved);
      })
    }else{
      //状态为 rejected 时，异步调用是失败回调
      setTimeout(()=>{
        handle(onRejected);
      })
    }
  })
}
```



### catch的实现

```js
  /**
  定义实例方法 catch 指定失败的回调函数
  返回一个行的promise对象
  */
  Promise.prototype.catch = function(onRejected){
    return this.then(undefined,onRejected);
  }
```

### finally 的实现

```js
Promise.prototype.finally = function(callback){
  return this.then(res=>{
    return Promise.resolve(f()).then(() => value);
  },err=>{
    return Promise.resolve(f()).then(() => {throw err});
  })
}
```



### 静态方法 resolve 和 reject 实现

静态方法 resolve 和 reject 会返回一个新的 promise 对象，并且如果方法接收的是一般类型的数据，返回的 promise 的结果就是接收的数据，对于 resolve 方法状态是成功的，对于reject方法状态是失败的；如果方法接收的是一个 promise 对象，那么返回的 promise 的结果由 接收的 promise 的结果决定，对于 resolve 方法状态是由 接收的 promise 状态决定，对于reject方法状态是失败的。

**resolve 实现**

```js
/**
	定义静态方法 resolve 
  返回一个新的成功的promise对象
  */
Promise.resolve = function (value){
  return new Promise((resolve,reject)=>{
    if(value instanceof Promise){
    	//是promise
      value.then(resolve,reject);
    }else{
			//普通数据
      resolve(value);
    }
  })
}
```



**reject 实现**

```js
/**
  定义静态方法 reject 
  返回一个新的失败的promise对象
  */
Promise.reject = function (reason){
  return new Promise((resolve,reject)=>{
    reject(reason);
  })
}
```





### 静态方法的 all 和 race 实现

all 可以接收一个 promise 的数组。all是执行所有的 promise 执行完毕或者中间有失败的 promise 时返回一个 promise 对象，如果全部的 promise 都是成功的则返回的 promise 对象的状态是成功的，结果是所有成功状态的 promise 的结果，否则是失败的，执行失败的回调；race和all一样，接收一个 promise 的数组。不同的是状态和结果由第一个执行完的 promise 决定。

**all 实现**

```js
/**
  定义静态方法 all 同时管理多个 promise 实例
  @param promises promise实例对象数组
  返回一个状态由 promises 的全部状态决定的新promise对象
  */
Promise.all = function (promises){
  //成功回答的结果
  const resolvedArr = new Array(promises.length);
  //成功的次数
  let resolvedCount = 0;
  return new Promise((resolve,reject) => {
    promises.forEach((p,index) => {
      //p.then( 不能处理传入的promises数组中混入了普通数据的情况，所以在 then 之前进行成功的 resolve Promise转化
      Promise.resolve(p).then(
        value => {
        	resolvedCount ++;
          resolvedaArr[index] = value;
          //全部完成
          if(resolvedCount === promiseArr.length){
            resolve(resolvedaArr);
          }
      	},
        reason => {
          reject(reason);
        }
      )
    })
  })
}
```



**race 实现**

```js
/**
  定义静态方法 race 同时管理多个 promise 实例
  @param promises promise实例对象数组
  返回一个状态由 promises 中第一个完成的promise状态决定的新promise对象，结果是第一个完成的 promise 的结果
  */
Promise.race = function (promises){
  return new Promise((resolve,reject) => {
    promises.forEach((p.index) => {
    	//p.then( 不能处理传入的promises数组中混入了普通数据的情况，所以在 then 之前进行成功的 resolve Promise转化
      Promise.resolve(p).then(
        value => {
        	resolve(value);
      	},
        reason => {
          reject(reason);
        }
      )
    })
  })
}
```



## 类式定义

```ts
interface Callback<T> {
    (value?: any): T;
}
interface Excutor {
    (resolve: Callback<any>, reject: Callback<any>): void;
}
interface CallBackItem {
    onResolved: Callback<any>;
    onRejected: Callback<any>
}
enum StateType{
    RESOLVERD = "resolved",
    REJECTED = "rejected",
    PENDING = "pending"
}

class MyPromise {

    private status: string = StateType.PENDING;
    private data: any = undefined;
    private callbacks: CallBackItem[] = [];

    constructor(excutor: Excutor) {
        try {
            excutor(this.resolve.bind(this), this.reject.bind(this));
        } catch (error) {
            this.reject(error);
        }
    }

    private resolve(value: any): void{
        if(this.status !== StateType.PENDING) return;
        this.status = StateType.RESOLVERD;
        this.data = value;
        if(this.callbacks.length > 0){
            this.callbacks.forEach( item => {
                setTimeout(() => {
                    item.onResolved(value);
                });
            })
        }
    }

    private reject(reason: any):void{
        if(this.status !== StateType.PENDING) return;
        this.status = StateType.REJECTED;
        this.data = reason;
        if(this.callbacks.length > 0){
            this.callbacks.forEach(item => {
                setTimeout(() => {
                    item.onRejected(reason);
                })
            })
        }
    }

    private handle(callback: Callback<any>, resolve: Callback<any>, reject: Callback<any>): void{
        try {
            let result = callback(this.data);
            if(result instanceof MyPromise){
                result.then(value => {
                    resolve(value);
                },reason => {
                    reject(reason);
                })
            }else{
                resolve(result);
            }
        } catch (error) {
            reject(error)
        }
    }

    public then(onResolved: Callback<any>, onRejected?: Callback<any>): MyPromise {
        onResolved = typeof onResolved === "function" ? onResolved : value => value ;
        onRejected = typeof onRejected === "function" ? onRejected : reason => {throw reason};
        return new MyPromise((resolve,reject) => {
            if(this.status === StateType.PENDING){
                this.callbacks.push({
                    onResolved:()=>{
                        this.handle(onResolved, resolve, reject);
                    },
                    onRejected:()=>{
                        this.handle(onRejected, resolve, reject);
                    }
                })
            }else if(this.status === StateType.RESOLVERD){
                setTimeout(()=>{
                    this.handle(onResolved, resolve, reject);
                })
            }else{
                setTimeout(() => {
                    this.handle(onRejected, resolve, reject);
                })
            }
        })
    }

    public catch(onRejected: Callback<any>): MyPromise { 
        return this.then(undefined,onRejected);
    }

    public static resolve(value: any): MyPromise { 
        return new MyPromise((resolve, reject) => {
            if(value instanceof MyPromise){
                value.then(res => {
                    resolve(res);
                },err => {
                    reject(err);
                })
            }else{
                resolve(value);
            }
        })
    }

    public static reject(reason: Error): MyPromise { 
        return new MyPromise((resolve, reject) => {
            reject(reason);
        })
    }

    public static all(promises: MyPromise[]): MyPromise { 
        let resolvedArr = new Array(promises.length);
        let resolvedCount = 0;
        return new MyPromise((resolve, reject) => {
            promises.forEach((p, index) => {
                MyPromise.resolve(p).then(res => {
                    resolvedCount ++;
                    resolvedArr.push(res);
                    if (resolvedCount === resolvedArr.length){
                        resolve(resolvedArr);
                    }
                },err => {
                    reject(err);
                })
            })
        })
    }

    public static race(promises: MyPromise[]): MyPromise { 
        return new MyPromise((resolve,reject) => {
            promises.forEach(p => {
                MyPromise.resolve(p).then(res => {
                    resolve(res);
                },err => {
                    reject(err);
                })
            })
        })
    }
}
```



# async 和 await 的理解

async 和 await 是 ES6 中生成器 generator 的语法糖，它配合 Promise 可以更加有效的将异步代码写成同步的形式，相比于 generator 语义性更好，更加易于理解。

## async 函数

`async function bar(){....}`

+ 函数的返回值为 promise 对象
+ promise 对象的结果由 async 函数执行的返回值决定

## await 表达式

```js
async function bar(){
  let result1 = await 1;
  let result2 = await foo();
}
```

await 右侧的表达式一般为 promise 对象，但是也可以是其他的值

+ 如果表达式是 promise 对象，await 返回的是 promise 成功的值
+ 如果表达式是其他的值，直接将值作为 await 的返回值

注意：

+ await  只能在写在 async 函数中。
+ 如果 await 的 promise 失败了，就会抛出异常，需要通过 try{}catch(){} 进行捕获处理。



# Promise async await 相关题

## 面试题1

```js
setTimeout(()=>{
  console.log(1);
  Promise.resolve().then(()=>{
    console.log(2);
  })
})
setTimeout(()=>{
  console.log(3);
},100)
Promise.resolve().then(()=>{
  console.log(4);
  Promise.resolve().then(()=>{
    console.log(5);
  })
})
Promise.resolve().then(()=>{
  console.log(6);
})
console.log(7);
```

```
7 4 6 5 1 2 3
```



## 面试题2

```js
const first = () => (new Promise((resolve,reject)=>{
  console.log(3);
  let p = new Promise((resolve, reject)=>{
    console.log(7);
    setTimeout(()=>{
      console.log(5);
      resolve(6)
    },0)
    resolve(1)
  })
  resolve(2)
  p.then(arg=>{
    console.log(arg)
  })
}))

first().then(arg=>{
  console.log(arg);
})
console.log(4)
```

```
3 7 4 1 2 5
```



## 面试题3

```js
setTimeout(()=>{
  console.log(0);
})
new Promise((resolve,reject)=>{
  console.log(1);
  resolve();
}).then(()=>{
  console.log(2)
  new Promise((resolve,reject)=>{
    console.log(3);
    resolve();
  }).then(()=>{
   	console.log(4);
  }).then(()=>{
   	console.log(5);
  })
}).then(()=>{
  console.log(6);
})

new Promise((resolve,reject)=>{
  console.log(7);
  resolve();
}).then(()=>{
  console.log(8);
})
```

```
1 7 2 3 8 4 6 5 0
```



## 面试题4

```js
async function a1(){
  console.log(0)
  await a2();
  console.log(1)
}

async function a2(){
  console.log(2)
}

console.log(3)

setTimeout(()=>{
  console.log(4);
},0)

Promise.resolve().then(()=>{
  console.log(5)
})

a1();

let promise2=new Promise(resolve=>{
  resolve(6)
  console.log(7)
})

promise2.then(res=>{
  console.log(res)
  Promise.resolve().then(()=>{
    console.log(8)
  })
})
console.log(9)
```

```
3 0 2 7 9 5 1 6 8 4
```



记住一个执行的原则：**先执行主代码（同步代码），在执行微任务队列，在执行宏任务队列，当在执行宏任务的时候会查看微任务队列是否已经为空队列，是则继续执行宏任务，否则执行微任务队列**