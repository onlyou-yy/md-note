## vue3新特性

### 组件可以有多个根节点

在 vue2 中有组件的根节点只能有一个，而在 vue3 中可以有多个根节点，如

```vue
<template>
	<h1>hello</h1>
	<span>world</span>
</template>
```



### createApp新api

在 vue2 中定义初始化vue实例需要使用 new 关键字，但是在vue3中则提供了更为方便的创建vue实例的 createApp 方法

vue2 版本在脚手架上创建vue实例

```js
import Vue from 'vue';
import router from '@/router/router'
import App form '@/app';

constvm=new Vue({
  el: '#app',
  router,
  render: h => h(App)
})
```

使用vue3的createApp创建vue实例

```js
import Vue,{crateApp} from 'vue';
import router from '@/router/router'
import App form '@/app';
//createApp 返回的是Vue,第二个参数传递的全局数据，在app的子组件全都可以通过 this.props.test 访问
createApp(App,{test:1}).use(router).mount('#app')
```

[vue3-创建应用createApp](https://www.jianshu.com/p/d2fa67f42b3c)



### 定义组件新方法 defineComponent 

vue3 更好地支持了 typeScript 所以使用 typescript 会更加流畅。对于创建组件的方式上vue3 为了在开发的时候给出 typescript 的类型提示也提出来新的方法——使用 defineComponent 方法创建，使用这个方法创建可以得到有typescript给出的语法以及参数提示。

vue2 的创建方式

```vue
<template>....</template>
<script>
	export default {....}
</script>
```

vue3 的新方式

```vue
<template>....</template>
<script lang='ts'>
	import {defineComponent} from 'vue'
  export default defineComponent({
    ....
  })
</script>
```



#### 定义异步组件 defineAsyncComponent

在vue3中不能使用 `const com=()=>import('./components/AsyncComponent.vue')`的组件懒加载写法，需要使用 defineAsyncComponent 

对于基本用法，`defineAsyncComponent` 可以接受一个返回 `Promise` 的工厂函数。Promise 的 `resolve` 回调应该在服务端返回组件定义后被调用。你也可以调用 `reject(reason)` 来表示加载失败。

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => import('./components/AsyncComponent.vue'))

app.component('async-component', AsyncComp)
```



### setup 和 ref 、reactive

#### **setup注册数据和方法**

+ 新的 option ，所有的组合 API 函数都在此处使用，只在初始化的时候执行一次
+ 函数如果返回对象，对象中的属性或者方法在模板中可以直接使用

```vue
<template>
	<h2>{{txt}}</h2>
</template>
<script lang='ts'>
    import {defineComponent} from 'vue'
    export default defineComponent({
        name:'test',
        setup(){
            return {
                txt:'hello'
            }
        }
    })
