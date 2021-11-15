# 1：基本原理

使用`Vue`时，我们只需要修改数据(`state`)，视图就能够获得相应的更新，这就是响应式系统。要实现一个自己的响应式系统，我们首先要明白要做什么事情：

1. 数据劫持：当数据变化时，我们可以做一些特定的事情
2. 依赖收集：我们要知道那些视图层的内容(`DOM`)依赖了哪些数据(`state`)
3. 派发更新：数据变化后，如何通知依赖这些数据的`DOM`

## 1. 数据劫持

几乎所有的文章和教程，在讲解`Vue`响应式系统时都会先讲：`Vue`使用`Object.defineProperty`来进行数据劫持。那么，我们也从数据劫持讲起，大家可能会对`劫持`这个概念有些迷茫，没有关系，看完下面的内容，你一定会明白。

`Object.defineProperty`的用法在此不多做介绍，不明白的同学可在[MDN](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FObject%2FdefineProperty)上查阅。下面，我们为`obj`定义一个`a`属性

```js
const obj = {}

let val = 1
Object.defineProperty(obj, a, {
  get() { // 下文中该方法统称为getter
    console.log('get property a')
    return val
  },
  set(newVal) { // 下文中该方法统称为setter
    if (val === newVal) return
    console.log(`set property a -> ${newVal}`)
    val = newVal
  }
})
复制代码
```

这样，当我们访问`obj.a`时，打印`get property a`并返回1，`obj.a = 2`设置新的值时，打印`set property a -> 2`。这相当于我们自定义了`obj.a`取值和赋值的行为，使用自定义的`getter`和`setter`来重写了原有的行为，这也就是`数据劫持`的含义。

但是上面的代码有一个问题：我们需要一个全局的变量来保存这个属性的值，因此，我们可以用下面的写法

```js
// value使用了参数默认值
function defineReactive(data, key, value = data[key]) {
  Object.defineProperty(data, key, {
    get: function reactiveGetter() {
      return value
    },
    set: function reactiveSetter(newValue) {
      if (newValue === value) return
      value = newValue
    }
  })
}

defineReactive(obj, a, 1)
复制代码
```

如果`obj`有多个属性呢？我们可以新建一个类`Observer`来遍历该对象

```js
class Observer {
  constructor(value) {
    this.value = value
    this.walk()
  }
  walk() {
    Object.keys(this.value).forEach((key) => defineReactive(this.value, key))
  }
}

const obj = { a: 1, b: 2 }
new Observer(obj)
复制代码
```

如果`obj`内有嵌套的属性呢？我们可以使用递归来完成嵌套属性的数据劫持

```js
// 入口函数
function observe(data) {
  if (typeof data !== 'object') return
  // 调用Observer
  new Observer(data)
}

class Observer {
  constructor(value) {
    this.value = value
    this.walk()
  }
  walk() {
    // 遍历该对象，并进行数据劫持
    Object.keys(this.value).forEach((key) => defineReactive(this.value, key))
  }
}

function defineReactive(data, key, value = data[key]) {
  // 如果value是对象，递归调用observe来监测该对象
  // 如果value不是对象，observe函数会直接返回
  observe(value)
  Object.defineProperty(data, key, {
    get: function reactiveGetter() {
      return value
    },
    set: function reactiveSetter(newValue) {
      if (newValue === value) return
      value = newValue
      observe(newValue) // 设置的新值也要被监听
    }
  })
}

const obj = {
  a: 1,
  b: {
    c: 2
  }
}

observe(obj)
复制代码
```

对于这一部分，大家可能有点晕，接下来梳理一下：

```shell
执行observe(obj)
├── new Observer(obj),并执行this.walk()遍历obj的属性，执行defineReactive()
    ├── defineReactive(obj, a)
        ├── 执行observe(obj.a) 发现obj.a不是对象，直接返回
        ├── 执行defineReactive(obj, a) 的剩余代码
    ├── defineReactive(obj, b) 
	    ├── 执行observe(obj.b) 发现obj.b是对象
	        ├── 执行 new Observer(obj.b)，遍历obj.b的属性，执行defineReactive()
                    ├── 执行defineReactive(obj.b, c)
                        ├── 执行observe(obj.b.c) 发现obj.b.c不是对象，直接返回
                        ├── 执行defineReactive(obj.b, c)的剩余代码
            ├── 执行defineReactive(obj, b)的剩余代码
代码执行结束
```

可以看出，上面三个函数的调用关系如下：

![img](vue2和vue3响应式数据原理/44ac0f532d584fcca79f5781d3ce6fcf_tplv-k3u1fbpfcp-watermark.awebp)

三个函数相互调用从而形成了递归，与普通的递归有所不同。 有些同学可能会想，只要在`setter`中调用一下渲染函数来重新渲染页面，不就能完成在数据变化时更新页面了吗？确实可以，但是这样做的代价就是：任何一个数据的变化，都会导致这个页面的重新渲染，代价未免太大了吧。我们想做的效果是：数据变化时，只更新与这个数据有关的`DOM`结构，那就涉及到下文的内容了：`依赖`

## 2. 收集依赖与派发更新

### 依赖

在正式讲解依赖收集之前，我们先看看什么是依赖。举一个生活中的例子：淘宝购物。现在淘宝某店铺上有一块显卡(~~空气~~)处于预售阶段，如果我们想买的话，我们可以点击`预售提醒`，当显卡开始卖的时候，淘宝为我们推送一条消息，我们看到消息后，可以开始购买。

将这个例子抽象一下就是发布-订阅模式：买家点击预售提醒，就相当于在淘宝上登记了自己的信息(订阅)，淘宝则会将买家的信息保存在一个数据结构中(比如数组)。显卡正式开放购买时，淘宝会通知所有的买家：显卡开卖了(发布)，买家会根据这个消息进行一些动作(~~比如买回来挖矿~~)。

