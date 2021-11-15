# JQuery（JavaScript的库）

## 使用jquery

### 下载

```tex
直接下载
https://jquery.com/download/

使用cdn
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.0/jquery.js"></script>

使用npm
npm i jquery -S
```



## DOM与jq对象的转换

```js
///dom-->jq
$(dom元素)

//jq-->dom
$(选择器)[index];
//eg
$('div')[0];
$('div')[1];
```



## js的onload和jq的ready的区别

+ js的onload在加载DOM元素和图片资源加载完毕之后才会执行

+ jq的ready在加载完DOM之后就会执行，不会等待图片资源加载

+ onload事件多次绑定，后面的会覆盖前面的

+ ready事件会依次执行绑定的事件

+ 写法

```js
//onload
window.onload=function (){
...
}

//ready
$(document).ready(function(){})
//或者
$(function(){});
```



## jq冲突问题

当多个不同的插件都使用了`$`符号做代表符就会出现冲突，不知道`$`到底代表的是什么。这时候就需要将`$`的操作权释放给其他的插件。

```js
jQuery.onConflict();
jQuery(function(){});
```

注意点：释放操作必须写在jq代码之前，释放后不能再使用`$`，改为使用jQuery。但是同样可以自定义访问符号

```js
var jq=jQuery.onCflict();
jq(function(){});
```



## jq核心函数	`$()`

核心函数`$()`，可以接收的参数

1. 接收一个函数：入口函数

2. 接收一个字符串

   2.1 选择器：`$('div')` 选择dom中的div元素

   ​		如果通过这种方式为元素添加事件，并且选择到的元素不止一个就是会遍历为		每个元素都添加事件

   2.2 html标签字符串：`$("<p>一个p</p>")` 创建一个p的dom元素

3. dom元素：将dom元素包装成jq对象



## jq对象

jq对象的本质其实是一个伪数组（拥有 length 属性，其它属性（索引）为非负整数，且不具有数组所具有的方法）



## jq静态方法

1.遍历

   ```js
   var arr=[1,2,3,5,6];
   var obj={0:1,1:2,2:3,length:3}
   //arr.forEach(function(item,index));数组的这个forEach函数只能遍历数组，但是不能遍历伪数组。
   //$.each();既可以遍历数组也可以遍历伪数组。
   $.each(arr,function(index,item){
     console.log(index,val)
   })
   
   //$.map()和$.each()一样既可以遍历数组也可以遍历伪数组。并且map函数返回一个新的数组，并且可以在函数中控制要返回的内容
   var res=$.map(arr,function(index,item){
     return item+'1';
   })
   ```

2.去除字符串前后的空格

   ```js
   var str=" sss "
   var s=$.trim(str);
   ```

3.判断是否是widow对象

   ```js
   $.isWindow(window);//true;
   $.isWindow([1,2,3]);//false;
   ```

4.判断是够是数组

   ```js
   $.isArray(window);//false;
   $.isArray([1,2,3]);//true;
   ```

5.判断是否是函数

   ```js
   $.isFunction(function(){});//true;
   $.isFunction($);//true;
   $.isFunction([1,2,3]);//false;
   ```

6.暂停入口函数执行

   ```js
   $.holdReady(true);//暂停
   $.holdReady(false);//继续
   $(function(){
     alert('alert')
   })
   ```



## jq选择器

和css选择器差不多，但是比css选择器要多。

```js
$("div:contains('我的')");匹配包含给定文本的元素
$("div:empty");匹配所有不包含子元素或者文本的空元素
$("div:has(p)");匹配含有选择器所匹配的元素的元素
$("div:parent");匹配含有子元素或者文本的元素
```

