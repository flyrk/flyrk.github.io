---
title: 获取浏览器窗口大小
date: 2017-08-08 20:57:31
categories:
- JS相关
tags:
- JS技巧
---

我们经常会遇到这样的需求：获取当前浏览器窗口大小，然后根据其大小来对页面进行样式设计。那么怎么去获取呢？

先来看这样一段代码：
```javascript
function getBrowserSize() {
  var de = document.documentElement;
  return {
    'width': (
      window.innerWidth
      || de && de.clientWidth
      || document.body.clientWidth
    ),
    'height': (
      window.innerHeight
      || de && de.clientHeight
      || document.body.clientHeight
    )
  };
}
```
这段代码就是用来获取浏览器窗口大小的，我们来看看其中用到的一些知识点。
<!--more-->
  - `documentElement`:`document.documentElement`获取的是页面中的`<html>`元素，也就相当于整个页面。
  - `innerWidth&&innerHeight`: innerWidth获取的是可视区域的宽度，但包括垂直滚动条；innerHeight则只包括可视区域高度，不包括功能框之类的。相比较之下，outerWidth和outerHeight则包含整个窗口的高度，包括功能框、滚动条之类的。
  - `document.body`:获取页面中body元素。
  - `Element.clientWidth`:获取元素的可视宽度（高度），它包括`padding`但不包括滚动条、`border`和`margin`。


## 实际测试
话不多说，直接上图：
![window-inner-outer](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/window-inner%26outer.jpg)

![browser-size-inner](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/browser-size-inner.jpg)

![browser-size-client](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/browser-size-client.jpg)

![browser-size-client-body](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/browser-size-client-body.jpg)

我们可以得出结论：
* `window.innerWidth`和`window.outerWidth`基本是一样的，因为innerWidth也包括了滚动条，但如果侧面有功能栏的话，innerWidth就会比outerWidth小；`window.innerHeight`和`window.outerHeight`差别较大，因为下面有调试栏。
* `document.documentElement.clientWidth(clientHeight)`获取的就是页面的可视窗口宽高，不包括滚动条和功能栏之类的。
* `document.body.clientWidth(clientHeight)`获取的是`body`元素的宽高，不包括滚动条，但因为`body`的内容高度有465px，所以比`document.documentElement.clientHeight`要高。

## 兼容性
`document.documentElement`几乎每个浏览器都兼容；`window.outerWidth`和`window.innerWidth`IE8及以下都不支持；`document.body`和`clientWidth`IE6之前不支持。

## 总结
获取浏览器当前窗口大小，一般用`window.innerWidth(innerHeight)`就好了，但如果不兼容，则次之用`document.documentElement.clientWidth(clientHeight)`，再不行就只能用`document.body`获取了。

前面提到的`getBrowserSize()`方法就是使用了这种理念。


> 参考资料:
> [documentElement](https://developer.mozilla.org/en-US/docs/Web/API/Document/documentElement)
> [innerWidth](https://developer.mozilla.org/en-US/docs/Web/API/Window/innerWidth)
> [outerWidth](https://developer.mozilla.org/en-US/docs/Web/API/Window/outerWidth)
> [document.body](https://developer.mozilla.org/en-US/docs/Web/API/Document/body)
> [clientWidth](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth)
> 《JavaScriptDOM高级程序设计》
