## 跨平台解决方案

在以往移动端的开发往往需要支持安卓和IOS平台，而这两个平台支持的语言和环境都是不一样的，这就意味着做一个App就需要开发两套代码，后台出现了跨平台解决方案，webView（坑多，效率低），React Native（性能还行，入门简单，但是开发不友好）。之后就出现了flutter。

flutter —— native —— react native 的区别

![1654092500928](flutter/1654092500928.png)

可以看出 他们之间的区别，flutter 和 Native 的流程几乎一样，flutter可以利用 Skia 绘画引擎直接通过 CPU、GPU进行绘制。而 react  native 就需要先将 js、jsx等其他的js代码通过 javaScript 虚拟机编译转化成对应的 原生组件之后才能通过 桥接 来和原生组件进行数据传输。

## flutter 的绘制原理

我们知道一秒播放16中图片的时候，会觉得这是一个流畅的动画效果，一秒中内可以播放多少中图片也叫做**帧率（fps）**，一般电影的帧率在24或者30。

要在显示设备上显示图像的话就需要频繁的刷新页面，保证图像时刻都是新的，这样就会用动画效果，这种每秒刷新的次数叫做**刷新率**，一般IPhone的是 60Hz，iPad 是120Hz。

**帧率和刷新率的关系**

CPU/GPU 向 Buffer 中生成图像，屏幕从 Buffer 中取 图像、刷新后显示。 这是一个典型的生产者——消费者模型。 理想的情况是帧率和刷新频率相等，每绘制一帧，屏幕显示一帧。 但是实际往往它们的大小是不同的。 如果没有锁来控制同步，很容易出现问题。 例如，当帧率大于刷新频率，当屏幕还没有刷新第 n-1 帧的时候，GPU 已经在生成第 n 帧了 从上往下开始覆盖第 n-1 帧的数据，当屏幕开始刷新第 n-1 帧的时候，Buffer 中的数据上半部分是第 n 帧数据， 而下半部分是第 n-1 帧的数据 显示出来的图像就会出现上半部分和下半部分明显偏差 的现象，我们称之为 “tearing”（撕裂）。

![1654095920721](flutter/1654095920721.png)

**双重缓存解决 tearing**

为了解决单缓存的“tearing”问题，就出现了 双重缓存和 VSync ； 两个缓存区分别为 Back Buffer 和 Frame Buffer。 GPU 向 Back Buffer 中写数据，屏幕从 Frame Buffer 中读数据。 VSync 信号负责调度从 Back Buffer 到 Frame Buffer 的复制操作 ，当然底层不是通过复制，而是通过交换内存地址方式，所以可以瞬 间完成，效率是非常高的； 

![1654096199807](flutter/1654096199807.png)

**工程流程**： 

+ 在某个时间点，一个屏幕刷新周期完成，VSync 信号产生，先完成 复制操作，然后通知 CPU/GPU 绘制下一帧图像。 
+ 复制操作完成后屏幕开始下一个刷新周期，即将刚复制到 Frame  Buffer 的数据显示到屏幕上。 
+ 在这种模型下，只有当 VSync 信号产生时，CPU/GPU 才会开始绘制。

所以 flutter 的绘制原理如下

![1654096580840](flutter/1654096580840.png)

1. GPU 将信号同步到UI线程
2. UI线程同Dart来构建图层树
3. 图层树在GPU线程进行合成
4. 合成后的视图数据提供给Skia引擎
5. Skia 引擎通过OpenGL或者Vulkan将显示内容提供给GPU
6. GPU 将图像绘制完成后 发出 Vsync 信号通过Back Buffer 进入到 Frame Buffer，GPU继续渲染 Frame Buffer 中的图像，并且 Vsync 通知另一个GPU继续绘制下一个图像。

## 安装

可以直接在官网下载安装程序或者压缩包就可以安装了，安装完成之后再配置环境变量，flutter 中的课执行文件是在`/bin`目录下的，所以需要在环境变量下配置.

```
安装目录/flutter/bin
```

因为flutter 还是依赖 Dart 所以还需要下载安装并配置 Dart

```
安装目录/dart-sdk/bin
```

之后就需要可以运行`flutter --version`检查是否安装成功了。

## 检查开发环境，以及一些坑

安装成功之后哦就可以通过`flutter doctor`命令来检查当前开发环境还缺少的东西了。做移动端开发肯定还需要用到手机模拟器的，一般在windows下可以使用 Android Studio 的模拟器也可以使用其他的模拟起，可以选择使用 **夜神模拟器** ，在MAC下可以使用Xcode中提供的模拟器。

