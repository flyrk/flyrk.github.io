---
title: javascript类型转换
date: 2017-08-02 13:55:02
toc: true
categories:
- JS相关
tags:
- JS原理
---

JavaScript中有七种类型，它们互相之间可以转换，但要搞清楚其中的转换关系可不简单。
<!--more-->

# JavaScript类型
## 类型种类
在JS中，有6种基本类型和一种引用类型，分别是：

基本类型：

- null
- undefined
- boolean
- number
- string
- symbol(ES6新增)

引用类型：

object

## 类型检测
一般来说，我们可以用typeof来检测某个值的数据类型，比如：

```javascript
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 24            === "number";    // true
typeof "24"          === "string";    // true
typeof { age: 24 }  === "object";    // true

// added in ES6!
typeof Symbol()      === "symbol";    // true
```

但是有一个例外: null,这也是一个bug

```javascript
typeof null === "object"; // true
```

如果想检测某个值是否是null，则可以这样：

```javascript
var a = null;
(!a && typeof a === "object"); // true
```

需要注意的是，function和array其实也是object，但它们有所区别：

```javascript
typeof function a(){ /* .. */ } === "function"; // true
typeof [1,2,3] === "object"; // true
```

如果要检测a是否是数组，可以用instanceof

```javascript
var a = [123];
a instanceof Array; // true
```

# 类型转换抽象操作
## ToString
当一个非String类型的值要转换为String，我们可以用`ToString`操作。

### toString()
对于基本类型来说，它们有自然的转换关系。如：null变成"null",undefined变成"undefined",true变成"true",number自然转换为数字字符串，但如果对于非常大的数，则会转换为指数形式。

对于object来说，`toString()`(代表着`Object.prototype.toString()`)将会返回`[[class]]`原型，比如说"object Object";但如果是数组，则重写了`toString()`方法，会返回一个以逗号分隔数组值的字符串。比如：

```javascript
var a = [1,2,3];
a.toString(); // "1,2,3"
```
### JSON.stringify()
另一种转换为字符串的方法就是JSON.stringify，对大多数基本类型，它的转换方法和toString是一样的：

1
2
3
4
```javascript
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (外面多加一层引号)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

但不同的是，JSON.stringify会忽略掉undefined、function和symbol。请看下面的例子：

```javascript
JSON.stringify( undefined );	// undefined
JSON.stringify( function(){} );	// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );	// "{"a":2}"
```

如果JSON.stringify调用了循环引用的object，则会抛出`Error`。如果JSON.stringify调用的对象有`toJSON()`方法，则会对toJSON()的返回值再进行stringify。

```javascript
var a = {
  b: 111
};
a.toJSON = function() {
	return { b: this.b };
};

JSON.stringify( a ); // "{"b":111}"

var c = {
	val: [1,2,3],

	toJSON: function(){
		return this.val.slice( 1 );
	}
};
JSON.stringify( c ); // "[2,3]"
```

## ToNumber
转换为number，我们可以用ToNumber操作。

### Number()
对于基本类型，转换规则为：true -> 1,false -> 0,undefined -> NaN,null -> 0，字符串如果包含字母则转换为NaN。

对于将某值转换为基本数据类型，通常会调用ToPrimitive操作，首先看该值有没有valueof()方法，如果没有则调用toString()方法，如果二者都没有，则抛出TypeError。

比如：

```javascript
var a = {
	valueOf: function(){
		return "12";
	}
};

var b = {
	toString: function(){
		return "12";
	}
};

var c = [1,2];
c.toString = function(){
	return this.join( "" );	// "12"
};

Number( a );			// 12
Number( b );			// 12
Number( c );			// 12
Number( "" );			// 0
Number( [] );			// 0
Number( [ "abc" ] );	// NaN
```

## ToBoolean
将一个值转换为Boolean值，我们先看一个false表：

```
undefined
null
false
+0, -0, and NaN
“”
```

任何不在这个表上的值都为true。
比如：

```javascript
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
Boolean( a && b && c );  // true
/***************/
var a = "false";
var b = "0";
var c = "''";
Boolean( a && b && c );  // true
/***************/
var a = [];		
var b = {};			
var c = function(){};
Boolean( a && b && c );  // true
```

# 显式转换
之前提到过一些显式转换，但其实还有以下几种：

## 字符串与数字的转换

```javascript
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

## 日期转换为数字
Date转换为的数字为从1 January 1970 00:00:00 UTC 到Date的毫秒数。

```javascript
var d = new Date( "Wed, 2 Aug 2017 08:53:06 CDT" );

+d; // 1501681986000
```