在`Vue`响应式系统中，显卡对应数据，那么例子中的买家对应什么呢？就是一个抽象的类: `Watcher`。大家不必纠结这个名字的含义，只需要知道它做什么事情：每个`Watcher`实例订阅一个或者多个数据，这些数据也被称为`wacther`的依赖(商品就是买家的依赖)；当依赖发生变化，`Watcher`实例会接收到数据发生变化这条消息，之后会执行一个回调函数来实现某些功能，比如更新页面(买家进行一些动作)。

![img](vue2和vue3响应式数据原理/ca1bba8bb8054c8dab993d84bfef898f_tplv-k3u1fbpfcp-watermark.awebp)

因此`Watcher`类可以如下实现

```js
class Watcher {
  constructor(data, expression, cb) {
    // data: 数据对象，如obj
    // expression：表达式，如b.c，根据data和expression就可以获取watcher依赖的数据
    // cb：依赖变化时触发的回调
    this.data = data
    this.expression = expression
    this.cb = cb
    // 初始化watcher实例时订阅数据
    this.value = this.get()
  }
  
  get() {
    const value = parsePath(this.data, this.expression)
    return value
  }
  
  // 当收到数据变化的消息时执行该方法，从而调用cb
  update() {
    this.value = parsePath(this.data, this.expression) // 对存储的数据进行更新
    cb()
  }
}

function parsePath(obj, expression) {
  const segments = expression.split('.')
  for (let key of segments) {
    if (!obj) return
    obj = obj[key]
  }
  return obj
}
复制代码
```

> 如果你对`Watcher`这个类什么时候实例化有疑问的话，没关系，下面马上就会讲到

其实前文例子中还有一个点我们尚未提到：显卡例子中说到，淘宝会将买家信息保存在一个数组中，那么我们的响应式系统中也应该有一个数组来保存买家信息，也就是`watcher`。

总结一下我们需要实现的功能：

1. 有一个数组来存储`watcher`
2. `watcher`实例需要订阅(依赖)数据，也就是获取依赖或者收集依赖
3. `watcher`的依赖发生变化时触发`watcher`的回调函数，也就是派发更新。

每个数据都应该维护一个属于自己的数组，该数组来存放依赖自己的`watcher`，我们可以在`defineReactive`中定义一个数组`dep`，这样通过闭包，每个属性就能拥有一个属于自己的`dep`

```js
function defineReactive(data, key, value = data[key]) {
  const dep = [] // 增加
  observe(value)
  Object.defineProperty(data, key, {
    get: function reactiveGetter() {
      return value
    },
    set: function reactiveSetter(newValue) {
      if (newValue === value) return
      value = newValue
      observe(newValue)
      dep.notify()
    }
  })
}
复制代码
```

到这里，我们实现了第一个功能，接下来实现收集依赖的过程。

### 依赖收集

现在我们把目光集中到页面的初次渲染过程中(暂时忽略渲染函数和虚拟`DOM`等部分)：渲染引擎会解析模板，比如引擎遇到了一个插值表达式，如果我们此时实例化一个`watcher`，会发生什么事情呢？从`Watcher`的代码中可以看到，实例化时会执行`get`方法，`get`方法的作用就是`获取`自己依赖的数据，而我们重写了数据的访问行为，为每个数据定义了`getter`，因此`getter`函数就会执行，如果我们在`getter`中把当前的`watcher`添加到`dep`数组中(淘宝低登记买家信息)，不就能够完成依赖收集了吗！！

> 注意：执行到`getter`时，`new Watcher()`的`get`方法还没有执行完毕。
>
> `new Watcher()`时执行`constructor`，调用了实例的`get`方法，实例的`get`方法会读取数据的值，从而触发了数据的`getter`，`getter`执行完毕后，实例的`get`方法执行完毕，并返回值，`constructor`执行完毕，实例化完毕。

> 有些同学可能会有疑惑：明明是`watcher`收集依赖，应该是`watcher`收集数据，怎么成了数据的`dep`收集`watcher`了呢？有此疑问的同学可以再看一下前面淘宝的例子(是淘宝记录了用户信息)，或者深入了解一下发布-订阅模式。

通过上面的分析，我们只需要对`getter`进行一些修改：

```js
get: function reactiveGetter() {
  dep.push(watcher) // 新增
  return value
}
复制代码
```

问题又来了，`watcher`这个变量从哪里来呢？我们是在模板编译函数中的实例化`watcher`的，`getter`中取不到这个实例啊。解决方法也很简单，将`watcher`实例放到全局不就行了吗，比如放到`window.target`上。因此，`Watcher`的`get`方法做如下修改

```js
get() {
  window.target = this // 新增
  const value = parsePath(this.data, this.expression)
  return value
}
复制代码
```

这样，将`get`方法中的`dep.push(watcher)`修改为`dep.push(window.target)`即可。

> 注意，不能这样写`window.target = new Watcher()`。因为执行到`getter`的时候，实例化`watcher`还没有完成，所以`window.target`还是`undefined`

> 依赖收集过程：渲染页面时碰到插值表达式，`v-bind`等需要数据等地方，会实例化一个`watcher`,实例化`watcher`就会对依赖的数据求值，从而触发`getter`，数据的`getter`函数就会添加依赖自己的`watcher`，从而完成依赖收集。我们可以理解为`watcher`在收集依赖，而代码的实现方式是在数据中存储依赖自己的`watcher`

> 细心的读者可能会发现，利用这种方法，每遇到一个插值表达式就会新建一个`watcher`，这样每个节点就会对应一个`watcher`。实际上这是`vue1.x`的做法，以节点为单位进行更新，粒度较细。而`vue2.x`的做法是每个组件对应一个`watcher`，实例化`watcher`时传入的也不再是一个`expression`，而是渲染函数，渲染函数由组件的模板转化而来，这样一个组件的`watcher`就能收集到自己的所有依赖，以组件为单位进行更新，是一种中等粒度的方式。要实现`vue2.x`的响应式系统涉及到很多其他的东西，比如组件化，虚拟`DOM`等，而这个系列文章只专注于数据响应式的原理，因此不能实现`vue2.x`，但是两者关于响应式的方面，原理相同。

### 派发更新

