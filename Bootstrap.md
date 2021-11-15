# Bootstrap

Bootstrap 采用了 `Normalize.css`来重置浏览器组件的基本样式，使用 `less` 进行开发的一个的**移动端优先**，基于**html5** 规则的优秀的css样式框架。

搭建基本的 Bootstrap 首先需要创建一个html页面，最好是提前安装规则创建，不然容易出现一些不知名的bug

```html
<!DOCTYPE html>
<html lang="zh-CN">
	<head>
		<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
 </head>
</html>
```



## 容器

Bootstrap 提供了两种容器，`container`与`container-fluid`，两种的区别在于`container`采用的是流体布局，也就是弹性盒子布局。而`container-fluid`采用的是**媒体查询**实现的是固定盒子容器。

`container-fluid`的相关阈值

| 断点标志 | 阈值 | 宽度 |
| :--: | :--: | :--: |
| lg(大屏pc) | windowWidth >= 1200 | 1170 |
| md(中屏pc) | 992 <= windowWidth < 1200 | 970 |
| sm(平板) | 768 <= windowWidth < 992 | 750 |
| xs(手机) | windowWidth < 768 | auto |

其他的一些样式也需要在容器中

容器的 less 源码如下

```less
.clearfix() {
  &:before,
  &:after {
    content: " "; // 1
    display: table; // 2
  }
  &:after {
    clear: both;
  }
}

.container-fixed(@gutter: @grid-gutter-width) {
  margin-right: auto;
  margin-left: auto;
  padding-left:  floor((@gutter / 2));
  padding-right: ceil((@gutter / 2));
  &:extend(.clearfix all);
}

.container {
  .container-fixed();
  @media (min-width: @screen-sm-min) {width: @container-sm;}
  @media (min-width: @screen-md-min) {width: @container-md;}
  @media (min-width: @screen-lg-min) {width: @container-lg;}
}
.container-fluid {
  .container-fixed();
}
```

容器只做了一个简单的操作，居中已经使用伪元素生成bfc环境。



## 栅格系统

Bootstrap 采用栅格系统，将容器内的内容分隔成 12 列，这样就可以很方便地为每个块分配空间，并且可以根据断点分配不同大小的的空间

+ `row`：在水平方向创建一组“列（column）”
+ `col-断点-空间数`：这个可以规定在不同的屏幕大小断点下占用的列数
+ `col-断点-offset-空间数`：通过这个样式可以指定当前元素向右偏移多少列，原理是使用`margin-x`的偏移原理
+ `col-断点-pull-空间数`：通过这个样式可以强制的控制元素的位置，向左边移动多少列，原理是使用`position`的定位属性`left,right`。
+ `col-断点-push-空间数`：通过这个样式可以强制的控制元素的位置，向右边移动多少列，原理是使用`position`的定位属性`left,right`。
+ `clearfix`：清除浮动，开启BFC，这个是很有用的样式，因为Bootstrap 的栅格系统是基于浮动`float:left`以及表格容器`display:table`实现的，所以清除浮动的样式是必须的。

栅格系统的部分源码

```less
//创建行容器
.row {
  .make-row();
}
.make-row(@gutter: @grid-gutter-width) {
  margin-left:  ceil((@gutter / -2));
  margin-right: floor((@gutter / -2));
  &:extend(.clearfix all);
}

//通过递归循环创建出全部断点全部列数的相关样式的样式类名，并设置基础样式
.make-grid-columns();
.make-grid-columns() {
  .col(@index) { 
    @item: ~".col-xs-@{index}, .col-sm-@{index}, .col-md-@{index}, .col-lg-@{index}";
    .col((@index + 1), @item);
  }
  .col(@index, @list) when (@index =< @grid-columns) {
    @item: ~".col-xs-@{index}, .col-sm-@{index}, .col-md-@{index}, .col-lg-@{index}";
    .col((@index + 1), ~"@{list}, @{item}");
  }
  .col(@index, @list) when (@index > @grid-columns) { 
    @{list} {
      position: relative;
      min-height: 1px;
      padding-left:  ceil((@grid-gutter-width / 2));
      padding-right: floor((@grid-gutter-width / 2));
    }
  }
  .col(1);
}

//递归循环创建出指定断点的全部列数的相关样式的样式类名，并设置宽度，pull，push，偏移offset的基本样式
.make-grid(xs);
.make-grid(@class) {
  .float-grid-columns(@class);
  .loop-grid-columns(@grid-columns, @class, width);
  .loop-grid-columns(@grid-columns, @class, pull);
  .loop-grid-columns(@grid-columns, @class, push);
  .loop-grid-columns(@grid-columns, @class, offset);
}
.float-grid-columns(@class) {
  .col(@index) { // initial
    @item: ~".col-@{class}-@{index}";
    .col((@index + 1), @item);
  }
  .col(@index, @list) when (@index =< @grid-columns) { // general
    @item: ~".col-@{class}-@{index}";
    .col((@index + 1), ~"@{list}, @{item}");
  }
  .col(@index, @list) when (@index > @grid-columns) { // terminal
    @{list} {
      float: left;
    }
  }
  .col(1); // kickstart it
}

.calc-grid-column(@index, @class, @type) when (@type = width) and (@index > 0) {
  .col-@{class}-@{index} {
    width: percentage((@index / @grid-columns));
  }
}
.calc-grid-column(@index, @class, @type) when (@type = push) and (@index > 0) {
  .col-@{class}-push-@{index} {
    left: percentage((@index / @grid-columns));
  }
}
.calc-grid-column(@index, @class, @type) when (@type = push) and (@index = 0) {
  .col-@{class}-push-0 {
    left: auto;
  }
}
.calc-grid-column(@index, @class, @type) when (@type = pull) and (@index > 0) {
  .col-@{class}-pull-@{index} {
    right: percentage((@index / @grid-columns));
  }
}
.calc-grid-column(@index, @class, @type) when (@type = pull) and (@index = 0) {
  .col-@{class}-pull-0 {
    right: auto;
  }
}
.calc-grid-column(@index, @class, @type) when (@type = offset) {
  .col-@{class}-offset-@{index} {
    margin-left: percentage((@index / @grid-columns));
  }
}

.loop-grid-columns(@index, @class, @type) when (@index >= 0) {
  .calc-grid-column(@index, @class, @type);
  // next iteration
  .loop-grid-columns((@index - 1), @class, @type);
}
```

通过阅读源码，会发现在容器的两边具有15px 的 padding ，行 两遍具有 -15px 的 margin，列两边具有 15px 的 padding。

为了能使槽宽的规则，列两边必须要有 15px 的 padding；为了能使列嵌套行，行两边必须要有 -15px 的 margin；为了让容器可以包裹行，容器两边必须要有15px 的padding。



## 响应式工具

响应式工具用于在不同屏幕的展示

+ `visible-断点-block`：控制元素在断点下以`display:block`的形式的可见
+ `visible-断点-inline`：控制元素在断点下以`display:inline`的形式的可见
+ `visible-断点-inline-block`：控制元素在断点下以`display:inline-block`的形式的可见
+ `visible-断点`：控制元素在断点在某断点时显示
+ `hidden-断点`：控制元素在断点在某断点时隐藏

上面显示的原理是通过控制`display`来进行控制的。 