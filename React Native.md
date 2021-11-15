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
> https://www.jianshu.com/p/abda49aac8b2
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



### 对于 IOS 环境