</script>
```

**注意**：steup返回的对象中的数据并不是响应式的数据，如果需要响应式的数据可以使用 ref 进行转化

> `<script setup>`是 setup 方式的语法糖，在这个标签下写 代码就相当于是在 options 的 setup中一样，而且数据不需要返回就可以直接在模版中使用，并且 还可以直接使用 await
>
> ```vue
> <template>
> 	<h2>{{txt}}</h2>
> </template>
> <script setup>
>   let txt = ref('test');
>   let result = await getData();
> </script>
> ```



#### **setup的一些细节问题**

**setup的执行时机**

- setup 是在 beforeCreate 生命周期回调之前执行的，而且只执行一次。因此在 setup 执行的时候组件还没有被创建出来，也就是所实例对象的 this 还不能用，this 指向的是 undefined
- 其实所有的 component API 相关回调函数中都不可以使用 this 的。

**setup的返回值问题**

- setup 中的返回值是一个对象，内部的属性和方法是给 html 模板使用的
- setup 中的对象内部的属性和方法和data函数中 return 对象的属性都可以坐在 html 模板中使用
- setup 中的对象中的属性和 data 函数中的对象中的属性会合并为组件对象的属性
- setup 中的对象中的方法和 methods 函数中的方法也会合并为组件对象的方法
- **注意**：
	- 一般不要混用，因为 methods 和 data 可以访问 setup 中的属性和方法，但是 setup 中不能访问 methods 和 data  
	- setup 不能是一个  async 函数，因为使用了async 返回的将不会是一个对象，而是一个 promise 实例

 **setup 中的参数**

setup(props,context)，接收两个参数

- props：是一个对象，里面有父级组件向自己组件传递的数据，而且是在自己组件中使用 props 接收到的所有属性，包含 props 配置声明且传入了的所有属性的对象
- context：是一个对象，里面有 attrs 对象（获取当前标签上的属性，但是该属性是在 props 中没有声明接收的所有的属性，相当于this.\$attrs）；emit 方法（分发事件的相当于 this.$emit）；slots 对象（包含所有传入的插槽内容的对象，相当于 this.\$slots）；expose 方法可以将属性或方法导出，当父组件中访问组件实例时这些属性和实例会被挂载到实例上。



#### **ref定义基本类型的响应式数据**

ref(initData) 是一个函数，它的作用是定义一个响应式的数据，返回的是一个 **Ref对象** ，对象中有一个 value 属性如果需要对数据进行操作，需要使用 Ref对象 调用 value 属性的方式进行数据的操作。但是在 html 模板中不需要写 value 就可以访问到。

```vue
<template>
	<h2>{{txt}}</h2>
	<h3>{{count}}</h3>
	<button @click='increment'>increment</button>
</template>
<script lang='ts'>
    import {defineComponent,ref} from 'vue'
    export default defineComponent({
        name:'test',
        setup(){
            let count=ref(0);
            function increment(){
                count.value++;
            }
            return {
                txt:'this count is :',
                count,
                increment
            }
        }
    })
</script>
```

**注意**：ref 一般用于定义一个**基本类型**的响应式数据



#### **reactive 定义复杂类型的响应式数据**

ref 只能对基本类型定义响应式数据，而 reactive 则可以为复杂类型的数据定义响应式数据。

+ reactive(obj) 接收一个普通对象然后返回该普通对象的响应数据代理器对象（Proxy）
+ 响应式转换是**深层的**：会影响对象内部所有嵌套的属性
+ 内部基于 ES6 的 proxy 实现，**通过代理对象操作**原对象内部数据都是响应式的

```vue
<template>
<h2>{{txt}}</h2>
<ul>
    <li>{{student.name}}</li>
    <li v-for="(item,i) in student.cars" :key='i'>{{item}}</li>
</ul>
<button @click='changeHandle'>changeHandle</button>
</template>
<script lang='ts'>
    import {defineComponent,reactive} from 'vue';
    import { nanoid } from 'nanoid'
    export default defineComponent({
        name:'test',
        setup(){
            let student=reactive({
                id:nanoid(),
                name:'jack',
                cars:['玛莎拉蒂','奔驰','宝马']
            });
            const changeHandle=()=>{
                student.name='老王';
                student.cars[0]='本田'
            }
            return {
                txt:'this student is :',
                student,
                changeHandle
            }
        }
    })