实现依赖收集后，我们最后要实现的功能是派发更新，也就是依赖变化时触发`watcher`的回调。从依赖收集部分我们知道，获取哪个数据，也就是说触发哪个数据的`getter`，就说明`watcher`依赖哪个数据，那数据变化的时候如何通知`watcher`呢？相信很多同学都已经猜到了：在`setter`中派发更新。

```js
set: function reactiveSetter(newValue) {
  if (newValue === value) return
  value = newValue
  observe(newValue)
  dep.forEach(d => d.update()) // 新增 update方法见Watcher类
}
复制代码
```

## 3. 优化代码

### 1. Dep类

我们可以将`dep`数组抽象为一个类:

```js
class Dep {
  constructor() {
    this.subs = []
  }

  depend() {
    this.addSub(Dep.target)
  }

  notify() {
    const subs = [...this.subs]
    subs.forEach((s) => s.update())
  }

  addSub(sub) {
    this.subs.push(sub)
  }
}
复制代码
```

`defineReactive`函数只需做相应的修改

```js
function defineReactive(data, key, value = data[key]) {
  const dep = new Dep() // 修改
  observe(value)
  Object.defineProperty(data, key, {
    get: function reactiveGetter() {
      dep.depend() // 修改
      return value
    },
    set: function reactiveSetter(newValue) {
      if (newValue === value) return
      value = newValue
      observe(newValue)
      dep.notify() // 修改
    }
  })
}
复制代码
```

### 2. window.target

在`watcher`的`get`方法中

```js
get() {
  window.target = this // 设置了window.target
  const value = parsePath(this.data, this.expression)
  return value
}
复制代码
```

大家可能注意到了，我们没有重置`window.target`。有些同学可能认为这没什么问题，但是考虑如下场景：有一个对象`obj: { a: 1, b: 2 }`我们先实例化了一个`watcher1`，`watcher1`依赖`obj.a`，那么`window.target`就是`watcher1`。之后我们访问了`obj.b`，会发生什么呢？访问`obj.b`会触发`obj.b`的`getter`，`getter`会调用`dep.depend()`，那么`obj.b`的`dep`就会收集`window.target`， 也就是`watcher1`，这就导致`watcher1`依赖了`obj.b`，但事实并非如此。为解决这个问题，我们做如下修改：

```js
// Watcher的get方法
get() {
  window.target = this
  const value = parsePath(this.data, this.expression)
  window.target = null // 新增，求值完毕后重置window.target
  return value
}

// Dep的depend方法
depend() {
  if (Dep.target) { // 新增
    this.addSub(Dep.target)
  }
}
复制代码
```

通过上面的分析能够看出，`window.target`的含义就是当前执行上下文中的`watcher`实例。由于`js`单线程的特性，同一时刻只有一个`watcher`的代码在执行，因此`window.target`就是当前正在处于实例化过程中的`watcher`

### 3. update方法

我们之前实现的`update`方法如下：

```js
update() {
  this.value = parsePath(this.data, this.expression)
  this.cb()
}
复制代码
```

大家回顾一下`vm.$watch`方法，我们可以在定义的回调中访问`this`，并且该回调可以接收到监听数据的新值和旧值，因此做如下修改

```js
update() {
  const oldValue = this.value
  this.value = parsePath(this.data, this.expression)
  this.cb.call(this.data, this.value, oldValue)
}
复制代码
```

### 4. 学习一下Vue源码

在[Vue源码--56行](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fblob%2Fdev%2Fsrc%2Fcore%2Fobserver%2Fdep.js)中，我们会看到这样一个变量：`targetStack`，看起来好像和我们的`window.target`有点关系，没错，确实有关系。设想一个这样的场景：我们有两个嵌套的父子组件，渲染父组件时会新建一个父组件的`watcher`，渲染过程中发现还有子组件，就会开始渲染子组件，也会新建一个子组件的`watcher`。在我们的实现中，新建父组件`watcher`时，`window.target`会指向父组件`watcher`，之后新建子组件`watcher`，`window.target`将被子组件`watcher`覆盖，子组件渲染完毕，回到父组件`watcher`时，`window.target`变成了`null`，这就会出现问题，因此，我们用一个栈结构来保存`watcher`。

```js
const targetStack = []

function pushTarget(_target) {
  targetStack.push(window.target)
  window.target = _target
}

function popTarget() {
  window.target = targetStack.pop()
}
复制代码
```

`Watcher`的`get`方法做如下修改

```js
get() {
  pushTarget(this) // 修改
  const value = parsePath(this.data, this.expression)
  popTarget() // 修改
  return value
}
复制代码
```

此外，`Vue`中使用`Dep.target`而不是`window.target`来保存当前的`watcher`，这一点影响不大，只要能保证有一个全局唯一的变量来保存当前的`watcher`即可

### 5.总结代码

现将代码总结如下：

```js
// 调用该方法来检测数据
function observe(data) {
  if (typeof data !== 'object') return
  new Observer(data)
}

class Observer {
  constructor(value) {
    this.value = value
    this.walk()
  }
  walk() {
    Object.keys(this.value).forEach((key) => defineReactive(this.value, key))
  }
}

// 数据拦截
function defineReactive(data, key, value = data[key]) {
  const dep = new Dep()
  observe(value)
  Object.defineProperty(data, key, {
    get: function reactiveGetter() {
      dep.depend()
      return value
    },
    set: function reactiveSetter(newValue) {
      if (newValue === value) return
      value = newValue
      observe(newValue)
      dep.notify()
    }
  })
}

// 依赖
class Dep {
  constructor() {
    this.subs = []
  }

  depend() {
    if (Dep.target) {
      this.addSub(Dep.target)
    }
  }

  notify() {
    const subs = [...this.subs]
    subs.forEach((s) => s.update())
  }

  addSub(sub) {
    this.subs.push(sub)
  }
}

Dep.target = null

const TargetStack = []

function pushTarget(_target) {
  TargetStack.push(Dep.target)
  Dep.target = _target
}

function popTarget() {
  Dep.target = TargetStack.pop()
}

// watcher
class Watcher {
  constructor(data, expression, cb) {
    this.data = data
    this.expression = expression
    this.cb = cb
    this.value = this.get()
  }

  get() {
    pushTarget(this)
    const value = parsePath(this.data, this.expression)
    popTarget()
    return value
  }

  update() {
    const oldValue = this.value
    this.value = parsePath(this.data, this.expression)
    this.cb.call(this.data, this.value, oldValue)
  }
}

// 工具函数
function parsePath(obj, expression) {
  const segments = expression.split('.')
  for (let key of segments) {
    if (!obj) return
    obj = obj[key]
  }
  return obj
}

// for test
let obj = {
  a: 1,
  b: {
    m: {
      n: 4
    }
  }
}

observe(obj)

let w1 = new Watcher(obj, 'a', (val, oldVal) => {
  console.log(`obj.a 从 ${oldVal}(oldVal) 变成了 ${val}(newVal)`)
})

复制代码
```

