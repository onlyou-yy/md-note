## flutter

### 安装

https://segmentfault.com/a/1190000041554528#:~:text=%E5%A6%82%E6%9E%9C%E5%87%BA%E7%8E%B0%E6%8A%A5%E9%94%99%20Android%20sdkmanager%20not%20found.%20Update%20to%20the,-%3E%20%E5%8B%BE%E9%80%89Android%20SDK%20Command-line%20Tools%20%28latest%29%20-%3E%20OK

https://blog.csdn.net/MarkeyMark/article/details/111031751

https://blog.csdn.net/adorable_/article/details/116590749

### 配置

### 配置Android环境

### 设置虚拟机

### 安装编辑器插件

安装vscode 的插件，有助于提高开发效率，开发flutter 常用的插件有

![image-20220508163113492](flutter/image-20220508163113492.png)

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

+ `ListView`多节点容器组件，一般用做列表显示，可以通过`scrollDirection`定义列表的方向。不过需要注意子节点的宽度（垂直列表）/高度（水平列表）会被强制铺满适应父容器的宽度/高度，如果要设置列表的宽度，就需要设置其父容器的宽度，或者通过设置padding属性将它进行挤压，还有就是默认情况下`ListView`下不能在使用`ListView`，如果要使用需要设置`shrinkWrap`属性为`true`;

	+ `ListTitle`文章列表组件，可以定义标题和子标题，通常配`ListView`使用，也可以通过`leading`定义列表项前图标，`trailing`在列表项后定义图标
	+ `ListView.builder`这个构造函数是用来创建动态列表的，会进行循环创建，通过`itemCount`定义循环的次数，通过`itemBuilder`来定义渲染函数。

+ `GridView`网格容器列表，相当定义了`display:grid`的`div`，可以设置`crossAxisSpacing`水平间距，`mainAxisSpacing`垂直间距等，和`ListView`一样拥有`GridView.builder`构造方法进行动态列表的生成，不过要定义水平间距等其他属性需要使用`gridDelegate:SliverGridDelegateWithFixedCrossAxisCount()`定义

+ `Wrap`子节点在一行/列放不下时会自动换行，通过`direction`属性定义接单排序方向，可以用来做瀑布流布局

+ `Stack`多子节点容器组件，其中有`children`属性可以设置多个字节点的列表，子节点之间是重在一起的，相当于给每个子组件都设置了绝对定位。可以使用`Positioned`或者`Align`来控制位置，也可以使用`alignment`属性定义内容的位置

+ `Positioned`绝对定义的容器，相当于设置了`position:absolute;`的`div`

+ `Align`设置位置的容器，可以通过设置`aligment`属性操作元素在容器中的位置

+ `Container`普通的单子节点容器组件，用来创建一个可见的矩形元素，可以通过设置`decoration`来设置背景色（color），背景图（image），边框（border），阴影（boxShader）等盒子类型效果。还可以设置`margin 外边距,padding 内边距`等。

+ `AspectRatio`可以设置相对宽高比的容器，通过`aspectRatio:宽/高`可以设置当前容器宽和高的比

+ `Icon`图标容器，方便快捷得生成图标。

+ `image`相当于`img`标签，可以用来加载网络图片（Image.network）、应用资源图片（Image.asset）、文件图片（Image.file）。并且可以通过`colorBlendMode`设置混合模式，一般都是作为其他容器的子容器使用，以便于设置其他样式，如圆角等.

  + 需要注意的是，在使用`Image.asset`、`AssetImage`加载资源图片时需要在根目录下新建`images`目录，并且还要新建`images/2.0x. images/3.0x images/4.0x`文件夹，并且在`images`下放入默认图片，`images/2.0x. images/3.0x images/4.0x`下放入对应分辨率的同名图片，这样flutter就能根据设备的分辨率加载不同的图片。最后还需要在`pubspec.yaml`下配置

    ```yaml
     assets:
     	- images/a.jpeg
     	- images/2.0x/a.jpeg
     	- images/  #加载全部
    ```

+ `Card`卡片容器组件

+ `ClipOval`裁剪容器，可以将容器裁剪成其他的形状，默认是处理成圆形/椭圆型，一般会配合`Image`使用

+ `SizeBox`空白的容器组件，通常用来做占位空间。

+ `Padding`内间距容器，相当于设置了`padding:xx`的`div`

+ `Expanded`弹性盒子容器，相当于是设置了`display:flex`的`div`，可以设置`flex`比例等

+ `Divider`分割线

**工具类**

