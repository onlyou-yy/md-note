## IOS原生开发

在IOS原生开发中，使用的语言是Objective-c，使用的UI框架是 UIKit 

## 创建IOS项目

使用xcode来创建项目

![1669455354164](IOS开发/1669455354164.png)

![1669456063147](IOS开发/1669456063147.png)

创建出来的项目如上图所示，在页面的左边是项目文件的目录结构，其中

+ `ViewController` 是项目视图的根管理类，用于管理根页面中所有的东西，包含按钮，文本等各个组件以及与用户的交互
+ `main.storyboard`是一个可视化拖动布局界面，在这个页面可以通过拖拽的方式给页面创建组件和添加绑定事件
+ `Assets`是项目的图片目录，项目中要使用到图片需要将拖到这个目录下
+ `LaunchScreen`是应用的启动页配置的地方
+ `info`是项目中一血基础信息的配置
+ `main.m`是项目的入口文件

然后第二块区域就是`main.storyboard`的拖动布局页面，第三块内容就是与当前布局页面相关联的页面文件，可以右上角的 `list->Assistant`打开

![1669455981265](IOS开发/1669455981265.png)

storyBoard 默认是与 ViewController 进行绑定的，当我们编辑 `storyBoard`的时候默认打开的代码就是根的 ViewController 如果需要修改可以在导航中选择自己的类

![1669557862632](IOS开发/1669557862632.png)



第四块内容就是UIView相关的信息配置了

在第四块内容顶部的左边的 `+`通过双击或拖拽可以快速在当前页面创建对应的UIView。

![1669473600592](IOS开发/1669473600592.png)

## 绑定事件和属性

通过拖拽的方式来完成把布局之后，我们还需要获取到相关的元素和给元素绑定事件，才能完成功能开发。

先选中要注册或要绑定事件的的元素，然后按住`ctrl`键，将它和代码进行连线，从而使他们能够联系起来，并自动生成代码（与定义相连，不是实现）

![1669469014713](IOS开发/1669469014713.png)

在生成代码的时候，connection为`outlet`表示定义（注册）属性，在绑定属性的时候`storage`一般都是设置成`weak`，因为 view 和 viewController 有一个互为引用的关系。`action`表示绑定事件，`arguments`是传入的参数，可以在信息中的tag传入信息

![1669474860953](IOS开发/1669474860953.png)

也可以先定义好代码之后再进行连线

![1669469675285](IOS开发/1669469675285.png)

**需要注意的是**，出现***This class is not key value coding-compliant for the key resultLabel.***，的报错一般都是因为，拖线后@property代码、事件处理方法被删除了。所以在删除来了代码后，UIView 的联系也要删除

![1669469726769](IOS开发/1669469726769.png)

```objc
#import "ViewController.h"

//一般都是定义私有的
@interface ViewController ()
- (IBAction)compute;
@property (weak, nonatomic) IBOutlet UITextField *input1;
@property (weak, nonatomic) IBOutlet UITextField *input2;
@property (weak, nonatomic) IBOutlet UILabel *resText;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
}


- (IBAction)compute {
    NSString *value1 = self.input1.text;
    NSString *value2 = self.input2.text;
    
    int num1 = [value1 intValue];
    int num2 = value2.intValue;
    
    int res = num1 + num2;
    
    self.resText.text = [NSString stringWithFormat:@"%d",res];
}
@end
```

### **IBAction和IBOutlet究竟有什么作用**

**IBAction**

从返回值角度上看，作用相当于void，只有返回值声明为IBAction的方法，才能跟storyboard中的控件进行连线

**IBOutlet**

只有声明为IBOutlet的属性，才能跟storyboard中的控件进行连线

### **Storyboard文件中箭头的含义**

程序的入口.新建一个ViewController后，设置***Is Initial View Controller***属性来让当前View Controller为默认启动项。

### **如何更换storyboard文件**

 项目 ->  General -> Deployment Info -> 改变Main.storyboard

![1669473523926](IOS开发/1669473523926.png)

