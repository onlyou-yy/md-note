## flutter

### 安装

### 配置

### 配置Android环境

### 设置虚拟机

### 安装编辑器插件

安装vscode 的插件，有助于提高开发效率，开发flutter 常用的插件有

![image-20220508163113492](/Users/a/Desktop/ljf/myfile/myGitServer/md-note/flutter/image-20220508163113492.png)

### 创建项目

创建 flutter 项目可以使用`flutter create 项目名`直接创建。

在安装了`flutter`插件之后，如果有手机或者虚拟机连接上的话，在右下角有个`no devices`，点击它就可以选择要连接的设备了，在连接上之后，就可以运行`flutter run`命令启动程序了。

### 程序入口

在dart 中程序的入口是一个`main`函数，在flutter 中入口是`runApp(根组件)`

```dart
void main(){
  runApp(Container(
    child:Center(
    	child:Text(
        'hello',
        fontSize:40
      )
    )
  ));
}
```

框架会强制让根 widget 铺满整个屏幕

### 组件

在 flutter 中几乎所有的组件都是 **Widget** （控件、部件、插件），可以理解为容器，几乎每个容器都会有一个 child 属性用来放置子节点，从而表现页面布局中的嵌套关系，在组件上还有一些其他的属性，比如`color,width,height`等都是对当前组件样式表现描述。

其实 flutter 中的组件和 html 中内联了样式的元素非常相似，比如下面的一个html

```html
<div style="background:yellow;width:100px;height:100px;">
	<div style="background:red;width:50px;height:50px;color:white;">
    hello world
  </div>
</div>
```

用flutter的组件表示就是

```dart
void main(){
  runApp(
  	Container(
      width:100,height:100,color:Color.yellow,
    	child:Container(
        width:50,height:50,color:Color.red,
        child:Text('hello world',color:Color.white,
        )
      )
    )
  );
}
```

其中`Container`基于相当于是一个`div`容器，而`Text`就相当于是第二个`div`中的文本节点（Text属于比较低级的组件，他的字节点就是文本内容，所以没有child）

**自定义组件**

一般来说，程序都不会那么简单，只用几个组件就能写好，所以上面的直接在`runApp`中写页面是不太现实的，所以一般都是自定一个组件来书写页面，自定组件首先需要继承一个容器组件，然后重写一下 build 方法（相当于react 中类式组件的 render 方法），这个方法返回的一个 widget。

```dart
void main(){
  runApp(MyApp());
}
class MyApp extends StatelessWidget {
  const MyApp({ Key? key }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      width:100,height:100,color:Color.yellow,
    	child:Container(
        width:50,height:50,color:Color.red,
        child:Text('hello world',color:Color.white)
      )
    );
  }
}
```

### 有状态组件和无状态组件

在自定义组件的时候我们需要继承一个 widget 的容器组件，常用有两个`StateLessWidget`和`StatefulWidget`，这两个的区别是前者是**无状态组件**后者是**有状态组件**，无状态组件表示在第一个次编译之后视图就固定下来了，之后即使修改了里面的状态数据页面也不会更新，所以在写无状态组件的时候通常都会使用`const`和`final`来定义变量和子节点组件。有状态组件则相反，在修改状态数据之后页面也会更新，比较适合做交互组件。

### 常用组件

**容器类**

+ `Text`文本组件，相当于就是一个普通`span`
+ `Center`上下左右居中容器组件，相当于一个设置了`display:flex;just-content:center;align-items:center;`的 `div`
+ `Row`多子节点容器组件，其中有`children`属性可以设置多个字节点的列表，子节点是横向排列，相当于一个设置了`display:flex;flex-direction:row`的 `div`
+ `Column`多子节点容器组件，其中有`children`属性可以设置多个字节点的列表，子节点是纵向排列，相当于一个设置了`display:flex;flex-direction:column`的 `div`
+ `Stack`多子节点容器组件，其中有`children`属性可以设置多个字节点的列表，子节点之间是重在一起的，相当于给每个子组件都设置了绝对定位。可以使用`Positioned`来控制位置
+ `Positioned`绝对定义的容器，相当于设置了`position:absolute;`的`div`
+ `Container`普通的单子节点容器组件，用来创建一个可见的矩形元素，可以通过设置`decoration`来设置背景色（color），背景图（image），边框（border），阴影（boxShader）等盒子类型效果。还可以设置`margin 外边距,padding 内边距`等。
+ `image`相当于`img`标签，可以用来加载网络图片（Image.network）、应用资源图片（Image.asset）、文件图片（Image.file）。并且可以通过`colorBlendMode`设置混合模式，一般都是作为其他容器的子容器使用，以便于设置其他样式
+ `MaterialApp`应用程序容器，可以理解成一个应用程序的盒子，可以这个这个盒子上设置路由来管理多个页面，设置全局数据，应用主题等等。总之就相当于是一个应用实例，功能类似于Vue中的`new Vue({})`。
+ `Scaffold`应用页面级模版容器，可以设置页面的标题栏，选项卡等。

**工具类**

+ `Color`用来定义颜色，主要使用的方法是`Color.fromARGB(a, r, g, b)`和`Color.fromRGBO( r, g, b,o)`,还可以`Color(0xFFFFFFFF)`来定义颜色
+ `Colors`也是用来定义颜色的，不过可以直接使用定义好的静态属性`Colors.red`
+ `TextDirection`用来描述`Text`组件的文字的文字排列方向
+ `BoxDecoration`用来给盒子类型的容器设置阴影，圆角，边框等盒子容器的属性
+ `NetworkImage,FileImage,AssetImage`加载图片资源
+ `TextStyle`为`Text`组件的文字设置样式、大小、加粗等
+ `EdgeInsets`可以为盒子容器的`margin padding`提供值
+ `Transform`设置平移，旋转，缩放等transform过度效果

**组件目录**

https://flutter.cn/docs/development/ui/widgets

https://flutter.cn/docs/reference/widgets

https://api.flutter-io.cn/flutter/widgets/widgets-library.html#classes

