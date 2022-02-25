---
title: 纯CSS绘制不同的图形
date: 2018-12-03 20:44:19
categories:
- CSS相关
tags:
- CSS技巧
---

为什么要用CSS绘制图形？我们知道，一般表示图案可以用`img`标签，直接用切好的图片，或者用`background`加载背景图。但是，当我们遇到一些小图标，比如说三角箭头、半圆、对话框、圆形等等。这些简单的图案其实可以完全使用CSS来生成，这样可以极大的减少图片资源的请求和加载，从而使网页加载速度更快。
<!--more-->

这里我列举一些比较感兴趣的图形。

# 基本图案

## 圆形
圆形可以说是除了方形以外最容易表达的图形了，多亏了CSS3的`border-radius`属性：

### 基本圆形
![circle](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/circle.jpg)
```css
.circle {
  width: 100px;
  height: 100px;
  background-color: red;
  border: 1px solid #000;
  border-radius: 50%;
}
```

### 椭圆形
![oval](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/oval.jpg)
```css
.oval {
  width: 50px;
  height: 100px;
  background-color: red;
  border-radius: 50px / 100px;
}
```

## 三角形
三角形其实利用的是`border`属性，利用不同方向的`border`宽度和颜色可以展现不同的三角形。

### triangle-top：
![triangle-top](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/triangle-top.jpg)
```css
.top-triangle {
  width: 0;
  height: 0;
  border-top: 100px solid red;
  border-right: 50px solid transparent;
  border-left: 50px solid transparent;
}
```

### triangle-bottom
![triangle-bottom](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/triangle-bottom.jpg)
```css
.bottom-triangle {
  width: 0;
  height: 0;
  border-bottom: 100px solid red;
  border-right: 50px solid transparent;
  border-left: 50px solid transparent;
}
```

### triangle-left
![triangle-left](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/triangle-left.jpg)
```css
.left-triangle {
  width: 0;
  height: 0;
  border-bottom: 50px solid transparent;
  border-top: 50px solid transparent;
  border-left: 100px solid red;
}
```

### triangle-right
![triangle-right](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/triangle-right.jpg)
```css
.right-triangle {
  width: 0;
  height: 0;
  border-bottom: 50px solid transparent;
  border-top: 50px solid transparent;
  border-left: 100px solid red;
}
```

其他的三角形也就是利用border属性进行的变形。

# 特殊图案
## 跳转图标
![border-top-left-radius](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/border-top-left-radius.png)
```css
.curvedarrow {
  position: relative;
  width: 0;
  height: 0;
  border-top: 26px solid transparent;
  border-right: 26px solid red;
  transform: rotate(10deg);
}
.curvedarrow:after {
  content: "";
  position: absolute;
  border: 0 solid transparent;
  border-top: 6px solid red;
  border-radius: 20px 0 0 0;
  top: -26px;
  left: -16px;
  width: 30px;
  height: 30px;
  transform: rotate(45deg);
}
```
这里主要利用的属性有两个，一个是`border-radius: 20px 0 0 0;`。为什么要这样用呢？其实`border-radius`最多支持四个值：`border-top-left-radius`, `border-top-right-radius`,`border-bottom-right-radius`, and `border-bottom-left-radius`。这里我们把`border-top-left-radius`设为20px，其他的都设为0，再加上只设`border-top`的宽度，最后出来是一个尾部有弧度的长条。

第二个属性则是`rotate(deg)`，将两个图形经过不同的角度旋转后再移动长条的位置进行组合，最后达到跳转箭头的图标效果。

## 等腰梯形
![trapezoid](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/trapezoid.png)
梯形其实很好理解，通过设置border的宽度和透明度就可以形成梯形：
```css
.trapezoid {
  width: 100px;
  border-left: 30px solid transparent;
  border-right: 30px solid transparent;
  border-bottom: 100px solid red;
}
```

## 平行四边形
![parallel](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/parallel.png)
很简单，直接利用`transform: skew(deg)`就可以得到任意角度的平行四边形：
```css
.parallel {
  width: 100px;
  height: 50px;
  background: red;
  transform: skew(30deg);
}
```

