---
title: 深刻理解闭包
date: 2017-04-06 20:45:59
tags: 闭包
categories: JS基础
---

### 变量的知识
---
- 变量的作用域：js里可以用函数来创造函数作用域，ES6中有块作用域，作用域里的变量是局部变量，作用域外面访问不到该变量，但在作用域里可以访问到作用域外面的变量。
- 变量的生存周期：全局变量的生存周期是永久的，除非主动销毁该全局变量。作用域里的变量当离开该作用域或者离开函数时，局部变量就会被垃圾回收装置自动销毁。
<!--more-->
### 什么是闭包？
---
我们先来看个例子
```javascript
var func = function() {
    var a = 0;
    return function() {
        a++;
        console.log(a);
    };
};
var f = func();
f();    // 1
f();    // 2
f();    // 3
```
当退出函数后，变量a并没有消失，因为调用var f = func();时，f返回了一个匿名函数的引用，而它可以访问到func()被调用时执行的环境，局部变量a则处在这个环境中，所以a可以被外界访问。这里就产生了所谓的闭包，局部变量a的生命周期被延长了。
我们知道，当func()执行后，通常会期待func()内部作用域全部被销毁，因为JS引擎有垃圾回收器用来释放不再使用的内存空间。当某个函数执行完后，一般我们都认为它已经不再使用了，自然会考虑对其回收。而闭包的作用就是让内部作用域依然存在，f()就是对该作用域的引用，而这个引用就叫做闭包。

### 闭包的作用
---
- 封装变量：把一些不需要暴露在全局的变量封装成“私有变量”。
- 延长局部变量寿命：使其不会在退出函数时就立即销毁。
- 将函数在定义时的词法作用域以外的地方调用使其可以继续访问定义时的词法作用域。

### 闭包与面向对象设计
---
- 对象以方法的形式包含了过程，而闭包则是在过程中以环境的形式包含了数据。
- 闭包实现命令模式:命令模式的意图是把请求封装为对象，从而分离请求的发起者和请求的接收者之间的耦合关系。用闭包实现命令模式如下:
```javascript
var createCommand = function(receiver){
    var excute = function(){
        return receiver.open();
    };
    var undo = function(){
        return receiver.close();
    };
    return {
        excute: excute,
        undo: undo
    };
};
```
- 闭包与内存管理:使用闭包的一部分原因是我们选择主动把一些变量封装在闭包中，因为可能在以后还需要使用这些变量，把这些变量放在闭包中和放在全局作用域中对内存方面的影响是一致的。如果在将来需要回收这些变量，可以手动把变量设置为null。

### 闭包的应用
---
在定时器、事件监听器、Ajax请求、跨窗口通信、Web Workders或者任何其他的异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包。例如：
```javascript
function wait(message) {
    setTimeout( function timer() {
        console.log( message );
    }, 1000 );
}

wait("This is a closure!");
```
在这里我们将一个内部函数timer传递给setTimeou，timer具有涵盖wait(...)作用域的闭包，因此包含对变量message的引用。wait(...)执行1000毫秒后，它的内部作用域并不会消失，timer依然保有wait(...)作用域的闭包。

### 经典闭包问题
---
考虑如下代码：
```javascript
for(var i = 0; i < 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
```
我们期待是输出0～5，每隔一秒输出一个数字。但实际上这段代码运行结果是每隔一秒输出一个5。想想就知道，setTimeout设置的延迟函数相当于异步调用，在循环后才执行，而循环结束后i为5,所以输出也就为5。那么问题出在哪呢？
我们以为循环时每个迭代都会“捕获”一个i的副本，但是根据作用域工作原理，尽管循环中的五个函数是在不同的迭代定义的，但它们共享一个全局作用域，因此只有一个全局变量i。
怎么解决？在循环的每个迭代创建一个闭包作用域。这里就用到IIFE函数：
```javascript
for(var i = 0; i < 5; i++) {
    (function(i) {
        setTimeout(function timer() {
            console.log(i);
        }, i * 1000);
    })(i);
}
```
这样问题就解决了，我们在每个循环创造一个IIFE匿名函数，并传入全局变量i，使得每个作用域都收到不同的局部变量i并用闭包保存下来，最后输出时就能正确输出i值了。
其实我们的解决方法本质上就是每次迭代都需要一个块作用域。在ES6里用let就能很好解决这个问题：
```javascript
for(let i = 0; i < 5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
```
因为let声明在每次迭代都会声明i，随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

### 闭包在模块中的应用
---
考虑以下代码：
```javascript
function CoolModule() {
    var sth = 1;
    var another = [1,2,3];
    function doSth() {
        console.log(sth);
    }
    function doAnother() {
        console.log(another.join(','));
    }

    return {
        doSth: doSth,
        doAnother: doAnother
    };
}

var foo = CoolModule();
foo.doSth();
foo.doAnother();
```
这个模式在js中被称为模块。CoolModule()是一个函数，我们必须通过调用它来创建一个模块实例。该函数返回的是一个对象，这个对象包含了对内部函数的引用，也就是一个闭包，而内部数据变量仍然是隐藏且私有状态。可以说这个对象的返回值就是模块的公共API。每次调用CoolModule()函数相当于创建了一个新的模块实例，然后可以用这个实例调用其内部函数API。
简单地说，模块模式需要具备两个必要条件：
    1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用创建一个新的模块实例）
    2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态
当只需要一个实例时，我们可以用如下方法实现单例模式：
```javascript
var foo = (function CoolModule() {
    var sth = 1;
    var another = [1,2,3];
    function doSth() {
        console.log(sth);
    }
    function doAnother() {
        console.log(another.join(','));
    }

    return {
        doSth: doSth,
        doAnother: doAnother
    };
})();

foo.doSth();
foo.doAnother();
```
我们通过一个IIFE立即执行模块函数并返回赋值给foo。通过在模块实例的内部保留对公共API对象的内部引用，可以从内部对模块实例进行修改，包括添加或删除方法和属性，以及修改它们的值。
