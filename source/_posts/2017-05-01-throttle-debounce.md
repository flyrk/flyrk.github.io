---
title: 浅谈函数节流与函数去抖
date: 2017-05-01 19:37:31
categories:
- JS基础
tags:
- Function
- throttle
- debounce
---

函数节流与函数去抖在很多场景都有着应用，对web性能有很大提升。但是这二者也有一些区别，在这里我简单的总结一些基本知识。
---
<!--more-->
# 函数节流:throttle
## 函数节流的目的
有些函数不是由用户直接控制触发的，函数有可能被**频繁地调用**而导致性能问题。一些常见的函数被频繁调用场景：
- window.onscroll事件。当绑定了scroll事件，只要浏览器的窗口滚动就会触发函数，触发频率非常高。如果绑定的函数里面还有DOM操作，则性能损耗非常大。
- mousemove事件。给div绑定该事件，当鼠标拖曳节点时触发函数。
- 键盘keydown监听
> 我们的目的就是减少函数触发的频率

## 函数节流原理
我们想到用setTimeout延迟时间执行函数，这样就可以避免函数在一段时间内执行多次。
```javascript
var throttle = function ( fn, delay ) {
    var _self = fn,             // 保存将要延迟执行的函数
          timer,                    // 定时器
          first_time = true;  // 是否第一次调用

    return function () {
        var args = [].slice.call(arguments),
              _me = this;
        if ( first_time ) {     // 如果第一次执行则直接调用
            _self.apply( _me, args );
            return first_time = false;
        }
        if( timer ) {           // 如果timer存在说明之前延迟的函数还没有执行
            return false;
        }
        timer = setTimeout( function () {
            clearTimeout( timer );
            timer = null;
            _self.apply( _me, args );
        }, delay );
    };
};

window.onscroll = throttle( function () {
    console.log("has changed");
}, 500 );
```

# 函数去抖:debounce
## 函数去抖的目的
函数去抖其实和节流很像，但不同的是去抖是让函数在一段时间内只触发执行一次，而节流是改变函数触发的调用周期。应用场景：
- 文字输入监听
- onscroll事件
- onresize事件。
## 函数去抖原理
也是利用setTimeout和闭包的原理。
```javascript
var debounce = function ( fn, delay ) {
    var context,
          args,
          timer;
    return function () {
        context = this, args = [].slice.call(arguments);
        if( timer ) {
            clearTimeout( timer );
        }
        timer = setTimeout( function () {
            fn.apply( context, args );
        }, delay );
    };
};

window.onresize = debounce( function () {
    console.log("has changed");
}, 600 );
```
# underscore源码
underscore里对throttle和debounce有更完整的实现，这里把代码贴出来
- throttle
```javascript
_.throttle = function(func, wait, options) {
    var context, args, result;
    var timeout = null;
    var previous = 0;
    if (!options) options = {};
    var later = function() {
      previous = options.leading === false ? 0 : _.now();
      timeout = null;
      result = func.apply(context, args);
      if (!timeout) context = args = null;
    };
    return function() {
      var now = _.now();
      if (!previous && options.leading === false) previous = now;
      var remaining = wait - (now - previous);
      context = this;
      args = arguments;
      if (remaining <= 0 || remaining > wait) {
        if (timeout) {
          clearTimeout(timeout);
          timeout = null;
        }
        previous = now;
        result = func.apply(context, args);
        if (!timeout) context = args = null;
      } else if (!timeout && options.trailing !== false) {
        timeout = setTimeout(later, remaining);
      }
      return result;
    };
 };
```

- debounce
```javascript
_.debounce = function(func, wait, immediate) {
    var timeout, args, context, timestamp, result;

    var later = function() {
      var last = _.now() - timestamp;       // 上一次函数触发的时间

      if (last < wait && last >= 0) {       // 如果没有到达规定的wait时间则继续延迟，相当于计时器
        timeout = setTimeout(later, wait - last);
      } else {
        timeout = null;
        if (!immediate) {
          result = func.apply(context, args);
          if (!timeout) context = args = null;
        }
      }
    };

    return function() {
      context = this;
      args = arguments;
      timestamp = _.now();
      var callNow = immediate && !timeout;
      if (!timeout) timeout = setTimeout(later, wait);
      if (callNow) {        // 如果是立即调用或者上一次函数已经调用完毕
        result = func.apply(context, args);
        context = args = null;
      }

      return result;
    };
 };
```