## 4. 注意事项

### 1. 闭包

`Vue`能够实现如此强大的功能，离不开闭包的功劳：在`defineReactive`中就形成了闭包，这样每个对象的每个属性就能保存自己的值`value`和依赖对象`dep`。

### 2. 只要触发getter就会收集依赖吗

答案是否定的。在`Dep`的`depend`方法中，我们看到，只有`Dep.target`为真时才会添加依赖。比如在派发更新时会触发`watcher`的`update`方法，该方法也会触发`parsePath`来取值，但是此时的`Dep.target`为`null`，不会添加依赖。仔细观察可以发现，只有`watcher`的`get`方法中会调用`pushTarget(this)`来对`Dep.target`赋值，其他时候`Dep.target`都是`null`，而`get`方法只会在实例化`watcher`的时候调用，因此，在我们的实现中，一个`watcher`的依赖在其实例化时就已经确定了，之后任何读取值的操作均不会增加依赖。

### 3. 依赖嵌套的对象属性

我们结合上面的代码来思考下面这个问题：

```js
let w2 = new Watcher(obj, 'b.m.n', (val, oldVal) => {
  console.log(`obj.b.m.n 从 ${oldVal}(oldVal) 变成了 ${val}(newVal)`)
})
复制代码
```

我们知道，`w2`会依赖`obj.b.m.n`， 但是`w2`会依赖`obj.b, obj.b.m`吗？或者说，`obj.b,和obj.b.m`，它们闭包中保存的`dep`中会有`w2`吗？答案是会。我们先不从代码角度分析，设想一下，如果我们让`obj.b = null`，那么很显然`w2`的回调函数应该被触发，这就说明`w2`会依赖中间层级的对象属性。

接下来我们从代码层面分析一下：`new Watcher()`时，会调用`watcher的get`方法，将`Dep.target`设置为`w2`，`get`方法会调用`parsePath`来取值，我们来看一下取值的具体过程：

```js
function parsePath(obj, expression) {
  const segments = expression.split('.') // 先将表达式分割，segments:['b', 'm', 'n']
  // 循环取值
  for (let key of segments) {
    if (!obj) return
    obj = obj[key]
  }
  return obj
}
复制代码
```

以上代码流程如下：

1. 局部变量`obj`为对象`obj`，读取`obj.b`的值，触发`getter`，触发`dep.depend()`(该`dep`是`obj.b`的闭包中的`dep`)，`Dep.target`存在，添加依赖
2. 局部变量`obj`为`obj.b`，读取`obj.b.m`的值，触发`getter`，触发`dep.depend()`(该`dep`是`obj.b.m`的闭包中的`dep`)，`Dep.target`存在，添加依赖
3. 局部变量`obj`为对象`obj.b.m`，读取`obj.b.m.n`的值，触发`getter`，触发`dep.depend()`(该`dep`是`obj.b.m.n`的闭包中的`dep`)，`Dep.target`存在，添加依赖

从上面的代码可以看出，`w2`会依赖与目标属性相关的每一项，这也是符合逻辑的。

## 5. 总结

总结一下：

1. 调用`observe(obj)`，将`obj`设置为响应式对象，`observe函数，Observe, defineReactive函数`三者互相调用，从而递归地将`obj`设置为响应式对象
2. 渲染页面时实例化`watcher`，这个过程会读取依赖数据的值，从而完成`在getter中获取依赖`
3. 依赖变化时触发`setter`，从而派发更新，执行回调，完成`在setter中派发更新`



