## new一个Object的实例

    实例化一个Object对其添加属性和方法

```js
var person = new Object();
person.name = "wz";
person.age = 21;
person.sex = "male";
person.sayName = function(){
   alert("this.name")
}
```

[js中的new()到底做了些什么？？](https://www.cnblogs.com/faith3/p/6209741.html)



## 对象字面量模式

    给人一种封装的感觉

```js
var person = {
	name:"wz";
	age:"21";
	sex:"male";
	sayName:function(){
		alert(this.name);
	}
}
```



## 工厂模式

    利用实例化Object或对象字面量很容易创建单个对象，但这些方式有个明显的缺点，使用同一个接口创建很多对象，会产生大量重复代码。

    在ECMA中无法创建类，用函数来封装特定接口创建对象的细节。

```js
function createPerson(name,age,sex){
	var o = new Object();
	o.name = "wz";
	o.age = "21";
	o.sex = "male";
	o.sayName = function(){
		alert(this.name)
	}
}
var person1 = createPerson("wz",21,"male");
var person1 = createPerson("逼天",21,"male")
```



    工厂模式的虽然解决了一个一个的创建对象的问题，但却没有解决对象识别问题(怎么知道一个对象的类型)返回的都是Object类型

## 构造函数模式

    ECMAScript中的构造函数可以用来创建特定类型的对象。像Object和Array这样的原生构造函数(-0-,原来这个new array()，new object(),中的array()和object()是构造函数)，也可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法。

```js
function Person(name,age,sex){//构造函数用大些开头，为了区分别的函数；构造函数本身也是函数，不过能创建对象而已。
	this.name = "wz";
	this.age = "21";
	this.sex = "male";
	this.sayName = function(){
		alert(this.name)
	}
}
var person1 =new Person("wz",21,"male");
var person2 =new Person("逼天",21,"male")
```



    任何函数，只要通过new操作符来调用，那它就可以作为构造函数；如果不用new操作符来调用，它就是一个普通函数的调用。

```js
//当作构造函数调用
var person = new Person("wz",21,"male");
person.sayName();//"wz"
//作为普通函数调用
Person("逼天",21,"male");//添加到window
window.sayName();//逼天
var o = new Object();
//在另一个对象的作用域被调用
Person.call(o,"武昭",21,"male");
o.sayName();//"武昭"
```



    构造函数虽然好用，但也不是没有缺点，使用构造函主要问题就是每个方法都要在每个实例上重新创建一遍前边的person1和person2虽然都有一个sayName的方法，但是那两个方法不是同一个function的实例。alert(person1.sayName == person2.sayName);//false
然而，创建两个完全同样任务的Function实例的确没有必要；况且又this对象在，根本不用在执行代码前就把函数绑定到特定对象上面。大可这样:

```js
function Person(name,age,job){
	this.name = name;
	this.age = age;
	this.sex = sex;
	this.sayName = sayName;
}
function sayName(){
	alert(this.name);
}
var person1 =new Person("wz",21,"male");
var person2 =new Person("逼天",21,"male")
```

    在这个例子中，我们把sayName()函数的定义转移到了构造函数外部。而在构造函数内部，我们将sayName属性设置成等于全局的sayName函数。这样一来，由于sayName包含的是一个指向函数的指针，因此person1和person2对象就共享了在全局作用域中定义的同一个sayName()函数。这样做确实解决了两个函数做同一件事的问题，可是新的问题又来了：在全局作用于中定义的函数实际上只能被某个对象调用，这样全局作用域有点名不副实。更让人无法接受的是：如果对象需要定义很多方法，那么就要定义很多个全局函数，这样我们这个自定义的引用类型就丝毫没有封装性可言了。

## 原型模式

    只有用new的函数才有prototype属性
    我们创建的每个函数都有一个prototype属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。按字面意思解释，prototype就是通过该构造函数创建的某个实例的原型对象，但是其实prototype是每个构造函数的属性而已，只能说万物皆对象罢了。
    原型对象的优点是：所有的对象实例都可以共享它包含的属性和方法。

```js
function Person(){}
Person.prototype.name = 'wz';
Person.prototype.age = 18;
Person.prototype.sex = 'male';
Person.prototype.sayName= function(){
    alert(this.name);
}

var person1 = new Person();
person1.sayName();   

var person2 = new Person();
person2.sayName();
//判断两个实例继承的方法和属性是否全等
console.log(person1.name=== person2.name);
console.log(person1.age === person2.age);
```

person1和person2访问的是同一个sayName();

    原型对象并不是没有缺点。首先，它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将去的相同的属性值。这还不是最大问题，原型模式的最大问题是由其共享的本性所导致的。

```js
function Person() {
    Person.prototype = {
        constructor: Person,
        name: "wz",
        age: 21,
        sex: "male",
        friends: ["w","h"],
        sayName: function () {
            alert(this.name);
        }
    };
}
var person1 = new Person();
var person2 = new Person();
person1.friends.push("y");//这push都报错
alert(person1.friends);
alert(person2.friends);
alert(person1.friends === person2.friends);
```
    本想给person1没想给person2加朋友

## 组合模式

    组合使用构造函数模式与原型模式

```js
function Person(name,age,sex) {
    this.name = name;
    this.age = age;
    this.sex = sex;
    this.friends = ["wz","hcy"];
}
Person.prototype = {
    constructor:Person,
    sayName:function(){
        alert(this.name);
    }
}
var person1 = new Person("wz",21,"male");
var person2 = new Person("hcy",21,"male");
person1.friends.push("y");
console.log(person1.friends);
console.log(person2.friends);
console.log(person1.friends === person2.friends);
```
[js创建对象的几种方式](https://blog.csdn.net/weixin_42519137/article/details/84202699)