</script>
```

**注意**：如果修改原数据的话，页面数据是不会更新的，但是如果是删除原对象的属性的话就会更新数据，所以一般想要有响应式效果的话就需要操作代理对象。

#### **reactive 和 ref 的一些细节问题**

+ 如果用 ref 处理对象/数组，内部会自动使用 reactive 来进行处理，最终返回的是一个 proxy 代理对象
+ ref 内部通过给value属性添加 getter/setter 来实现对数据的劫持
+ reactive 内部通过使用 proxy 来实现对对象内部所有数据的劫持，并通过 Reflect操作对象内部数据
+ ref 的数据操作在 js 中要使用 .value ，在模板中不需要（内部模板解析的时候会自动添加上 .value）



### vue2 和 vue3 的响应式对比

**vue2 响应式的实现核心方法：**

+ 对象：通过`Object.defineProperty`对对象的已有属性值的读取和修改进行劫持（监视 / 拦截）
+ 数组：通过重写数组更新数组一系列更新元素的方法来实现元素的修改和劫持

```js
Object.defineProperty(data,'count',{
  set(newVal){},
  get(){},
})
```

使用数据劫持方法出现的问题

+ 对象直接新添加属性或者删除已有属性，界面不会自动更新
+ 直接通过下标替换袁术或者更新length，界面不会自动更新，如 arr[1] = 0



**vue3 的响应式实现核心方法**

+ 通过Proxy（代理）：拦截对data任意属性的任意操作，包或属性值的读写，属性的添加、删除等
+ 通过Reflect（反射）：动态对被代理的对象的相应的属性进行特地的操作

```js
new Proxy(data,{
    get(target,prop){
        return Reflect.get(target,prop)
    },
    set(target,prop,val){
        return Reflect.set(target,prop,val)
    },
    deleteProperty(target,prop){
        return Reflect.get(target,prop)
    },
})
```

proxy 对对象的监听是深层的，对象中的属性值变化或者是数组中直接通过下标修改值的情况也是能监听到的，相比于 vue2 中只能监听浅层对象的情况要好得多。



### **vue3 中的计算属性和监视**

在 vue3 中可以导出制造计算属性的的函数 computed 和监听器 watch / watchEffect 这两个属性一般是用在setup 中制作计算属性和给属性添加监视器的

```vue
<template>
  <fieldset>
    <legend>姓名操作</legend>
    <input type="text" v-model="nameStr.firstname"><br />
    <input type="text" v-model="nameStr.lastname"><br />
    </fieldset>
  <fieldset>
    <legend>计算属性和监视</legend>
    <input type="text" v-model="fullname1"><br />
    <input type="text" v-model="fullname2"><br />
  </fieldset>
</template>

<script lang="ts">
import {defineComponent,ref,reactive,watch,computed} from 'vue'
export default defineComponent({
    name:'Test',
    setup(){
        const nameStr=reactive({
            firstname:'',
            lastname:''
        })
        let fullname1=computed({
            get(){
                return nameStr.firstname+"_"+nameStr.lastname
            },
            set(val){
                console.log(val);
            }
        })
        let fullname2=ref('');
        watch(nameStr,({firstname,lastname})=>{
            console.log(nameStr,nameStr.firstname)
            fullname2.value=firstname+"_"+lastname
        }，{immediate:true,deep:true})
        return {
            nameStr,
            fullname1,
            fullname2,
        }
    }
})
</script>
```

+ computed：在定义的时候会被执行一次
+ watch：会生成一个监视器对象，在定义的时候是会立即执行一次的，如果想要立即执行可以在第三个参数添加 immediate 属性，deep 属性是开启深度监听。
	+ watch 可以监听多个数据，但是只能监听响应式的数据，如果要监听非响应式的数据需要写错回调形式

```js
//加入data1是响应式的数据，data2是非响应式数据
const data1=reactive({
  a:1,
  b:2
})
const data2=0;
watch([data1,data1.a,()=>data2],(newVal,oldVal)=>{
  ....
})
```

这里的 data1 和 data2 是被注册了的响应式数据watch可以监听到，但是 data1.a 中 a 是data1 的proxy对象进行管理的对 data1.a 这是一个响应式的数据，但是对于 a 一个属性的话就不是响应式的数据，所以watch是不会监视 data1.a 的。但是 watch 并不会自动检测 深层的对象这时候就需要使用回调的形式或者开启深监视模式才行。

watchEffect：和 watch 一样，但是会在定义的时候立即执行一次



### vue3 生命周期函数

vue2 的声明周期函数图

![Vue 实例生命周期](Vue3/lifecycle.png)

对于 vue3 的生命周期其实并并没有太大的变化，只是生命周期都换了个名字而已，但是在 vue3 中还是可以使用 vue2 的生命周期函数的。

| 2.0           | 3.0             |
| ------------- | --------------- |
| beforeCreate  | use setup()     |
| created       | use setup()     |
| beforeMount   | onBeforeMount   |
| mounted       | onMounted       |
| beforeUpdate  | onBeforeUpdate  |
| updated       | onUpdated       |
| beforeDestroy | onBeforeUnmount |
| destroyed     | onUnmounted     |
| errorCaptured | onErrorCaptured |

新的生命周期函数都是些组合api 使用前需要在 vue 中引入才行，如

```vue
<script lang="ts">
	import {defineComponent,onBeforeMount} from 'vue;
  export default defineComponent({
    name:'Test',
    setup(){
      onBeforeMount(()=>{
        ...
      })
    }
  })
