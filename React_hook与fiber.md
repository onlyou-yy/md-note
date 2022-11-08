## Fiber 前置知识

### react为什么要用Fiber

Fiber 架构是在React 16的时候才被提出的，在这之前 React 执行任务是使用协调的完成的

+ React 会递归对比 虚拟DOM树，找出需要变动的节点，然后同步更新它们，这个过程 React 称为**协程**（Reconcilation）
+ 在 Reconcilation 期间，Rect会一直占用浏览器资源，一则会导致用户出发的事件得不到响应，二则会导致掉帧，用户会感觉到卡顿



### 屏幕刷新率

大多数设备的屏幕刷新率为 60 次/秒，浏览器渲染动画或者页面每一帧的速率也需要和设备屏幕的刷新率保持一致，而浏览器页面是一帧一帧绘制出来的，当每秒绘制的帧数（FPS）达到 60 时页面就比较流畅，小于这个值的时候页面会卡顿，每个帧绘制的时间时 1/60 = 16.6ms ，所以每一帧分到的时间时 16.6ms 。所以我们在书写代码的时候要尽量将一帧的工作量控制在 16ms 内

### 帧

![image-20221108142025620](/Users/gcb/Desktop/ljf_new/file/md-note/React_hook与fiber/image-20221108142025620.png)

一帧包含了用户的交互、js的执行、以及`requestAnimationFrame`的调用，布局计算以及页面的重绘等工作。 假如某一帧里面要执行的任务不多，在不到16ms的时间内就完成了上述任务的话，那么这一帧就会有一定的空闲时间，这段时间就恰好可以用来执行`requestIdleCallback`的回调，相对应的如果这一帧的任务很多没有剩余的时间就不会执行`requestIdleCallback`的回调，并且只有等到有帧有剩余时间的时候才会执行

> `requestIdleCallback(fn,timeout)`还可以接受一个超时时间，如果超过这个时间还没有执行就会立即执行，并且在回调函数中可以接受到一个 deadline 对象，这个对象有两个属性，`deadline.timeRemaining()` 可以返回当前帧还剩下多少时间可以使用，`didTimeout`当前 fn 是超时

```JS
function sleep(delay){
  for(let now=Date.now();Date.now() - now > delay;){}
}
const works = [
  ()=>{
    console.log("task 1 start")
    sleep(20)
    console.log("task 1 end")
  },
  ()=>{
    console.log("task 2 start")
    sleep(20)
    console.log("task 2 end")
  },
  ()=>{
    console.log("task 3 start")
    sleep(20)
    console.log("task 3 end")
  },
]

requestIdleCallback(workLoop,{timeout:1000});
function workLoop(deadline){
 console.log(`deadline,timeRemain:${deadline.timeRemaining()},didTimeout:${deadline.didTimeout}`)
  if((deadline.timeRemaining() > 0 || deadline.didTimeout) && works.length){
    performUnitOfWork()
  }
  if(works.length){
    requestIdleCallback(workLoop,{timeout:1000});
  }
}
function performUnitOfWork(){
  works.shift()();
}
```

目前 react 的 fiber 就是运用了`requestIdleCallback`的原理将任务进行时间分片分割成多个小任务，然后按照优先级来在绘制之后执行。但是目前`requestIdleCallback`只有Chrome支持，所以React利用`MessageChannel`来模拟了`requestIdleCallback`

### MessageChannel

`MessageChannel`API允许我们创建一个新的消息通道，并通过它的两个 `MessagePort` 属性发送数据，`MessageChannel`创建了一个通信管道，这个管道有两个端口，每个端口都可以通过`postMessage`发送数据，而另一个端口通过绑定`onmessage`事件就可以接受到来自另一个端口的数据，同时MessageChannel 是一个宏任务。



## 链表

Fiber 中很多地方都使用到链表结构

```js
class Update{
  constructor(payload,nextUpdate){
    this.payload = payload;
    this.nextUpdate = nextUpdate;
  }
}
class UpdateQueue{
  constructor(){
    this.baseState = null;//原状态
    this.firstUpdate = null;//第一个更新
    this.lastUpdate = null;//最后一个更新
  }
  enqueueUpdate(update){
    if(this.firstUpdate){
      this.firstUpdate = this.lastUpdate = update
    }else{
      this.lastUpdate.nextUpdate = update
      this.lastUpdate = update
    }
  }
  //便利链表进行更新
  forceUpdate(){
    let currentState = this.baseState || {}
    let currentUpdate = this.firstUpdate;
    while(currentUpdate){
      let nextState = typeof currentUpdate.payload === 'function' ? 
          currentUpdate.payload(currentState) : currentUpdate.payload;
      currentState = {...currentState,...nextState};//合并状态
      currentUpdate = currentUpdate.nextUpdate;//继续下一个更新
    }
    //更新完成之后重置链表
    this.firstUpdate = this.lastUpdate = null;
    this.baseState = currentState;
    return currentState;
  }
}
let queue = new UpdateQueue()
queue.enqueueUpdate(new Update({name:"jack"}))
queue.enqueueUpdate(new Update({age:12}))
queue.enqueueUpdate(new Update((state)=>({age:state.age + 1})))
queue.enqueueUpdate(new Update((state)=>({age:state.age + 1})))
queue.forceUpdate()
```





## 参考

+ [React 技术揭秘](https://kasong.gitee.io/just-react/)
+ [这可能是最通俗的 React Fiber(时间分片) 打开方式](https://juejin.cn/post/6844903975112671239)
+ [你应该知道的requestIdleCallback](https://juejin.cn/post/6844903592831238157)
+ [手写React的Fiber架构，深入理解其原理](https://juejin.cn/post/6844904197008130062)