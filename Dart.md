# Dart

flutter 用Dart语言来进行编写的是，所以在学习flutter 之后需要先对Dart进行了解，Dart是谷歌公司开发的一种针对客户优化的语言，可在任何平台上开发快速的应用程序。Dart 语言是类型安全的；它使用静态类型检查来确保变量的值 **始终** 与变量的静态类型相匹配。如果学习过typeScript、java 等强类型语言的话会接得他们非常相似。



## Dart 程序执行入口

Dart程序必须要用一个入口函数`void main(){}`,Dart将从这个函数开始执行程序。



## Dart 中的数据类型

在 dart 中有个概念——***一切数据皆为对象***，也就是所有变量引用的都是 **对象**，每个对象都是一个 **类** 的实例。数字、函数以及 `null` 都是对象。除去 `null` 以外， 所有的类都继承于 **Object** 类。

Dart 常用的内置类型有

+ `int`整型
+ `double`浮点数
+ `String`字符和字符串
+ `bool`布尔值
+ `List`列表，类式于js中数组，但是List 中只能放同一类型的数据
+ `Set`与js中的Set一致
+ `Map`与js中的Map一致



## Dart 中的定义变量

因为 dart 是一种强类型语言，所以在定义变量的时候需要指定变量的类型

```dart
String str = 'string';
int num1 = 12;
double num2 = 12.0;
bool b = true;
```

如果不想自己指定数据类型的话还可以使用`var`来定义变量，这样的话Dart自动推断数据类型。

```dart
var val = 15;
```

在定义`List Set Map`的时候，可以指定存储的元素的类型

```dart
List ls1 = ['jack','rocy'];
List<int> ls2 = [1,2];
List ls2 = <int>[1,2];

Set setVal1 = {'jack','rocy','junp'};
Set<double> setVal2 = {1.2,2.5,3.4};
Set setVal2 = <double>{1.2,2.5,3.4};

Map mapVal1 = {'jack':12,'rocy':13};
Map<String,int> mapVal2 = {'jack':12,'rocy':13};
Map mapVal2 = <String,int>{'jack':12,'rocy':13};
```

需要注意的是，在定义变量的时候必须要先赋初始值，不然Dart 就会认为这个值是 null，不符合你定义的类型导致报错。也可以使用`int? num;`来告诉Dart这个变量可以为null



## Dart 变量修饰关键字

在dart中可以使用一些关键字来对变量做一些修饰，比如定义一个常量需要使用`const`来指定。

+ `final`，表示变量是一个常量，且在一次赋值之后就不能再改变，在定义的时候可以先不赋值，且如果是`List Map Set`这些引用类型的数据的话之后还可以添加、删除、修改里面元素数据
+ `const`，也表示一个常量，但是定义的时候必须要进行赋值。且也不能对引用类型的数据里面的元素进行操作了。
+ `late`，表示当前变量不进行赋值那么快，之后再赋值，在时候的时候才进行赋值。



## Dart 中的函数

在dart 中定义函数类似于C语言`返回类型 函数名(){}`，如果该函数没有返回值的时候返回类型就是`void`

```dart
String foo(){
  return 'hello';
}
```

**函数参数**

在Dart 中函数的参数除了固定的参数外，还有可选参数，可选两种形式，一种是可选位置参数，一种是可选命名参数，

**可选位置参数**

可选位置参数通过`[]`定义，在里面的参数在调用函数的时候可以不传。

```dart
String strAdd(String a,String b,[int? i1,int i2 = 12,int? i3]){
 	return '$a,$b,$i1,$i2,$i3';
}
print(strAdd('hh', 'gg', 11));
```

**可选命名参数**

可选命名参数通过`{}`定义，这些参数可可以不按位置顺序进行输入，通过参数名字可以指定参数的值

```dart
String nameAdd(String firstName, {String? lastName, int age = 18}) {
  return '$firstName-$lastName,age:$age';
}
print(nameAdd('Well', age: 22, lastName: 'jack'));
```

**匿名函数**

```dart
Function foo = () {
  return 'hello xxx';
};
print(foo());
```

**箭头函数**

