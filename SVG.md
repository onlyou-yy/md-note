# SVG

svg 是 一种基于 XML 语法的图像格式，全程是 **可缩放矢量图（scalable vector graphics）** ，其他图像是基于像素处理的，SVG 则是属于对图像的形状描述，所以它的本质还是文本，体积较小，且不管放大多少倍都不会失真，也由于它是一种 XML 语法的图像格式，也就是一种结构化的文本格式，其实就是和 HTML 相差不多，因此 SVG 的图像元素也可以像HTML中的元素一样可以被 DOM 操作。

当然，即使绘制一个简单的 SVG 图代码可以也会是非常复杂的，所以一般都是使用[在线工具](https://c.runoob.com/more/svgeditor/)直接生成 SVG 图，或者在 [iconfont](https://www.iconfont.cn/) 这种库直接下载，也可以使用 AI ，figma 这种制图软件进行制作。

## SVG 标签属性

`<svg>`标签是一个容器，所有的 svg 绘图标签都需要被 `<svg>`包裹才能生效，相当与告诉浏览器其中的代码按 SVG 规范解析。

```html
<svg width="100%" height="100%" version="1.1" viewbox="0 0 100 100"
xmlns="http://www.w3.org/2000/svg">

<circle cx="100" cy="50" r="40" stroke="black"
stroke-width="2" fill="red"/>

</svg>
```

属性：

+ width：画布的宽度
+ height：画布的高度
+ viewBox：定义画布要显示的区域`"0 0 100 100"`分别表示，x坐标，y坐标，宽度，高度。需要主要的是当前区域是会被放大为整个画布那么大
+ xmlns：表示定义了命名空间

如果这个 svg 是以`.svg`的形式存在的话可以在前面加上

```html
<?xml version="1.0" standalone="no"?>

<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" 
"http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
```

第一行包含了 XML 声明。请注意 standalone 属性！该属性规定此 SVG 文件是否是“独立的”，或含有对外部文件的引用。

standalone="no" 意味着 SVG 文档会引用一个外部文件 sss- 在这里，是 DTD 文件。

第二和第三行引用了这个外部的 SVG DTD。该 DTD 位于 “http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd”。该 DTD 位于 W3C，含有所有允许的 SVG 元素。



## 在HTML 中使用 svg 

svg可以直接插入在 HTML 中，也可以当成是普通的图片一样由`<img src="test.svg" />`进行引入，也可以使用`<embed>、<object> 、 <iframe>`进行引入使用如

```html
<embed src="rect.svg" width="300" height="100" 
type="image/svg+xml"
pluginspage="http://www.adobe.com/svg/viewer/install/" />

<object data="rect.svg" width="300" height="100" 
type="image/svg+xml"
codebase="http://www.adobe.com/svg/viewer/install/" />

<iframe src="rect.svg" width="300" height="100">
</iframe>
```



## 绘制图形

svg 中的形状标签有 `矩形 <rect>,圆形 <circle>,椭圆 <ellipse>,线 <line>,折线 <polyline>,多边形 <polygon>,路径 <path>`

这么多种形状标签他们都有一些共同的属性，如`x（x坐标）,y（y坐标）,width,height,fill（填充颜色）,stroke（描边颜色）,stroke-width（描边宽度）`[等等](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute)，而且形状标签的属性可以直接写在标签中，也可以写在样式中，完全可以件svg 中的图像标签可做是普通的HTML 标签看待。

### 文本`<text>`

普通文本：

```html
<svg>
  <text x="0" y="15" fill="red">I love SVG</text>
</svg>
```

按路径显示文本

```html
<svg>
   <defs>
    <path id="path1" d="M75,20 a1,1 0 0,0 100,0" />
  </defs>
  <text x="10" y="100" style="fill:red;">
    <textPath xlink:href="#path1">I love SVG I love SVG</textPath>
  </text>
</svg>
```

文本块

```html
<svg>
  <text x="10" y="20" style="fill:red;">Several lines:
    <tspan x="10" y="45">First line</tspan>
    <tspan x="10" y="70">Second line</tspan>
  </text>
</svg>
```



### 矩形 `<rect>`

```html
<svg>
	<rect x="20" y="20" width="100" height="100" fill="red" stroke-width="10" stroke="black"></rect>
</svg>
```

```html
<style>
#rect1{
	width:100px;
	height:100px;
	fill:red;
	stroke-width:10px;
	stroke:black;
	x:20px;
	y:20px;
}
</style>
<svg>
	<rect id="rect1"></rect>
</svg>
```

### 圆形`<circle>`

```html
<svg width="100%" height="100%">
	<circle x="20" y="20" cx="100" cy="50" r="40" stroke="black"
stroke-width="2" fill="red"/>
</svg>
```

`cx cy`：圆心坐标

`r`：半径长，单位是像素 px

### 椭圆`<ellipse>`

```html
<svg width="100%" height="100%">
	<ellipse cx="300" cy="150" rx="200" ry="80"
style="fill:rgb(200,100,50);
stroke:rgb(0,0,100);stroke-width:2"/>
</svg>
```

`rx`：横向半径长

`ry`：纵向半径长

### 直线`<line>`

```html
<svg width="100%" height="100%">
	<line x1="0" y1="0" x2="300" y2="300"
style="stroke:rgb(99,99,99);stroke-width:2"/>
</svg>
```

`x1 y1`：起始坐标位置，`x2 y2`：结束点坐标

### 多边形`<polygon>`

```html
<svg width="100%" height="100%">
	<polygon points="220,100 300,210 170,250"
style="fill:#cccccc;
stroke:#000000;stroke-width:1"/>
</svg>
```

`points`：坐标点集，`x y` 以`,`相隔，两坐标之间用` `相隔。结束时会将第一个坐标和最后一个坐标连接闭合。

### 折线`<polyline>`

```html
<svg width="100%" height="100%">
	<polyline points="0,0 0,20 20,20 20,40 40,40 40,60"
style="fill:white;stroke:red;stroke-width:2"/>
</svg>
```

`points`：坐标点集，`x y` 以`,`相隔，两坐标之间用` `相隔。结束时第一个坐标和最后一个坐标不会闭合。

### 路径`<path>`

```html
<svg width="100%" height="100%" version="1.1"
xmlns="http://www.w3.org/2000/svg">
<path d="M250 150 L150 350 L350 350 Z" />
</svg>
```

`d`：绘制命令集

命令有很多，但是一般常用的就 M L H V Z 这几个

> M：moveTo  移动起点到某一点的意思
>
> L：lineTo 划线到哪一点的意思
>
> H：horizontal lineto 水平划线到哪一点
>
> V ： vertical lineto 垂直划线到哪一点
>
> Z：闭合路径



## 线型

一般没人的 stroke 描边样式都是方形的，使用 `stroke-dasharray` 等属性改变线型。

+ `stroke-width="10"`控制线的宽度
+ `stroke-linecap="round|butt|square"`控制线两端的形状 round 圆形，butt 方形（默认），square 增加方形``
+ ``stroke-dasharray="10,15,10,5"`设置线为虚线，没两值为一组，分别表示 `每段虚线长,虚线的间距`
+ `stroke-linejoin="miter|round|bevel"`描边转角的表现方式 miter（默认）方形，round 圆形，bevel 增加方形
+ `stroke-dashoffset="100"`虚线起点相对于路径起点的偏移量。

stroke-dasharray和stroke-dashoffset相结合可以做出很炫酷的效果

```html
<style>
 #shape {
     stroke-width: 6px;
     fill: transparent;
     stroke: #009FFD;
     stroke-dasharray: 85 400;
     stroke-dashoffset: -220;
     transition: 1s all ease
    }
    svg:hover #shape {
        stroke-dasharray: 70 0;
        stroke-width: 3px;
        stroke-dashoffset: 0;
        stroke: #06D6A0
    }
</style>
<svg height="40" width="150">
    <rect id="shape" height="40" width="150" />
</svg>
```



## 容器，渐变，动画，滤镜

[容器，渐变，动画，滤镜](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/defs)