### **如何退出键盘？**

1. `[第一响应者 resignFirstResponder];`，当叫出键盘的那个控件(第一响应者)调用这个方法时，就能退出键盘
2. `[self.view endEditting:YES]` 只要调用这个方法的控件内部存在第一响应者，就能退出键盘

Company Identifier和Bundle Identifier的作用



## IOS 的基本架构

### view和viewController

**UIView**

+ 屏幕上能看得见摸得着的东西就是UIView，比如屏幕上的按钮、文字、图片
+ 一般翻译叫做：视图\控件\组件
+ UIButton、UILabel、UITextField都继承自UIView
+ 每一个UIView都是一个容器，能容纳其他UIView（比如右图中的整个键盘是一个UIView，里面容纳很多小格子的数字UIView）

**UIViewController**

每一个新的界面都是一个新的UIView，在切换过程中，涉及到了UIView的创建和销毁，UIView与用户的交互，其实每当显示一个新的页面时，首先会创建一个新的 UIViewController 对象，然后创建一个对应的全屏UIView，UIViewController负责管理这个UIView，UIViewController 就是UIView的大管家

+ 负责创建、显示、销毁UIView，负责监听UIView内部事件，负责处理UIView与用户的交互
+ UIViewController内部有个UIView属性，就是它负责管理的UIView对象

***UIView与UIViewController的关系***

+ UIView只负责对数据的展示，采集用户的输入、监听用户的事件等
+ 其他的操作比如：每个UIView的创建、销毁、用户触发某个事件后的事件处理程序等这些都交给UIViewController来处理



## IOS应用的运行流程

1. 读取Main.storyboard文件
2. 创建的ViewController对象
3. 根据storyboard文件中描述创建ViewController的UIView对象
4. 将UIView对象显示到用户眼前



## 设置UIView的位置和大小

```objc
- (IBAction)change:(UIButton *)sender {
  	NSLog(@"%@"，sender.tag);//UIView 在info中绑定的tag
    CGRect btnFrame = self.headBtn.frame;
    btnFrame.origin.y -= 10;
  	btnFrame.size.width = 100;
    self.headBtn.frame = btnFrame;
}
```

OC 语法规定： **不允许直接修改对 象的结构体属性的成员**

```objc
self.headBtn.frame.origin.y -= 10;
```

frame 其实就是一个 `CGRect`，可以改变大小和位置，其实还有两个东西可以修改对象的位置和大小

+ `center`，是一个`CGPoint`，可以修改UIView的位置，不过这个位置是UIView的中心点
+ `bounds.size`，是一个`CGSize`，可以修改UIView大小



## UIView控件的常用属性方法

### 常用属性

`self`：指的是 ViewController

`subviews`：子view数组

```objc
self.view.subviews[0].backgroundColor = [UIColor redColor];
```

`superview`：父view

```objc
self.input1.superview.backgroundColor = [UIColor yellowColor];
```

`frame`获取view的大小和位置

```objc
CGRect frame = self.button.frame;
frame.origin.x;
frame.origin.y;
frame.size.width;
frame.size.height;
```

`enter`获取view的中心点位置

```objc
CGPoint center = self.button.center;
center.x;
center.y;
```

`bounds`获取view大小

```objc
CGRect bounds = self.button.bounds;
bounds.width;
bounds.height;
```

`tag` view标志，可以用来区分不同的view

```objc
button.tag = 100;
```



### 常用方法

`addSubview` 添加子控件

```objc
self.view.addSubview(button);
```

`removeFromSuperview` 从父控件删除

```objc
[button removeFromSuperview];
```

`viewWithTag` 如果给View设置了`tag`属性，就可以通过这个属性来查找到对应的View

```objc
UIButton *button = (UIButton *)[self.view viewWithTag:100];
```

`addTarget` 为view 添加事件