+ `Color`用来定义颜色，主要使用的方法是`Color.fromARGB(a, r, g, b)`和`Color.fromRGBO( r, g, b,o)`,还可以`Color(0xFFFFFFFF)`来定义颜色
+ `Colors`也是用来定义颜色的，不过可以直接使用定义好的静态属性`Colors.red`
+ `TextDirection`用来描述`Text`组件的文字的文字排列方向
+ `BoxDecoration`用来给盒子类型的容器设置阴影，圆角，边框等盒子容器的属性
+ `NetworkImage,FileImage,AssetImage`加载图片资源
+ `TextStyle`为`Text`组件的文字设置样式、大小、加粗等
+ `EdgeInsets`可以为盒子容器的`margin padding`提供值
+ `Transform`设置平移，旋转，缩放等transform过度效果

**MaterialApp UI库类**

+ `MaterialApp`应用程序容器，可以理解成一个应用程序的盒子，可以这个这个盒子上设置路由来管理多个页面，设置全局数据，应用主题等等。总之就相当于是一个应用实例，功能类似于Vue中的`new Vue({})`。
+ `Scaffold`应用页面级模版容器，可以设置页面的标题栏，选项卡等。
+ `AppBar`应用页面顶部选项卡容器。
+ `DefaultTabController`在`AppBar`下方的导航标签的容器，可以定义多个标签，并且可以通过`TabBarView`容器定义标签对应的页面。
+ `TabBar`标签容器，通过`tabs`定义多个标签，一般定义在`AppBar`的`bottom`或者`title`上
+ `TabBarView`定义`TabBar`中标签对应的内容。
+ `TabController` 可以定义自定义或者监听标签的行为，不过需要注意的是使用这个组件必须要使用动态组件，并且混入`SingleTickerProviderStateMixin`，而且在`TabBar TabBarView`中的`controller`都定义赋值给同一个`TabController`,之后就可以通过`TabController.addLisener`进行变化的监听了
+ `Drawer`抽屉式侧边栏，可以通过`MaterialApp`的`drawer endDrawer`分别定义左右两边的侧边栏。
+ `DrawerHeader`抽屉式侧边栏顶部容器，可以通过调用 `Navigator.pop` 关闭打开的抽屉。
+ `UserAccountDrawerHeader`抽屉式侧边栏顶部容器，提供和很多可选项，可以快速实现用户侧边栏布局。

**组件目录**

https://flutter.cn/docs/development/ui/widgets

https://flutter.cn/docs/reference/widgets

https://api.flutter-io.cn/flutter/widgets/widgets-library.html#classes



### 交互

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

+ `TextButton`凸起按钮
+ `OutlinedButton`线框按钮
+ `IconButton`图表按钮
+ `ButtonBar`按钮组
+ `FloatingActionButton`浮动按钮

这些按钮都用自己的默认宽高，如果要设置大小的话需要设置父容器的大小来实现，并且可以通过`style`来设置按钮的相关样式

### 动画

### 路由

在`Scaffold`组件中可以通过`bottomNavigationBar`属性可以定义底部的导航选项卡，这个属性的值容器是`BottomNavigationBar`，通过`items`选项定义选项卡列表项，`currentIndex`定义选中当前选中的项的索引，`onTab`定义点击选项卡的回调函数所以可以通过`BottomNavigationBar`来控制`Scaffold`组件的`body`显示的组件来实现路由的效果。需要注意的是，如果`items`超过3项就需要设置`type: BottomNavigationBarType.fixed`选项卡才会显示

```dart
class Tab extends StatefulWidget {
  const Tab({Key? key}) : super(key: key);

  @override
  State<Tab> createState() => _TabState();
}

class _TabState extends State<Tab> {
  int currentIndex = 0;
  List<Widget> pageList = [Page1(), Page2(), Page3()];
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('hello world'),
      ),
      body: this.pageList[this.currentIndex],
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

#### Navigator

在 flutter 中几乎都是组件，所以flutter 的路由跳转实际上就是将当前的页面替换成其他的页面组件

flutter 中有两种路由模式，一种是普通路由通过`Navigator`实现，一种是命名路由，通过`MaterialApp`的`routers`属性实现。

**常用的方法**

+ `Navigator.of(context).push()`普通跳转
+ `Navigator.of(context).pop()`跳转到栈定的页面，相当于返回
+ `Navigator.of(context).pushReplacementNamed()`跳转到另外一个页面，并替换当前记录
+ `Navigator.of(context).pushAndRemoveUntil()`跳转到另外一个页面，对以往的历史记录做移除操作，比如跳转到`Page1`并移除跳转记录`Navigator.of(context).pushAndRemoveUntil(MaterialPageRoute(builder:(context)=>Page1())),(route)=>route == null`

**普通路由**

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

**命名路由**

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



### 父子组件通信

### 状态管理

### flutter 生命周期

### 移动端适配方案

