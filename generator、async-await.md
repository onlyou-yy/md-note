# async/await实现

 在多个回调依赖的场景中，尽管Promise通过链式调用取代了回调嵌套，但过多的链式调用可读性仍然不佳，流程控制也不方便，ES7 提出的async 函数，终于让 JS 对于异步操作有了终极解决方案，简洁优美地解决了以上两个问题。

> 设想一个这样的场景，异步任务a->b->c之间存在依赖关系，如果我们通过then链式调用来处理这些关系，可读性并不是很好，如果我们想控制其中某个过程，比如在某些条件下，b不往下执行到c，那么也不是很方便控制

```js
Promise.resolve(a)
  .then(b => {
    // do something
  })
  .then(c => {
    // do something
  })
```

> 但是如果通过async/await来实现这个场景，可读性和流程控制都会方便不少。

```js
async () => {
  const a = await Promise.resolve(a);
  const b = await Promise.resolve(b);
  const c = await Promise.resolve(c);
}
```



那么我们要如何实现一个async/await呢，首先我们要知道，**async/await实际上是对Generator（生成器）的封装**，是一个语法糖。由于Generator出现不久就被async/await取代了，很多同学对Generator比较陌生，因此我们先来看看Generator的用法：

> ES6 新引入了 Generator 函数，可以通过 yield 关键字，把函数的执行流挂起，通过next()方法可以切换到下一个状态，为改变执行流程提供了可能，从而为异步编程提供解决方案。

```JS
function* myGenerator() {
  yield '1'
  yield '2'
  return '3'
}

const gen = myGenerator();  // 获取迭代器
gen.next()  //{value: "1", done: false}
gen.next()  //{value: "2", done: false}
gen.next()  //{value: "3", done: true}
```

也可以通过给`next()`传参, 让yield具有返回值

```js
function* myGenerator() {
  console.log(yield '1')  //test1
  console.log(yield '2')  //test2
  console.log(yield '3')  //test3
}

// 获取迭代器
const gen = myGenerator();

gen.next()
gen.next('test1')
gen.next('test2')
gen.next('test3')
```

我们看到Generator的用法，应该️会感到很熟悉，`*/yield`和`async/await`看起来其实已经很相似了，它们都提供了暂停执行的功能，但二者又有三点不同：

- `async/await`自带执行器，不需要手动调用next()就能自动执行下一步
- `async`函数返回值是Promise对象，而Generator返回的是生成器对象
- `await`能够返回Promise的resolve/reject的值

**我们对async/await的实现，其实也就是对应以上三点封装Generator**

## 1.自动执行

我们先来看一下，对于这样一个Generator，手动执行是怎样一个流程

```js
function* myGenerator() {
  yield Promise.resolve(1);
  yield Promise.resolve(2);
  yield Promise.resolve(3);
}

// 手动执行迭代器
const gen = myGenerator()
gen.next().value.then(val => {
  console.log(val)
  gen.next().value.then(val => {
    console.log(val)
    gen.next().value.then(val => {
      console.log(val)
    })
  })
})

//输出1 2 3
```

我们也可以通过给`gen.next()`传值的方式，让yield能返回resolve的值

```js
function* myGenerator() {
  console.log(yield Promise.resolve(1))   //1
  console.log(yield Promise.resolve(2))   //2
  console.log(yield Promise.resolve(3))   //3
}

// 手动执行迭代器
const gen = myGenerator()
gen.next().value.then(val => {
  // console.log(val)
  gen.next(val).value.then(val => {
    // console.log(val)
    gen.next(val).value.then(val => {
      // console.log(val)
      gen.next(val)
    })
  })
})
```

显然，手动执行的写法看起来既笨拙又丑陋，我们希望生成器函数能自动往下执行，且yield能返回resolve的值，基于这两个需求，我们进行一个基本的封装，这里`async/await`是关键字，不能重写，我们用函数来模拟：

```js
function run(gen) {
  var g = gen()                     //由于每次gen()获取到的都是最新的迭代器,因此获取迭代器操作要放在_next()之前,否则会进入死循环

  function _next(val) {             //封装一个方法, 递归执行g.next()
    var res = g.next(val)           //获取迭代器对象，并返回resolve的值
    if(res.done) return res.value   //递归终止条件
    res.value.then(val => {         //Promise的then方法是实现自动迭代的前提
      _next(val)                    //等待Promise完成就自动执行下一个next，并传入resolve的值
    })
  }
  _next()  //第一次执行
}
```

对于我们之前的例子，我们就能这样执行：

```js
function* myGenerator() {
  console.log(yield Promise.resolve(1))   //1
  console.log(yield Promise.resolve(2))   //2
  console.log(yield Promise.resolve(3))   //3
}

run(myGenerator)
```