dart 中也有箭头函数，这个箭头函数和js中的箭头函数是不一样的，js中的箭头函数主要解决的是this的指向问题，而dart中的箭头函数则主要是用来简化函数的书写方式而已，dart 的箭筒函数只有在`方法体只包含一个表达式时，可以使用箭头表达式方法进行简写`。

```dart
Function bar = () => print('xxx');
bar();
```



## Dart 中的类

在Dart 中定义类也是使用 `class`关键字的，同样具有 `封装、继承、多态`的特性。但是在Dart 的类中并没有`public private protect`等权限描述符，在Dart 中只用共用和私有两种概念，通过`_`来表示私有变量，并且这种私有变量只有通过包引入的情况下才是有效的，在同一文件下还是可以访问的。

**定义类**

```dart
class Person{
  String name = 'person';
  int? age;
  //默认构造函数
  Person();
}
```

如果不定义构造函数的话，dart 也会绑定创建一个同类名的默认构造函数`Person();`所以我们也可以自己定义构造函数。

```dart
class Person{
  String name = 'person';
  int? age;
  //构造函数
  Person(this.name,this.age);
}
```

而且定义多个具名的构造函数

```dart
class Person{
  String name = 'person';
  int? age;
  //构造函数
  Person(this.name,this.age);
  //具名构造函数
  Person.fromJson({String name = 'jack',int age = 18}){
   	this.name = name;
    this.age = age;
  }
}
```

> 如果要使用单例模式来返回实例，就需要使用`factory`来定义返回，说明返回的结果并不一定会是一个新的实例
>
> ```dart
> class Person{
> //...
> Person ins = null;
> factory Person.single(){
>  if(!this.ins){
>    this.ins = Person('jack',13);
>  }
>  return this.ins;
> }
> }
> ```

**初始化构造参数**

有时，当你在实现构造函数时，您需要在构造函数体执行之前进行一些初始化。例如，final 修饰的字段必须在构造函数体执行之前赋值。在初始化列表中执行此操作，该列表位于构造函数的签名与其函数体之间：

```dart
class Point{
  final double x;
  final double y;
  Point(this.x,this.y);
  Point.fromJson(Map<String, double> json)
    : x = json['x']!,
  y = json['y']! {
    print('In Point.fromJson(): ($x, $y)');
  }
}
```

 **重定向构造方法**

有时一个构造方法仅仅用来重定向到该类的另一个构造方法。重定向方法没有主体，它在冒号（`:`）之后调用另一个构造方法

```dart
class Automobile {
  String make;
  String model;
  int mpg;
  Automobile(this.make, this.model, this.mpg);
  Automobile.hybrid(String make, String model) : this(make, model, 60);
  Automobile.fancyHybrid() : this.hybrid('Futurecar', 'Mark 2');
}
```

### 继承

dart 中的基础同样使用`extends`，如果要在子类中调用父类的构造函数需要使用`super()`方法，子类中重写父类方法时最好使用`@override`来标识一下

```dart
class Animal{
 	String? name;
  void eat(String food){
    print('eat a $food');
  };
  Animal(this.name);
}

class Cat extends Animal{
  Cat(name):super(name){
    print('create a cat');
  }
  @override
  String? name;
  @override
  void eat(String food){
    print('$name eat a $food');
  };
}
```



### 抽象类/多态

多态就是父类定义一个方法不去实现，让继承他的子类去实现，每个子类有不同的表现。在dart 中也是同样是用 `abstract`来定义抽象类的，而且dart中抽象类既可以当作普通父类继承使用、抽象类中以及实现的方法将会成为公共方法，也可以当作接口使用，通过 `implements` 实现。但是需要注意，抽象类本身不能被实例化。

```dart
abstract class Animal{
 	String? name;
  void eat(String food);//抽象方法
  void run(){
    print('$name is running');
  }
  Animal(this.name);
}

//继承情况下不用实现 run
class Cat extends Animal{
  Cat(name):super(name)
  @override
  String? name;
  @override
  eat(String food){
    print('eat a $food');
  }
}

//实现情况下必须要实现全部
class Dog implements Animal{
  @override
  String? name;
  @override
  void eat(String food) {}

  @override
  void run() {}
}
```

抽象类是可以进行多实现的。

