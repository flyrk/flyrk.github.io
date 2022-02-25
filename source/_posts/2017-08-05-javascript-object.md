---
title: 详解JavaScript对象
date: 2017-08-05 14:32:12
categories:
- JS相关
tags:
- 对象
---

JavaScript中的对象是一种引用类型，也就是说一般新建一个对象给它赋给一个值a，则a只是指向该对象地址的一个“指针”，对象的存储方式是堆。

<!--more-->

# 对象的复制

## JSON方法
对于 可以被序列化为一个 JSON 字符串并且可以根据这个字符串解析出一个结构和值完全一样的对象来说，有一种巧妙的复制方法:
```javascript
var newObj = JSON.parse(JSON.stringify( someObj ));
```
这种方法的好处是实现了 **深复制**，但必须要求原对象可以被解析成JSON字符串。
## `Object.assign()`方法
ES6定义了`Object.assign()`方法实现 **浅复制**。该方法第一个参数是目标对象，之后可以跟若干个源对象。它会遍历源对象的所有可枚举的自有键并把它们复制到目标对象，比如说：
```javascript
var myObj = {
  a: 12
  b: anotherObj,
  c: anotherArray,
  d: anotherFunction
};

var newObj = Object.assign( {}, myObject);
newObj.a; // 12
newObj.b === anotherObj; // true
newObj.c === anotherArray; // true
newObj.d === anotherFunction; // true
```
但要注意的是，浅复制只是简单的使用`=`操作符赋值，对于引用类型值来说，指向的是同一个对象。也就是说，`newObj.b === myObject.b`。

# 属性描述符
ES5开始，所有的属性都有属性描述符。通过`Object.getOwnPropertyDesciptor(obj, prop)`可以获取对象的某个属性的属性描述符。比如：
```javascript
Object.getOwnPropertyDescriptor(myObj, "a");
//Object {
//  value: 12,
//  writable: true,
//  enumerable: true,
//  configurable: true
//  }
```
一般默认值`writable`、`enumerable`、`configurable`都为`true`，我们也可以用`Object.defineProperty()`来添加一个新属性或者修改一个已有属性（如果它是`configurable`）。比如：
```javascript
var myObject = {};
Object.defineProperty( myObject, "a", {
  value: 2,
  writable: true,
  configurable: true,
  enumerable: true
});
myObject.a; // 2
```
## writable
```javascript
var myObject = {};
Object.defineProperty( myObject, "a", {
  value: 2,
  writable: false,
  configurable: true,
  enumerable: true
});
myObject.a = 3;
myObject.a; // 2
```
可以看到，当我们修改`writable`为false，则该属性不可写了。
## configurable
```javascript
var myObject = {};
Object.defineProperty( myObject, "a", {
  value: 2,
  writable: true,
  configurable: false, // 不可配置
  enumerable: true
});
myObject.a; // 2
myObject.a = 5;
myObject.a; // 5
delete myObject.a;
myObject.a; // 5

Object.defineProperty( myObject, "a", {
  value: 2,
  writable: true,
  configurable: true,
  enumerable: true
}); // TypeError
```
一旦我们修改`configurable`为false，就无法撤销，并且不能重新配置属性，也无法删除该属性。
## enumerable
这个属性描述符代表属性是否会出现在对象的属性枚举中，比如`for ..in`循环。如果设置`enumerable: false`，则该属性不会出现在枚举中。

# 不变性
JavaScript不像其他语言，它是动态的，也就是可变的。但如果我们想属性或者对象是不可改变的，该怎么实现呢？
## 对象常量
结合`writable: false`和`configurable: false`可以创建一个常量属性：
```javascript
var myObject = {};
Object.defineProperty(myObject, "CONST_VARIABLE", {
  value: 100,
  writable: false,
  configurable: false
});
```
## 禁止扩展
如果想保留对象已有的属性并禁止向对象添加新的属性，则可以这样：
```javascript
var myObject = { a: 12 };
Object.preventExtensions( myObject );

myObject.b = 13;
myObject.b; // undefined
```
## 密封
使用`Object.seal(...)`会创建一个“密封”对象，相当于在对象上调用`Object.preventExtensions()`并且设置所有属性为`configurable: false`。

所以，密封之后的对象只能修改已有属性的值，而不能添加新属性，也不能重新配置或删除已有属性。
## 冻结
使用`Object.freeze(...)`会创建一个冻结对象，相当于在对象上调用`Object.seal()`并且设置所有属性为`writable: false`。

这个方法完全使对象不可变，会禁止对于对象本身及其任意直接属性的修改。
> 需要注意的是，之前所有的方法创建的都是浅不可变，也就是说，它们只会影响目标对象和它的直接属性。如果目标对象引用了其他对象(数组、对象、函数，等)，其他对象的内容不受影响，仍然是可变的。

