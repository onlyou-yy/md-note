# canvas

canvas 是 h5 新增的东西，用于绘制图像，长久以来在web上运行的动画和图像都是以 flash 的形式出现的，但是 flash 不但使用要求高（需要安装adobe flash player），开发成本也高（需要使用 actionscript），漏洞多，卡顿，不流畅等等问题。

canvas 是一种轻量级的画布，使用 canvas 进行 JavaScript 的编程，不需要增加额外的插件，性能也很好，不卡顿，在手机中也很流畅，并且后续会支持 3D 场景（experimental-webgl）的开发。并且使用 canvas 也非常简单，因为 h5 中 canvas 只是一个标签`<canvas>` 而且属性要只有` width height`这两个主要属性和一些`id class style`等通用属性而已，但是 canvas 有个缺点就是在画布中的元素是不支持使用 DOM 操作的。



## 基本使用

要使用 canvas 首先就是要创建 canvas 标签，然后获取到 canvas 画布环境，所有的图像绘制都是通过 ctx 设置的，和canvas 没有太大关系。

```html
<canvas id="can" width="200" height="300">
    （IE6/7/8）浏览器暂不支持 canvas 功能</canvas>
<script>
	let can = document.querySelector("#can");
	let ctx = can.getContext("2d",{alpha:true});//alpha 画布背景是否透明，默认是透明的
</script>
```

**注意**：canvas 的 width 和 height 不要用 css 来设置，因为通过css设置的是指**canvas元素**的宽高，而 `<canvas id="can" width="200" height="300">` 中的宽高指的是**画布场景**的宽高，如果使用了，就会导致画布被缩放或者变形。

## canvas 特点

**像素化**：使用 canvas 绘制一个图形，一点绘制成功，canvas 就像素化了他们。所以我们是不能从画布上再次得到这个图形，也就是我们不能再操作它了。这个就是 canvas 比较轻量的原因。

因此如果我们需要让这个canvas 图形做动画效果，必须使用`清屏->更新->渲染`的步骤进行编程，总而言之就是需要重新再画一次。

```js
let can = document.querySelector("#can");
let ctx = can.getContext("2d");
let left = 0;
ctx.fillStyle = "blue";
let timeId = setInterval(()=>{
	ctx.clearRect(0,0,700,700);
	left++;
	if(left>=100){
		clearInterval(timeId);
	}
    ctx.fillRect(left,150,20,20);
},10);
```



## 绘制图形API

### **绘制矩形**

+ `ctx.fillRect(x,y,w,h)`绘制填充的一个矩形
+ `ctx.fillStyle = "red"`设置填充图形的颜色值，可以接受 颜色名，16位颜色值，rgb，rgba等颜色值
+ `ctx.strokeRect(x,y,w,h)`绘制一个矩形边框
+ `ctx.clearRect(x,y,w,h)`清除指定矩形区域，让清除部分完全透明
+ `ctx.rect(x,y,w,h)`绘制一个矩形，但是如果不用`ctx.fill()`填充颜色或者`ctx.stroke()`描边是看不到的。
	+ `ctx.fill()`有两种模式可以选择 ，**"evenodd"**不填充相交的部分，**nonzero**全填充（默认）

> 清屏的方式除了使用 `ctx.clearRect(x,y,w,h)`之外还可以使用`canvas.width = val`重新设置canvas 的宽来清除屏幕上的东西。

### **绘制路径**

+ `ctx.beginPath()`用于创建一个路径，并把之前的路径关闭
+ `ctx.moveTo(x,y)`移动路径起点到x,y
+ `ctx.lineTo(x,y)`从上一个路径点移动到 x, y
+ `ctx.closePath()`关闭路径绘制，如果画的路径是没有闭合的是用这个方法会将起点和终点连线闭合
+ `ctx.stroke()`描边
+ `ctx.strokeStyle="red"`设置描边颜色

画好**路径**后还需要使用`ctx.stroke()`描边之后路径才会显示出来。也可以同是使用`ctx.fill()`来填充不规则图形。

### **绘制圆弧**

+ `ctx.arc(x,y,r,startRadian,endRadian,isInverse)`绘制一个**圆弧路径**，`ctx.arc(200,200,50,0,1,false)`在**圆心**为(200,200)处，在**弧度**0开始到1顺时针画一个**半径**为 50的圆弧。`0 <= startRadian|endRadian <= 2π`，需要注意：半径必须 大于 0

