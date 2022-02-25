---
title: JS中"类"的构造与继承
date: 2017-08-10 21:46:33
categories:
- JS相关
tags:
- 类
---

有句话说“JavaScript里一切都是对象”，这句话当然是错的，因为它还有六种基本类型：`String`、`Number`、`Boolean`、`Null`、`undefined`、`Symbol`。对象才有属性和方法，当我们想要操作基本类型，比如获取length、截取长度等方法，JS就会自动把基本类型转换为对应的对象实例，我们就在对象实例上操作。
<!--more-->

我们知道，JS里没有类似于C++、Java一样的`class`类，于是也就没有私有、公有之说。但这并不代表我们不能模拟一个类，因为类的思想在面向对象编程里是很重要的一点，对于对象的继承与方法的引用和代码的重构等等都有极大好处。所以很多人就在JS里模拟类的用法，ES6甚至官方定义了一个`class`语法，但这也不是真正的类，只是`class`的语法糖。

说了这么多，JS里到底怎么实现类呢？利用函数的特性：所有的函数默认都会拥有一个名为prototype的公有且不可枚举的属性，它会指向另一个对象，也就是原型。


# 构造函数
什么是构造函数？我们知道`function`也是对象的一种，而当我们想要声明一个构造函数，可以使用下列形式：
```javascript
function myConstructor() {
  this.name = 'Simon';
  var car = 'BMW';
  function getCar() {
    console.log(car);
  }
  getCar();
}

var myObj = new myConstructor();
```
使用上述方法我们就声明了一个构造函数`myConstructor`，但其实构造函数也是普通的对象，它和java里类的构造函数完全不一样。当我们使用`new`创建一个对象，实际上是把myObj的[[Prototype]]链接关联到myConstructor.prototype指向的对象。

我们知道，真正的面向类的语言中，类可以被复制或者实例化很多次，每创建一个实例相当于把类的方法属性复制到新实例中。但javascript没有类似的复制机制，要想让新对象与构造的“类”对象有关系，就必须把两个对象的prototype关联起来。

实际上并不存在所谓的“构造函数”，我们只是对函数的“构造调用”。

当使用new来进行构造函数调用时，会执行以下四个步骤：
  1. 创建一个全新的对象
  2. 新对象会执行[[Prototype]]链接到constructor对象的prototype
  3. 新对象会绑定到函数调用的this
  4. 如果构造函数没有返回其他对象，那么new的函数调用会返回这个新对象

这样一来，我们就把新对象与构造函数对象关联起来了。所以说函数不是构造函数，只是当且仅当使用new时，函数调用会变成“构造函数调用”。
```javascript
function Foo () {
  //.....
}

var a = new Foo();

a.constructor === Foo; // true
Foo.prototype.constructor === Foo; // true
```
当我们创建了Foo函数，Foo.prototype默认有一个公有且不可枚举的属性constructor，这个属性引用的是对象关联的函数（也就是Foo）。而调用a.constructor，实际上只是通过[[Prototype]]委托给了Foo.prototype，所以指向了Foo。

如果我们创建了新的原型对象，那么新对象并不会自动获得constructor属性，比如说：
```javascript
function Foo() {
  //....
}
Foo.prototype = { /*....*/ }; // 创建了新的原型对象

var a = new Foo();
a.constructor === Foo; // false
a.constructor === Object; // true
```


但问题来了，运行以下代码：
```javascript
myObj.name; // "Simon"
myObj.car; // undefined
myObj.getCar(); // Uncaught TypeError: myObj.getCar is not a function
```
为什么会这样呢？原因就是构造函数有作用域。`this.name = 'Simon'`相当于给构造函数这个对象的`name`属性赋值`"Simon"`，这里的`this`指代构造函数对象原型。而在构造函数里使用`var`、`function`创建的变量和函数都相当于 *局部变量*，也就是“私有方法”，因为作用域的问题，这些变量和属性在构造函数之外是不能访问的，“私有方法”中的函数可以访问其他的私有变量，构造函数的实例却不能访问这些方法。