</script>
```

组合式 API 的生命周期钩子函数基本都是传入 函数作为参数的

并且这些组合 API 可以在 setup 中使用。虽然 vue3 和 vue2 的生命周期函数作用都是一样的，但是 vue3 中生命周期函数会比 vue2 中的生命周期函数更快执行。



### vue3 中的 hook

+ 使用 vue3 的组合API封装的可复用的功能函数
+ 自定义 hook 的作用类似于vue2中的mixin技术（将公共的属性或者方法注入到组件中）
+ 自定义的 hook 的优势L很清楚复用功能代码的来源，更清楚易懂

vue3 中的 hook 可以理解为是一个使用 vue 中方法专注于逻辑层面的一个公共组件方法，也就是可以使用 ref 、reactive 、watch 这些api 去实现组件中公共方法的函数。 

如，收集用户鼠标点击的页面坐标  `./hooks/useMousePosition.ts`

```ts
import {ref,onMounted,onBeforeUnmount} from 'vue';
export default function useMousePostion(){
  const x = ref(0);
  const y = ref(0);
  const updatePosition = (e:MouseEvent)=>{
    x.value=e.pageX;
    y.value=e.pageY;
  }
 	onMounted(){
    document.addEventListener('click',updatePosition);
  }
  onBeforeUnmount(){
    document.removeEventListener('click',updatePosition);
  }
  return {
    x,y
  }
}
```

在组件中使用

```vue
<script lang=ts>
	import useMousePostion from './hooks/useMousePosition';
  export default defineComponent({
    name:'Test',
		setup(){
      let {x,y}=useMousePostion();
      return {
        x,y
      }
    }
  })
</script>
```

调用 useMousePostion 的使用就会把其中的 变量和声明周期函数一同注入到组件中，如果有多个hook 使用相同的声明周期的话也是没有问题的，因为会被作用域分开，但是都会被执行。



### toRefs使用

toRefs 的作用：

+ 把一个响应式对象转换成普通对象，改普通对象的每个 property 都是一个ref
+ 应用：当从合成函数返回响应式对象的时候，toRefs 非常有用，这样消费组件就可以中不丢失响应式的情况下对返回的对象进行分解使用。简单来说就是如果有一个函数返回的是一个响应式的数据 a ，并且 a 也在页面中有被渲染使用的话，我们在编辑 a 的时候 本来不需要更新的页面也会随之更新，所以在不修改（或者说污染） a 的前提下要得到响应式的数据就需要使用到 toRefs 方法。（并且 toRefs 也是可以作用于普通对象的）
+ 问题：reactive对象取出的所有属性值都是非响应式的
+ 解决：利用 toRefs 可以将一个响应式的 reactive 对象的所有原始属性转换成响应式的 ref 属性



### ref 的其他作用

ref 可以像 vue2 中的标签属性 ref 那样获取 页面中的元素

```vue
<template>
  <h2>ref的作用</h2>
	<input type='text' ref="inputRef"/>
</template>

