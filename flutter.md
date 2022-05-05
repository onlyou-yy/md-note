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

```js
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

### 抽象类

### 继承

### 多态

### Mixin

## 异步支持 async/await

## 常用操作符

is 和 as

模板字符串

List、Set、Map常用方法及属性

避空运算符：`??	?.	 类型?   值！`

级联操作符`..`

## 包管理工具



# flutter

安装

配置

配置Android环境

设置虚拟机

真机调试注意点

常用组件