转载自[争霸爱好者链接的0年前端的Vue响应式原理学习总结1：基本原理](https://juejin.cn/post/6932659815424458760)

# 2：数组的处

## 1. 为什么要对数组特殊处理

上一篇文章讲到了`vue`数据响应式的基本原理，结尾提到，我们要对数组进行一个单独的处理。很多人可能会有疑问了，为什么要对数组做特殊处理呢？数组不就是`key`为数值的对象吗？那我们不妨来尝试一下

```js
function defineReactive(obj, key, val) {
  Object.defineProperty(obj, key, {
    get() {
      console.log('get: ', val)
      return val
    },
    set(newVal) {
      console.log('set: ', newVal)
      newVal = val
    }
  })
}

const arr = [1, 2, 3]
arr.forEach((val, index, arr) => {
  defineReactive(arr, index, val)
})
复制代码
```

如果我们访问和获取`arr`的值，`getter`和`setter`也会被触发，这不是可以吗？但是如果`arr.unshift(0)`呢？数组的每个元素要依次向后移动一位，这就会触发`getter`和`setter`，导致依赖发生变化。由于数组是顺序结构，所以索引(key)和值不是绑定的，因此这种护理方法是有问题的。

那既然不能监听数组的索引，那就监听数组本身吧。`vue`对`js`中7种会改变数组的方法进行了重写：`push, pop, unshift, shift, splice, reverse, sort`。那么，我们就要对`Observer`进行修改了：

```js
class Observer {
  constructor(value) {
    this.value = value
    if (Array.isArray(value)) {
      // 代理原型...
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  walk(obj) {
    Object.keys(obj).forEach((key) => defineReactive(obj, key, obj[key]))
  }
  // 需要继续监听数组内的元素(如果数组元素是对象的话)
  observeArray(arr) {
    arr.forEach((i) => observe(i))
  }
}
复制代码
```

关键的处理就是代理原型

## 2. 代理原型

相信大家对原型链都比较熟悉了，当我们调用`arr.push()`时，实际上是调用了`Array.prototype.push`，我们想对`push`方法进行特殊处理，除了重写该方法，还有一种手段，就是设置一个代理原型

![img](vue2和vue3响应式数据原理/97b621913e6e4310829ae11d5c757871_tplv-k3u1fbpfcp-watermark.awebp)

我们在数组实例和`Array.prototype`之间增加了一层代理来实现派发更新(依赖收集部分后面单独讲)，数组调用代理原型的方法来派发更新，代理原型再调用真实原型的方法实现原有的功能：

```js
// Observer.js
if (Array.isArray(value)) {
  Object.setPrototypeOf(value, proxyPrototype) // value.__proto__ === proxyPrototype
  this.observeArray(value)
}

// array.js
const arrayPrototype = Array.prototype // 缓存真实原型

// 需要处理的方法
const reactiveMethods = [
  'push',
  'pop',
  'unshift',
  'shift',
  'splice',
  'reverse',
  'sort'
]

// 增加代理原型 proxyPrototype.__proto__ === arrayProrotype
const proxyPrototype = Object.create(arrayPrototype)

// 定义响应式方法
reactiveMethods.forEach((method) => {
  const originalMethod = arrayPrototype[method]
  // 在代理原型上定义变异响应式方法
  Object.defineProperty(proxyPrototype, method, {
    value: function reactiveMethod(...args) {
      const result = originalMethod.apply(this, args) // 执行默认原型的方法
      // ...派发更新...
      return result
    },
    enumerable: false,
    writable: true,
    configurable: true
  })
})
复制代码
```

问题来了，我们如何派发更新呢？对于对象，我们使用`dep.nofity`来派发更新，之所以能够拿到`dep`数组，是因为我们利用`getter`和`setter`形成了闭包，保存了`dep`数组，并且这样能够保证每个属性都有属于自己的`dep`。那数组呢？如果再`array.js`中定义一个`dep`，那所有的数组都会共享这一个`dep`，显然是不行的，因此，`vue`在每个对象身上添加了一个自定义属性：`__ob__`，这个属性保存自己的`Observer`实例，然后再`Observer`上添加一个属性`dep`不就可以了吗！

## 3. `__ob__`属性

对`observe`做一个修改：

```js
// observe.js
function observe(value) {
  if (typeof value !== 'object') return
  let ob
  // __ob__还可以用来标识当前对象是否被监听过
  if (value.__ob__ && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else {
    ob = new Observer(value)
  }
  return ob
}
复制代码
```

`Observer`也要做修改：

```js
constructor(value) {
  this.value = value
  this.dep = new Dep()
  // 在每个对象身上定义一个__ob__属性,指向每个对象的Observer实例
  def(value, '__ob__', this)
  if (Array.isArray(value)) {
    Object.setPrototypeOf(value, proxyPrototype)
    this.observeArray(value)
  } else {
    this.walk(value)
  }
}

// 工具函数def，就是对Object.defineProperty的封装
function def(obj, key, value, enumerable = false) {
  Object.defineProperty(obj, key, {
    value,
    enumerable,
    writable: true,
    configurable: true
  })
}

复制代码
```

这样，对象`obj: { arr: [...] }`就会变为`obj: { arr: [..., __ob__: {} ], __ob__: {} }`

```js
// array.js
reactiveMethods.forEach((method) => {
  const originalMethod = arrayPrototype[method]
  Object.defineProperty(proxyPrototype, method, {
    value: function reactiveMethod(...args) {
      const result = originalMethod.apply(this, args)
      const ob = this.__ob__ // 新增
      ob.dep.notify()        // 新增
      return result
    },
    enumerable: false,
    writable: true,
    configurable: true
  })
})
复制代码
```

还有一个小细节，`push, unshift, splice`可能会向数组中增加元素，这些增加的元素也应该被监听：

```js
Object.defineProperty(proxyPrototype, method, {
  value: function reactiveMethod(...args) {
    const result = originalMethod.apply(this, args)
    const ob = this.__ob__
    // 对push，unshift，splice的特殊处理
    let inserted = null
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        // splice方法的第三个及以后的参数是新增的元素
        inserted = args.slice(2)
    }
    // 如果有新增元素，继续对齐进行监听
    if (inserted) ob.observeArray(inserted)
    ob.dep.notify()
    return result
  },
  enumerable: false,
  writable: true,
  configurable: true
})
复制代码
```

到这里，我们通过在对象身上新增一个`__ob__`属性，完成了数组的派发更新，接下来是依赖收集

## 4. 依赖收集

在将依赖收集之前，我们先看一下`__ob__`这个属性

```js
const obj = {
  arr: [
    {
      a: 1
    }
  ]
}
复制代码
```

执行`observe(obj)`后，`obj`变成了下面的样子

```js
obj: {
  arr: [
    {
      a: 1,
      __ob__: {...} // 增加
    },
    __ob__: {...}   // 增加
  ],
  __ob__: {...}     // 增加
}
复制代码
```

我们的`defineReactive`函数中，为了递归地为数据设置响应式，调用了`observe(val)`，而现在的`observe()`会返回`ob`，也就是`value.__ob__`，那我们不妨接收一下这个返回值

```js
// defineReactive.js
let childOb = observe(val) // 修改

set: function reactiveSetter(newVal) {
  if (val === newVal) {
    return
  }
  val = newVal
  childOb = observe(newVal) // 修改
  dep.notify()
}
复制代码
```

那这个`childOb`是什么呢？比如看`obj.arr`吧：

1. 执行`observe(obj)`会触发`new Observer(obj)`，设置了`obj.__ob__`属性，接下来遍历`obj`的属性，执行`defineReactive(obj, arr, obj.arr)`
2. 执行`defineReactive(obj, arr, obj.arr)`时，会执行`observe(obj.arr)`，返回值就是`obj.arr.__ob__`

也就是说，每个属性(比如`arr`属性)的`getter`和`setter`不仅通过闭包保存了属于自己的`dep`，而且通过`__ob__`保存了自己的`Observer`实例，`Observer`实例上又有一个`dep`属性。如果添加以下代码：

```js
// defineReactive.js
get: function reactiveGetter() {
  if (Dep.target) {
    dep.depend()
    childOb.dep.depend() // 新增
  }
  return val
}
复制代码
```

就会有下面的情况：**每个属性的getter和setter通过闭包保存了dep，这个dep收集了依赖自己的watcher， 闭包中还保存了chilOb，childOb.dep也保存了依赖自己的watcher，这两个属性保存的watcher相同，那前文讲到的派发更新就能够实现了**。

> `obj.prop`闭包中保存的`childOb`就是`obj.prop.__ob__`，闭包中的`dep`与`childOb.dep`保存的内容相同

但是`dep`和`childOb.dep`保存的`watcher`并不完全相同，看`obj[arr][0].a`，由于这是一个基本类型，对它调用`observe`会直接返回，因此所以没有`__ob__`属性，但是这个属性闭包中的`dep`能够收集到依赖自己的`watcher`。所以上述代码可能会报错，故做如下修改

```js
get: function reactiveGetter() {
  if (Dep.target) {
    dep.depend()
    if (childOb) {
      childOb.dep.depend() // 新增  
    }
  }
  return val
}
复制代码
```

## 5. 依赖数组就等于依赖了数组中所有的元素

看这个例子：

```js
const obj = {
  arr: [
    { a: 1 }
  ]
}

observe(obj)

// 数据监听后的obj
obj: {
  arr: [
    {
      a: 1,
      __ob__: {...}
    },
    __ob__: {...}
  ],
  __ob__: {...}
}
复制代码
```

新建一个依赖`arr`的`watcher`，如果为`obj.arr[0]`增加一个属性：

```js
Vue.set(obj.arr[0], 'b', 2) // 注意是Vue.set
复制代码
```

这样是不会触发`watcher`回调函数的。因为我们的`watcher`依赖`arr`，求值时触发了`obj.arr`的`getter`，所以`childOb.dep(arr.__ob__.dep)`中收集到了`watcher`。但是`obj.arr[0].__ob__`中并没有收集到`watcher`，所以为其设置新值不会触发更新。但是`Vue`认为，**只要依赖了数组，就等价于依赖了数组中的所有元素**，因此，我们需要进一步处理

```js
// defineReactive.js
get: function reactiveGetter() {
  if (Dep.target) {
    dep.depend()
    if (childOb) {
      childOb.dep.depend()
      // 新增
      if (Array.isArray(val)) {
        dependArray(val)
      }
    }
  }
  return val
}

function dependArray(array) {
  for (let e of array) {
    e && e.__ob__ && e.__ob__.dep.depend()
    if (Array.isArray(e)) {
      dependArray(e)
    }
  }
}
复制代码
```

以上新增代码的作用，就是当依赖是数组时，遍历这个数组，为每个元素的`__ob__.dep`中添加`watcher`。

## 6. 总结

我们通过设置代理原型，让数组执行变异方法来完成响应式，并且为每个属性设置了`__ob__`属性，这样，我们在闭包之外就能访问`dep`了，从而完成派发更新，包括`Vue.set`方法也利用了这个属性。

其实对于数组来说，依赖收集阶段依然是在`getter`中完成的，只不过是借助了`arr.__ob__`来完成，并且需要对数组的所有项都添加依赖，而更新派发是在变异方法中完成的，依然借助了`__ob__`。



转载自[争霸爱好者链接的0年前端的Vue响应式原理学习总结2：数组的处理](https://juejin.cn/post/6932843824515383303)

# 3：渲染watcher

## 什么是渲染Watcher

`vue`中有多种`watcher`，我们之前实现的`watcher`类似于`Vue.$watch`，当依赖变化时执行回调函数。而渲染`watcher`不需要回调函数，渲染`watcher`接收一个渲染函数而不是依赖的表达式，当依赖发生变化时，自动执行渲染函数

```js
new Watcher(app, renderFn)
复制代码
```

那么如何做到依赖变化时重新执行渲染函数呢，我们要先对`Watcher`的构造函数做一些改造

```js
constructor(data, expOrFn, cb) {
  this.data = data
  // 修改
  if (typeof expOrFn === 'function') {
    this.getter = expOrFn
  } else {
    this.getter = parsePath(expOrFn)
  }
  this.cb = cb
  this.value = this.get()
}

// parsePath的改造，返回一个函数
function parsePath(path) {
  const segments = path.split('.')
  return function (obj) {
    for (let key of segments) {
      if (!obj) return
      obj = obj[key]
    }
    return obj
  }
}
复制代码
```

这样，`this.getter`就是一个取值函数了，`get`修改

```js
get() {
  pushTarget(this)
  const data = this.data
  const value = this.getter.call(data, data) // 修改
  popTarget()
  return value
}
复制代码
```

要想依赖变化时重新执行渲染函数，就要在派发更新阶段做一个更新，因此，`update`方法也要进行修改：

```js
update() {
  // 重新执行get方法
  const value = this.get()
  // 渲染watcher的value是undefined，因为渲染函数没有返回值
  // 因此value和this.value都是undefined，不会进入if
  // 如果依赖是对象，要触发更新
  if (value !== this.value || isObject(value)) {
    const oldValue = this.value
    this.value = value
    this.cb.call(this.vm, value, oldValue)
  }
}

function isObject(target) {
  return typeof target === 'object' && target !== null
}
复制代码
```

大家可能会有疑问了，为什么不能直接用`this.getter.call(this.data)`来重新执行渲染函数呢，这就涉及到下文要提到的重新收集依赖了。但是在此之前，要先解决一个问题：依赖的重复收集

## 重复的依赖

看这样一个例子

```html
<div>
  {{ name }} -- {{ name }}
</div>
复制代码
```

如果我们渲染这个模板，那么渲染`watcher`就会依赖两次`name`。因为解析该模板时，会读取两次`name`的值，就会触发两次`getter`，此时`Dep.target`都是当前`watcher`，在`depend`方法中，

```js
depend() {
  if (Dep.target) {
    dep.addSub(Dep.target)
  }
}
复制代码
```

依赖会被收集两次，`name`变化时就会触发两次重新渲染。因此`vue`采用了以下方式

首先为每个`dep`添加一个`id`

```js
let uid = 0

constructor() {
  this.subs = []
  this.id = uid++ // 增加
}
复制代码
watcher`修改的地方比较多，首先为增加四个属性`deps, depIds, newDeps, newDepIds
this.deps = []             // 存放上次求值时存储自己的dep
this.depIds = new Set()    // 存放上次求值时存储自己的dep的id
this.newDeps = []          // 存放本次求值时存储自己的dep
this.newDepIds = new Set() // 存放本次求值时存储自己的dep的id
复制代码
```

> ​	每次取值完毕后，会交换`dep`与`newDep`，并将`newDep`清空，下文会讲到

我们的思路是，当需要收集`watcher`时，由`watcher`来决定自己是否需要被`dep`收集。在上面的例子中，假设对`name`取值时，`watcher`被`dep1`收集，第二次对`name`取值时，`watcher`发现自己已经被`dep1`收集过了，就不会重新收集一遍，代码如下

```js
// dep.depend
depend() {
  if (Dep.target) {
    Dep.target.addDep(this) // 让watcher来决定自己是否被dep收集
  }
}

// watcher.addDep
addDep(dep) {
  const id = dep.id
  // 如果本次求值过程中，自己没有被dep收集过则进入if
  if (!this.newDepIds.has(id)) {
    // watcher中记录收集自己的dp
    this.newDepIds.add(id)
    this.newDeps.push(dep)
    if (!this.depIds.has(id)) {
      dep.addSub(this)
    }
  }
}
复制代码
```

现在解释一下最后一个`if`，考虑重新渲染的情况：`watcher`依赖`name`，`name`发生了变化，导致`watcher`的`get`方法执行，会重新对`name`取值，进入`addDep`方法时，`newDepIds`是空的，因此会进入`if`，来到最后一个`if`，因为第一次取值时，`dep`已经收集过`watcher`了，所以不应该再添加一遍，这个`if`就是这个作用。

> 《Vue技术内幕》总结的很好：
>
> 1. `newDeps`和`newDepIds`用来再一次取值过程中避免重复依赖，比如：`{{ name }} --  {{ name }}`
> 2. `deps`和`depIds`用来再重新渲染的取值过程中避免重复依赖

再执行`get`方法最后会清空`newDeps,newDepIds`

```js
cleanUpDeps() {
    // 交换depIds和newDepIds
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    // 清空newDepIds
    this.newDepIds.clear()
    // 交换deps和newDeps
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    // 清空newDeps
    this.newDeps.length = 0
  }
复制代码
```

## 依赖的重新收集

我所理解的依赖重新收集包括两部分内容：收集新的依赖和删除无效依赖。其实收集新依赖再上面的代码中已经有所体现了，虽然前面的代码中对重复依赖做了很多判断，但是能够收集到依赖的基本前提是`Dep.target`存在，从`Watcher`的代码中可以看出，只有在`get`方法执行过程中，`Dep.target`是存在的，因此，我们在`update`方法中使用了`get`方法来重新触发渲染函数，而不是`getter.call()`。并且重新收集依赖是必要的，比如使用了`v-if`的情况，因此，现在的响应式系统比之前的`固定依赖版本`又有了很大进步。

至于删除无效依赖部分，可以在`cleanUpDeps`中添加如下代码

```js
cleanUpDeps() {
  // 增加
  let i = this.deps.length
  while (i--) {
    const dep = this.deps[i]
    if (!this.newDepIds.has(dep.id)) {
      dep.removeSub(this)
    }
  }
  let tmp = this.depIds
  // ...
}
复制代码
```

在求值结束(也就是依赖收集结束)后，如果本次求值过程中，发现有些`dep`在上次求值时收集了自己，但是这次求值时没有收集自己，说明该数据已经不需要自己了，将自己从`dep`中删除即可

```js
// Dep.js
removeSub(sub) {
  remove(this.subs, sub)
}

function remove(arr, item) {
  if (!arr.length) return
  const index = arr.indexOf(item)
  if (index > -1) {
    return arr.splice(index, 1)
  }
}
复制代码
```

这样，我们的响应式系统就比较完整了

## 总结

其实所谓的渲染`watcher`和其他的`watcher`区别不大，只是依赖变化时自动执行渲染函数而已，上文中提到的重复依赖的处理，依赖重新收集是通用的。



转载自[争霸爱好者链接的0年前端的Vue响应式原理学习总结3：渲染watcher](https://juejin.cn/post/6933041656765612039)

# 4：最终章

## MVVM类

我们会像`vue`一样，建立一个类，叫做`MVVM`，接收一个配置参数：

```js
class MVVM {
  constructor(options) {
    this.$el = options.el
    this.$data = options.data
    // 将数据变为响应式
    observe(this.$data)
    // 模板编译
    if (this.$el) new Compiler(this.$el, this)
  }
}
复制代码
```

在`Vue`中，我们可以直接用`vue`实例访问数据，所以，我们也可以实现这样的功能，定义一个方法：

```js
proxyData(data) {
  Object.keys(data).forEach((key) => {
    Object.defineProperty(this, key, {
      get() {
        return data[key]
      },
      set(newVal) {
        if (data[key] === newVal) return
        data[key] = newVal
      }
    })
  })
}
复制代码
```

实例化时执行该方法即可

```js
constructor(options) {
  this.$el = options.el
  this.$data = options.data
  // 先代理数据，这样之后需要数据时就不需要用this.$data.prop了，直接使用this.prop即可
  this.proxyData(this.$data)
  observe(this.$data)
  if (this.$el) new Compiler(this.$el, this)
}
复制代码
```

## 模板编译器

将数据设置为响应式后，开始模板编译。模板编译思路如下：先将`el`从页面上取出来放到`fragment`中，编译完成后再放回页面中。

```js
class Compiler {
  constructor(el, vm) {
    // el可以是选择器或元素节点
    this.el = isElementNode(el) ? el : document.querySelector(el)
    this.vm = vm
    let fragment = node2Fragment(this.el)
    this.compile(fragment)
    this.el.appendChild(fragment)
  }
}

function isElementNode(node) {
  return node.nodeType === 1
}

function node2Fragment(node) {
  let fragment = document.createDocumentFragment()
  let firstChild = node.firstChild
  while (firstChild) {
    fragment.appendChild(firstChild)
    firstChild = node.firstChild
  }
  return fragment
}
复制代码
```

`compile`方法就是主编译方法

```js
compile(node) {
  let childNodes = Array.from(node.childNodes)
  childNodes.forEach((c) => {
    if (isElementNode(c)) {
      this.compileElementNode(c)
    } else {
      this.compileTextNode(c)
    }
  })
}
复制代码
```

元素节点编译方法如下

```js
compileElementNode(node) {
  // 获取元素节点的属性来找出指令
  Array.from(node.attributes).forEach(({ name, value: expression }) => {
    if (isDirective(name)) {
      // 指令以v-开头
      const directive = name.split('-')[1]
      // 渲染watcher
      new Watcher(
        this.vm,
        // 相当于解析指令的渲染函数
        directiveCompiler[directive](node, expression, this.vm)
      )
    }
  })
  // 如果该节点时元素节点，应该递归编译该节点内部的节点
  this.compile(node)
}

function isDirective(str) {
  return str.startsWith('v-')
}
复制代码
```

指令解析器如下：

```js
function setValue(vm, expression, value) {
  const keys = expression.split('.')
  let obj = vm.$data
  for (let i = 0; i < keys.length - 1; i++) {
    obj = obj[keys[i]]
  }
  obj[keys.slice(-1)] = value
}

// 只处理了v-model指令
export default {
  model(node, expression, vm) {
    // 监听输入框的input方法，实现双向绑定
    node.addEventListener('input', (e) =>
      setValue(vm, expression, e.target.value)
    )
    const value = parsePath(expression).call(vm, vm)
    return function () {
      node.value = value
    }
  }
}
复制代码
```

文本解析方法

```js
compileTextNode(node) {
  if (/\{\{(.+?)\}\}/g.test(node.textContent)) {
    // 渲染watcher
    new Watcher(this.vm, textCompiler(node, this.vm))
  }
}
复制代码
```

`textCompiler`方法如下

```js
function textCompiler(node, vm) {
  const text = node.textContent
  return function () {
    const content = text.replace(/\{\{(.+?)\}\}/g, (...args) => {
      const path = args[1].trim()
      const val = parsePath(path).call(vm, vm)
      if (isObject(val)) {
        // 如果模板内是对象，使用JSON.stringify来显示
        // JSON.stringify也会访问对象内部的属性
        // 这样就完成了对该对象所有属性的依赖收集
        return JSON.stringify(val, null, 1) // 第三个参数：空格数量
      }
      return val
    })
    node.textContent = content
  }
}
复制代码
```

这样，我们就实现了一个简单模板编译器，实例化`MVVM`后，就有一个响应式的应用了！！

```js
// index.js
import MVVM from './MVVM'

window.vm = new MVVM({
  el: '.app',
  data: {
    obj: {
      a: 1,
      b: 2
    },
    a: 1,
    arr: [
      {
        a: 1
      }
    ]
  }
})

复制代码
<div class="app">
  {{ obj }}
  <br />
  {{ arr }}
  <br />
  <input type="text" v-model="obj.a" />
</div>
复制代码
```

## 两个全局方法

不知道大家是否还记得，前面的文章中提到过`Vue.$set`方法就使用了`__ob__`属性来添加响应式数据，我们来看一下

```js
$set(target, key, value) {
  // 对于数组利用splice实现添加元素
  if (Array.isArray(target)) {
    // 如果splice索引超过数组长度会报错
    target.length = Math.max(target.length, key)
    target.splice(key, 1, value)
    return value
  }
  // 对于对象，如果该属性已经存在，直接赋值
  if (key in target && !(key in Object.prototype)) {
    target[key] = value
    return value
  }
  const ob = target.__ob__
  // 如果目标对象不是响应式对象，直接赋值
  if (!ob) {
    target[key] = value
    return value
  }
  // 设置响应式属性
  defineReactive(target, key, value)
  // 派发更新
  ob.dep.notify()
  return value
}
复制代码
```

此外，`Vue.$delete`方法也是如此

```js
$delete(target, key) {
  // 对于数组用splice方法删除元素
  if (Array.isArray(target)) {
    target.splice(key, 1)
    return
  }
  const ob = target.__ob__
  // 如果对象没有该属性，直接返回
  if (!target.hasOwnProperty(key)) return
  delete target[key]
  // 如果不是响应式对象，则不需要派发更新
  if (!ob) return
  // 对于响应式对象，删除属性后要派发更新
  ob.dep.notify()
}
复制代码
```

因此，我们也可以总结出下面的结论

> 1. `getter`和`setter`闭包中保存的`dep`用来存储依赖**纯对象的属性**的`watcher`，只有这个闭包能够访问到这个变量
> 2. `getter`和`setter`闭包中保存的`childOb`，就是与这个属性同级的`__ob__`属性，这个属性存储一个`Observer`实例，实例上也有一个`dep`属性，这个属性可以保存依赖数组的`watcher`，并且外部的方法也可以通过`__ob__`属性来派发更新



转载自[争霸爱好者链接的0年前端的Vue响应式原理学习总结4：最终章](https://juejin.cn/post/6933142007489658894)

项目地址[https://gitee.com/sbkwind/vue-responsive-summary](https://gitee.com/sbkwind/vue-responsive-summary)

