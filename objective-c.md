## objective-c 基本概念

objective-c 简称 oc ，是在C的基础上新增了1小部分面向对戏那个的语法，将C的复杂、繁琐的语法封装的更为简单，而且OC是完全兼容 C
+ `#import` 指令：同1个文件无论#import多少次. 最终都只会包含1次 不会出现重复包含的情况.
+ NSLog函数：自动换行，会输出一些其他的调试信息.

## 数据类型
oc 中的数据类型有是完全兼容 c 中的类型的，在oc 中也有很多自己的数据类型
+ BOOL : YES ｜ NO
+ Boolean : true | false
+ class : 类
+ id : 万能指针
+ nil : 空指针
+ SEL : 选择器
+ block : 块

## 类和对象

### 定义类

类的声明
```objc
@interface 类名 : NSObject
{
  //属性默认是私有的
  //使用@public 之后的变量都是public的
  类型 属性;
}
- (方法返回值类型)方法名;
- (方法返回值类型)方法名:(参数类型)参数名1 :(参数类型)参数名2;
- (方法返回值类型)方法名:(参数类型)参数名1 and:(参数类型)参数名2;
@end
```
类的实现
```objc
@implementation 类名
- (方法返回值类型)方法名{
  //todo...
}
- (方法返回值类型)方法名:(参数类型)参数名1 :(参数类型)参数名2{
  //todo...
}
- (方法返回值类型)方法名:(参数类型)参数名1 and:(参数类型)参数名2{
  //todo...
}
@end
```
使用
```objc
#import <Foundation/Foundation.h>

@interface Person : NSObject
{
  //属性默认是私有的
  //使用@public 之后的变量都是public的
  @public
  NSString *_name;
  int _age;
  float _height;
  BOOL _sex;
}
- (void)say;
- (void)saySomething:(NSString *)something;
- (void)eatSomething:(int)count :(NSString *)something;
- (void)eatWith:(NSString *)food1 and:(NSString *)food2;
@end

@implementation Person
- (void)say
{
  // 在方法中可以直接访问 属性
  // 谁调用了这个方法，那么访问到的变量就是谁的
  NSLog(@"Person say 'my name is %@'",_name);
}
/*
方法名叫 saySomething:
参数 (NSString *)something
*/
- (void)saySomething:(NSString *)something
{
  NSLog(@"Person say %@",something);
}
/*
方法名叫 eatSomething: :
参数 (int)count (NSString *)something
*/
- (void)eatSomething:(int)count :(NSString *)something
{
  NSLog(@"Person eat %d %@",count,something);
}
/*
方法名叫 eatWith: and:
参数 (NSString *)food1 and:(NSString *)food2
*/
- (void)eatWith:(NSString *)food1 and:(NSString *)food2
{
  NSLog(@"Person eat %@ and %@",food1,food2);
}
@end


int main(int args,char *argv[]){
  //实例化就是调用 new方法的过程
  Person *p = [Person new];
  p->_name = @"jack";
  p->_age = 1;
  (*p)._height = 180.f;

  NSLog(@"Person _name:%@",p->_name);
  NSLog(@"Person _age:%d",p->_age);
  NSLog(@"Person _age:%f",p->_height);

  [p say];
  [p saySomething:@"爱你"];
  [p eatSomething:3 :@"fish"];
  [p eatWith:@"coffe" and:@"fish"];

  return 0;
}
```
**总结**
+ 访问属性 可以使用 `对象->属性名` 或者 `指针的. (*对象).属性名`
+ 访问属性需要用 `[对象 方法名:参数1 :参数2]`

+ 对于属性的名方法和规范
  + `类型 _变量名`;

+ 对于方法的命名方法和规范
  + `- (返回类型)方法名:(参数类型)参数名 :(参数类型)参数名`
  + 方法名的命名规范：`动作With:(参数类型)参数名 and:(参数类型)参数名`
+ 在类中可以使用 `self`来访问自己的方法，`self`是一个指针，指向类的实例对象

## 对象在内存中的存储

### 内存中的五大区域
+ 栈：存储局部变量.
+ 堆：程序员手动申请的字节空间 malloc calloc realloc函数.
+ BSS段：存储未被初始化的全局变量 静态变量.
+ 数据段(常量区)：存储已被初始化的全局 静态变量 常量数据.
+ 代码段：存储代码. 存储程序的代码.

### 类加载
+ 在创建对象的时候 肯定是需要访问类的
+ 在创建对象的时候 肯定是需要访问类的

在程序运行期间 当某个类第1次被访问到的时候. 会将这个类存储到内存中的代码段区域.这个过程叫做类加载.只有类在第1次被访问的时候,才会做类加载. 一旦类被加载到代码段以后. 直到程序结束的时候才会被释放.

### 对象在内存中究竟是如何存储的
```objc
Person *p1 = [Person new];
```
+ Person *p1; 会在栈内存中申请1块空间. 在栈内存中声明1个Person类型的指针变量p1。p1是1个指针变量.  那么只能存储地址.
+ `[Person new]` 真正在内存中创建对象的其实是这句代码.

**new做的事情**
+ 在堆内存中申请1块合适大小的空间.
+ 在这个空间中根据类的模板创建对象 
  + 类模板中定义了什么属性.就把这些属性依次的声明在对象之中.对象中还有另外1个属性 叫做isa 是1个指针. 指向对象所属的类在代码段中的地址