<script lang="ts">
import {defineComponent,ref,onMounted} from 'vue'
export default defineComponent({
    name:'Test',
    setup(){
      const inputRef=ref<HTMLElement | null>(null);
      onMounted(()=>{
        inputRef.value&&inputRef.value.focus();
      })
      return {
        inputRef
      }
    }
})
</script>
```

被 ref 标记后的元素会在页面被挂载之后赋值给 inputRef



### shallowReactive 和 shallowRef

+ shallowReactive ：只处理了对象内最外层的响应式
+ shallowRef：只处理了value的响应式，不进行对象的reactive处理

reactive 和 ref 处理对象的时候返回的是一个**深度响应的数据**，如果使用 shallowReactive 或者 shallowRef 处理对象的话得到的则会使浅响应的数据

什么时候使用浅响应式数据？

+ 一般都是使用 ref 和 reactive 深度响应式数据
+ 如果是一个对象数据，结构比较深，但是变化时只是外层属性变化，那么使用 shallowReactive
+ 如果是一个对象数据，后面会产生新的对象来替换 ，那么使用 shalloRef



### readonly 和 shallowReadonly

readonly 是一个组合API，会将一个数据进行深度只读操作

```js
const state=reactive({
  name:'jack',
  age:12,
  cars:['玛莎拉蒂','宝马']
})
const readonlyState=readonly(state);
```

相对的 也会有个浅度只读的 shallowReadonly 

readonly：

+ 深度只读数据
+ 获取一个对象（响应式或者纯对象）或者 ref 并返回原始代理的只读代理
+ 只读代理是深层的：访问的任意嵌套 property 也是只读的

shallowReadonly

+ 浅只读数据
+ 创建一个代理，是其自身的 property 为只读，但不执行嵌套对象的深度只读转换



### toRaw 和 markRaw

reactive 和 ref 是将对象变成响应式的代理对象，相对的也有方法可以将这些代理对象重新变回普通的对象或者数据。组合API toRaw 和 markRaw 就是这一过程的api

```js
const state=reactive({
  a:1
})
const rawState=toRaw(state);
const makeRawState=markRaw(state);
```

toRaw 

+ 返回由 reactive 或 readonly 方法转换成响应式代理的普通对象
+ 只是一个还原方法，可以用于临时兑取，访问不会被代理/跟踪，写入时也不会触发界面更新

markRaw

+ 标记一个对象，使其永远不会转换为代理，返回对象本身
+ 应用场景
	+ 有些值不应该被设置为响应式的，例如复杂的第三方类实例或者 Vue 组件对象
	+ 当渲染具有不可变数据源的大列表时，跳过代理转换可以提高性能



### toRef

+ 为源响应式对象上的某个属性创建一个 ref 对象，二者内部操作的是同一个数据值，更新时二者是同步的
+ 与 ref 的区别：ref是拷贝一份新的数据值单独操作，更新时互相不影响
+ 应用：当要将某个 prop 的 ref 传递给复合函数时使用（有些库可能需要传递进去的是数值可能是 Ref 对象，但是通过 v-bind 传递给子组件的是一个数组，如果传入的是一个有 ref 转化的数据那个这个数据交由函数处理后处理变化将不会实时响应的页面和源数据上，所以需要使用 toRef 类似于将传值改为传址）

```vue
<template>
    <div>
      <Child :age="age"></Child>
      <hr />
      <button @click="updateAge">updateAge</button>
    </div>
</template>
<script lang='ts'>
import {computed, defineComponent,Ref,ref,toRef} from 'vue';
export default defineComponent({
    name:'Parent',
    setup(){
        const nameStr=reactive({
          age:98
        })
        const age=toRef(nameStr,'age')
        const updateAge=()=>{
                    age.value++;
              }
        return {
          age
        }
    }
})
</script>

//++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
<template>
    <div>
        <h2>this props is：{{age}}</h2>
      	<h2>this length is：{{length}}</h2>
    </div>