```dart
abstract class PrintA{
  void printA();
}
abstract class PrintB{
  void printA();
}
class Print implements PrintA,PrintB{
  @override
  void printA() {}

  @override
  void printA() {}
}
```



### Mixin

mixin 可以将一些特定的类混入到当前类中，实现多基础，如果混入的类中同名的属性或者方法的话会按混入倒叙优先，混入类使用的是`with`关键字.

```dart
class A{
  String info = "this is A";
  void printA(){
    print("A");
  }
}
class B{
  String info = "this is B";
  void printB(){
    print("B");
  }
}
class C{
  String infoC = "this is C";
}
class Cc{
  String infoC = "Ccc";
}
class Print extends Cc with A,B,C{
  void printInfo(){
    printA();//A
    printB();//B
    print(this.info);//this is B
    print(this.infoC);//Ccc
  }
}
```

> 需要注意
>
> 1. 作为mixins的类只能继承于Object，不能继承其他类
> 2. 作为mixins的类不能有构造函数
> 3. 一个类可以mixins多个mixins类
> 4. mixins 不是继承，也不是借口，而是一种新特性

mixin 可以是特定的类，也可以通过`mixin`关键字定义，作用和`class`关键字相似，并且可以使用`on`关键字指定这个mixins只能被哪些类使用

```dart
mixin Musical{
  bool canPlayPiano = false;
  void entertainMe() {
    print('Humming to self');
  }
}
//--------------------------
class Musician {
  // ...
}
//指定 Musical 只能被 Musician 使用
mixin Musical on Musician {
  // ...
}
class SingerDancer extends Musician with Musical {
  // ...
}
```



## 异步支持 async/await

开发中很多时候我们都需要处理一些异步的操作，比如发送请求，I/O 操作等，我们需要在获取到结果之后在进行处理，这个并不好处理，因为我们不知道结果何时会返回。异步编程通常使用回调方法来实现，但是 Dart 提供了其他方案：Future和 Stream 对象。 Future 类似与 JavaScript 中的 Promise ，代表在将来某个时刻会返回一个结果。 Stream 类可以用来获取一系列的值，比如，一系列事件。

**对于Future**

```dart
// 假如 HttpRequest.getString(url) 返回一个 Future
HttpRequest.getString(url).then((String res){
  print(res);
}).catchError(e=>print(e))
```

Futurn 和 Promise 一样他们的then 都接收两个回调函数，第一个是成功回调，第二个是失败回调。并且最终错误使用`catchError`进行捕获

为了更加方便的解决异步问题，dart 引入了 async / await 关键字，使得异步操作可以已同步的方式进行书写。并且 aysnc 方法返回的是一个 Future，同样的 await 必须用于 async 中

```dart
Futurn<void> getData() async {
  var data = await fetchData();
  print(data);
}
```

使用了 async 的函数返回的将会是一个Futurn，并且会在调用后立即返回，所以其中的内容才是真正的异步操作。

可以使用`Future.wait()`来实现 Promise.all 的效果

```dart
Future<void> deleteLotsOfFiles() async =>  ...
Future<void> copyLotsOfFiles() async =>  ...
Future<void> checksumLotsOfOtherFiles() async =>  ...

await Future.wait([
  deleteLotsOfFiles(),
  copyLotsOfFiles(),
  checksumLotsOfOtherFiles(),
]);
print('Done with all the long steps!');
```



**关于 Stream**

Stream 用来表示一系列数据，HTML 中的按钮点击就是通过 stream 传递的。同样也可以将文件作为数据流来读取。一般是有`listen(cb)`来见监听数据的回调。可以通过`await for`、生产器、Stream 类来生产Stream数据。



## 常用操作符

### is 和 as

is 用来做类型判断

```dart
var ls = ['ss','fff','sss','ddd'];
print(ls is List<int>);//false
print(ls is List<String>);//true
```

as 用来做类型断言，可以强制指指定值的类型

```dart
//这样是会报错的，因为ls被推断为 List<String>
var ls = ['fasd','adf','sdfadf'];
int a = ls[1];
//这样就不会了，不过逻辑上是错误的
var ls = ['fasd','adf','sdfadf'] as List<int>;
int a = ls[1];
```

### 字符串插值