```objc
// self表示你 参数中所加的方法（farmImage：）是加到button本身的
// handleClick 是当前ViewController中的方法
[button addTarget:self action:@selector(handleClick) forControlEvent:UIControlEventTouchUpInside]
-(void)handleClick{NSLog(@"click handle")}

// 如果要传递参数可以将参数绑定或者设置在 view 上，因为在绑定的方法中会将当前view传入
button.tag = 100;
[button addTarget:self action:@selector(handleClick:) forControlEvent:UIControlEventTouchUpInside]
-(void)handleClick:(id)sender{//sender 就是button
  int tag = sender.tag;
  NSLog(@"click handle tag",tag);
}
```



## 动态创建控件

可以通过代码的方式动态创建控件，`UIView`也是一个控件，不过这个控件是空白的，一般用来做容器使用

```objc
UIView *cont = [UIView new];
UIButton *button = [UIButton new];
[cont addSubview:button];
```

```objc
#import "ViewController.h"
@interface ViewController ()
@end
@implementation ViewController
// 当要显示一个页面的时候，首先创建这个界面对应的控制器
// 控制器创建好以后，接着创建控制器所管理的个View，当这个View加载完成之后就开始执行 viewDidLoad 方法
// 所以只要viewDidLoad方法执行，就表示控制器所管理的view创建好了
- (void)viewDidLoad{
	[super viewDidLoad];
	
	//创建控件 按钮
	UIButton *button = [[UIButton alloc] init];
	//设置文本和状态，使用默认状态
	[button setTitle:@"click me" forState:UIControlStateNormal];
	//高亮状态下的按钮
	[button setTitle:@"light" forState:UIControlStateHighlighted];
	
	//设置文本颜色
	[button setTitleColor:[UIColor redColor] forState:UIControlStateNormal];
	
	//设置默认状态下按钮的背景,会在 Assets.中查找
  //如果图片是 png 就可以省略后缀
	UIImage *imgNormal = [UIImage imageNamed:@"btn_01"];
	UIImage *imgLight = [UIImage imageNamed:@"btn_02"];
	[button setBackgroundImage:imgNormal forState:UIControlNormal];
	[button setBackgroundImage:imgLight forState:UIControlStateHighlighted];
	
	//设置控件的大小位置
	button.frame = CGRectMake(10,10,50,50);
	
	//为控件添加事件
	[button addTarget:self action:@selector(handleClick) forControlEvent:UIControlEventTouchUpInside]
	
	//将控制器添加到 view
	[self.view addSubView:button];
}
- (void)handleClick{
	NSLLog(@"click button");
}
@end
```



## 平移

```objc
//相对于原来位置
self.button.transform = CGAffineTransformMakeTranslation(0,-50);//向上平移
//相对于原来的transform进行平移
self.button.transform = CGAffineTransformTranslate(self.button.transform,0,50);//向下平移
```

**注意：** `transform`相对于原来位置时，如果联系执行上面代码两次，button不会上移 100 ，因为本来就已经是 50 了。相对于原来的transform进行平移则可以移动100



## 加载文件

### 加载`.plist`文件

在加载文件的时候，必须要传递一个路径的参数，如果传递的是一个文件名就会默认在`Assets`目录中找，如果传递一个绝对路径就会在指定的绝对路径中找。

ios应用是被安装到不用手机使用的，所以他们的根目录是不确定的，所以我们在读取项目中文件的时候需要先获取当前项目的根路径，通过`[NSBundle mainBundle];`可以获取到项目根路径

```objc
- (void)data{
  if(_data == nil){
    NSString *path = [[NSzBundle mainBundle] pathForResource:@"data.plist" ofType:nil];
    NSArray *arr = [NSArray arrayWithContentsOfFile:path];
    _data = arr;
  }
  return _data;
}
```

### 加载 Image 图片

加载图片有两种方式

使用`[UIImage imageNamed:imgName]`

```objc
//如果图片是 png 就可以省略后缀，会在Assets中查找img.png
UIImage img = [UIImage imageNamed:@"img"];
```

这种方式的特点是，加载进来的图片**会被缓存起来，不会被释放**