</template>
<script lang='ts'>
import {computed, defineComponent,Ref,ref,toRef} from 'vue';
function useAgeLength(data:Ref){
    return computed(()=>{
       return data.value.toString().length;
    })
}
export default defineComponent({
    name:'Child',
    props:{
        age:{
            type:Number
        }
    },
    setup(props){
        const length=useAgeLength(toRef(props,'age'));
        // const length=useAgeLength(ref(props.age));
        return {
            length
        }
    }
})
</script>
```



### customRef 自定义 ref 

`customRef`用于自定义一个 ref ，可以显式地控制依赖跟踪和触发响应，接收一个工厂函数，两个参数分别是用于追踪的`track`与用于触发响应的`trigger`，并返回一个带有`get`和`set`的属性对象。

`track`与`trigger`其实是 proxy 中的方法，在 customRef 中被再次封装了一遍，在proxy 中他们的作用分别是：

+ track：当有一个 API 能够在某些内容发生变化时更新最终值，我们必须在内容发生变化时设置新的值。可以使用 `track` 函数执行此操作，该函数可以传入 `target` 和 `key` 两个参数。简单来说就是定位更新得到最新的值，并且**通知vue要最终监视该值**
+ trigger：当需要更新 Proxy 的值 value 的时候需要使用 trigger 方法方法进行更新并**告诉vue更新界面**，该函数可以传入 `target` 和 `key` 两个参数。

[Vue3 的响应式和以前有什么区别，Proxy 无敌？（源码级详解）](https://blog.csdn.net/weixin_40906515/article/details/106168778)

但是在 customRef 中不需要传参数也可以，因为 ref 只控制一个值

可以使用 customRef 制作带防抖功能的 v-model

```html
<input v-model="text" />
```

```js
function useDebouncedRef(val, delay = 200) {
  let timeout
  return customRef((track, trigger) => {
    return {
      get() {
        track();//
        return value
      },
      set(newValue) {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
          value = newValue
          trigger()
        }, delay)
      }
    }
  })
}

export default {
  setup() {
    return {
      text: useDebouncedRef('hello')
    }
  }
}
```



### provide 和 inject 跨层级组件通信

以往实现组件间通信可以使用，v-bind + @事件 、事件车（eventBus.\$on + eventBus.\$emit）、vuex 实现。现在多了一种更加方便的方式 provide + inject 方式，这种方式通过 provide 定义一个全局后代组件可以访问的数据，然后后代组件可以通过 inject 进行访问。

**用于组件选项**

**privide 提供数据**

```js
//顶层vue实例
const app = Vue.createApp({})
app.component('todo-list', {
  data() {
    return {
      todos: ['Feed a cat', 'Buy tickets']
    }
  },
  provide: {
    user: 'John Doe',
  },
})
```

如果需要 provide 一些实例属性数据的话，需要返回一个对象，并且

```js
provide: {
    return {
      todoLength: this.todos.length,
    }
},
```

**注意**：这种暴露出去的数据并不是响应式的，如果想要是响应式的话可以将数据用 ref 或 reactive 、computed 进行处理。

**inject 接收数据**

```js
//在后代组件中
app.component('todo-list-statistics', {
  inject: ['user'],
  created() {
    console.log(`Injected property: ${this.user}`) // > 注入 property: John Doe
  }
})
```



**组合式api使用**

+ provide(key,value)，一次只能暴露一个

```js
import { provide, reactive, ref } from 'vue'
import MyMarker from './MyMarker.vue

export default {
  components: {
    MyMarker
  },
  setup() {
    const location = ref('North Pole')
    const geolocation = reactive({
      longitude: 90,
      latitude: 135
    })

    provide('location', location)
    provide('geolocation', geolocation)
  }
}
```

+ inject(key,defaultVal)，当key不存在时使用默认值 defaultVal

```js
import { inject } from 'vue'

export default {
  setup() {
    const userLocation = inject('location', 'The Universe')
    const userGeolocation = inject('geolocation')

    return {
      userLocation,
      userGeolocation,
    }
  }
}
</script>
```



### 响应式数据的判断

+ isRef：检查一个值是否为一个对象
+ isReactive：检查一个对象是否是由 reactive 创建的响应式对象
+ isReadonly：检查一个对象是否是由 readonly 创建的只读对象
+ isProxy：检查一个对象是否是由 reactive 或者 readonly 创建的代理



### ref 、reactive等的拦截数据原理

#### 1) shallowReactive 与 reactive

```js
const reactiveHandler = {
  get (target, key) {

    if (key==='_is_reactive') return true

    return Reflect.get(target, key)
  },

  set (target, key, value) {
    const result = Reflect.set(target, key, value)
    console.log('数据已更新, 去更新界面')
    return result
  },

  deleteProperty (target, key) {
    const result = Reflect.deleteProperty(target, key)
    console.log('数据已删除, 去更新界面')
    return result
  },
}