下面是一些可能回遇到的坑和解决方法，也可以去[中文社区](https://flutter-io.cn/)看看

https://segmentfault.com/a/1190000041554528#:~:text=%E5%A6%82%E6%9E%9C%E5%87%BA%E7%8E%B0%E6%8A%A5%E9%94%99%20Android%20sdkmanager%20not%20found.%20Update%20to%20the,-%3E%20%E5%8B%BE%E9%80%89Android%20SDK%20Command-line%20Tools%20%28latest%29%20-%3E%20OK

https://blog.csdn.net/MarkeyMark/article/details/111031751

https://blog.csdn.net/adorable_/article/details/116590749

## 常用key命令

在运行项目`flutter run`之后。我们可以在控制台输入一些key来让项目做出对应的处理，比如输入`r`热更新，

+ `r`热更新，主要执行组件的`build`方法
+ `R`重载项目
+ `o`切换平台（Android，Mac）
+ `p`切换构造线的显示
+ `i`切换小部件检查器
+ `v`打开devtool，如果需要输入连接url的话，可以输入运行项目后提示的第二个地址，如下。第一个是控制台的网页地址
+ `c`清除屏幕
+ `q`退出项目

![image-20220520121443016](/Users/a/Desktop/ljf/myfile/myGitServer/md-note/flutter/image-20220520121443016.png)

## 安装编辑器插件

安装vscode 的插件，有助于提高开发效率，开发flutter 常用的插件有

![image-20220508163113492](flutter/image-20220508163113492.png)

[vscode中Flutter开发中常用的快捷键](https://juejin.cn/post/6998726767074623496/)



## 创建项目

创建 flutter 项目可以使用`flutter create 项目名`直接创建。

在安装了`flutter`插件之后，如果有手机或者虚拟机连接上的话，在右下角有个`no devices`，点击它就可以选择要连接的设备了，在连接上之后，就可以运行`flutter run`命令启动程序了。

## 程序入口

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

框架会强制让根 widget 铺满整个屏幕，并且 `runApp(根组件)`中根组件必须指明排版顺序。

## 组件

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

## 有状态组件和无状态组件

在自定义组件的时候我们需要继承一个 widget 的容器组件，常用有两个`StateLessWidget`和`StatefulWidget`，这两个的区别是前者是**无状态组件**后者是**有状态组件**

无状态组件表示在第一个次编译之后视图就固定下来了，之后即使修改了里面的状态数据页面也不会更新，所以在写无状态组件的时候通常都会使用`const`和`final`来定义变量和子节点组件。

有状态组件则相反，在修改状态数据之后页面也会更新，比较适合做交互组件。

> 其实在 flutter 中所有的 widget 都是不能改变的，因为几乎所有的 widget 都继承于 Widget 类，而 Widget 类是被 `@immutable`装饰，表示不可变，所以无论是在`StatelessWidget`中还是`StatefulWidget`中都是不能直接定义可修改的状态的，而且其中的变量都应该使用 final 或者 const 来进行定义，如果要定义状态需要在`StatefulWidget`中的 `createStatus`方法返回的 继承于`State`的组件中定义。
>
> 在 StatefulWidget 中是没有 build 方法的，StatefulWidget 组件的 build 方法是放在 State 组件中的，这是因为：
>
> 1. build 出来的 widget 是需要依赖 State 中的变量；
> 2. 在 Flutter 的运行过程中，widget 是不断的销毁和创建的，当我们自己的状态发送改变时并不希望创建一个新的 State



## 生命周期

生命周期函数可以让我们在合适的时机做正确的事，比如在初始化的时候获取数据，初始化数据，进行组件事件监听等，在 flutter 中无状态组件 StatelessWidget 的生命周期之后构造函数和build方法。

```dart
class Test extends StatelessWidget{
	Text(){
    print('实例化完成');
  }
  @override
  Widget build(BuildContext context){
    print('调用build方法');
    return Text("hello");
  }
}
```

有状态组件 StatefulWidget 的生命周期函数如下：

- 在下图中，灰色部分的内容是Flutter内部操作的，我们并不需要手动去设置它们；
- 白色部分表示我们可以去监听到或者可以手动调用的方法；

![image-20220607152848376](/Users/a/Desktop/ljf/myfile/myGitServer/md-note/flutter/image-20220607152848376.png)

首先，执行**StatefulWidget**中相关的方法：

1. 执行StatefulWidget的构造函数（Constructor）来创建出StatefulWidget；
2. 执行StatefulWidget的createState方法，来创建一个维护StatefulWidget的State对象；

其次，调用createState创建State对象时，执行State类的相关方法：

1. 执行State类的构造方法（Constructor）来创建State对象；
2. 执行initState，我们通常会在这个方法中执行一些数据初始化的操作，或者也可能会发送网络请求；

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/O8xWXzAqXuswRnN2TtnykKuZ5VJibJcVKLBOvkpcqFXPENRFyDlDgb6JBgJZvaFFGpoOyLbIcEdd0IKwYy3WU6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

- - 注意：这个方法是重写父类的方法，必须调用super，因为父类中会进行一些其他操作；
  - 并且如果你阅读源码，你会发现这里有一个注解（annotation）：@mustCallSuper

3. 执行didChangeDependencies方法，这个方法在两种情况下会调用

- - 情况一：调用initState会调用；
  - 情况二：从其他对象中依赖一些数据发生改变时，比如 InheritedWidget

4. Flutter执行build方法，来看一下我们当前的Widget需要渲染哪些Widget；
5. 当前的Widget不再使用时，会调用dispose进行销毁；
6. 手动调用setState方法，会根据最新的状态（数据）来重新调用build方法，构建对应的Widgets；
7. 执行`didUpdateWidget`方法是在当父Widget触发**重建**（rebuild）时，系统会调用`didUpdateWidget`方法；

## 常用组件

### **容器类**

**单子节点容器**

+ `Text`文本组件，相当于就是一个普通`span`，（最终渲染的并不是 Text ，而是一个`RichText`）
+ `Text.rich`富文本组件，第一个参数可以放`TextSpan`，这是一个多节点容器，里面可以放多个文本组件，比如文本容器`TextSpan`，组件容器`WidgetSpan`。
+ `Center`上下左右居中容器组件，相当于一个设置了`display:flex;just-content:center;align-items:center;`的 `div`
+ `Positioned`绝对定义的容器，相当于设置了`position:absolute;`的`div`
+ `Align`设置位置的容器，可以通过设置`aligment`属性操作元素在容器中的位置，通过`widthFactory，heightFactory`设置容器宽高是子容器的大小的倍数。（其实在一些组件中设置的布局排序属性`alignment`，本质上也是使用`Align`再给子元素包裹一层的）
+ `Container`普通的单子节点容器组件，用来创建一个可见的矩形元素，可以通过设置`decoration`来设置背景色（color），背景图（image），边框（border），阴影（boxShader）等盒子类型效果。还可以设置`margin 外边距,padding 内边距`等。
+ `AspectRatio`可以设置相对宽高比的容器，通过`aspectRatio:宽/高`可以设置当前容器宽和高的比
+ `Icon`图标容器，方便快捷得生成图标，是一种矢量图。
+ `Card`卡片容器组件
+ `ClipOval`裁剪容器，可以将容器裁剪成其他的形状，默认是处理成圆形/椭圆型，一般会配合`Image`使用
+ `SizeBox`空白的容器组件，通常用来做占位空间。
+ `Padding`内间距容器，相当于设置了`padding:xx`的`div`
+ `Flexible`弹性盒子容器，可以通过设置`flex`来按比例分配空间，设置`fit`来确定剩余空间的分配使用，如果设置了`fit:FlexFit.loose`（默认），表示是分配了但是不使用，`fit:FlexFit.tight`就是使用。
+ `Expanded`弹性盒子容器，继承于`Flexible`，相当于是设置了`display:flex`的`div`，可以设置`flex`比例，不能设置`fit`，默认是`tight`
+ `Divider`分割线
+ `Theme`主题容器，可以快速为子容器设置主题，比如颜色风格
+ `SafeArea`安全区域容器，不会遮挡状态栏和底部菜单

**多子节点容器**

+ `Row`多子节点容器组件，其中有`children`属性可以设置多个字节点的列表，子节点是横向排列，且默认占满一行，可以通过`mainAxisSize`来设置占据空间的大小；相当于一个设置了`display:flex;flex-direction:row`的 `div`
+ `Column`多子节点容器组件，其中有`children`属性可以设置多个字节点的列表，子节点是纵向排列，相当于一个设置了`display:flex;flex-direction:column`的 `div`
+ `Flex`弹性盒子布局容器，就是相当于一个`display:flex`的`div`，可以设置 flex 的相关属性，比如排列方向`direction`，当设置`direction:Axis.vertical`时就相当于是一个 `Column`，当设置`direction:Axis.horizontal`时就相当于是一个 `Row`。
+ `PageView`页面滚动容器组件，效果类似于抖音等视频的单页面视频滚动效果，每个子元素都相当于是一个页面。
+ `Wrap`子节点在一行/列放不下时会自动换行，通过`direction`属性定义接单排序方向，可以用来做瀑布流布局
+ `Stack`多子节点容器组件，其中有`children`属性可以设置多个字节点的列表，子节点之间是重在一起的，相当于给每个子组件都设置了绝对定位。可以使用`Positioned`或者`Align`来控制位置，也可以使用`alignment`属性定义内容的位置。通过`fit`可以设置子元素的填充模式
+ `ListView`多节点容器组件，一般用做列表显示，可以通过`scrollDirection`定义列表的方向。不过需要注意子节点的宽度（垂直列表）/高度（水平列表）会被强制铺满适应父容器的宽度/高度，如果要设置列表的宽度，就需要设置其父容器的宽度，或者通过设置padding属性将它进行挤压，还有就是默认情况下`ListView|Column`下不能在使用`ListView`，如果要使用需要设置`shrinkWrap`属性为`true`;
  + `ListTile`文章列表组件，可以定义标题和子标题，通常配`ListView`使用，也可以通过`leading`定义列表项前图标，`trailing`在列表项后定义图标
  + `ListView.builder`这个构造函数是用来创建动态列表的，会进行循环创建，通过`itemCount`定义循环的次数，通过`itemBuilder`来定义渲染函数，**当节点显示时才会被创建**。
  + `ListView.separated`这个构造函数是用来创建带分割组件的动态列表的
  + `ListView(children:List.generate(count,genFn))`，`List.generate(count,genFn)`可以生成一个固定长度的widget列表，但是这种方式会比较消耗性能，因为**无论节点是否显示都会被创建**。
+ `GridView`网格容器列表，相当定义了`display:grid`的`div`，可以用`gridDelegate:SliverGridDelegateWithFixedCrossAxisCount()`定义设置`crossAxisSpacing`水平间距，`mainAxisSpacing`垂直间距等，和`ListView`一样拥有`GridView.builder`构造方法进行动态列表的生成（`flutter_staggered_grid_view`库可以实现瀑布流布局）
  + `GridView.builder`创建动态列表
  + `GridView.count`默认设置了`gridDelegate:SliverGridDelegateWithFixedCrossAxisCount()`的`GridView`
  + `GridView.Extent`默认设置了`gridDelegate:SliverGridDelegateWithFixedCrossAxisExtent()`的`GridView`


其实`ListView/GridView/PageView`等可滚动的视图都是由各种`sliver`组件包装而来，不过这些都是flutter 帮我们定义好了的滚动列表，如果要实现一些比较复杂或者自定义的可滚动列表的话就需要使用`CustomScrollView`来进行设计，比如说我要在一个`ListView`中嵌套`ListView、GridView、PageView`等其他的列表的话就不容易实现了，因为一个不确定大小的容器是不能在嵌套一个不确定大小的容器的（虽然可以定义`shrinkWrap`让列表大小等于内容大小，但是会比较消耗性能）。

[Flutter - 循序渐进 Sliver](https://juejin.cn/post/6844904155195129864)       [Flutter-Sliver](https://www.jianshu.com/p/c892587fed97)

+ `CustomScrollView`自定义的可滚动容器，不过这个容器的子元素不再是`Text`等简单的容器了，而应该是`sliver`。常用的Sliver容器
  + `SliverSafeArea`安全区域，滚动时滚出容器区域的内容不会被隐藏，如果是`SafeArea`就会隐藏
  + `SliverGrid`类似于`GridView`也是创建一个网格布局的列表
  + `SliverList`类似于`ListView`也是创建一个列表
  + `SliverChildBuilderDelegate`是创建`Sliver`节点的方法，和`ListView.builder`类似都是循环创建，当节点显示时才会被创建，通过`delegate`参数创建
  + `SliverChildListDelegate`是创建`Sliver`节点的方法，和`ListView(children:List.generate(count,genFn))`一致，通过`delegate`参数创建
  + `SliverAppBar`页面顶部的导航
  + `SliverPadding`设置内边距，滚动出区域的容器不会隐藏

> 单子节点`Sliver`容器的子节点参数应该是`sliver`，多子节点`Sliver`容器的子节点参数一般是`slivers`，而且`Sliver`系列组件需要在`CustomScrollView`中使用

```dart
class SliverDemo extends StatelessWidget {
  const SliverDemo({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: CustomScrollView(
          slivers: [
            SliverAppBar(
              pinned: true,
              expandedHeight: 300,
              flexibleSpace:FlexibleSpaceBar(
                title: Text('hello'),
                background: Image.network("https://picsum.photos/300/300",fit: BoxFit.cover,),
              ),
            ),
            SliverSafeArea(
              sliver: SliverPadding(
              padding: EdgeInsets.all(10),
              sliver: SliverGrid(
                  delegate: SliverChildBuilderDelegate(
                    (context, index) {
                      return Container(
                        color: Color.fromARGB(255, Random().nextInt(256),
                            Random().nextInt(256), Random().nextInt(256)),
                      );
                    },
                    childCount: 20,
                  ),
                  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                    crossAxisCount: 3,
                    childAspectRatio: 1,
                    mainAxisSpacing: 8,
                    crossAxisSpacing: 8,
                  )),
            )),
            SliverList(delegate: SliverChildListDelegate(
              List.generate(10, (index) => Text("list $index",textScaleFactor: 4,))
            ))
          ],
        ),
      ),
    );
  }
}

```



### 媒体类

+ `image`相当于`img`标签，可以用来加载网络图片（Image.network）、应用资源图片（Image.asset）、文件图片（Image.file）。并且可以通过`colorBlendMode`设置混合模式，一般都是作为其他容器的子容器使用，以便于设置其他样式，如圆角等.

  + 需要注意的是，在使用`Image.asset`、`AssetImage`加载资源图片时需要在根目录下新建`images`目录，并且还要新建`images/2.0x. images/3.0x images/4.0x`文件夹，并且在`images`下放入默认图片，`images/2.0x. images/3.0x images/4.0x`下放入对应分辨率的同名图片，这样flutter就能根据设备的分辨率加载不同的图片。最后还需要在`pubspec.yaml`下配置

    ```yaml
     assets:
     	- images/a.jpeg
     	- images/2.0x/a.jpeg
     	- images/  #加载全部
    ```

+ `NetworkImage,FileImage,AssetImage`加载图片资源

+ `FadeInImage`占位图容器，可以用来设置在加载真正要显示的图片之前显示另外一张图片，等图片加载完成之后再淡入显示。

> flutter 会默认进行图片缓存（限制是 1000 张，100MB），如果flutter 发现要加载的图片的地址和缩放和之前加载过的图片是一样的话就会使用缓存中的图片不会重复加载。

### 控制器

+ `ScrollController`滚动列表的监控控件，可以通过滚动列表的controller参数进行绑定
  + `ins.addListener()`添加滚动事件回调函数，`ins.offset`可以获取滚动的距离
  + `ins.animationTo()`动态跳转到指定位置
+ `NotificationListener`监听子节点事件的组件，可以通过`onNotification`来定义事件回调，如监听滚动事件`onNotification:(ScrollNotification noti){}`

### **工具类**

+ `Color`用来定义颜色，主要使用的方法是`Color.fromARGB(a, r, g, b)`和`Color.fromRGBO( r, g, b,o)`,还可以`Color(0xFFFFFFFF)`（前面两位是透明度）来定义颜色
+ `Colors`也是用来定义颜色的，不过可以直接使用定义好的静态属性`Colors.red`
+ `Icons`定义了很多图标的 IconData
+ `TextDirection`用来描述`Text`组件的文字的文字排列方向
+ `BoxDecoration`用来给盒子类型的容器设置阴影，圆角，边框等盒子容器的属性
+ `TextStyle`为`Text`组件的文字设置样式、大小、加粗等
+ `EdgeInsets`可以为盒子容器的`margin padding`提供值
+ `Transform`设置平移，旋转，缩放等transform过度效果
+ `Alignment`相对于父容器进行位置设置，如果是`Alignment(x,y)`，x和y的值取值范围是`[-1,1]`，如果比这个范围大就会跑出容器范围，`Alignment(0,0)`表示居中

### **MaterialApp UI库类**

+ `MaterialApp`应用程序容器，可以理解成一个应用程序的盒子，可以这个这个盒子上设置路由来管理多个页面，设置全局数据，应用主题等等。总之就相当于是一个应用实例，功能类似于Vue中的`new Vue({})`，为App确定一个设计的风格，应用的排序方向。
+ `Scaffold`应用页面级模版容器，可以设置页面的标题栏，选项卡等。
+ `AppBar`应用页面顶部选项卡容器。
+ `DefaultTabController`在`AppBar`下方的导航标签的容器，可以定义多个标签，并且可以通过`TabBarView`容器定义标签对应的页面。
+ `TabBar`标签容器，通过`tabs`定义多个标签，一般定义在`AppBar`的`bottom`或者`title`上
+ `TabBarView`定义`TabBar`中标签对应的内容。
+ `TabController` 可以定义自定义或者监听标签的行为，不过需要注意的是使用这个组件必须要使用动态组件，并且混入`SingleTickerProviderStateMixin`，而且在`TabBar TabBarView`中的`controller`都定义赋值给同一个`TabController`,之后就可以通过`TabController.addLisener`进行变化的监听了
+ `Drawer`抽屉式侧边栏，可以通过`MaterialApp`的`drawer endDrawer`分别定义左右两边的侧边栏。
+ `DrawerHeader`抽屉式侧边栏顶部容器，可以通过调用 `Navigator.pop` 关闭打开的抽屉。
+ `UserAccountDrawerHeader`抽屉式侧边栏顶部容器，提供和很多可选项，可以快速实现用户侧边栏布局。

### **表单类**

+ `TextFiled`文本输入框，类型`input`标签，可以通过`controller: TextEditingController实例`来设置初始输入以及之后获取输入框组件的数据，通过`onChanged`来监听输入改变的事件
+ `CheckBox`多选盒子，需要通过定义`onChanged`事件之后才能改变。
+ `CheckboxListTitle`可定义大小标题的可选中的列表项，类似于`ListTile`但是在右侧多一个`CheckBox`
+ `Radio`单选按钮，可以通过`groupValue`来指定同一变量来为多个单选按钮规划到一个组上
+ `RadioListTitle`和`CheckboxListTitle`相似
+ `Switch`开关

**模态弹窗**

+ `showDialog`不是一个组件，而是一个用来显示模态弹窗的方法，通过`builder`定义弹窗的内容，这个方法返回一个 `Future`对象，可以使用 then 来订阅模态弹窗的操作返回
+ `AlertDialog`普通提示模态弹窗，通过 `actions`定义操作按钮
+ `SimpleDialog`可以自定义多个子节点内容的模态弹窗
+ `showModalBottomSheet`不是一个组件，而是一个用来显示底部的模态弹窗的方法，和`showDialog`类似
+ `toast`是第三方的包

可以通过`Navigation.pop()`来手动关闭，这也说明了显示显示一个 Dialog 其实是打开一个新的页面，只不过这个页面是一个半透明的状态。

**自定义`Dialog`**

自定义dialog其实是自定义一个新的页面，首先需要继承Dialog，然后实现里面的build方法，这个方法其实是一个半透明的`Material`页面，然后在调用`showDialog`方法将dialog内容显示出来。

```dart
class MyDialog extends Dialog {
  String title = "";
  String msg = "";
  MyDialog({this.title = "", this.msg = "", Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return Material(
      type: MaterialType.transparency,//设置页面背景为半透明色
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,//主轴居中
        crossAxisAlignment: CrossAxisAlignment.center,//侧轴居中
        children: [
          Container(
            width: 200,
            height: 200,
            color: Colors.white,
            child: Column(
              children: [
                Padding(
                    padding: EdgeInsets.fromLTRB(10, 4, 10, 4),
                    child: Stack(
                      children: [
                        Align(
                          alignment: Alignment.topCenter,
                          child: Text(
                            "$title",
                            style: TextStyle(fontSize: 16),
                          ),
                        ),
                        Align(
                          alignment: Alignment.topRight,
                          child:
                              InkWell(child: Icon(Icons.close), onTap: () {
                                Navigator.pop(context);//关闭页面，相当于放回上一个页面
                              }),
                        )
                      ],
                    )),
                Divider(
                  height: 10,
                ),
                Container(
                  width: double.infinity,
                  padding: EdgeInsets.all(10),
                  child: Text("$msg"),
                )
              ],
            ),
          )
        ],
      ),
    );
  }
}
```



### **按钮**

交互就意味着页面会做动态改变，所以要在flutter 中进行交互的话首先组件必须要是有状态的组件，所以在自定义页面组件的时候就需要基础`StatefulWidget`组件，并且实现其中的创建状态的方法，其次真正的子组件其实是继承于`State`类的，这个类中有`setState`方法能管理页面的状态，动态更新页面

```dart
class Test extends StatefulWidget{
  Test({Key:key}):super(key:key);
  _TestState createState() => _TestState();
}
class _TestState extends State<Test>{
  int count = 0;
  @override
  Widget build(BuildContext context){
    return Container(
    	child:Column(
      	children:[
         	Text('数字是：${count}'),
          ElevatedButton(
          	child:Text('+++'),
            onPressed(){
              this.setState((){
                this.count++;
              });
            }
          )
        ]
      )
    )
  }
}
```

在调用`setState`来改变数据的时候，其实是 `build`方法再执行一遍。

flutter 中有多种已经定义好了样式的按钮，比如说上面用到的`ElevatedButton`表示的是一个 凸起的按钮，除此之外还有

+ `ElevatedButton` 凸起的按钮
+ `TextButton`文本按钮
+ `OutlinedButton`线框按钮
+ `IconButton`图表按钮
+ `ButtonBar`按钮组
+ `FloatingActionButton`浮动按钮
+ `InkWell`无样式按钮，通过`onTab`来绑定事件

这些按钮都用自己的默认宽高，如果要设置大小的话需要设置父容器的大小来实现，也可以在外层包裹一个`ButtonTheme`组来设置最小宽高，并且可以通过`style`来设置按钮的相关样式，默认情况想 Button 上下有一定的间距，可以通过设置`materialTapTargetSize`来取消

### 组件使用技巧

flutter中的组件很多，并不需要去记，只需要知道有哪些常用的组件就好，不过官网的组件例子写得不太好，还都是英文的不好看，所以推荐去看**(老孟的 300 多个组件例子)[http://laomengit.com/]**，在平时给组件设置属性的时候需要我们传递固定类型的数据比如设置`Container`组件的`decoration`需要使用`Decoration`类。

![image-20220608145749937](/Users/a/Desktop/ljf/myfile/myGitServer/md-note/flutter/image-20220608145749937.png)

但是发现`Decoration`是一个抽象类，也没有工厂构造函数（可以令抽象类也可以被实例化），所以我们需要去找实现或者继承了`Decoration`的类。此时可以点击`ctrl+F12`就可以看到实现的类了，再结合文档就可以知道应该如何使用

![image-20220608150305374](/Users/a/Desktop/ljf/myfile/myGitServer/md-note/flutter/image-20220608150305374.png)



## 网络请求

flutter 中的异步处理是采用事件循环和非阻塞IO的模式的（其实就JavaScript的模式）。

flutter 中的网络请求方法是由`dart:io`库提供的，前端发请求需要使用其中的`HttpClient`类。而且这个类支持多种请求方式，比如`get post head put patch delete`。

### 发送`get`请求

```dart
let client = HttpClient();
//发起请求
HttpClientRequest request = await client.get('localhost',80,'/file.txt');
//请求完毕后关闭请求获得数据
HttpClientResponse response = await request.close();
//得到数据后还需要解析成真实数据
final stringData = await response.transform(utf8.decoder).join();
```

在拿到数据之后，这些数据一般是json格式的数据，如果直接存储的话，在之后的使用中可能不是很方便，因为不会有类型检查以及代码提示，此时我们可以将这些数据转换成一个数据类，通过[`JSON to Dart`](https://jsontodart.com/)可以快速生成这个类。

### json数据转换

将`Map List`转换成JSON 字符串，使用`dart:convert`的`json.encode(d)`或者`jsonEncode(d)`进行转化，

```dart
Map m = {"key1": "val1"};
List l = [1, 2, 3, 4];
print(json.encode(m));
print(jsonEncode(l));
```

如果要将JSON字符串转化成Map、List，就可以是使用`json.decode(s)`或者`jsonDecode(s)`

```dart
String s = '{"key1": "val1"}';
String al = '[1,2,3,4]';
print(json.decode(s));
print(jsonDecode(al)[1]);
```

### 处理Uri

通过`Uri`类可以对url进行处理，比如将一个url 处理成一个对象，或者说将一个对象处理成Url

```dart
httpsUri = Uri(
    scheme: 'https',
    host: 'example.com',
    path: '/page/',
    queryParameters: {'search': 'blue', 'limit': '10'});
print(httpsUri); // https://example.com/page/?search=blue&limit=10

final uri = Uri.parse(
    'https://dart.dev/guides/libraries/library-tour#utility-classes');
print(uri); // https://dart.dev
print(uri.isScheme('https')); // true
```

### Dio

dio 是一个类似于 axios 的强大的网络请求库

使用前首先需要在`pubspec.yaml`中安装dio库，或者直接运行`pub get dio`

简单使用

```dart
final dio = Dio();
dio.get("http://httpbin.org/get").then(res => print(res));
dio.post("http://httpbin.org/post").then(res => print(res));
```

为了方便维护，一般都不会直接去使用第三方库，而是对他做一层封装

```dart
//基础配置类
class HttpConfig{
  static const String baseURL = "http://httpbin.org";
  static const int timeout = 5000;
}
//封装类
class HttpRequest{
  //基础配置数据
  static final BaseOptions baseOptions = BaseOptions(baseUrl:HttpConfig.baseURL,connectTimeout:HttpConfig.timeout);
  static final Dio dio = Dio(baseOptions);
  
  //公共请求方法
  static Future<T> request<T>(String url,{
    String method = "get",
    Map<String,dynamic> params,
    Interceptor inters,
  }) async {
    //创建单独配置
    final option = Options(method:method);
    
    //创建默认的全局拦截器
    Interceptor DInter = InterceptorsWrapper(
    	onRequest:options=>options,
      onResponse:response => response,
      onError:err => err;
    )
    List<Interceptor> inters = [dInter];
    if(inter != null){
      inters.add(inter)
    }
    
    //发送请求
    try{
      Response response = await dio.request(url,queryParamters:params,options:options);
      return response.data;
    }on DioError catch(e){
      return Future.error(e);
    }
  }
}
```



## 路由

在`Scaffold`组件中可以通过`bottomNavigationBar`属性可以定义底部的导航选项卡，这个属性的值容器是`BottomNavigationBar`，通过`items`选项定义选项卡列表项，`currentIndex`定义选中当前选中的项的索引，`onTab`定义点击选项卡的回调函数所以可以通过`BottomNavigationBar`来控制`Scaffold`组件的`body`显示的组件来实现路由的效果。需要注意的是，如果`items`超过3项就需要设置`type: BottomNavigationBarType.fixed`选项卡才会显示。

还有需要显示的页面可以通过`body`来定义，在 flutter 中有个组件可以根据输入的index来显示对应索引的页面 `IndexedStack`，当也可以使用一个`List`来存储所有的页面，然后通过修改索引来显示页面的。

```dart
class Tab extends StatefulWidget {
  const Tab({Key? key}) : super(key: key);

  @override
  State<Tab> createState() => _TabState();
}

class _TabState extends State<Tab> {
  int currentIndex = 0;
  //List<Widget> pageList = [Page1(), Page2(), Page3()];
  Widget pages = IndexedStack(index:this.currentIndex,children:[Page1(), Page2(), Page3()]);
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('hello world'),
      ),
      //body: this.pageList[this.currentIndex],
      body: pages,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: currentIndex,
        type: BottomNavigationBarType.fixed,//如果items超过3项就需要设置
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home_outlined), label: '首页'),
          BottomNavigationBarItem(
              icon: Icon(Icons.ac_unit_rounded), label: '血'),
          BottomNavigationBarItem(
              icon: Icon(Icons.access_alarm_sharp), label: '时间'),
        ],
        onTap: (index) {
          this.setState(() {
            this.currentIndex = index;
          });
        },
      ),
    );
  }
}
class Page1 extends StatelessWidget {
  const Page1({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Center(
        child: Text("Page1"),
      ),
    );
  }
}
class Page2 extends StatefulWidget {
  const Page2({Key? key}) : super(key: key);
  @override
  State<Page2> createState() => _Page2State();
}
class _Page2State extends State<Page2> {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Center(
        child: Text("Page2"),
      ),
    );
  }
}
class Page3 extends StatelessWidget {
  const Page3({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Center(
        child: Text("Page3"),
      ),
    );
  }
}
```

通过`BottomNavigationBar`可以实现简单的路由效果，但是这个并不是真正的路由，因为这种方式搞多依赖于UI，其实在flutter中，路由是通过`Navigator`类来实现的

### Navigator

在 flutter 中几乎都是组件，所以flutter 的路由跳转实际上就是将当前的页面替换成其他的页面组件

flutter 中有两种路由模式，一种是普通路由通过`Navigator`实现，一种是命名路由，通过`MaterialApp`的`routers`属性实现。

**常用的方法**

+ `Navigator.of(context).push()`普通跳转
+ `Navigator.of(context).pop()`跳转到栈定的页面，相当于返回，返回时还可以传递传递参数`pop('xxx')`，之后可以在进入当前页面的那个路由中通过then接收，比如要返回的A页面`Navigator.of(context).push(MaterialPageRoute(builder:(context)=>A())).then(res=>print(res))`
+ `Navigator.of(context).pushReplacementNamed()`跳转到另外一个页面，并替换当前记录
+ `Navigator.of(context).pushAndRemoveUntil()`跳转到另外一个页面，对以往的历史记录做移除操作，比如跳转到`Page1`并移除跳转记录`Navigator.of(context).pushAndRemoveUntil(MaterialPageRoute(builder:(context)=>Page1())),(route)=>route == null`

### **普通路由**

将上面的 `Page1`改成

```dart
class Page1 extends StatefulWidget {
  const Page1({Key? key}) : super(key: key);
  @override
  State<Page1> createState() => _Page1State();
}
class _Page1State extends State<Page1>{
 	@override
  Widget build(BuildContext context){
    return Container(
      child: ElevatedButton(
        child:Text('点我跳转'),
      	onPressed(){
          // 路由跳转，of方法可以定义执行上下文，如果不使用of的话,可以在push的第二个参数传入 Navigator.push(context,...)
         	Navigator.of(context).push(
          	MaterialPageRoute(
            	builder:(BuildContext context) => Page2(),
            )
          )
        }
      )
    );
  }
}
```

上面就通过`Navigator.of().push`实现了一个简单的路由跳转，一般路由跳转都会携带参数的，在 flutter 中传递参数一般都是通过给构造函数传递参数实现的，路由和父子组件之间通信也是如此

那么在`Page1` 向`page2`传递参数就是这样

```dart
Navigator.of(context).push(
  MaterialPageRoute(
    builder:(BuildContext context) => Page2('这是一个参数'),
  )
)
```

同时`Page2`也需要做一下修改

```dart
class Page2 extends StatefulWidget {
  String? params;
  Page2(this.params,{ Key? key }) : super(key: key);

  @override
  State<Page2> createState() => _Page2State();
}

class _Page2State extends State<Page2> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('page2'),
      ),
      body: Container(
        child: Text('Page2 get params:${widget.params}'),
      ),
    );
  }
}
```

需要注意的是，如果是无状态组件的话可以直接使用传过来的构造参数，但是如果是有状态组件的话就需要通过`widget`来访问，因为 有状态组件的实体内容（状态类`_Page2State`）和构造类(`Page2`)是分开的，`widget`默认存在于状态类中，表示的就是`Page2`实例。

### **命名路由**

flutter 中的命名路由需要在`MaterialApp`的`routes`属性中配置路由映射，之后就可以使用`Navigator.pushNamed(context,'路由名')`方法就能跳转到对应的页面

```dart
class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'hello',
      home: MyTab(),
      routes:{
        '/page1':(context)=>Page1(),
        '/page2':(context)=>Page2(),
      }
    );
  }
}
```

之后在需要跳转的时候调用`Navigator.pushNamed(context,'/page1')`就能跳转到`Page1`了

命名路由的传参方式有两种，一种是直接通过`Navigator.pushNamed()`的第三个参数`argumentss`来传递，然后在页面中通过`final args = ModalRoute.of(context)!.settings.arguments;`就可以获取到了

### `onGenerateRoute`

还有一种方式是通过`MaterialApp`的`onGenerateRoute`来进行统一的路由管理，也可以用来做路由拦截的功能。在调用`Navigator.pushNamed()`会先执行`onGenerateRoute`回调，这个回调接收一个`settings`的参数（包含要跳转的url和要传递的参数），在这里我们可以通过返回一个`MaterialPageRoute`对象来决定要跳转到的页面

```dart
const routes = {
  '/':(context)=>Home(),
  '/page1':(context)=>Page1(),
  '/page2':(context,{arguments})=>Page2(arguments:arguments),
}
MaterialApp(
  //初始路由，开始的时候会加载 Home();
  initialRoute:'/',
  onGenerateRoute: (settings) {
    //settings.name 路由名，settings.arguments路由参数
    final Function pageBuilder = this.routes[settings.name];
    return MaterialPageRoute(
      builder: (context) {
        //Page2 需要在构造函数中接收参数
        return pageBuilder(context,settings.arguments);
      },
    );
  },
)
```

`onUnknowRoute`

当匹配不到想的路由的时候就会触发这个方法，可以在这个方法中放返回一个 404 的页面

```dart
MaterialApp(
	onUnknowRoute:(settinggs){
    return MaterialPageRoute(
    	builder:(context) => Page404()
    )
  }
)
```



## 动画

## 状态管理

### 父子组件通信

父子组件之间的传值主要是通过 组件的构造函数参数进行传递，如果是父组件传递给子组件，那么就可以直接将数据传递过去；如果是子组件传给父组件就可以通过父组件传递过来的方法来将数据传递出去

**通过构造函数参数传递数据**

父组件

```dart
class A extends StatefulWidget {
  A({Key? key}) : super(key: key);
  @override
  State<A> createState() => _AState();
}

class _AState extends State<A> {
  String parentData = "parent";
  String childData = '';
  //父组件接收子组件传递过来的数据
  void getChildData(data) {
    setState(() {
      childData = data;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ListView(
        children: [
          Text('child data:$childData'),
          Divider(),
          B(
            data: parentData,//父组件传递数据给子组件
            cb: (val) => {getChildData(val)},
          )
        ],
      ),
    );
  }
}
```

子组件

```dart
class B extends StatefulWidget {
  late String data;
  late Function cb;
  B({required String data, required Function cb, Key? key}) : super(key: key) {
    this.data = data;
    this.cb = cb;
  }
  String childData = "child";
  @override
  State<B> createState() => _BState();
}

class _BState extends State<B> {
  int count = 0;
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('parent data:${widget.data}'),
        ElevatedButton(
            onPressed: () {
              //传递数据给父组件
              count++;
              this.widget.cb("${widget.childData}${count}");
            },
            child: Text('send data to parent'))
      ],
    );
  }
}
```

显然这种方式是非常麻烦的，当层级高了之后如果比层级差距比较高的两个组件想要进行通信的话，就需要中间的每个组件都传递一些不需要的数据，而且当状态更新时不相关的组件也会更新。

**通过`inheritedWidget`实现数据共享**

在高层级的组件树中通过 构造函数进行数据传递的方法是非常麻烦且耗费性能的，这是可以通过`inheritedWidget`组件创建一个状态数据中心提供给其他的子组件，然后在需要使用的组件中通过`context.dependOnInheritedWidgetOfExactType<A>().xxxx`来获取数据中心的数据。

例如，有`Home -> A -> B -> C`，其中A作为数据中心

```dart
class InherHome extends StatefulWidget {
  int counter = 0;
  InherHome({Key? key }) : super(key: key);
  @override
  State<InherHome> createState() => _InherHomeState();
}

class _InherHomeState extends State<InherHome> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("data share"),
      ),
      body:A(
        counter: this.widget.counter,
        child: B(
          child: C(),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: ()=>{
          this.setState(() {
            this.widget.counter ++;
          })
        },
      )
    );
  }
}
//###########################################
class A extends InheritedWidget{
  int counter = 0;//需要共享的数据
  A({required Widget child,required this.counter}):super(child: child);
  // 为了方便获取A的数据，在这里定一个静态方法来返回
  static A? of(BuildContext context){
    return context.dependOnInheritedWidgetOfExactType<A>();
  }
  //控制状态改变时是否更新页面
  @override
  bool updateShouldNotify(covariant A oldWidget) {
    return oldWidget.counter != this.counter;
  }
}
//###########################################
class B extends StatefulWidget {
  final Widget child;
  const B({required this.child, Key? key }) : super(key: key);

  @override
  State<B> createState() => _BState();
}
class _BState extends State<B> {
  @override
  Widget build(BuildContext context) {
    print("B Build");
    return Container(
      child: this.widget.child,
    );
  }
}
//###########################################
class C extends StatefulWidget {
  const C({ Key? key }) : super(key: key);

  @override
  State<C> createState() => _CState();
}
class _CState extends State<C> {
  //当共享数据更新（updateShouldNotify 返回true）时执行 
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print("C didChangeDependencies");
  }
  @override
  Widget build(BuildContext context) {
    print("C Build");
    return Container(
      child: Text('从 Home-> 获取到的值是：${A.of(context)?.counter}'),
    );
  }
}
```

这样就把A作为了一个数据中心，当`counter`发送改变的时候，页面就会更新。也可以在A中定义修改状态的方法，方便修改状态时调用.

> 这种方式其实并不是完整的数据中心，因为其中还确实了更新状态的方法，在`inheritedWidget`中是没有`setState`方法的，所以如果要实现一个真中的数据中心的话就需要在外层在包裹一层`StatefulWidget`组件，为数据中心提供一个初始状态数据和修改状态的方法，但是因为状态是顶层组件提供的，调用`setState`来更新状态时就意味着整个组件树都要调用build方法重构，是非常损耗性能的。

但是当更新状态的时候会发现控制台中会输出

```
B Build
C didChangeDependencies
C Build
```

也就是说 不相关的B组件也执行了`build`重建了组件，这就导致了性能损耗，可以通过`ValueNotifier`来解决

```dart
class A extends InheritedWidget {
  // int counter = 0; //需要共享的数据
  late ValueNotifier<int> _valueNotifier;
  A({required Widget child, required int counter}) : super(child: child) {
    this._valueNotifier = new ValueNotifier(counter);
  }
  ValueNotifier<int> get valueNotifier => this._valueNotifier;
  // 为了方便获取A的数据，在这里定一个静态方法来返回
  static A of(BuildContext context) {
    // return context.dependOnInheritedWidgetOfExactType<A>();
    return context.getElementForInheritedWidgetOfExactType<A>()?.widget as A;
  }

  add(int step) {
    _valueNotifier.value += step;
  }

  //控制状态改变时是否更新页面
  @override
  bool updateShouldNotify(covariant A oldWidget) {
    return false;
  }
}
//#################################
class C extends StatefulWidget {
  const C({Key? key}) : super(key: key);

  @override
  State<C> createState() => _CState();
}

class _CState extends State<C> {
  //当共享数据更新时执行
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print("C didChangeDependencies");
  }

  @override
  Widget build(BuildContext context) {
    print("C Build");
    return ValueListenableBuilder(
      valueListenable: A.of(context).valueNotifier,
      builder: (BuildContext context, int value, Widget? child) {
        return Container(
          child: Column(
            children: [
              Text('从 Home-> 获取到的值是：${value}'),
              ElevatedButton(
                  onPressed: () => {A.of(context).add(2)}, child: Text('添加2'))
            ],
          ),
        );
      },
    );
  }
}
```



## 移动端适配方案

Flutter的三棵树渲染机制和原理(https://juejin.cn/post/6916113193207070734)

https://juejin.cn/post/7056646298073563166



## bug

黄色警告空间

hasSize错误



## 参考

[codewhy 老师的flutter系列文章](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg5MDAzNzkwNA==&action=getalbum&album_id=1566028536430247937&scene=173&from_msgid=2247483705&from_itemidx=1&count=3&nolastread=1#wechat_redirect)

[老孟的 300 多个组件例子](http://laomengit.com/)

**组件目录**

https://flutter.cn/docs/development/ui/widgets

https://flutter.cn/docs/reference/widgets

https://api.flutter-io.cn/flutter/widgets/widgets-library.html#classes