## 五角星
其原理是利用三个三角形通过旋转不同的角度和绝对定位拼接而成。
![star-five](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/star-five.png)
具体代码如下：
```css
.star-five {
  margin: 50px 0;
  position: relative;
  display: block;
  color: red;
  width: 0px;
  height: 0px;
  border-right: 100px solid transparent;
  border-bottom: 70px solid red;
  border-left: 100px solid transparent;
  transform: rotate(35deg);
}
.star-five:before {
  border-bottom: 80px solid red;
  border-left: 30px solid transparent;
  border-right: 30px solid transparent;
  position: absolute;
  height: 0;
  width: 0;
  top: -45px;
  left: -65px;
  display: block;
  content: '';
  transform: rotate(-35deg);
}
.star-five:after {
  position: absolute;
  display: block;
  color: red;
  top: 3px;
  left: -105px;
  width: 0px;
  height: 0px;
  border-right: 100px solid transparent;
  border-bottom: 70px solid red;
  border-left: 100px solid transparent;
  transform: rotate(-70deg);
  content: '';
}
```

## 六角星
利用两个三角形拼接而成。
![star-six](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/star-six.png)
```css
.star-six {
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-bottom: 100px solid red;
  position: relative;
}
.star-six:after {
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-top: 100px solid red;
  position: absolute;
  content: "";
  top: 30px;
  left: -50px;
}
```

## 心形
利用两个半圆长条形状旋转一定角度后拼装成心形。
![heart](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/heart.png)
```css
.heart {
  position: relative;
  width: 100px;
  height: 90px;
}
.heart:before,
.heart:after {
  position: absolute;
  content: "";
  left: 50px;
  top: 0;
  width: 50px;
  height: 80px;
  background: red;
  border-radius: 50px 50px 0 0;
  transform: rotate(-45deg);
  transform-origin: 0 100%;
}
.heart:after {
  left: 0;
  transform: rotate(45deg);
  transform-origin: 100% 100%;
}
```

## 月亮
非常简单，利用了`box-shadow`属性，`box-shadow`一般支持四个值：`offset-x`、`offset-y`、`blur-radius`、`spread-radius`、`color`。还有一个值`inset`表示shadow是否内嵌。这里我们通过设置`border-radius`使其为圆形然后令`box-shadow`便宜一定值得到月亮形状。
![moon](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/moon.png)
```css
.moon {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  box-shadow: 15px 15px 0 0 red;
}
```

## 阴阳图案
刚一看到这个图案感觉很神奇，不知道怎么实现的，但其实很简单，分三步：

首先，利用`border-width`画出一个一半深、一半浅的圆形：
![yin-yang1](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/yin-yang1.png)
```css
.yin-yang {
  width: 96px;
  box-sizing: content-box;
  height: 48px;
  background: #eee;
  border-color: black;
  border-style: solid;
  border-width: 2px 2px 50px 2px;
  border-radius: 100%;
  position: relative;
}
```

然后，利用伪元素的`content`和`border`，生成一个小铜钱然后定位到适合的位置：
![yin-yang2](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/yin-yang2.png)
```css
.yin-yang:before {
  content: "";
  position: absolute;
  top: 50%;
  left: 0;
  background: #eee;
  border: 18px solid black;
  border-radius: 100%;
  width: 12px;
  height: 12px;
  box-sizing: content-box;
}
```

最后，在对称的位置上在生成一个小铜钱，颜色与之前的相反。
![yin-yang3](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/yin-yang3.png)
```css
.yin-yang:after {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  background: black;
  border: 18px solid #eee;
  border-radius: 100%;
  width: 12px;
  height: 12px;
  box-sizing: content-box;
}
```

两个一拼接，就形成了想要的阴阳图案：
![yin-yang](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/yin-yang.png)

# 总结
利用CSS，我们可以实现很多简单甚至是较复杂的图案，这样省去了切图的烦恼，还减少了http请求，资源加载速度变快，唯一不足的可能就是基本上都是用的CSS3属性，有兼容性问题。但是，既然能直接用CSS画出图案，当然是用起来！

> 更多图案参见[参考资料](https://css-tricks.com/the-shapes-of-css/)