# 继承
那么，如果我们想要实例能继承构造函数的属性或者方法怎么办呢？JS使用的机制为 *原型继承*，请看以下代码：
```javascript
myConstructor.prototype.car = "Farrari";
myConstructor.prototype.getCar = function () {
  console.log(this.car);
};

myObj.car; // "Farrari"
myObj.getCar(); // "Farrari"
```
一旦我们在构造函数的原型上添加了属性car，当新对象myObj调用car属性时，它本身没有这个属性，于是就通过委托，在原型链上找，最后在myConsrtuctor.prototype上找到。

但使用原型继承来实现子类继承父类更好的方法是这样：
```javascript
function Foo(name) {
  this.name = name;
}
Foo,prototype.getName = function() {
  console.log(this.name);
};

function Bar(name, type) {
  Foo.call(this, name); // 相当于ES6的super(name);
  this.type = type;
}

Bar.prototype = Object.create(Foo.prototype); // 相当于ES6的extends
Bar.prototype.constructor = Bar; // 这里需要修复consructor

Bar.prototype.getType = function () {
  console.log(this.type);
};

var a = new Bar('a', 'obj a');
a.getName(); // 'a'
a.getType(); // 'obj a'
```
这里用到的核心语句就是`Bar.prototype = Object.create(Foo.prototype);`，调用Object.create(obj)会凭空创建一个新对象并把新对象的[[Prototype]]关联到obj上。

我们来看看Object.create()的polyfill:
```javascript
if (!Object.create) {
  Object.create = function (proto, propsObj) {
    if (!(proto === null || typeof proto === 'object' || typeof proto === 'function')) {
      throw TypeError('Arguments must be object, or Null');
    }
    var temp = new Object();
    temp.__proto__ = proto;
    if (propsObj) {
      Object.defineProperty(temp, propsObj);
    }
    return temp;
  };
}
```
有的人可能会说，为什么不能直接`Bar.prototype = Foo.prototype;`呢？这样并不会创建一个关联到Foo.prototype的新对象，只是让Bar.prototype引用Foo.prototype对象，我们执行`Bar.prototype.getType = ....`也会在Foo.prototype上修改，跟预料的结果不一样。那这样根本不需要Bar，还不如直接在Foo上修改就好了。

而之前介绍的`Bar.prototype = new Foo()`虽然也能实现原型继承，但它会有一些副作用。比如Foo如果有（写日志、修改状态、注册到其他对象、给this添加属性等等）的行为，就会影响到Bar的子类，造成无法预料的结果。

所以，要创建一个关联的“子类”对象，我们必须使用Object.create()，但这样有一个缺点是要创建一个新对象而把旧对象抛弃。

好在ES6添加了`Object.setPrototypeOf(...)`方法，可以修改关联。
```javascript
Bar.prototype = Object.create(Foo.prototype); // ES6之前需要抛弃默认对象

Object.setPrototypeOf(Bar.prototype, Foo.prototype); // ES6可以直接修改Bar.prototype对象
```

# 检查“类”的关系
考虑以下代码：
```javascript
function Foo() {
  //...
}
Foo.prototype.blah = ...;

var a = new Foo();
```
我们如果想要判断a的原型是否与Foo关联起来了，我们可以有以下几种方法：
## instanceof
```javascript
a instanceof Foo; // true
```
但这样有个缺点是只能处理对象和函数之间的关系。如果我们想判断两个对象之间是否通过[[Prototype]]关联。可以用下列方法：
## isPrototypeOf()
```javascript
Foo.prototype.isPrototypeOf(a); // true
```
要判断两个对象之间的关系，更直接：
```javascript
b.isPrototypeOf(c); // 判断b是否出现在c的原型链中
```
## Object.getPrototypeOf()
```javascript
Object.getPrototypeOf(a) === Foo.prototype; // true
```
## __proto__
我们可以直接用非标准的`__proto__`属性，`__proto__`实际上存在于内置的Object.prototype中，且不可枚举。
```javascript
a.__proto__ === Foo.prototype; // true
```

# 总结
要想创建类似于“类”的对象，可以使用构造函数调用，给构造函数添加公有方法，在构造函数的`prototype`上添加属性。新对象想要继承构造函数，使用Javascript的原型继承机制。
