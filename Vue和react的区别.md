## 相同点

+ 都有组件化开发 和 虚拟dom
+ 都支持 通过标签属性 props 进行父子组件间通信
+ 都支持数据驱动视图，不直接操作真实dom，更新状态数据就自动更新页面
+ 都支持服务端渲染，vue 有 nuxt，react 有 next
+ 都支持原生 native 方案，react 的 react native，vue的 weex

## 不同点

+ 数据绑定，vue 实现的是数据的双向绑定，react 数据流动是单向的
+ 组件的写法不一样，React 推荐的做法是 JSX，也就是把 html、css、js都写到JavaScript里面，即 *all in js*；Vue推荐的做法是 webpack+vue-loader 的单文件组件格式，即写在同一文件中
+ state 对象在react应用中不可变，需要使用 setState 方法更新状态；在vue中state对象不是必须的，数据有data属性在vue对象中管理
+ 虚拟dom不一样，vue会跟踪每一个组件的依赖关系，不需要重新渲染整个组件树，而对于react而言，每当应用的状态被改变时全部组件都会被重新渲染，所以react 中需要 shouldComponentUpdate 这个生命周期函数方法来进行控制
+ react 严格上只针对MVC 中的View层，Vue则是MVVM模式