Date.now()方法就是用的这种转换：

```javascript
if (!Date.now) {
	Date.now = function() {
		return +new Date();
	};
}
```

## ~操作符
Js里有"\~"操作符，它首先将值转换为32bit的数字，然后在对它进行取补码，也就是说，"\~x"相当于"-(x+1)"。

于是，~有一个用处就是判断-1，比如我们对字符串使用indexOf()时如果找不到则返回-1，我们可以这样写：

```javascript
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- truthy!

if (~a.indexOf( "lo" )) {	// true
  // 找到了
}

~a.indexOf( "ol" );			// 0    <-- falsy!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// 没有找到
}
```

除此之外，~~可以用来对小数进行取整，但要注意的是，它是舍弃小数位数，而不是四舍五入，相当于Math.ceil()：

```javascript
Math.floor( -49.6 );	// -50
Math.ceil(-49.6);  // -49
~~-49.6;		   // -49
```
## parseInt()方法
该方法可以将字符串转换为数字，而且可以接受第二个参数，表示要转换的进制。但要注意的是，它有几个特别的地方：

```javascript
parseInt( 0.000008 );		// 0   ("0" from "0.000008")
parseInt( 0.0000008 );		// 8   ("8" from "8e-7")
parseInt( false, 16 );		// 250 ("fa" from "false")
parseInt( parseInt, 16 );	// 15  ("f" from "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```

所以，要谨慎使用parseInt。

## 转换为Boolean
我们已经知道Boolean()方法可以将值转换为Boolean，但还有一个更快的方法，就是使用!!操作符：

```javascript
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

!!a;	// true
!!b;	// true
!!c;	// true

!!d;	// false
!!e;	// false
!!f;	// false
!!g;	// false
```

# 隐式转换
## 数字转换为字符串
因为字符串可以用+连接，所以当我们把数字和字符串用+连接时，数字会强制转换为字符串：

```javascript
var a = 122;
var b = a + '';  
b;  // "122"
```

## 字符串转换为数字
因为-只在数字运算符中才有定义，所以对字符串使用-会被强制转换为数字：

```javascript
var a = "122";
var b = a - 0;  
b;  // 122
```

## 数组转换为数字
同样的，如果数组中只有一个值，使用-时，会先把数组转换为字符串，再转换为数字：

```javascript
var a = [3];
var b = [1];
a - b; // 2
```

# 操作符的妙用
## 操作符||和&&
我们都知道||代表‘或’，&&代表‘和’，但是与Java、C++等语言不同的是，使用这两个操作符时，它们返回的不是true或false，而是本身的值，举个例子：

```javascript
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

于是我们就可以利用这两个操作符进行一些赋值操作：

```javascript
a || b;
// 等价于：
a ? a : b;

a && b;
// 等价于：
a ? b : a;
```

## 操作符==与===
这两个符号我们应该很熟悉了，==代表值的比较，如果二者类型不同，会进行强制转换；===则是二者值与类型都相同才为true。这里需要注意一些问题：

> NaN不等于它本身
> +0等于-0

### 字符串与数字比较
二者比较时，使用===结果肯定为false，因为二者类型不同，但如果使用==，则需要进行转换，简单来说就是把字符串转换为数字再对二者的值进行比较：

```javascript
var a = 22;
var b = "22";

a === b;	// false
a == b;		// true
```
### 任何值与Boolean比较
当任何值与Boolean使用==比较时，首先会把Boolean转换为数字，然后再进行比较。也就是说：

```javascript
var x = true;
var y = "42";

x == y; // false
```

所以，在任何时候，都不要使用== true或者== false这样的语句。

### null与undefined比较
当使用==比较null和undefined时，结果总为true。

### Object与非Object比较
当object/function/array与String或者Number进行==比较时，会先把Object转换为String或Number，然后再对值进行比较，而Boolean值会被转换为Number再进行比较。

### 一些值得注意的比较

```javascript
"" == [null];	// true
[] == ![]; // true
0 == "\n"; // true
"0" == false;	// true
false == 0;		// true
false == "";	// true
false == [];	// true
"" == 0;		// true
"" == [];		// true
0 == [];		// true
```

# 总结
需要特别注意的就是隐式转换，将对象转换为基本类型时，会先调用`valueOf()`方法，没有才调用`toString()`方法，如果我们给自定义的对象添加`valueOf()`方法，并自己给返回值，也能能起到转换的效果。还有就是，在进行等于比较时，还是用===比较安全。

> 参考资料： [You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS/)