# Getter和Setter
当访问某个对象属性时，实际上实现了`[[Get]]`操作，对象默认的内置`[[Get]]`操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值。如果没有找到，则会遍历原型链，如果还是没找到，则返回`undefined`。

当设置某个对象属性的值，则有`[[Put]]`操作。`[[Put]]`会检查以下内容：
  1. 属性是否存在setter，存在则调用setter
  2. 属性的`writable`是否是`false`，是的话则赋值失败
  3. 如果都不是，则对属性进行赋值

我们可以给对象一个属性定义getter、setter，当访问或赋值对象的属性时，会忽略属性的`value`和`writable`特性，而关注`get`和`set`特性：
```javascript
var myObject = {
  get a() { // 给属性a定义一个getter
    return 11;
  }
};

Object.defineProperty( myObject, "b", {
  get: function() { return this.a * 2; }, // 给属性b设置一个getter
  enumerable: true
});
myObject.a; // 11
myObject.b; // 22
myObject.a = 3;
myObject.a; // 11
```
当访问属性时会自动调用`get`函数，其返回值会被当作属性访问的返回值；注意的是，由于我们没有设置`set`，所以赋值操作被忽略了。而由于我们定义的`get`始终返回2，即使设置了`set`也没有意义。

我们可以这样设置`getter`和`setter`：
```javascript
var myObject = {
  get a() { // 给属性a定义一个getter
    return this._a;
  },
  set a(val) {
    this._a = val * 5;
  }
};

myObject.a = 2;
myObject.a; // 10
```

# 属性的存在性
当我们访问`object.a`返回的是`undefined`，可能是属性中本来存储的就是`undefined`，也可能是属性不存在，那么怎么判断属性是否存在呢？

我们可以用`in`或者`Object.hasOwnProperty()`方法：
```javascript
var myObject = { a: 1 };

("a" in myObject); // true
("b" in myObject); // false

myObject.hasOwnProperty("a"); // true
myObject.hasOwnProperty("b"); // false
```
这里要注意的是，`in`会检查属性是否在对象及其原型链中，而`hasOwnProperty`只会检查属性是否在对象中，不会检查原型链。

# 枚举
要检查某个属性是否可枚举，通常有两种方法：
第一种方法，使用`for...in`循环：
```javascript
var myObject = { a: 1 };

Object.defineProperty(myObject, "b", {
  value: 2,
  enumerable: false
});

myObject.b; // 2
("b" in myObject); // true
myObject.hasOwnProperty("b"); // true

for (let k in myObject) {
  console.log(`${k}:${myObject[k]}`); // a:1
}
```
第二种方法：
```javascript
var myObject = { a: 1 };

Object.defineProperty(myObject, "b", {
  value: 2,
  enumerable: false
});

myObject.propertyIsEnumerable("a"); // true
myObject.propertyIsEnumerable("b"); // false

Object.keys(myObject); // ["a"]
Object.getOwnPropertyNames(myObject); // ["a", "b"]
```
`propertyIsEnumerable()`可以判断给定的属性名是否直接存在于对象中且`enumerable: true`；`Object.keys(..)` 会返回一个数组，包含所有可枚举属性，`Object.getOwnPropertyNames(..)`会返回一个数组，包含所有属性，无论它们是否可枚举。

要注意的是，`Object.keys(..)`和`hasOwnProperty()`和`Object.getOwnPropertyNames(..)`不会查找原型链，而`in`和`for...in`会查找原型链。

# 遍历
对于数组来说，最常用的遍历就是`for`循环。然而javascript内置了一些数组的迭代器，包括`forEach(..)`、`every(..)`、`some(..)`、`map(..)`。它们都接受一个回调函数并把它应用到每个元素上，区别就是它们对回调函数返回值的处理方式不同。

`forEach(..)`遍历所有元素并忽略返回值；`every(..)`会一直运行直到返回值为`false`；`some(..)`会一直运行直到返回值为`true`；`map(..)`遍历所有元素并且用返回值代替当前元素。

ES6还新增了遍历数组的`for...of`循环语法：
```javascript
var myArray = [1,2,3];

for (var v of myArray) {
  console.log( v );
}
// 1
// 2
// 3
```
`for..of`循环首先会向被访问对象请求一个 **迭代器** 对象，然后通过调用迭代器对象的next() 方法来遍历所有返回值。由于数组有内置的`@@iterator`，所以`for..of`可以直接应用在数组上，如果想让对象也使用`for..of`循环，则需要自定义`@@iterator`，这里就不多展开了。

# 总结
JavaScript对象是引用类型，我们可以给对象添加属性并修改属性描述符来控制属性的特征，还可以用`get`和`set`来修改属性的访问和赋值操作，对象的属性还可以枚举和遍历。

> 参考资料：[You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch3.md)