> **弧度=(Math.PI/180)\*角度。**

## 透明度

+ `ctx.globalAlpha = 1`设置绘画的透明度也可以通过设置颜色的`rgba`来设置透明度。

## 线型

+ `ctx.lineWidth = 10`设置线的宽度
+ `ctx.lineCap = "butt"|"round"|"square"`设置线端点样式，分别为矩形（默认），圆角，矩形（但是会在两端增加一个矩形区域）。
+ `ctx.lineJoin = "bevel"|"round"|"square"`设置线与线之间连接点的样式，平角，圆角，矩形（默认）；
+ `ctx.setLineDash([线长,间隔....])`设置描边线为虚线，并且设置可以设置单个虚线的长度与间隔，可以设置多个，每两个为一组。
+ `ctx.lineDashOffset = 10`设置虚线便宜量，> 0 时表示的是想左便宜， < 0 时表示的向右偏移
+ `ctx.isPointInPath(x,y)`判断点是否在路径上，也可以用来判断是否命中一个图形，但是不能判断是否命中图片
+ `ctx.isPointInStroke(x,y)`判断点是否在图像的描边上。

## 绘制文本

+ `ctx.fillText(txt,x,y)`将文本txt，在 x,y 处绘制出来
+ `ctx.strokeText(txt,x,y)`将文本txt，在 x,y 处绘制出来，是描边文本
+ `ctx.font = "10px sans-serif"`设置文本字体样式，可以用来设置字体大小
+ `ctx.textAlign = "start"|"left"|"right"|"center"|"end"`设置文本的对其方式，默认是 start，需要注意的是 这个的对其是相对 x 而言的，也就是说 如果是 center 那个文本将会向左便宜50%，使 x 位于中间。
+ `ctx.textBaseline ="top"|"hanging"|"middle"|"alphabetic"|"ideographic"|"bottom"` 基线对齐选项
+ `ctx.direction ="ltr"|"rtl"|"inherit"`文本方向
+ `ctx.measureText('foo')`将返回一个 TextMetrics 对象的宽度、所在像素，这些体现文本特性的属性

## 渐变

+ `ctx.createLinearGradient(x1,y1,x2,y2)`由点（x1，y1） 到点（x2，y2）创建一个具有方向的线性渐变区域，只有在这个区域上的图才应用到。返回 Gradients 对象。这个对象可以用做 fillStyle 和 strokeStyle 的值。
+ `ctx.createRadialGradient(x1,y1,r1,x2,y2,r2)`前三个定义一个以 (x1,y1) 为原点，半径为 r1 的圆，后三个参数则定义另一个以 (x2,y2) 为原点，半径为 r2 的圆。这个可以在三维空间上理解。
+ `Gradients.addColorStop(persent,color)`，定义渐变的颜色占比，persent 取值范围为 0~1 ，表示在渐变方向上的百分之几由color 颜色开始渐变。

![image-20210422180351831](canvas/image-20210422180351831.png)

> 填充还可以使用图片进行。`ctx.createPattern()`可以创建一个贴图器，同样返回的值也可以给 `ctx.fillStyle`使用
>
> ```js
> function draw() {
>     var ctx = document.getElementById('canvas').getContext('2d');
>     // 创建新 image 对象，用作图案
>     var img = new Image();
>     img.src = 'https://mdn.mozillademos.org/files/222/Canvas_createpattern.png';
>     img.onload = function() {
>        // 创建图案
>        var ptrn = ctx.createPattern(img, 'repeat');
>        ctx.fillStyle = ptrn;
>        ctx.fillRect(0, 0, 150, 150);
>     }
> }
> ```



## 阴影

+ `ctx.shadowOffsetX = 10`设阴影x轴的偏移量
+ `ctx.shadowOffsetY = 10`设阴影y轴的偏移量
+ `ctx.shadowBlur = 10`设阴影的模糊度
+ `ctx.shadowColor = "red"`设阴影的颜色



## 使用图片

`ctx.drawImage()`方法会根据不同的参数数量起到不同的作用。这个方法也经常配合`canvas.toDataURL("image/jpeg", 质量(0~1));`被用于**前端图片压缩**。