/* 
自定义shallowReactive
*/
function shallowReactive(obj) {
  return new Proxy(obj, reactiveHandler)
}

/* 
自定义reactive
*/
function reactive (target) {
  if (target && typeof target==='object') {
    if (target instanceof Array) { // 数组
      target.forEach((item, index) => {
        target[index] = reactive(item)
      })
    } else { // 对象
      Object.keys(target).forEach(key => {
        target[key] = reactive(target[key])
      })
    }

    const proxy = new Proxy(target, reactiveHandler)
    return proxy
  }

  return target
}


/* 测试自定义shallowReactive */
const proxy = shallowReactive({
  a: {
    b: 3
  }
})

proxy.a = {b: 4} // 劫持到了
proxy.a.b = 5 // 没有劫持到


/* 测试自定义reactive */
const obj = {
  a: 'abc',
  b: [{x: 1}],
  c: {x: [11]},
}

const proxy = reactive(obj)
console.log(proxy)
proxy.b[0].x += 1
proxy.c.x[0] += 1
```



#### 2) shallowRef 与 ref

```js
/*
自定义shallowRef
*/
function shallowRef(target) {
  const result = {
    _value: target, // 用来保存数据的内部属性
    _is_ref: true, // 用来标识是ref对象
    get value () {
      return this._value
    },
    set value (val) {
      this._value = val
      console.log('set value 数据已更新, 去更新界面')
    }
  }

  return result
}

/* 
自定义ref
*/
function ref(target) {
  if (target && typeof target==='object') {
    target = reactive(target)
  }

  const result = {
    _value: target, // 用来保存数据的内部属性
    _is_ref: true, // 用来标识是ref对象
    get value () {
      return this._value
    },
    set value (val) {
      this._value = val
      console.log('set value 数据已更新, 去更新界面')
    }
  }

  return result
}

/* 测试自定义shallowRef */
const ref3 = shallowRef({
  a: 'abc',
})
ref3.value = 'xxx'
ref3.value.a = 'yyy'


/* 测试自定义ref */
const ref1 = ref(0)
const ref2 = ref({
  a: 'abc',
  b: [{x: 1}],
  c: {x: [11]},
})
ref1.value++
ref2.value.b[0].x++
console.log(ref1, ref2)
```



#### 3) shallowReadonly 与 readonly

```js
const readonlyHandler = {
  get (target, key) {
    if (key==='_is_readonly') return true

    return Reflect.get(target, key)
  },

  set () {
    console.warn('只读的, 不能修改')
    return true
  },

  deleteProperty () {
    console.warn('只读的, 不能删除')
    return true
  },
}

/* 
自定义shallowReadonly
*/
function shallowReadonly(obj) {
  return new Proxy(obj, readonlyHandler)
}

/* 
自定义readonly
*/
function readonly(target) {
  if (target && typeof target==='object') {
    if (target instanceof Array) { // 数组
      target.forEach((item, index) => {
        target[index] = readonly(item)
      })
    } else { // 对象
      Object.keys(target).forEach(key => {
        target[key] = readonly(target[key])
      })
    }
    const proxy = new Proxy(target, readonlyHandler)

    return proxy 
  }

  return target
}

/* 测试自定义readonly */
/* 测试自定义shallowReadonly */
const objReadOnly = readonly({
  a: {
    b: 1
  }
})
const objReadOnly2 = shallowReadonly({
  a: {
    b: 1
  }
})

objReadOnly.a = 1
objReadOnly.a.b = 2
objReadOnly2.a = 1
objReadOnly2.a.b = 2
```



#### 4) isRef, isReactive 与 isReadonly

```js
/* 
判断是否是ref对象
*/
function isRef(obj) {
  return obj && obj._is_ref
}

/* 
判断是否是reactive对象
*/
function isReactive(obj) {
  return obj && obj._is_reactive
}

/* 
判断是否是readonly对象
*/
function isReadonly(obj) {
  return obj && obj._is_readonly
}

/* 
是否是reactive或readonly产生的代理对象
*/
function isProxy (obj) {
  return isReactive(obj) || isReadonly(obj)
}