dart 中可以很方便的先字符串中插入值，有点像js中模版字符，在普通的字符中通过`${}`就可以插入值了，如果仅仅只是一个变量时可以直接`$变量名`，如果是一个表达式就必须是`${表达式}`

```dart
var a = 1;
print('number is $a');
print('number add is ${a + 1}');
```

如果需要输入字符串的格式可以使用`'''`或者`"""`包裹

```dart
var str = '''
sdfsafsf
		fsdff. dsfsdf 
''';
print(str);
```

如果不希望字符串中的特殊字符被转化，比如`\n,\t`等。可以在字符串前加一个 `r`,需要注意的加上后`${}`也会被原样输出。

```dart
var str = r'不能换行 \n 呵呵呵';
```

### List、Set、Map常用方法及属性

**List**

+ `isEmpty`是否为空列表
+ `length`列表长度
+ `reversed`列表翻转
+ `add(el)`添加元素
+ `addAll([])`添加多个元素
+ `clear()`清空列表
+ `contains(el)`判断是否包含某个元素
+ `elementAt(index)`查找索引值的元素
+ `every(cb)`遍历列表，是否每个元素都符合条件
+ `forEach(cb)`遍历列表
+ `reduce(cb)`和js中的reduce相似，用来累计数据
+ `where(cb)`和js中的some相似，是否一个元素是符合条件的
+ `map(cb)`和js中的map相似
+ `indexOf(el)`查找元素所在索引
+ `inset(i,el)`在索引 i 处插入值
+ `insetAll(i,[])`在索引 i 处插入值
+ `join(str)`拼接列表成字符串
+ `remove(el)`移除某个元素
+ `removeAt(index)`移除某个位置的元素
+ `sublist(start,end)`截取列表的内容
+ `toSet()`将列表转化成Set
+ `toList()`转化成列表，相当于复制

**Set**

+ `isEmpty`是否为空
+ `length`长度
+ `add(el)`
+ `addAll([])`
+ `clear()`
+ `contains(el)`
+ `containsAll({el})`
+ `difference({el})`对比返回不一样的新的set
+ `every(cb)`
+ `forEach(cb)`
+ `reduce(cb)`
+ `where(cb)`
+ `map(cb)`
+ `remove(el)`
+ `removeAt(index)`
+ `toList()`转化成列表
+ `toSet()`转化成Set，相当于复制

**Map**

+ `isEmpty`是否为空
+ `length`长度
+ `keys`取出全部键，返回的并不是一个列表，而是一个Iterable对象，可以使用全部的遍历方法以及for
+ `values`取出全部键，返回的是Iterable对象
+ `entries`获取map的Iterable形式
+ `addAll({})`
+ `clear()`
+ `containsKey(key)`是否包含某个key
+ `containsValue(value)`是否包含某个value
+ `forEach(cb)`
+ `map(cb)`
+ `putIfAbsent(k,v)`获取某个key的值，如果没有这个key就添加并复制为v
+ `remove(el)`
+ `update(key,update(v),v)`使用更新函数更新某个key的值
+ `updateAll(update(k,v))`使用更新函数更新全部

在Dart 中也有扩展运算符`...`，并且对List，Map，Set都有用

```dart
var ls = ['fasd','adf','sdfadf'];
var map = {'ss':22};
var set = {1,2,4};
var ls2 = [...ls];
var map2 = {...map};
var set2 = {...set};
```



### 避空运算符：`??	?.	 类型?   值！`

```dart
String? str;//使 str可以为空
var hasStr = str??'default string';//如果str不存在就赋值为 default string

var obj = {k:'v'};
print(obj?.k);//obj是否存在，存在的话就继续

var val1 = 'ss';
var val2 = val1!;//指定val2 必定不能为空，如果为空就报错
```

### 级联操作符`..`

要对同一对象执行一系列操作，可以使用`..`操作符，级连操作符可以将指针指回到第一个对象上

