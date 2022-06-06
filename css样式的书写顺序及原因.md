记得刚开始学习前端的时候，每次写css样式都是用到什么就在样式表后添加什么，完全没有考虑到样式属性的书写顺序对网页加载代码的影响。后来逐渐才知道正确的样式顺序不仅易于查看，并且也属于css样式优化的一种方式。那么是怎么个顺序呢？

1. 定位属性：`position  display  float  left  top  right  bottom   overflow  clear   z-index`
2. 自身属性：`width  height  padding  border  margin   background`
3. 文字样式：`font-family   font-size   font-style   font-weight   font-varient   color   `
4. 文本属性：`text-align   vertical-align   text-wrap   text-transform   text-indent    text-decoration   letter-spacing    word-spacing    white-space   text-overflow`
5. css3中新增属性：`content   box-shadow   border-radius  transform……`

一般都是按照上述`1-2-3-4-5`的顺序进行书写。这样做的目的是**减少浏览器reflow（回流）**，提升浏览器渲染dom的性能



因为浏览器的渲染流程为：

1. 解析html构建dom树，解析css构建css树：将html解析成树形的数据结构，将css解析成树形的数据结构
2. 构建render树：DOM树和CSS树合并之后形成的render树。
3. 布局render树：有了render树，浏览器已经知道那些网页中有哪些节点，各个节点的css定义和以及它们的从属关系，从而计算出每个节点在屏幕中的位置。
4. 绘制render树：按照计算出来的规则，通过显卡把内容画在屏幕上。

css样式解析到显示至浏览器屏幕上就发生在1，2，3步骤，可见浏览器并不是一获取到css样式就立马开始解析而是根据css样式的书写顺序将之按照dom树的结构分布render样式，完成第2步，然后开始遍历每个树结点的css样式进行解析，此时的css样式的遍历顺序完全是按照之前的书写顺序。在解析过程中，一旦浏览器发现某个元素的定位变化影响布局，则需要倒回去重新渲染正如按照这样的书写书序：

```css
width: 100px;
height: 100px;
background-color: red ;
position: absolute;
```

当浏览器解析到position的时候突然发现该元素是绝对定位元素需要脱离文档流，而之前却是按照普通元素进行解析的，所以不得不重新渲染，解除该元素在文档中所占位置，然而由于该元素的占位发生变化，其他元素也可能会受到它回流的影响而重新排位。最终导致③步骤花费的时间太久而影响到④步骤的显示，影响了用户体验。

所以规范的的css书写顺序对于文档渲染来说一定是事半功倍的！



**注意**：

+ render树的结构不等同于DOM树的结构，一些设置display:none的节点不会被放在render树中，但会在dom树中。
+ 有些情况，比如修改了元素的样式，浏览器并不会理科reflow或repaint，而是把这些操作积攒一批，然后做一次reflow，这也叫做异步reflow.但在有些情况下，比如改变窗口，改变页面默认的字体等，对于这些情况，浏览器会马上进行reflow.
+ 为了更好的用户体验，渲染引擎将会尽可能早的将内容呈现到屏幕上，并不会等到所有的html都解析完成之后再去构建和布局render树。它是解析完一部分内容就显示一部分内容，同时，可能还在通过网络下载其余内容。



原文链接：https://blog.csdn.net/qq_36060786/article/details/79311244