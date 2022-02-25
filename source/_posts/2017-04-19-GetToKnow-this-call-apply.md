---
title: 浅谈this和call、apply
date: 2017-04-19 21:46:45
categories:
- JS相关
tags:
- JS原理
---

this的指向一直是很多人头疼的问题，我之前也经常弄混，后来整理了一下，感觉开朗了许多，在这里列出来以供参考
<!--more-->
## this的指向
 一般可分为四种情况：
   1. 作为普通函数调用：此时this总是指向全局对象，比如
    ```
    var func = {
        name: 'Simon',
        getName: function() {
            console.log(this.name);
        }
    };
    var name = 'Yuk';
    var getMyName = func.getName;
    getMyName();  // 输出'Yuk'
    ```
   2. 作为对象方法调用：此时this指向该对象，如
    ```
    var func = {
        name: 'Simon',
        getName: function() {
            console.log(this.name);
        }
    };
    var name = 'Yuk';
    func.getName();  // 输出'Simon'
    ```
   3. 构造器调用：当用new运算符调用函数时，函数会返回一个对象，此时this就指向这个返回的对象。但如果构造器显式地返回了一个object类型的对象，则this会指向这个显式对象。
   4. bind、call、apply方法调用：此时this指向方法中指定的obj。

---
## call和apply
  1. call和apply的作用基本相同，区别仅在于传入的参数形式不同。call传入的参数数量不固定，第一个参数是代表函数体内this的指向，从第二个参数开始往后，每个参数被依次传入函数；apply接受两个参数，第一个也是代表函数体内this的指向，第二个参数为一个包含参数的数组或者类数组(arguments)。
  2. 当使用call或者apply，如果传入的第一个参数为null，函数体内的this会指向默认的宿主对象，浏览器中是window，node中是global。
  3. call和apply的用途：
   * 改变this的指向
   * Function.prototype.bind的实现：
```javascript
Function.prototype.bind = function() {
   var _this = this,   // 保存原函数
       context = [].shift.call(arguments), // 需要绑定的this的上下文
       args = [].slice.call(arguments);  // 剩余的参数转成数组
   return function() {     // 返回一个新的函数
       return _this.apply(context, [].concat.call(args, [].slice.call(arguments) ) );
           //执行新的函数的时候，会把之前传入的context当做新函数体内的this
           //并且组合两次分别传入的参数，作为新函数的参数 
   };
};
```
   * 借用其它对象的方法：在操作arguments的时候，我们经常找Array.prototype对象借用方法。
```javascript
[].slice.call(arguments);   // 将arguments转换为数组
[].shift.call(arguments);   // 截去arguments列表中的头一个元素
[].push.call(arguments, item); // 向arguments里添加新元素
```


