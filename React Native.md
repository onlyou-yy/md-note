## app的分类

目前 app 分为原生app ，web App，以及Hybird App（混合app），他们三者的主要区别是

**原生app**：基于手机本地环境进行开发，可以调度、访问、使用手机上资源，但是对于不同的平台需要使用不同的语言和工具进行开发，开发难度和开发周期会比较长。

**web app**：使用 html5，css，javascript 等纯前端技术进行开发，开发难度低，且开发周期短，但是web app是运行在手机浏览器上的并且资源都存储在云端服务器上，这也导致了每次打开app 都需要去请求资源，用户体验略差，并且也不能调用手机上的 资源和功能（位置，摄像头等）。

**Hybird App**：结合了原生app资源使用自由和web app 开发成本低的优点，开发成本低，可以调用手机上的资源。Hybird App 的底层使用的是一套编写好的手机原生框架，可以开放给其他的框架进行接入，所以开发 Hybird App 的方式也很多 ，如 react native，uniapp，taro 等。

![1610768109783](React Native/1610768109783.png)



## **环境搭建**

跟着React Native中文网的[搭建开发环境](https://reactnative.cn/docs/environment-setup)教程来搭建就可以了，不过有几个需要注意的地方

### 对于 Android  SDK

按照教程来搭建基本不会有什么问题，最主要的问题就是在安装Android SDK的时候，如果没有使用科学上网工具的话，一般是下载不成功的，所以最好是根据教程上写的用科学上网工具来代理下载，一般可用的科学上网工具有很多比如[蓝灯](https://github.com/getlantern/lantern)，[佛跳墙](https://github.com/getfotiaoqiang/fotiaoqiang)等等。

### 对于 Android 模拟器

我觉得 Android Studio 里面的模拟器已经很够用了不用安装 genymotion 等其他的模拟器了，可以在Android Studio中的 `Tools -> Android AVD`中创建和管理模拟器。将项目的`AwesomeProject/android`导入到项目中编译并运行到模拟器，然后再运行`yarn android`即可。

> **Android的一些常用命令**
>
> + `android list avd`  查看现在有哪些模拟器可以使用，
> + `emulator -avd 模拟器名字`或者`emulator @模拟器名字`  启动相应的模拟器。
> + `adb devices`  查看当前运行中的模拟器
> + `android list target` 列出当前可用的SDK版本
> + `android create avd -n <name> -t <targetID> [-<option> <value>]` 创建模拟器，最好是在Android studio 中创建
> + `adb push D:\test.txt /sdcard/` 将D盘的`test.txt`文件添加到模拟器的`/sdcard/`下
> + `adb pull /sdcard/test.txt D:\ `将模拟器的`test.txt` 文件复制到`D:\`
> + `adb shell pm setInstallLocation 1`默认安装在手机内存
> + `adb shell pm setInstallLocation 2`默认安装在SD卡
> + `adb shell input keyevent 82` 打开模拟器调试面板
>
> 其他的常用[命令….](https://blog.csdn.net/gabbzang/article/details/9393981)

### 其他的一些问题

> [VSCode终端不能使用命令](https://blog.csdn.net/weixin_51781586/article/details/114577951)
>
> [使用第三方模拟器运行react native项目](https://www.jianshu.com/p/3ed09488058c)
>
> [使用VsCode开发调试React Native笔记](https://www.jianshu.com/p/dfa3d8ea6d90)
>
> [react-native系列(3)入门篇：使用VSCode及RN的代码调试过程](https://blog.csdn.net/zeping891103/article/details/85860149)
>
> [RN Android "unabled to connect with remote debugger"](https://www.jianshu.com/p/abda49aac8b2)



### 对于 IOS 环境

只要跟着React Native中文网来做的话都不会有太大的问题，最难搞的就是使用`pod install`的时候出现的一些不明所以的问题.

**错误一：**

```shell
[!] Oh no, an error occurred.
Search for existing github issues similar to yours:
https://github.com/CocoaPods/CocoaPods/search?q=invalid+byte+sequence+in+US-ASCII&type=Issues
```

主要原因应该是，Profile 文件的编码方式被改变了，所以修改一下即可

```shell
$ export LANG=en_US.UTF-8
$ export LANGUAGE=en_US.UTF-8
$ export LC_ALL=en_US.UTF-8
```



**错误二：**

```shell
[!] Oh no, an error occurred.  
  
Search for existing GitHub issues similar to yours:  
https://github.com/CocoaPods/CocoaPods/search?q=No+such+file+or+directory+-+%2FUsers%2Frwx-mac%2FDesktop%2FHe%2FHeAmap%2FPods%2FAMapSearch%2FAMapSearchKit.framework%2FResources&type=Issues  
```

主要的原因应该是pod的版本问题

解决方式一：从新安装pod的repos/master

```shell
cd ~/.cocoapods/repos
rm -rf master
pod setup
```

解决方式二：更新cocoapods

```shell
$ sudo gem update --system
$ sudo gem install cocoapods -n/usr/local/bin
```

解决方式三：重新安装cocoapods

```shell
sudo gem uninstall cocoapods
sudo gem install cocoapods
pod setup
```

解决方式四：安装0.38.1版本cocoapods

```shell
sudo gem uninstall cocoapods
sudo gem install cocoapods -v 0.38.1
sudo rm -rf ~/.cocoapods && sudo rm -fr ~/.cocoapods/repos/master && pod setup && pod install
```



**错误三：**

<img src="../md-note/React Native/react-native.png" alt="image-20211115120343028" style="zoom:50%;" />

这个错误一般是在安装某个模块的时候出现的，是因为网络差导致的，毕竟pod也是优先使用国外的资源的，所以要解决这个问题可以修改数据源

对于旧版的 CocoaPods 1.3之前 可以使用如下方法使用 tuna 的镜像：

```shell
$ pod repo remove master
$ pod repo add master https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git
$ pod repo update
```

新版的 CocoaPods 1.3之后 不允许用pod repo add直接添加master库了，但是依然可以：

```shell
$ cd ~/.cocoapods/repos 
$ pod repo remove master
$ git clone https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git master
```

最后进入自己的工程，在自己工程的podfile第一行加上：

```
source 'https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git'
```

如果你用的是cocoapods1.8，可以还会报CDN错误，可以再执行一下这个

```shell
pod repo remove trunk
```

如果这样改完之后下载速度还是很慢的话建议使用一下git/npm代理工具进行加速，如边车[dev-sidecar](https://gitee.com/docmirror/dev-sidecar)



**错误四：**

```shell
[!] /bin/bash -c 
set -e
#!/bin/bash
# Copyright (c) Facebook, Inc. and its affiliates.
#
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.

set -e

PLATFORM_NAME="${PLATFORM_NAME:-iphoneos}"
CURRENT_ARCH="${CURRENT_ARCH}"
.....
```

解决方式

```shell
sudo xcode-select --switch /Applications/Xcode.app
```



## 关于安装第三方库

对于一些不依赖于与原生代码的库可以直接使用`npm`或`yarn`安装之后使用，有一些库基于一些原生代码实现，你必须把这些文件添加到你的应用，否则应用会在你使用这些库的时候产生报错

下载某个库到本地

```shell
npm install ******
```

链接某个库到项目中

```shell
react-native link *****
```

React Native 0.60 及更高版本链接是自动的，但是对于Mac开发IOS可能还需要在项目中运行`cd ios && pod install`进行链接



## 关于布局

在react-native中大多数容器默认已经开启为`display:flex;`布局，并且默认的排列方向（主轴方向）是`flex-directive:column`垂直方向。所以在react-native中可以直接给元素定义flex的相关属性。



## 关于样式

在react-native中支持使用样式类名（也就是className）来定义样式，仅支持通过style来设置样式，并且样式的设置需要使用对象来设置`style={{color:'white',backgroundColor:'blue'}}`，react-native中也不全支持css属性，每个组件也都有自己特定的属性，所以具体样式还是要参考文档。

### 样式的继承

react-native提倡的是每个组件都相互独立互不影响，所以react-native中大多数属性是不可继承的。不过也有例外，React Native 实际上还是有一部分样式继承的实现，不过仅限于文本标签的子树。在下面的代码里，第二部分会在加粗的同时又显示为红色：

```react
<Text style={{ fontWeight: 'bold' }}>
  I am bold
  <Text style={{ color: 'red' }}>and red</Text>
</Text>
```

### 抽离公共样式

有时候多个组件使用的样式是同一基本样式，但是组件之间又略有不同，这时候可以使用数组的方式来设置组件的样式，将公共的样式抽离出来然后添加到各组件的样式数组，并且后面的样式回覆盖前面的样式。

```react
const commonStyle = {fontSize:30,color:'skyblue'};
<Text style={[backgroundColor:'yellow',commonStyle]}>and red</Text>
<Text style={[backgroundColor:'green',commonStyle]}>and red</Text>
```



## 基础组件的使用

### View

View 组件相当于是 div 标签，就是一个普通的容器，不过不可以插入文本节点`<View>hello</View>`，文本节点需要使用Text来包裹`<View><Text>hello</Text></View>`

### Text

Text 组件相当于是一个 span 标签，不过里面的布局不是按flexbox进行布局的，而是文本排列布局。这意味着`<Text>`内部的元素不再是一个个矩形，而可能会在行末进行折叠。

### Image与ImageBackground

两个组件都是用来加载图片的，不过Image加载的是普通图片，ImageBackground用来加载背景图片，不过Image标签中不能插入内容，而ImageBackground则是相当于一个带背景的容器。通过设置`source`来确定图片，`source`接收的是一个对象值

对于本地的图片，需要使用`require`来导入图片（`require`返回的也是一个对象，并且也包含了宽高）

```react
<Image source={require('./assets/a.png')}>
<ImageBackground source={require('./assets/a.png')}>
```

对于网络图片或者base64的图片可以使用`{uri:'https://picsum.photos',width:30,height:30}`来设置（记得要设置宽高，否则图片不会显示），这里的`width height`指的是图片的宽高，而不是容器的宽高，容器的宽高需要通过`style`来设置

```react
<Image source={{uri:'https://picsum.photos',width:30,height:30}}>
<ImageBackground source={{uri:'https://picsum.photos'},style={{width:30,height:30}}}>
```

如果图片大小和容器大小不一致，可以使用`style={{resizeMode:'cover'}}`或者`<Image source={require('./assets/a.png') resizeMethod='scale'}>`来设置图片的缩放模式



### Button

Button组件是一个简单的跨平台的按钮组件，它是`TouchableOpacity、TouchableHighlight`或`TouchableNativeFeedback`组件的上层封装，可以理解为`TouchableOpacity、TouchableHighlight`或`TouchableNativeFeedback`是仅有反馈而仅带基础样式的组件，而Button则是带有反馈并且有定制样式的组件。需要注意的是Button在 IOS 和 Android 上的样式表现是不一样的，这样就会让界面在IOS和Android上表现不同意，为了解决这个问题，一般需要我们自己在`TouchableOpacity、TouchableHighlight`或`TouchableNativeFeedback`上封装一个样式统一的组件，或者使用第三方button库（如`react-native-action-button`），或者使用第三方UI库（`react-native-elements`、`ant-design-mobile-rn`）



### ScrollView

是一个滚动组件，这个容器支持纵向滚动，也支持横向滚动，但是必须给组件设置宽高，当内容的宽高超出了容器的宽高才会开启滚动。

**对于子元素容器**
且子元素将会被全部包裹到一个容器元素中，可以通过`contentContainerStyle`来设置容器的样式。考虑到性能问题，一般用来做引导图、轮播图等数据量不大的。

**下拉刷新操作**
对于垂直方向的列表，`ScrollView` 提供了一个`refreshControl`的属性来给提供下拉刷新的功能，`refreshControl`接受一个组件作为值，这个组件必须有两个属性`refreshing`（是否处于刷新状态）和`onRefresh`（刷新时的事件），`react-native`也已经提供给了这个组件`RefreshControl`。



### FlatList | SectionList | VirtualizedList

`ScrollView` 组件比较适合处理数据量少（节点少）的情况，而如果处理数据量比价大的时候就容易出现卡顿的现象，对于数据量大节点多的情况应该使用`FlatList | SectionList | VirtualizedList`。`FlatList | SectionList`都继承于`VirtualizedList`，并且同时拥有`VirtualizedList`与`ScrollView`的所有`props`。

`FlatList`与`SectionList`相差无几，不同的是`SectionList`支持分组，`FlatList`支持多列布局。



### StatusBar

状态栏，一个应用中只有一个状态栏，如果有多个的话后面的会把前面的覆盖掉。



## 网路请求

在软件开发里面不存在跨域的问题（跨域主要是因为浏览ajax引擎的同源策略导致的，而在应用中不存在ajax引擎所以不会有跨越问题）。在react-native中网络请求不再是使用xhr了，而是使用[fetch](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)来发送网络请求了。不过仍然可以使用第三方的网络请求框架如[frisbee](https://github.com/niftylettuce/frisbee)或是[axios](https://github.com/mzabriskie/axios)等。

fetch的简单使用：发送get请求

```js
fetch('https://mywebsite.com/mydata.json').then(res=>res.json()).then(res=>console.log(res));
```

发送post请求

```js
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    firstParam: 'yourValue',
    secondParam: 'yourOtherValue'
  })
});
```



使用axios的话需要先安装

```shell
npm i axios -S
```

然后参考文档进行封装使用就好了。



## 使用导航（react-navigation）

目前 react-native 常用的导航库有`react-navigation`和`react-native-navigation`，使用导航就可以开始进行多页面的开发以及进行页面的跳转了。

### react-navigator

**安装：**最好是安装[官网教程](https://reactnavigation.org/docs/getting-started)来进行安装，因为每个版本要安装的东西都不太一样，显示的是6.x的

```shell
yarn add @react-navigation/native @react-navigation/native-stack react-native-screens react-native-safe-area-context 
```

如果用的是react-native0.59及以下的版本还需要运行

```shell
react-native link @react-navigation/native
react-native link @react-navigation/native-stack
react-native link react-native-screens
react-native link react-native-safe-area-context
```

> 如果还需要进行IOS端的开发，还需要运行`cd ios && pod install`（记得确保网络稳定，不然会失败）

进行安卓开发还需要在`android/app/src/main/java/<your package name>/MainActivity.java`文件下添加

```java
import android.os.Bundle;
------------------------------------------
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(null);
}
```



**基本使用**





## 使用状态管理（redux/mobx）



## UI框架



## 常用文档

[React-Native 中文网](https://reactnative.cn/)

[React-Native 学习指南](https://github.com/reactnativecn/react-native-guide)

[React-Native 组件库](https://js.coach/)

[React Native Directory](https://reactnative.directory/)



