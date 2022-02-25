---
title: 原生JS实现回到顶部的效果
date: 2017-08-22 14:31:45
categories:
- JS相关
tags:
- JS技巧
---

我们在浏览网页时通常会看到有返回顶部的按钮。当我们移动鼠标滑轮或者方向键时使页面开始滚动，如果我们滚动了一会想回到页面顶部时，这个时候就需要回到顶部按钮，那这个按钮是怎么实现的呢？
<!--more-->
# JS实现滚动
其实很简单，利用下列函数就可以实现：
```javascript
/*
 * func: 实现页面滚动到顶部的效果，
 * 离顶部越近滚动速度越慢
 * @acceleration: 滑动的加速度
 * @time: 延迟时间 
 */
function goTop(acceleration, time) {
  let xScroll = document.documentElement.scrollLeft || document.body.scrollLeft || window.scrollLeft || 0,   // 获取水平滚动坐标
      yScroll = document.documentElement.scrollTop ||document.body.scrollTop || window.scrollTop || 0,  // 获取垂直滚动坐标
      speed = 1 + acceleration; // 滚动速度

  window.scrollTo(Math.floor(xScroll / speed), Math.floor(yScroll / speed)); // 屏幕滚动到某个坐标，因为speed大于1，所以x、y轴的坐标越来越小

  
  if (xScroll > 0 || yScroll > 0) { // 如果没有滚动到顶部就设置延迟time后继续滚动
      setTimeout(() => {
          goTop(acceleration, time);
      }, time);
  }
}
```

在 *返回顶部* 的按钮上绑定`onClick="goTop();"`就可以实现返回顶部操作。

这里用到了`xScroll = document.documentElement.scrollLeft || document.body.scrollLeft || window.scrollLeft || 0;`，其实这三个值的效果都差不多，都代表滚动条水平移动的像素，只是浏览器兼容性可能有所不同，所以这样写兼容性更好。

然后就是`window.scrollTo(x, y)`方法。该方法使当前窗口滚动到指定的x、y像素坐标。之所以除以speed，这样就以一定的速度慢慢减小，达到缓慢的动画效果。

之后判断如果滚动条距离顶部还有一段距离的话，就继续循环调用该函数，并设置一定的延迟时间，以达到平滑滚动的动画效果。这里的`time`值要设置的比较小，超过100就显得比较不自然了。

# 按钮的样式设置
滚动特效实现了，但一般我们只有当页面发生滚动并且滚动到一定距离时才想要回到顶部，我们希望回到顶部的按钮在一定情况下才出现，于是我们需要设置按钮何时出现。通过在`scroll`事件上绑定函数：
```javascript
let scrollBtn = document.getElementsByClassName('scroll2Top-btn')[0];
  window.addEventListener('scroll', () => { // 这里用了ES6语法
    let contentTop = document.documentElement.clientHeight || window.innerHeight, // 获取当前可视窗口的高度
      scrollTop = document.documentElement.scrollTop || document.body.scrollTop;  // 获取垂直滚动条距离页面顶部的距离
    
    if (contentTop < scrollTop) { // 如果可视窗口的高度小于垂直滚动的距离，说明已经向下滚动超过一个可视窗口的距离，也就是说看不到顶部了，于是就设置按钮可见
      scrollBtn.style.display = "block";
    } else {
      scrollBtn.style.display = "none";
    }
  });
```
---

通过这两个简单的函数，实现了回到顶部的滚动效果，全部用原生JS编写，简单又实用。