```dart
querySelector('#confirm')
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

## 库和包管理工具

### 常用库

在 Dart 中有一些核心库可以使用，常用的有

+ `dart:async`支持通过使用 Future 和 Stream 这样的类实现异步编程。
+ `dart:collection`提供 `dart:core` 库中不支持的额外的集合操作工具类。
+ `dart:convert`用于提供转换不同数据的编码器和解码器，包括 JSON 和 UTF-8。
+ `dart:core`每一个 Dart 程序都可能会使用到的内置类型、集合以及其它的一些核心功能。使用时无需导入
+ `dart:math`包含算术相关函数和常量，还有随机数生成器。
+ `dart:io`用于支持非 Web 应用的文件、Socket、HTTP 和其它 I/O 操作。
+ `dart:html`为 Web 应用开发所提供的 HTML 元素和其它资源。
+ `dart:js_util`补充 `dart:html` 或 `js` 包中缺少的功能性 API。
+ `dart:web_gl`用于浏览器的 3D 编程。
+ `dart:svg`用于可缩放的矢量图形 (SVG)。

`import` 和 `library` 关键字可以帮助你创建一个模块化和可共享的代码库。代码库不仅只是提供 API 而且还起到了封装的作用：以下划线（`_`）开头的成员仅在代码库中可见。 **每个 Dart 程序都是一个库**，即便没有使用关键字 `library` 指定。

**导入**

导入普通文件

```dart
import 'res/data.dart';
```

导入核心库

```dart
import 'dart:io';
```

导入自己的dart文件，比如`test/Animal.dart`

```dart
import 'package:test/Animal.dart';
```

可以使用 `as`定义别名/命名空间，其实就是库包裹到一个对象中。

```js
import 'package:test/Animal.dart' as ani;
ani.Animal();
```

还可以使用`show`只导入部分内容，或者使用`hide`隐藏部分内容

```dart
// 只导入foo.
import 'package:lib1/lib1.dart' show foo;

// 除了 foo 其他的都导入
import 'package:lib2/lib2.dart' hide foo;
```

可以使用`deferred as `来延迟加载库

```dart
import 'package:greetings/hello.dart' deferred as hello;
//在需要使用到库到时候再加载
Future<void> greet() async {
  await hello.loadLibrary();//加载库
  hello.printGreeting();
}
```

**导出**

导出关键字`export`只能用来做一些库的集合导出，比如要导出文件`A.dart`的全部方法，导出`B.dart `的say，run函数，导出`C.dart`中除了say函数的其他函数

```dart
//pack.dart
export './A.dart';
export './B.dart' show say,run;
export './C.dart' hide say;
```

之后就可以直接导入`pack.dart`来间接导入所需要的三个文件中的方法了

```dart
import 'package:lib1/pack.dart' show foo;
```



### 包管理工具

除了可以使用dart 提供的核心库外，还可以在 [https://pub.flutter-io.cn/](https://pub.flutter-io.cn/) 上下载和使用别人开发的一些库。那么对于这些第三方库我们需要使用一个包管理器来管理和下载相应的库。

dart 提供了`dart pub`命令来管理包，在使用包管理工具之前需要先在项目根目录下创建`pubspec.yaml`文件，并在文件中写入一些必要的参数

```yaml
name: my_app
description: 项目描述
# 项目依赖/第三方库
dependencies: 
  js: ^0.6.0
  intl: ^0.17.0
  
# 开发依赖
dev_dependencies: 
  test: '>=1.15.0 <2.0.0'

# 尝试查找其 SDK 约束适用于您已安装的 Dart SDK 版本的包的最新版本，从 Dart 2.12 开始，省略 SDK 约束是一个错误
environment: 
  sdk: '>=2.10.0 <3.0.0'
  flutter: ^0.1.2
```

在一些编辑器中，保存`pubspec.yaml`文件的时候会执行`dart pub get`命令下载依赖，如果没有执行的话我们可以收执行`dart pub get`进行下载。

> 第三方库一般会被下载到 pub 缓存中。默认情况下，此目录位于`.pub-cache` 您的主目录下（在 macOS 和 Linux 上）或`%LOCALAPPDATA%\Pub\Cache`（在 Windows 上）

**pub 常用命令**

+ `dart pub add 库名`下载安装第三方库，默认时生产依赖，如果要作为开发依赖就可以用`--dev`参数
+ `dart pub remove 库名`移除相应的第三方库
+ `dart pub upgrade 库名`更新/升级相应的第三方库，如果不指定库名就是升级全部
+ `dart pub get`下载`pubspec.yaml`中全部依赖的第三方库