+ 初始化对象的属性
  + 如果属性的类型是基本数据类型 那么就赋值为0
  + 如果属性的类型是C语言的指针类型 那么就赋值为NULL
  + 如果属性的类型是OC的类指针类型9 那么就赋值为nil
+ 返回对象的地址

### 为什么不把方法存储在对象之中?
因为每1个对象的方法的代码实现都是一模一样的  没有必要为每1个对象都保存1个方法 这样的话就太浪费空间了.既然都一样 那么就只保持1份.

### nil与NULL
+ NULL：只能作为指针变量的值. 如果1个指针变量的值是NULL值代表. 代表这个指针不指向内存中的任何1块空间，NULL其实等价于0  NULL其实是1个宏. 就是0
+ nil：只能作为指针变量的值. 代表这个指针变量不指向内存中的任何空间. nil其实也等价于0 也是1个宏. 就是0. 所以, NULL和nil其实是一样的 。
+ 使用建议：虽然使用NULL的地方可以是nil 使用 nil的地方可以使用NULL 但是不建议大家去随便使用.

**C指针用NULL**
```objc
int *p1 = NULL; //p1指针不指向内存中的任何1块空间.
```
**OC的类指针用nil**
```objc
Person *p1 = nil; p1指针不指向任何对象.
```

**注意**

+ 对象中只有属性,而没有方法. 自己类的属性外加1个isa指针指向代码段中的类.
+ 如何访问对象的属性，
  + 指针名->属性名; 根据指针 找到指针指向的对象 再找到对象中的属性来访问
+ 如何调用方法
  + `[指针名 方法名];`先根据指针名找到对象,对象发现要调用方法 再根据对象的isa指针找到类.然后调用类里的方法.



## 分组导航标记

分组导航标记:

+ `#pragma mark 分组名`
	​       就会在导航条对应的位置显示1个标题.
+  `#pragma mark -`
	       就会在导航条对应的位置显示1条水平分隔线.
+  `#pragma mark - 分组名`
	       就会在导航条对应的位置先产生1条水平分割线.再显示标题.



## 方法与函数

我们之前在C中学习的函数,就叫做函数，在OC类中写的方法.就叫做方法

### 相同点

都是用来封装1段代码的. 将1段代码封装在其中, 表示1个相对独立的功能， 函数或者方法只要被调用.那么封装在其中的代码就会被自动执行.

### 不同点

+ 语法不同.
+ 定义的位置不一样，OC方法的声明只能写在@interface的大括号的外面,实现只能写在@implementation之中，函数除了在函数的内部和@interface的大括号之中 其他的地方都是可以写，就算把函数写在类中 这个函数仍然不属于类 所以创建的对象中也没有这个函数，注意; 函数不要写到类中.虽然这样是可以的 但是你千万不要这么做 因为这么做是极度的不规范的.
+ 调用的方式也不一样，函数可以直接调用，但是 方法必须要先创建对象 通过对象来调用.
+ 方法数是属于类的

### 注意点

+ `@interface`是类的声明. `@implementation`是类的实现 他们之间不能相互嵌套
+ 类必须要先声明然后再实现
+ 类的声明和实现必须都要有 就算没有方法 类的实现也不必不可少的
+ 类的声明必须要放在使用类的前面  实现可以放在使用类的后面
+ 声明类的时候 类的声明和实现必须要同时存在，特殊情况下可以只有实现 没有声明（虽然可以这样,但是我们平时在写类的时候千万不要这么写 因为这么写是极度不规范的）
+ 属性名一定要以下划线开头 这是规范. 否则后面的知识点你就对不上号， 类名 每1个单词的首字母大写
+ 属性不允许声明的时候初始化 在为类写1个属性的时候 不允许在声明属性的时候为属性赋值.
+ OC方法必须要创建对象通过对象名来调用
+ 方法只有声明 没有实现
	+ 如果方法只有声明 没有实现  编译器会给1个警告  不会报错.
	+ 如果指针指向的对象 有方法的声明 而没有方法的实现 那么这个时候通过指针来调用这个方法，在运行的时候  就会报错（`unrecognized selector sent to instance 0x100420510` 只要你看到了这个错误.说明要么对象中根本就没有这个方法. 要么只有方法的声明而没有方法的实现）



## 多文件开发

把1个类写在1个模块之中. 而1个模块至少包含两个文件.

` .h 头文件`

写的类声明 因为要用到Foundation框架中的类 NSObject 所以在这个头文件中要引入Foundation框架的头文件 然后将类的声明的部分写在.h文件中

` .m 实现文件`

先引入模块的头文件 这样才会有类的声明,再写上类的实现。如果要用到类. **只需要引入这个了模块的头文件就可以直接使用了**

> **添加类模块的更简洁的方式**
>
>  `NewFile->Cocoa Class` 自动生成模块文件  .h  .m 。自动的将类的声明和实现写好



### 枚举或者结构体定义在什么地方

如果只是1个类要用。那么就定义在这个类的头文件中。 如果多个类要用，那么就定义在单的头文中，谁要用谁就去引用。



https://blog.csdn.net/panjiye82?type=blog&year=2022&month=05

https://www.bilibili.com/video/BV1NJ411T78u?p=10&spm_id_from=pageDriver&vd_source=41ed998ac767425fb616fd9071ce9682