具体参考[jQuery中文文档](http://jquery.cuishifeng.cn/id.html)



## jq属性操作

1. 什么是属性：

> 对象身上保存的变量就是属性

2. 如何操作属性

> 对象.属性名=值
>
> 对象[属性名]=值
>
> 对象.属性
>
> 对象[属性]

3. 什么是属性节点

> `<span name="code">code</span>`
>
> 在html代码中添加的属性就是属性节点

4. 操作属性节点

> DOM元素.setAttribute(“属性名”，值);
>
> DOM元素.getAttribute(“属性名”);

5. 属性和属性节点之间的区别

> 任何对象都有属性，只用dom对象操作属性节点

6. jq中操作属性节点

> 设置：`$(dom).attr(‘属性名’,值);`设置全部匹配到的元素的属性节点
>
> 获取：`$(dom).attr(‘属性名’);`只会获取第一个匹配的元素的属性节点
>
> 移除：`$(dom).removeAttr(‘属性名1 属性名2’);`移除匹配到的没有元素的属性1和属性2

这种放方式操作属性节点是在dom对象的attr内的属性

7. jq操作属性

> 设置：`$(dom).prop(‘属性名’,值);`设置匹配到的第一个元素的属性
>
> 获取：`$(dom).prop(‘属性名’);`只会获取第一个匹配的元素的属性
>
> 移除：`$(dom).removeProp(‘属性名1 属性名2’);`移除匹配到的没有元素的属性1和属性2

这只放操作的是dom元素的属性

注意点：

+ prop不仅能操作属性，而且可以操作属性节点

```js
$('div').prop('class','box');
```

+ 在要操作具有true和false两个属性的属性节点，如checked，selected或者disabled时使用prop(),其他用attr()



## jq操作类

1. 添加类，`dom元素.addClass('类名1 类名2');`

   ```html
   <style>
     .red{color:red;}
     .blue{color:blue;}
     .yellow{color:yellow;}
   </style>
   <span>我要变色了</span>
   <button>
     变色
   </button>
   <script>
   let color={
     0:"red",
     1:"blue",
     2:"yellow"
   }
   $('button').click(function(){
     let key=Math.floor(Math.rand()*3);
     $('span').addClass(color[key]);
   })
   </script>
   ```

   

2. 移除类`dom元素.removeClass('类名1 类名2');`

3. 切换类`dom元素.toggleClass('类名1 类名2');`有就删除没有就添加



## jq操作文本值

```js
//获取
$(dom).html();//获取到的是整个元素节点
$(dom).text();//获取到的是文本节点
$(dom).val();//获取到的dom元素的value属性节点的值，没有就是undefined

//设置
$(dom).html(值);//值可以是文本也可以是html代码
$(dom).text(值);//值只能是文本
$(dom).val(值);//值是文本
```



## jq操作css样式

```js
//设置
$(dom).css('width','100px');
$(dom).css('width','100px').css('height','100px');
$(dom).css({width:'100px',height:'100px'})

//获取
$(dom).css(属性名);
```



## jq操作位置和尺寸

```js
//获取位置和尺寸
$(dom).width();宽
$(dom).height();高
$(dom).offset().left;左边距
$(dom).offset().top;上边距

//获取position,不能设置
$(dom).position().left;
//可以通过操作css或者style进行设置
$(dom).css({left:'10px'})

//设置
$(dom).width('100px');宽
$(dom).height('100px');高
$(dom).offset({left:'10px',top:'10px'})

//获取scroll相关属性
//容器需要有宽度和高度，并且需要设置overflow属性
//获取页面中元素内容滚出的距离
$(dom).scrollTop();

//设置
$(dom).scrollTop(300);

//获取网页滚动的距离，要解决浏览器兼容问题
$("html").scrollTop()+$("body").scrollTop();
//设置
$("html,body").scrollTop(300);
```



## jq事件操作

```js
//绑定事件
//$(dom).事件名(fn);
$(dom).click(fn);//有些js的事件无法通过这种形式绑定
$(dom).on(fn);//可以绑定任意事件

//绑定的事件不会覆盖，并且同一元素可以绑定多个事件

//事件解绑
$(dom).off();//解绑所有事件
$(dom).off('click');//移除所有click类型的事件
$(dom).off('click'，test);//移除类型为click，方法为test的指定事件
```

**阻止事件冒泡**
可以在子元素中使用 return false 或者使用e.stopPropagation();

**阻止默认行为**
可以在目标元素的事件中用return false 或者e.preventDefault();

**自动触发事件**

```js
$(dom).trigger('click');
$(dom).triggerHandler('click');

//区别
//trigger会触发事件冒泡
//trigger会触发默认行为
//triggerHandler不会触发事件冒泡
//triggerHandler不会触发默认行为
```

**自定义事件**

```js
$(dom).on('事件名',fn);
$(dom).trigger('事件名');
```

事件触发和自定义事件原理：

[javascript事件触发器fireEvent和dispatchEvent](https://www.cnblogs.com/tiger95/p/6962059.html)

**事件命名空间**

当多个人对同一项目的同一元素添加同一事件的时候，需要对事件进行区分，确定要触发的是谁定义的事件，这时候就需要使用命名空间了

```js
$(dom).on('click.zs',fn);//这是张三定义的事件
$(dom).on('click.ls',fn);//这是张三定义的事件

$(dom).triggle('click.zs');//触发张三定义的事件
```

来个面试题试试

```html
<div class="father" style="background:red;width:200px;height:200px">
  <div class='son' style="background:blue;width:100px;height:100px"></div>
</div>

<script>
$(function(){
  $('.father').on('click.ls',function(){
    alert('f1')
  })
  $('.father').on('click',function(){
    alert('f2')
  })
  $('.son').on('click.ls',function(){
    alert('s')
  })
  
  $('.son').trigger('click.ls');
})
</script>
```

点击son时输出什么？s	f1，因为触发了命名空间事件，并且会触发事件冒泡，如果是`$('.son').trigger('click');`就会触发所有的click事件输出s	f1	f2

**事件委托**

```html
<ul>
	<li>1</li>
  <li>2</li>
</ul>
<button onclick="addTag()">
  添加
</button>
<script>
$(function(){
  $('ul>li').click(function(){
    console.log($(this).text());
  })
  $('button').click(function(){
    $('ul').append($('<li>new</li>'))
  })
})
</script>
```

这时点击元素的话可以正常输出，但是如果是动态添加的新的li就不会被绑定click事件了也无法输出new了。

因此需要使用到事件委托为新添加的元素添加事件

```js
$('ul').delegate('li','click',function(){
  console.log($(this).text())
})
```

将li的点击事件委托给父元素，其实就是利用事件冒泡机制。

**移入移出事件**

> ```js
> $(dom).on('mouseover',function(){})
> $(dom).on('mouseout',function(){})
> ```
>
> 这种一移入移出事件会触发事件冒泡

> ```js
> $(dom).on('mouseenter',function(){})
> $(dom).on('mouseleave',function(){})
> ```
>
> 这种则不会触发事件冒泡

> ```js
> $(dom).hover(移入事件函数，移出事件函数);
> $(dom).hover(移入移出事件函数);
> ```
>
> hover事件其实就是mouseenter和mouseleave的结合体，因此这个也不会触发事件冒泡。
>
> 如果只写一个事件处理函数的话，就是移入移出都是使用这个函数进行处理



## **jq动画效果**

**显示隐藏**

```js
//隐藏：display:none
$('div').hide(时间（毫秒）,动画函数,执行完毕回调函数);
//显示: display:block
$('div').show(时间（毫秒）,动画函数,执行完毕回调函数);
//切换
$('div').toggle(时间（毫秒）,动画函数,执行完毕回调函数);
```

案例：对联广告

动画函数：swing（默认）,linear，speed，easing

**展开和收起**

````js
//展开
$('div').slideDown(时间（毫秒）,动画函数,执行完毕回调函数);
//收起
$('div').slideUp(时间（毫秒）,动画函数,执行完毕回调函数);
//切换
$('div').slideToggle(时间（毫秒）,动画函数,执行完毕回调函数);
````

**淡入淡出**

```js
//展开
$('div').fadeIn(时间（毫秒）,动画函数,执行完毕回调函数);
//收起
$('div').fadeOut(时间（毫秒）,动画函数,执行完毕回调函数);
//切换
$('div').fadeTo(时间（毫秒）,透明值（0-1）,动画函数,执行完毕回调函数);
//切换
$('div').fadeToggle(时间（毫秒）,动画函数,执行完毕回调函数);
```

**自定义动画**

animate

```js
$('div').animate({
  要操作的css属性，font-size => fontSize
},时间（毫秒）,动画函数,执行完毕回调函数){
  TODO....
}

//可以使用链式编程做连续动画
$('div').animate({width:'100px'},500).animate({height:'100px'},500)
```

累加属性

```js
//如果div的width是100px,没次动画增加100px
$('div').animate({
  width:'+=100'
},1000);
```

stop和delay

```js
//停止当前元素动画
//立即停止当前的，继续执行后续的
$('div').stop();
$('div').stop(false);
$('div').stop(false,false);
//立即停止当前和后续的所有动画
$('div').stop(true);
$('div').stop(true,false);
//立即完成当前的，继续执行后续的
$('div').stop(true,true);
//立即完成当前的，停止执行后续的
$('div').stop(true,true);

//为元素动画增加执行延迟
$('div').animate({width:'100px'},500).delay(1000).animate({height:'100px'},500)
```



## jq节点操作

添加节点

```html
<div>1</div>
<script>
//内部插入
$('div').append($('<p>new</p>'))		//<div>1<p>new</p></div>
$('<p>new</p>').appendTo('div')			//<div>1<p>new</p></div>
$('div').prepend($('<p>new</p>'))		//<div><p>new</p>1</div>
$('<p>new</p>').prependTo('div')		//<div><p>new</p>1</div>
  
//外部插入
$('div').after($('<p>new</p>'))			//<div>1</div><p>new</p>
$('<p>new</p>').insertAfter('div')	//<div>1</div><p>new</p>
$('div').before($('<p>new</p>'))		//<p>new</p><div>1</div>
$('<p>new</p>').insertBefore('div')		//<p>new</p><div>1</div>
</script>

```

删除节点

```html
<div>
	<p>this is a pagrape</p>
  <div class="div1">this is div</div>
</div>
<script>
$('div').remove() //会将包或自己在内的元素删除 显示的是空
$('div').remove('.div1') //<div><p>this is a pagrape</p></div>
//detach和remove一样
  
$('div').empty() //删除内容，但是不会删除自身<div></div>

</script>
```

替换节点

```html
<div>
  <p>
    this is a p tag
  </p>
</div>
<script>
//替换所有匹配的元素为指定元素
$('p').replaceWith($('<h2>this is a head</h2>'));
$('<h2>this is a head</h2>').replaceAll('p')
</script>
```

复制节点

```html
<div>
  <p>
    this is a p tag
  </p>
</div>
<script>
  //元素复制并且添加到末尾
  let clone=$('div').clone(true)
  $('div').after(clone);
  //true:深复制，会复制元素，会复制事件
  //false：浅复制,只会复制元素，不会复制事件
</script>
```





[jquery中文手册](http://jquery.cuishifeng.cn/index.html)