/* 测试判断函数 */
console.log(isReactive(reactive({})))
console.log(isRef(ref({})))
console.log(isReadonly(readonly({})))
console.log(isProxy(reactive({})))
console.log(isProxy(readonly({})))
```





### 新组件

#### 1) Fragment(片断)

- 在Vue2中: 组件必须有一个根标签
- 在Vue3中: 组件可以没有根标签, 内部会将多个标签包含在一个Fragment虚拟元素中
- 好处: 减少标签层级, 减小内存占用

```vue
<template>
    <h2>aaaa</h2>
    <h2>aaaa</h2>
</template>
```



#### 2) Teleport(瞬移)

- Teleport 提供了一种干净的方法, 让组件的html在父组件界面外的特定标签(很可能是body)下插入显示

ModalButton.vue

```vue
<template>
  <button @click="modalOpen = true">
      Open full screen modal! (With teleport!)
  </button>

  <teleport to="body">
    <div v-if="modalOpen" class="modal">
      <div>
        I'm a teleported modal! 
        (My parent is "body")
        <button @click="modalOpen = false">
          Close
        </button>
      </div>
    </div>
  </teleport>
</template>

<script>
import { ref } from 'vue'
export default {
  name: 'modal-button',
  setup () {
    const modalOpen = ref(false)
    return {
      modalOpen
    }
  }
}
</script>


<style>
.modal {
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  background-color: rgba(0,0,0,.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.modal div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: white;
  width: 300px;
  height: 300px;
  padding: 5px;
}
</style>
```

App.vue

```vue
<template>
  <h2>App</h2>
  <modal-button></modal-button>
</template>

<script lang="ts">
import ModalButton from './ModalButton.vue'

export default {
  setup() {
    return {
    }
  },

  components: {
    ModalButton
  }
}
</script>
```



#### 3) Suspense(不确定的)

- 它们允许我们的应用程序在等待异步组件时渲染一些后备内容，可以让我们创建一个平滑的用户体验

```vue
<template>
  <Suspense>
    <template v-slot:default>
      <AsyncComp/>
      <!-- <AsyncAddress/> -->
    </template>

    <template v-slot:fallback>
      <h1>LOADING...</h1>
    </template>
  </Suspense>
</template>

<script lang="ts">
/* 
异步组件 + Suspense组件
*/
// import AsyncComp from './AsyncComp.vue'
import AsyncAddress from './AsyncAddress.vue'
import { defineAsyncComponent } from 'vue'
const AsyncComp = defineAsyncComponent(() => import('./AsyncComp.vue'))
export default {
  setup() {
    return {
     
    }
  },

  components: {
    AsyncComp,
    AsyncAddress
  }
}
</script>
```

- AsyncComp.vue

```vue
<template>
  <h2>AsyncComp22</h2>
  <p>{{msg}}</p>
</template>

<script lang="ts">

export default {
  name: 'AsyncComp',
  setup () {
    // return new Promise((resolve, reject) => {
    //   setTimeout(() => {
    //     resolve({
    //       msg: 'abc'
    //     })
    //   }, 2000)
    // })
    return {
      msg: 'abc'
    }
  }
}
</script>
```

- AsyncAddress.vue

```vue
<template>
<h2>{{data}}</h2>
</template>

<script lang="ts">
import axios from 'axios'
export default {
  async setup() {
    const result = await axios.get('/data/address.json')
    return {
      data: result.data
    }
  }
}
</script>
```



## 总结

2020年9月发布的正式版vue3，vue3 支持大多数的vue2特性，并且vue3 中设计了一套强大的组合API 代替了 vue2 中的 option API 复用性更强了。并且更好的支持TS 。最主要的是 vue3 中使用 Proxy 配合 Reflect 代替了 vue2 中的 Object.defineProperty() 方法实现数据的响应式原理）。重写了虚拟DOM，速度更快了，并且设计了个新的脚手架 vite 。提供了新的组件 Fragment / Teleport / Suspense。



## 问题

### 为什么需要虚拟DOM？

[为什么需要虚拟DOM？