+ `ctx.drawImage(imgObj,x,y)`将图片加载在 x，y 处。
+ `ctx.drawImage(imgObj,x,y,w,h)`将图片加载在 x，y 处，并设置宽高，会缩放。`ctx.imageSmoothingEnabled = false;`可以使用这个控制是否开启平滑缩放
+ `ctx.drawImage(imgObj,x,y,w,h,clipX,clipY,clipW,clipH)`前4个是定义图像源的切片位置和大小，后4个则是定义切片的目标显示位置和大小。将图片进行切片。

### 图片跨域

**imgObj** 可以是 `Image`对象`new Image()`（但是在获取数据的时候可能会出现跨域的问题，这时候可以为img添加`img.crossOrigin = 'anonymous';`属性解决跨域问题），也可以是`HTMLImageElement`(`<img>`)，`HTMLVideoElement`(`<video>`)，`<canvas>`对象。

[一个关于image访问图片跨域的问题](https://juejin.cn/post/6844903795726483463)

[Image跨域问题](https://www.jianshu.com/p/40567c0500ba)

> `ImageBitmap`是一个高性能的位图，可以低延迟地绘制，它可以从上述的所有源以及其它几种源中生成。
>
> ```js
> const imageBitmapPromise = createImageBitmap(image[, options]);
> const imageBitmapPromise = createImageBitmap(image, sx, sy, sw, sh[, options]);
> ```
>
> `image`： 一个图像源, 可以是一个`<img>`, `SVG `，`<image>`，`<video> `，`<canvas>`，`HTMLImageElement`，`HTMLVideoElement`，`HTMLCanvasElement`，`Blob`，`ImageData`，`ImageBitmap`，`OffetscreenCanvas`对象
>
> sx：裁剪点x坐标.
>
> sy：裁剪点y坐标.
>
> sw：裁剪宽度,值可为负数.
>
> sh：裁剪高度,值可为负数.



## 状态保存和恢复

- `ctx.sava()`可以将当前的环境状态保存下来，保存的方式是栈，也就是说恢复的时候是出栈的方式恢复的。
  - 可保存的状态有：应用的变形，strokeStyle，fillStyle，globalAlpha，lineWidth，lineCap，lineJoin，miterLimit，lineDashOffset，shadowOffsetX，shadowOffsetY，shadowBlur，shadowColor，globalCompositeOperation，font，textAlign，textBaseline，direction，imageSmoothingEnabled，当前的裁切路径（clipping path）
- `txt.restore()`恢复上一次保存的状态，将状态栈做出栈操作。



## 变形

- `ctx.translate(x,y)`移动的是整个 canvas 的坐标系到一个不同的位置。需要注意的是如果你是在一个循环中做位移但没有保存和恢复 canvas 的状态，很可能到最后会发现怎么有些东西不见了，那是因为它很可能已经超出 canvas 范围以外了。
- `ctx.rotate(弧度)`将整个 canvas 的坐标系按原点旋转。
- `ctx.scale(sx,sy)`方法可以缩放画布的水平和垂直的单位。
- `ctx.transform(a, b, c, d, e, f)`这个是上面的几个机集合，传入的是一个变换矩阵。在条件允许的时候可以尽量使用这个方法进行变形，因为这个方式可以**开启GPU加速**
	- a：水平缩放。
	- b：垂直倾斜。
	- c：水平倾斜。
	- d：垂直缩放。
	- e：水平移动。
	- f：垂直移动。



## 合成

+ `ctx.globalCompositeOperation = type`这个方法可以设置图像之间的合成模式，类似于PS的图层合并方式。[在这里查看类型和效果](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Compositing)。利用这个强大的特性可以用来做ps的图层合并效果等等。比如使用`destination-out`就可以轻松实现刮刮卡效果。



## 裁剪与遮罩

+ `ctx.clip()`这个方法的作用和`ctx.fill() ctx.stroke()`很像，都是对图像的一种填充方式，`fill()`是填充整个图像，`stroke()`是对图像进行描边，而`cilp()`则是裁剪图像，准确来说应该是创建一个遮罩，只用在遮罩内的内容才会被看见。



## 图像操作

+ `ctx.createImageData(w,h)`创建一个宽度为 w 高度为 h 的 ImageData 对象，也可以直接通过 ImageData 对象克隆一个新的 ImageData 对象 ctx.createImageData(imageData)
+ `ctx.getImageData(x,y,w,h)`获取画布上某一区域上的 ImageData 对象
+ `ctx.putImageData(imagedata, dx, dy[, dirtyX, dirtyY, dirtyWidth, dirtyHeight])`将 ImageData 源图像中点（dirtyX, dirtyY）宽高为（dirtyWidth, dirtyHeight）的区域在画布的（dx, dy）绘制出来。

> ImageData ：这个对象中有三个属性 {width，height，data}，其中data就是该图像的像素描述（数字数组，每四个为一组分别表示为 r，g，b，a），**是可以被遍历修改的**，通过修改这个数组中的数据可达到控制像素的效果。最后可以通过 putImageData 方法将图像输出到屏幕上。
>
> imageData.data 是**Unit8ClampedArray**类型的数据，是一个包含图片上所有像素点数据的**一维数组**，除了直接修改外，还可以使用`imageData.data.set(arr)`方法来设置。



## Canvas动画

因为Canvas 像素话的特点，图像在被绘制到画布上之后就不能被操作了，所以想要实现Canvas的动画的话，就需要按照`清屏->更新->渲染`的步骤重复改变画布上的内容才行。这样的话就可以选择使用循环、setTimeout，setInterval、requestAnimationFrame(fn) （fn接收一个参数，是执行的时间）方法实现重复渲染更新了。

> #### requestAnimationFrame(fn) 比起 setTimeout、setInterval的优势主要有两点：
>
> 1. requestAnimationFrame 会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率，一般来说，这个频率为每秒60帧。
> 2. 在隐藏或不可见的元素中，requestAnimationFrame将不会进行重绘或回流，这当然就意味着更少的的cpu，gpu和内存使用量。

同时在做比较复杂的动画的时候可以考虑将动画进行分层，然后采用多个 canvas 画布来进行重叠起来形成一个完整的动画效果，这样不仅能大大减低开发难度，而且也提升了性能。



## canvas，file，blob，DataURL 之间的转换

+ `canvas.toDataURL("image/jpeg",quality)`可以将 canvas 装换为 DataURL 对象，这个对象表示的是一张 jpeg 格式的图片，quality 表示转换品质（0-1）；
+ `canvas.toBlob(callback, type, encoderOptions);`可以将 canvas 装换为 Blob 对象。type 指定图片格式，默认格式为`image/png`。encoderOptions 值在0与1之间，当请求图片格式为`image/jpeg`或者`image/webp`时用来指定图片展示质量。

**canvas 转 DataURL** :`canvas.toDataURL("image/jpeg",quality)`

**canvas 转 Blob**：`canvas.toBlob(callback, type, encoderOptions)`

**file / Blob 转 DataURL**：file对象其实也是blob对象，所以两者转换为dataURL的方法一样：

```js
let reader = new FileReader();
reader.onload = (e) => {
    console.log(e.target.result);
}
reader.readAsDataURL(blob|file);
```

**DataURL 转 Blob / File**：由于 DataURL 的格式是`data:text/plain;base64,YWFhYWFhYQ==`这样的，前面的`data:text/plain;base64`表示的数据的类型，后面的则是数据的编码内容，可以通过`new Blob([u8arr], {type:mime});`来创建 Blob 的数据

> **Uint8Array** 数组类型表示一个8位无符号整型数组，创建时内容被初始化为0。创建完后，可以以对象的方式或使用数组下标索引的方式引用数组中的元素。

```js
function dataURLtoBlob(dataurl) {
    let arr = dataurl.split(',');
    let mime = arr[0].match(/:(.*?);/)[1];
    let bstr = atob(arr[1]);
    let n = bstr.length; 
    let u8arr = new Uint8Array(n);
    while(n--){
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new Blob([u8arr], {type:mime});
}

function dataURLtoFile(dataurl, filename) {
    let arr = dataurl.split(',');
    let mime = arr[0].match(/:(.*?);/)[1];
    let bstr = atob(arr[1])
    let n = bstr.length;
    let u8arr = new Uint8Array(n);
    while(n--){
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new File([u8arr], filename, {type:mime});
}
```

**DataURL，file，Blob 图片转 canvas**：可统一将 file | Blob 转化为 DataUrl 之后再通过 `ctx.drawImage(img)`进行转化

```js
let img = new Image();
img.onload = function(){
    ctx.drawImage(img,0,0);
}
img.src = DataURL;
```



## **解析 Blob**

可以通过 `URL.createObjectURL(File|Blob|MediaSource)`解析为`DOMString`，但是**要记得**使用`URL.revokeObjectURL()`释放资源。

> [JS中的Blob对象](https://www.jianshu.com/p/b322c2d5d778)
>
> [理解DOMString、Document、FormData、Blob、File、ArrayBuffer数据类型](https://blog.csdn.net/hefeng6500/article/details/99481636)



在实际个开发过程中其实是很少会自己写原生的 canvas 来实现效果，除了前端压缩已以及做简单水印可以自己写之外一般都会使用 canvas 的库来实现，web前端使用到 canvas 地方一般都是图表，而做图表的库可以使用`echarts.js`，如果要使用 canvas 做图像开发的话可以选择`Knova`或者[Fabric.js](http://jspang.com/detailed?id=20)



## 将HTML绘制到Canvas

HTML 页面虽然不能直接绘制到 Canvas 上，但是SVG可以转成图片并绘制到canvas上，而且 SVG 是可以通过`foreignObject`来内嵌 html 代码，所以我们只需要在 SVG 上内嵌自己的HTML 代码，之后再将SVG 转化成图片之后绘制到Canvas上即可

```html
<svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
  <style>
    img{
      border-radius:50%;
    }
  </style>
  <foreignObject x="20" y="20" width="160" height="160">
    <div xmlns="http://www.w3.org/1999/xhtml">
      <img class="showImg" src="https://picsum.photos/seed/picsum/200/300" width="100" height="100">
      时代峰峻哦啊睡觉哦发送的地方怕送发票开店铺上的佛啊睡觉哦的佛阿斯顿发
      Lorem ipsum dolor sit amet consectetur adipisicing elit. Neque cum exercitationem quia provident molestias, omnis perspiciatis rerum ducimus. Sint ab ullam cupiditate molestias? Consequatur itaque soluta perferendis sapiente aspernatur ad.
    </div>
  </foreignObject>
</svg>
```

> 需要注意 图片链接协议转化成 DataURL

```js
fetch('https://picsum.photos/seed/picsum/200/300?'+ Date.now())
  .then(response => response.blob())
  .then(data => {
  let fileReader = new FileReader();
  fileReader.onloadend = function(){
    let imgs = document.querySelectorAll(".showImg")
    for(let img of imgs){
      img.src =fileReader.result;
    }
    covertSVG2Image(document.querySelector("svg"),"fff",300,300,"jpg")
  }
  fileReader.readAsDataURL(data)
});
function covertSVG2Image(node, name, width, height, type = 'png'){
  let serializer = new XMLSerializer()
  let htmlString = serializer.serializeToString(node);
  let source = '<?xml version="1.0" standalone="no"?>\r\n' + htmlString
  let image = new Image()
  image.src = 'data:image/svg+xml;charset=utf-8,' + encodeURIComponent(source)
  let canvas = document.createElement('canvas')
  canvas.width = width
  canvas.height = height
  let context = canvas.getContext('2d')
  context.fillStyle = '#fff'
  context.fillRect(0, 0, 10000, 10000)
  image.onload = function () {
    context.drawImage(image, 0, 0)
  }
  document.body.appendChild(canvas)
}
```



## 用Canvas生成浏览器指纹

通过 Canvas 代码生成的唯一像素会因使用的系统和浏览器不同而不同，也就是说只要在同一设备上相同浏览器生成的出来的像素数据是相同的，这个相同的数据可以作为用户的一种标识，即使用户频繁更换账号，依然可以知道同一个人。

```js
const canvas = document.createElement("canvas");
function bin2hex(s) {
  var i,l,n,o = "";
  s += "";

  for (i = 0, l = s.length; i < l; i++) {
    n = s.charCodeAt(i).toString(16);
    o += n.length < 2 ? "0" + n : n;
  }

  return o;
}
const b64 = canvas.toDataURL().replace("data:image/png;base64,", "");
// window.atob 用于解码使用 base-64 编码的字符串
const bin = atob(b64);
const crc = bin2hex(bin.slice(-16, -12));
```

还有很多中方式可以生成浏览器指纹，具体可以在 https://browserleaks.com/ 查看相关的内容。

[基于 H5 Canvas "指纹识别" 技术 【浏览器指纹 VS Canvas指纹】](https://juejin.cn/post/7032988392480571406)



## 其他问题

### **Html2canvas 图片跨域问题**

第一种：后端需要在服务器IIS上的HTTP响应标头设置，最简单粗暴的方法就是全部设置成*，不过这样安全性也低，自己可以根据自己需求设置：

```js
access-control-allow-credentials:true
Access-Control-Allow-Headers:*
access-control-allow-origin:*
```

第二种：在img标签上设置 `crossorigin="anonymous"` 属性，但是这样还是不行的，还是会报跨域问题，最后在src图片地址后面加上时间戳

```html
<img :src="staffQrCodeDialogBg1 + '?' + new Date().getTime()" alt="" crossOrigin="anonymous" />
```

如果还是不行可以将 图片下载下来之后 转化成 DataURL

```js
let img = new Image();
img.crossOrigin = "anonymous"
img.onload = function(){
  let canvas = document.createElement("canvas");
  canvas.width = img.naturalWidth;
  canvas.height = img.naturalHeight;
  let ctx = canvas.getContext("2d");
  ctx.drawImage(img,0,0);
  let url = canvas.toDataURL("image/png");
  imgEl.src = url;
}
```

### **保存的图片比较模糊**

如果保存的图片模糊，大概率是因为在绘制的时候尺寸进行了压缩，可将要保存的元素的 图片（是img元素，不是保存下来的图片）的尺寸设置得大点，这样就没那么容易模糊

### 绘制道canvas的图片模糊

图片有个属性`naturalWidth naturalHeight`表示图片的原始尺寸，这是是图片的真实大小，而通过css设置的`width height`是样式尺寸，当样式尺寸和原始尺寸不一样的时候，图片就会被缩放，从而导致绘制到canvas的图片变模糊，还有当页面缩放的时候即使样式尺寸和原始尺寸一致也可能回出现模糊的情况（实际上只有当 devicePixelRatio 不为 1 的时候才容易模糊），要解决这个问题就需要保持原始尺寸和样式尺寸一致

```
原始尺寸=样式尺寸x缩放倍率(window.devicePixelRatio)
```

devicePixelRatio 受页面缩放和设备的影响，所以即使页面没有缩放也可能不是 1。

同样的 canvas 也是一个图片，而`canvas.width canvas.heigth`就是他的原始尺寸，因为样式尺寸会随着页面缩放而变化（不会表现在devtool上，），所以在样式尺寸固定的时候，canvas的原始尺就需要乘上一个`devicePixelRatio`，包括之后画的元素的尺寸也是。

https://blog.csdn.net/harry_yaya/article/details/105964081

https://juejin.cn/post/7067415002289799205

### ios上偶尔会出现空白

如果你的页面有多个canvas并且在window，android浏览器上渲染一切正常，但是在ios/macos系统的浏览器上运行有时候会渲染不出来，那很可能会cancas大小超出了浏览器的内存限制

[从一个 bug 中延伸出 canvas 最大内存限制和浏览器渲染原理](https://zhuanlan.zhihu.com/p/540761999)



## 参考

[canvas 多个图形可视化操作:拖拽、缩放、旋转](https://juejin.cn/post/6894433110491201543)

[制作一个画板来学习canvas](https://juejin.cn/post/6920397865873309704)

[fabricjs](http://fabricjs.com/)

[canvas绘制图片，图片变模糊](https://blog.csdn.net/harry_yaya/article/details/105964081)

[Canvas 交互](https://www.jianshu.com/p/1fc2282ca15e)



## 例子以及应用

### **刮刮卡**

### **时钟**

### **发射小球**

### **小球连线**

### **弹幕**

### **画板**

### **物理小球**

### **小球跟随**

### **图片压缩**

### **图片裁剪**

### **图片加水印**

