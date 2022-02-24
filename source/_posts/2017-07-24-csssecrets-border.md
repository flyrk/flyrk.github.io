---
title: CSS边框的应用
date: 2017-07-24 13:34:47
toc: true
categories:
- CSS
tags:
- CSS技巧
---

CSS中盒模型里重要的一个部分就是border，常见的border方法就是设置一重边框：border: 1px solid #494949，但如果我们想实现其他效果，该怎么用呢？

<!--more-->
# CSS实现多重边框
一般的边框设置border只能达到一重边框的效果，如果想达到多重边框，一般要使用各种hack，比如用多个元素来模拟边框。那么有没有更好的办法呢？有。以下方法也能实现多重边框，并且不需要添加多余元素来污染dom结构。

## box-shadow方案
我们知道box-shadow一般用来生成投影，MDN的box-shadow是这样定义的：

> Specify a single box-shadow using:
> - Two, three, or four values.
> - If only two values are given, they are interpreted as values.
> - If a third value is given, it is interpreted as a .
> - If a fourth value is given, it is interpreted as a .
> - Optionally, the inset keyword.
> - Optionally, a value.

我们注意到，一般前两个值代表x和y方向的偏移，第三个值代表模糊值，而第四个参数代表扩张半径，通过指定该值为正或负可以让投影面积加大或减小。于是我们可以实现这样的效果：
```css
background: lightblue;
box-shadow: 0 0 0 10px #533;
```

当然这和border的效果没什么区别，但box-shadow的好处在于，它支持逗号分隔语法，我们可以创建任意数量的投影。于是我们可以实现多重投影：
```css
background: lightblue;
box-shadow: 0 0 0 10px #533, 0 0 0 15px #2f4, 0 2px 5px 15px rgba(0, 0, 0, .6);
```

注意的是，box-shadow是层层叠加的，第一层投影位于最顶层，依此类推。所以我们如果想在之前的10px外圈上加一道5px的外框，则需要指定扩张半径的值为15px。

需要注意的是：

- 投影的行为和边框不完全一致，因为它不会影响布局，也不会受到box-sizing的影响。
- box-shadow创建出的边框不会响应鼠标事件。如果一定要响应的话，应该给box-shadow加上inset关键字，来使投影绘制在元素的内圈。

## outline方案
某些情况下可能只需要两层边框，这个时候可以用一层border再加上outline实现，好处是outline可以实现虚线边框，而box-shadow只能实现实线边框。

```css
background: lightblue;
border: 10px solid #533;
outline: 5px solid #2f4;
```

需要注意的是：

它只适用于双层边框，而且不一定会贴合border-radius属性产生的圆角。

# 边框内圆角实现
有时候我们需要实现一个容器，只在内部有圆角，而边框外侧四个角则是直角。要实现这个效果，最容易想到的方法就是使用两个元素。

```html
<div class='container'>
  <div>This is a box</div>
</div>
```

```css
.container {
  background: #655;
  padding: .6em;
}

.container > div {
  background: tan;
  padding: 1em;
  border-radius: .8em;
}
```

但使用两个元素总是有点不好，能不能只使用一个元素呢？答案是可以。我们可以利用box-shadow和outline达到同样的效果。

```css
.container {
  background: tan;
  padding: 1em;
  border-radius: .8em;
  box-shadow: 0 0 0 .5em #655
  outline: .6em solid #655;
}
```
实现的效果和之前一样。

因为box-shadow会跟着元素的圆角走，而outline不会，所以利用两者叠加，box-shadow正好把border和outline之间的空隙填补了。

注意的是，box-shadow的扩张值不一定等于outline的宽度，一般使用稍小一点的值。根据学过的勾股定理，最小的扩张值一般要大于(&radic;2-1)*border-radius，为了方便，一般取border-radius的一半。