第二中方式是使用`[UIImage imageWithContentsOfFile:path]`，其中 path 一般都是绝对路径

```objc
NSString *path = [[NSBundle mainBundle] pathForResource:@"img" ofType:nil];
UIImage img = [UIImage imageWithContentsOfFile:path];
```

这种方式图片就不会被缓存，在加载完之后就会释放内存，下次新的数据就可以使用这块内存了



## 动画

### 简易动画

简易动画大致有2种方式

头尾式

```objc
[UIView beginAnimations:nil context:nil]; // 开启动画
[UIView setAnimationDuration:1]; // 设置动画执行时间
/** 需要执行动画的代码 **/
self.headBtn.frame = btnFrame;
[UIView commitAnimations]; // 提交动画
```

block式

```objc
[UIView animateWithDuration:0.5 animations:^{
    /** 需要执行动画的代码 **/
  self.headBtn.frame = btnFrame;
}];
```



### 帧动画

帧动画相关属性和方法，要使用 `UIImageView`作为控件

```objc
// 要播放的序列帧图片数组
@property(nonatomic,copy) NSArray *animationImages;
// 持续时间
@property(nonatomic) NSTimeInterval animationDuration;
// 重复次数
@property(nonatomic) NSInteger animationRepeatCount;

// 开始播放
-(void)startAnimating;
// 停止动画
-(void)stopAnimating;
// 是否正在播放动画
-(void)isAnimating;
```

```objc
-(void)animate{
  NSMutableArray *imgs = [];
  for(int i = 0;i < 11;i++){
    NSString *name = [NSString stringWithFormat:@"img_%02d",i];
    NSString *path = [[NSBundle mainBundle] pathForResource:name ofType:nil];
    UIImage img = [UIImage imageWithContentsOfFile:path];
    [imgs addObject:img];
  }
  self.frameImage.animationImages = imgs;
  self.frameImage.animationDuration = 3;
  self.frameImage.animationRepeatCount = 1;
  
  [self.frameImage startAnimating];
  
  //动画执行完成之后要修改存储图片的数组
  [self.frameImage performSelector:@selector(setAnimationImages:) withObject:nil afterDelay:3]
}
```



## xib 和 storyBoard

xib 和 storyBoard 一样都是用来描述软件界面的文件，不同的是 xib 是一个比较轻量级的view描述文件，适合用来写组件。而 storyBoard  是一个比较重的页面描述文件，适合用来做页面，以及页面的跳转关系。

xib 也可以进行可视化的开发，在加载 xib 的时候，系统会自动生成对应的代码来创建组件类。

### 通过xib 创建组件

假设我们需要定义一个 头像`AvatarView`组件（模板 | 控件），因为要区分自己的类和系统的类，避免重名，我们使用前缀进行规范，

所以我们创建一个新的 xib 文本，可以在xcode

创建`MYAvatar.h`和`MYAvatar.m`

```objc
// MYAvatar.h
#import <Foundation/Foundation.h>
@interface MYAvatar : NSObject{
  @property(nonatomic,copy)NSString *name;
  @property(nonatomic,copy)NSString *icon;
}
-(instancetype)initWithDict:(NSDictionary *)dict;
+(instancetype)avatarWithDict:(NSDictionary *)dict;
@end
```

```objc
// MYAvatar.m
#import "MYAvatar.h"
@implementation MYAvatar
-(instancetype)initWithDict:(NSDictionary *)dict{
  if(self = [super init]){
    self.name = dict[@"name"];
    self.icon = dict[@"icon"];
  }
  return self;
}
+(instancetype)avatarWithDict:(NSDictionary *)dict{
  return [[self alloc] initWithDict:dict];
}
@end
```





## 生命周期

https://www.jianshu.com/p/fcfbd4919b0b

https://blog.csdn.net/qq_27597629/article/details/99962258

https://www.jianshu.com/p/66dff5221f4b





## 文档

[中文简要文档](https://developer.apple.com/cn/documentation/)

[苹果开发文档](https://developer.apple.com/documentation/technologies)

