# 常见的定位方案

1. 普通流
   - 元素按照其在HTML中的先后位置由上而下布局
2. 浮动布局（float）
   - 元素首先按照普通流的位置出现，然后根据浮动的方向尽可能的向左或者向右偏移，脱离文档流
3. 绝对定位（absolute）
   - 开启绝对定位，元素会相对于第一个 position 不为 static  的父级元素进行定位，脱离文档流

# BFC 概念

**BFC( block formatting contexts )** 块级格式上下文。属于定位方案中的普通流

**特点**：具有BFC 特性的元素可以看作是隔离的独立容器，容器里面的元素不会再布局上影响到外面的元素。简单理解就是一个封闭的大箱子。



# 开启BFC方法

1. body元素，本身就是一个 BFC
2. 浮动元素：float 除 none 以外的值。
3. 绝对定位元素：position ：absolute / fixed
4. display 为 inline-block，table-cell，flex;
5. overflow：除了 visible 以外的其他值。

BFC 一般都是使用在不想受外部布局

# BFC 的特性以及应用

1. 同一个 BFC 环境下，元素在垂直方向上的外边距会折叠

```html
<style>
    div{
        width:100px;
        height:100px;
        background-color:red;
        margin:100px;
    }
</style>
<body>
    <div></div>
    <div></div>
</body>
```

解决：放在不同的 BFC 中，这里两个 div 都存在于同一个 BFC（body） 中，所以只需要将其中一个div 放入到另外一个 bfc 中即可

```html
<style>
    .div-box{
        width:100px;
        height:100px;
        background-color:red;
        margin:100px;
    }
    .bfc-box{
        overflow:hidden;
    }
</style>
<body>
    <div class="bfc-box">
        <div class="div-box"></div>
    </div>
    <div class="div-box"></div>
</body>
```

2. BFC可以包含浮动元素（清除浮动）

```html
<style>
    div{
        border:1px solid red;
    }
    .div-box{
        width:100px;
        height:100px;
        background-color:red;
        float:left;
    }
    .bfc-box{
        overflow:hidden;/*不开启BFC bfc-box将不能被div-box 撑开*/
    }
</style>
<body>
    <div class="bfc-box">
        <div class="div-box"></div>
    </div>
</body>
```

3. BFC 可以阻止元素被浮动元素覆盖

```html
<style>
    .div-box{
        width:100px;
        height:100px;
        background-color:red;
        float:left;
    }
    .text-box{
        width:200px;
        height:200px;
        background-color:black;
        color:white;
    }
    .bfc-box{
        overflow:hidden;/*不开启BFC bfc-box将被div-box 覆盖并且会有文字环绕的效果*/
    }
</style>
<body>
    <div class="div-box"></div>
    <div class="bfc-box text-box">
        不开启BFC bfc-box将被div-box 覆盖并且会有文字环绕的效果
        不开启BFC bfc-box将被div-box 覆盖并且会有文字环绕的效果
        不开启BFC bfc-box将被div-box 覆盖并且会有文字环绕的效果
        不开启BFC bfc-box将被div-box 覆盖并且会有文字环绕的效果
        不开启BFC bfc-box将被div-box 覆盖并且会有文字环绕的效果
        不开启BFC bfc-box将被div-box 覆盖并且会有文字环绕的效果
    </div>
</body>
```
