# Vuex

> Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能

**状态管理是什么？**

> 其实就是将多个组件共享的变量全部存储在一个对象里面，然后将这个对象存储在顶层的vue实例中，让其他的组件可以使用。

**vuex 解决了什么问题？**

在 vue 中父子组件间的通信使用通过属性绑定和`$emit`实现的，这种方式在组件层级嵌套不深的情况下确实是一个不错的解决方案，当时当遇到组件嵌套深或者是兄弟组件间通信的时候就会比较麻烦了。vuex 就是为了解决 vue 中兄弟组件，爷孙组件间通行的问题，可以简单将 vuex 理解为数据中心（数据中心中的数据也是响应式的）。

一般在 vuex 中可以存放一些公共的数据，比如 token，用户信息，购物车数据，商品收藏数据等等。

> 如果您不打算开发大型单页应用，使用 Vuex 可能是繁琐冗余的。确实是如此——如果您的应用够简单，您最好不要使用 Vuex。一个简单的 [store 模式](https://cn.vuejs.org/v2/guide/state-management.html#简单状态管理起步使用)就足够您所需了。
>
> 如果相关比较简单的话完全可以使用更简单的方法去管理数据，比如上面说的 store模式，或者是事件车模式（发布订阅模式）等。



## vuex 工作及使用流程

![vuex](Vuex/vuex.png)



## 使用 Vuex 



### 安装

```js
npm i vuex -S
```



### 创建 vuex 实例

一般是将 vuex 放到 项目的 store 文件夹下，创建`src/store/store.js`

```js
import Vue from "vue";
import Vuex from "vuex";

//注册 vuex 插件到 vue
Vue.use(Vuex);

//创建vue实例
const store = new Vuex.Store({
    state:{
        count:0,
        students:[],
    },
    mutations:{
        increment(state,data){
            state.count += data
        },
        updateStudents(state,data){
            state.student = data;
        }
    },
    getters:{
        getCount(state,getters){
            return state.count;
        }
    },
    actions:{
        async getStudents(ctx,data){
            const students = await axios("/getStudents");
            ctx.commit("updateStudents",students);
        }
    },
    modules:{
        a:{
            state:{},
            getters:{},
        },
        b:{
            state:{},
            mutations:{},
        }
    }
});

//导出数据
export default store;
```



### 挂载到vue实例上

```js
import Vue from "vue";
import store from "./src/store/store.js";
import App from "./App";

new Vue({
    render:h=>h(App),
    store,
}).$mount("#app");
```



## Vuex 中的属性的

- `state`：用来存储所有需要共享的数据，但是**不要直接访问或者修改里面的值**，因为这种修改的方式是不会被 devtools 监听到的，这样就很难调试。可以使用 `mutations`来修改，用`getters`来访问。
	- 单一状态树：也叫单一数据源，官方推荐一个项目里面只使用一个 store ，不要创建多个store，这样会比较容易维护，如果实在是要做分区可以使用 `modules`来实现。
- `mutations`：用来修改 state 中数据的值，不过这里的方法只能做同步操作，异步操作的话 devtools 可能监听不到变化
	- `mutations`里面的方法接收两个参数，第一个是 store 中的 state ，第二个是传入的数据
	- `this.$store.commit("mutation方法名",数据)`：执行 mutaitions 里面的方法 。也可以这样写`this.$store.commit({type:"mutation方法名",数据})`，不过这样写的时候在mutation 接收到的数据就是是一个对象。
- `getters`：用来获取 state 中数据值，可以按照计算属性的形式进行使用。
	- `getters`里面的方法接受两个个参数，第一个是 store 中的 state ，第二个是其他的 `getters`对象。
	- `this.$store.getters.getter方法名`：访问 state 中的变量。如果想要按函数的方式使用的话，getters 可以返回一个函数来实现。
- `actions`：用来做一些异步的操作，一般是获取后台数据，获取到数据之后可以使用 `mutaitions` 来将数据同步到 state 上。
	- `actions`中的方法接受两个参数。第一个是 vuex 的 实例对象，也是vuex 的执行上下文 ，第二个是传入的数据。
	- `this.$store.dispatch("action方法名",数据)`执行`actions`中的方法。也可以这样写`this.$store.dispatch({type:"action方法名",数据})`，不过这样写的时候在 actions 接收到的数据就是是一个对象。
	- 有时我们需要知道`action`什么时候结束，在结束的时候做一些操作，这是可以在`actions`使用`promise`并放回，之后使用`then`绑定回调事件
- `modules`：命名空间，如果项目比较大的话会使用到。里面就是一个个小的vuex实例。具体使用还是看[官网](https://vuex.vuejs.org/zh/guide/modules.html)比较好，哪里有详细的解释



## vuex 中的响应式数据

vuex 中的响应式数据和 vue 的中基本一样，一开始在 state 中定义好的数据才是响应式的，后面动态加入的数据就不会加入到 vue 的响应式系统中。

### 增加响应式数据

如果想要将数据加入到响应式系统中可以使用vue的set方法`Vue.set(目标对象,key,value)`，

+ 目标对象可以使用对象也可以是数组。
+ key：目标对象是对象时，就是属性，如果是数组就是索引
+ value：要设置的初始值

### 删除响应式数据

如果想要删除一个data或者state中定义的对象的数据里面的属性的话可以使用两种方法，一个是从新赋值一个新的对象，这个对象里面排除掉要删除的属性。还有一个方法就是使用 vue 的 delete 方法`Vue.delete(目标对象,key)`，参数和`Vue.set`一样。



### 辅助工具

在vue中使用vuex时，访问都是需要写一堆很长的东西，比如说使用`getters`需要这样写`this.$store.getters.geters方法名`这样显然是分成不优雅的，而且也分成难写，而使用`this.$store.state.state属性名`的方式也可能会被误修改。

vuex提供了下面的几个方法可以将vuex中的数据和方法注入到 vue 中。其中`mapState`和`mapGetters`一般放在`computed`中，`mapMutations`和`mapActions`一般放到`methods`中。

#### `mapState`

```js
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 `mapState` 传一个字符串数组。

```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```



#### `mapGetters`

```js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

如果你想将一个 getter 属性另取一个名字，使用对象形式：

```js
...mapGetters({
  // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```



#### `mapMutations`

```js
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```



#### `mapActions`

```js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```