这样我们就初步实现了一个`async/await`。上边的代码只有五六行，但并不是一下就能看明白的，我们之前用了四个例子来做铺垫，也是为了让读者更好地理解这段代码。 简单来说，我们封装了一个run方法，run方法里我们把执行下一步的操作封装成`_next()`，每次Promise.then()的时候都去执行`_next()`，实现自动迭代的效果。在迭代的过程中，我们还把resolve的值传入`gen.next()`，使得yield得以返回Promise的resolve的值

> 这里插一句，是不是只有`.then方法`这样的形式才能完成我们自动执行的功能呢？答案是否定的，yield后边除了接Promise，还可以接`thunk函数`，thunk函数不是一个新东西，所谓thunk函数，就是**单参的只接受回调的函数**，详细介绍可以看[阮一峰Thunk 函数的含义和用法](https://link.juejin.cn?target=https%3A%2F%2Fp1-jj.byteimg.com%2Ftos-cn-i-t2oaga2asx%2Fgold-user-assets%2F2020%2F3%2F15%2F170dc5e88df6c208~tplv-t2oaga2asx-image.image)，无论是Promise还是thunk函数，其核心都是通过**传入回调**的方式来实现Generator的自动执行。thunk函数只作为一个拓展知识，理解有困难的同学也可以跳过这里，并不影响后续理解。

## 2.返回Promise & 异常处理

虽然我们实现了Generator的自动执行以及让yield返回resolve的值，但上边的代码还存在着几点问题：

1. **需要兼容基本类型**：这段代码能自动执行的前提是`yield`后面跟Promise，为了兼容后面跟着基本类型值的情况，我们需要把yield跟的内容(`gen().next.value`)都用`Promise.resolve()`转化一遍
2. **缺少错误处理**：上边代码里的Promise如果执行失败，就会导致后续执行直接中断，我们需要通过调用`Generator.prototype.throw()`，把错误抛出来，才能被外层的try-catch捕获到
3. **返回值是Promise**：`async/await`的返回值是一个Promise，我们这里也需要保持一致，给返回值包一个Promise

我们改造一下run方法：

```js
function run(gen) {
  //把返回值包装成promise
  return new Promise((resolve, reject) => {
    var g = gen()

    function _next(val) {
      //错误处理
      try {
        var res = g.next(val) 
      } catch(err) {
        return reject(err); 
      }
      if(res.done) {
        return resolve(res.value);
      }
      //res.value包装为promise，以兼容yield后面跟基本类型的情况
      Promise.resolve(res.value).then(
        val => {
          _next(val);
        }, 
        err => {
          //抛出错误
          g.throw(err)
        });
    }
    _next();
  });
}

```

然后我们可以测试一下：

```js
function* myGenerator() {
  try {
    console.log(yield Promise.resolve(1)) 
    console.log(yield 2)   //2
    console.log(yield Promise.reject('error'))
  } catch (error) {
    console.log(error)
  }
}

const result = run(myGenerator)     //result是一个Promise
//输出 1 2 error

```

到这里，一个`async/await`的实现基本完成了。最后我们可以看一下babel对async/await的转换结果，其实整体的思路是一样的，但是写法稍有不同：

```js
//相当于我们的run()
function _asyncToGenerator(fn) {
  // return一个function，和async保持一致。我们的run直接执行了Generator，其实是不太规范的
  return function() {
    var self = this
    var args = arguments
    return new Promise(function(resolve, reject) {
      var gen = fn.apply(self, args);

      //相当于我们的_next()
      function _next(value) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'next', value);
      }
      //处理异常
      function _throw(err) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'throw', err);
      }
      _next(undefined);
    });
  };
}

function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
  try {
    var info = gen[key](arg);
    var value = info.value;
  } catch (error) {
    reject(error);
    return;
  }
  if (info.done) {
    resolve(value);
  } else {
    Promise.resolve(value).then(_next, _throw);
  }
}

```

使用方式：

```js
const foo = _asyncToGenerator(function* () {
  try {
    console.log(yield Promise.resolve(1))   //1
    console.log(yield 2)                    //2
    return '3'
  } catch (error) {
    console.log(error)
  }
})

foo().then(res => {
  console.log(res)                          //3
})

```

有关`async/await`的实现，到这里就告一段落了。但是直到结尾，我们也不知道await到底是如何暂停执行的，有关await暂停执行的秘密，我们还要到Generator的实现中去寻找答案

# Generator实现

我们从一个简单的Generator使用实例开始，一步步探究Generator的实现原理：

```js
function* foo() {
  yield 'result1'
  yield 'result2'
  yield 'result3'
}
  
const gen = foo()
console.log(gen.next().value)
console.log(gen.next().value)
console.log(gen.next().value)

```

我们可以在[babel官网](https://link.juejin.cn?target=https%3A%2F%2Fbabeljs.io%2Frepl%2F%23%3Fbrowsers%3D%26build%3D%26builtIns%3Dfalse%26spec%3Dfalse%26loose%3Dfalse%26code_lz%3DGYVwdgxgLglg9mAVAAmHOAKAlMg3gKGWQE8YBTAGwBNkByAJzIGcQKoBGWwk86uxlmwBMXIqUo0GzVlADMXAL7d8EBEyjIA5mTDIAvKnTYVauBTIA6CnE0ZtYC2DIAPKNgsA3AIYUQZLCZgTGaW1rb2ji5uWJ4-fgGqQSFWNnY6ka7u3r7-QA%26debug%3Dfalse%26forceAllTransforms%3Dfalse%26shippedProposals%3Dfalse%26circleciRepo%3D%26evaluate%3Dfalse%26fileSize%3Dfalse%26timeTravel%3Dfalse%26sourceType%3Dmodule%26lineWrap%3Dtrue%26presets%3Des2015%2Creact%2Cstage-2%26prettier%3Dfalse%26targets%3D%26version%3D7.5.5%26externalPlugins%3D)上在线转化这段代码，看看ES5环境下是如何实现Generator的：

```js
"use strict";

var _marked =
/*#__PURE__*/
regeneratorRuntime.mark(foo);

function foo() {
  return regeneratorRuntime.wrap(function foo$(_context) {
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
          _context.next = 2;
          return 'result1';

        case 2:
          _context.next = 4;
          return 'result2';

        case 4:
          _context.next = 6;
          return 'result3';

        case 6:
        case "end":
          return _context.stop();
      }
    }
  }, _marked);
}

var gen = foo();
console.log(gen.next().value);
console.log(gen.next().value);
console.log(gen.next().value);

```

代码咋一看不长，但如果仔细观察会发现有两个不认识的东西 —— `regeneratorRuntime.mark`和`regeneratorRuntime.wrap`，这两者其实是 regenerator-runtime 模块里的两个方法，regenerator-runtime 模块来自facebook的 regenerator 模块，完整代码在[runtime.js](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Fregenerator%2Fblob%2Fmaster%2Fpackages%2Fregenerator-runtime%2Fruntime.js)，这个runtime有700多行...-_-||，因此我们不能全讲，不太重要的部分我们就简单地过一下，重点讲解暂停执行相关部分代码

> 个人觉得啃源码的效果不是很好，建议读者拉到末尾先看结论和简略版实现，源码作为一个补充理解

## regeneratorRuntime.mark()

`regeneratorRuntime.mark(foo)`这个方法在第一行被调用，我们先看一下runtime里mark()方法的定义

```js
//runtime.js里的定义稍有不同，多了一些判断，以下是编译后的代码
runtime.mark = function(genFun) {
  genFun.__proto__ = GeneratorFunctionPrototype;
  genFun.prototype = Object.create(Gp);
  return genFun;
};

```

这里边`GeneratorFunctionPrototype`和`Gp`我们都不认识，他们被定义在runtime里，不过没关系，我们只要知道`mark()方法`为生成器函数（foo）绑定了一系列原型就可以了，这里就简单地过了

## regeneratorRuntime.wrap()

从上面babel转化的代码我们能看到，执行`foo()`，其实就是执行`wrap()`，那么这个方法起到什么作用呢，他想包装一个什么东西呢，我们先来看看wrap方法的定义：

```js
//runtime.js里的定义稍有不同，多了一些判断，以下是编译后的代码
function wrap(innerFn, outerFn, self) {
  var generator = Object.create(outerFn.prototype);
  var context = new Context([]);
  generator._invoke = makeInvokeMethod(innerFn, self, context);

  return generator;
}

```

wrap方法先是创建了一个generator，并继承`outerFn.prototype`；然后new了一个`context对象`；`makeInvokeMethod方法`接收`innerFn(对应foo$)`、`context`和`this`，并把返回值挂到`generator._invoke`上；最后return了generator。**其实wrap()相当于是给generator增加了一个_invoke方法**

这段代码肯定让人产生很多疑问，outerFn.prototype是什么，Context又是什么，makeInvokeMethod又做了哪些操作。下面我们就来一一解答：

> `outerFn.prototype`其实就是`genFun.prototype`，

这个我们结合一下上面的代码就能知道

> `context`可以直接理解为这样一个全局对象，用于储存各种状态和上下文：

```js
var ContinueSentinel = {};

var context = {
  done: false,
  method: "next",
  next: 0,
  prev: 0,
  abrupt: function(type, arg) {
    var record = {};
    record.type = type;
    record.arg = arg;

    return this.complete(record);
  },
  complete: function(record, afterLoc) {
    if (record.type === "return") {
      this.rval = this.arg = record.arg;
      this.method = "return";
      this.next = "end";
    }

    return ContinueSentinel;
  },
  stop: function() {
    this.done = true;
    return this.rval;
  }
};

```

> ```
> makeInvokeMethod`的定义如下，它return了一个`invoke方法`，invoke用于判断当前状态和执行下一步，其实就是我们调用的`next()
> ```

```js
//以下是编译后的代码
function makeInvokeMethod(innerFn, context) {
  // 将状态置为start
  var state = "start";

  return function invoke(method, arg) {
    // 已完成
    if (state === "completed") {
      return { value: undefined, done: true };
    }
    
    context.method = method;
    context.arg = arg;

    // 执行中
    while (true) {
      state = "executing";

      var record = {
        type: "normal",
        arg: innerFn.call(self, context)    // 执行下一步,并获取状态(其实就是switch里边return的值)
      };

      if (record.type === "normal") {
        // 判断是否已经执行完成
        state = context.done ? "completed" : "yield";

        // ContinueSentinel其实是一个空对象,record.arg === {}则跳过return进入下一个循环
        // 什么时候record.arg会为空对象呢, 答案是没有后续yield语句或已经return的时候,也就是switch返回了空值的情况(跟着上面的switch走一下就知道了)
        if (record.arg === ContinueSentinel) {
          continue;
        }
        // next()的返回值
        return {
          value: record.arg,
          done: context.done
        };
      }
    }
  };
}

```

> 为什么`generator._invoke`实际上就是`gen.next`呢，因为在runtime对于next()的定义中，next()其实就return了_invoke方法

```js
// Helper for defining the .next, .throw, and .return methods of the
// Iterator interface in terms of a single ._invoke method.
function defineIteratorMethods(prototype) {
    ["next", "throw", "return"].forEach(function(method) {
      prototype[method] = function(arg) {
        return this._invoke(method, arg);
      };
    });
}

defineIteratorMethods(Gp);

```

## 低配实现 & 调用流程分析

这么一遍源码下来，估计很多读者还是懵逼的，毕竟源码中纠集了很多概念和封装，一时半会不好完全理解，让我们跳出源码，实现一个简单的Generator，然后再回过头看源码，会得到更清晰的认识

```js
// 生成器函数根据yield语句将代码分割为switch-case块，后续通过切换_context.prev和_context.next来分别执行各个case
function gen$(_context) {
  while (1) {
    switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 'result1';

      case 2:
        _context.next = 4;
        return 'result2';

      case 4:
        _context.next = 6;
        return 'result3';

      case 6:
      case "end":
        return _context.stop();
    }
  }
}

// 低配版context  
var context = {
  next:0,
  prev: 0,
  done: false,
  stop: function stop () {
    this.done = true
  }
}

// 低配版invoke
let gen = function() {
  return {
    next: function() {
      value = context.done ? undefined: gen$(context)
      done = context.done
      return {
        value,
        done
      }
    }
  }
} 

// 测试使用
var g = gen() 
g.next()  // {value: "result1", done: false}
g.next()  // {value: "result2", done: false}
g.next()  // {value: "result3", done: false}
g.next()  // {value: undefined, done: true}

```

这段代码并不难理解，我们分析一下调用流程：

1. 我们定义的`function* `生成器函数被转化为以上代码
2. 转化后的代码分为三大块：
   - `gen$(_context)`由yield分割生成器函数代码而来
   - `context对象`用于储存函数执行上下文
   - `invoke()方法`定义next()，用于执行gen$(_context)来跳到下一步
3. 当我们调用`g.next()`，就相当于调用`invoke()方法`，执行`gen$(_context)`，进入switch语句，switch根据context的标识，执行对应的case块，return对应结果
4. 当生成器函数运行到末尾（没有下一个yield或已经return），switch匹配不到对应代码块，就会return空值，这时`g.next()`返回`{value: undefined, done: true}`

从中我们可以看出，**Generator实现的核心在于`上下文的保存`，函数并没有真的被挂起，每一次yield，其实都执行了一遍传入的生成器函数，只是在这个过程中间用了一个context对象储存上下文，使得每次执行生成器函数的时候，都可以从上一个执行结果开始执行，看起来就像函数被挂起了一样**


作者：写代码像蔡徐抻
链接：https://juejin.cn/post/6844904096525189128