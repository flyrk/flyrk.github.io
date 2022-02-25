---
title: 用原生JS写轮播图
date: 2017-08-27 16:53:36
categories:
- JS相关
tags:
- JS技巧
---

我们经常可以在网页上看到轮播图的效果，这是一个很常见的应用，但是，要想比较完美地实现这个功能，还是需要花点时间的。
<!--more-->
# 要实现的功能
首先我们来看看要实现这么一个轮播图需要哪些功能，这里我把我想到的都列出来了。
1. 页面加载后轮播图自动开始播放，每张图片停几秒钟。
2. 图片与图片之间实现平滑过渡动画效果，不显突兀。
3. 鼠标悬停到当前图片时轮播动画停止，鼠标离开图片后继续开始轮播。
4. 图片上有左右翻页功能按钮，点击左边按钮图片往左滑动，点击右边按钮图片右滑。
5. 图片下端有显示图片个数的小圆点，当前图片是第几个，则第几个小圆点“点亮”。
6. 离开当前页面后轮播动画停止，回到当前页面后轮播动画继续。

我能想到的要实现的功能就这么多，接下来就一步步开始实现。

# HTMl结构
先不管JS、CSS部分，我们先把整体结构定下来。这里直接贴上主体部分代码：
```html
<div class="wrap">
  <div class="loop-container">
    <img src="./assets/cat5.jpg" alt="5" class="loop-image">
    <img src="./assets/cat1.jpg" alt="1" class="loop-image">
    <img src="./assets/cat2.jpg" alt="2" class="loop-image">
    <img src="./assets/cat3.jpg" alt="3" class="loop-image">
    <img src="./assets/cat4.jpg" alt="4" class="loop-image">
    <img src="./assets/cat5.jpg" alt="5" class="loop-image">
    <img src="./assets/cat1.jpg" alt="5" class="loop-image">
  </div>
  <div class="buttons">
    <span class="on">1</span>
    <span>2</span>
    <span>3</span>
    <span>4</span>
    <span>5</span>
  </div>

  <div class="arrow arrow-left">
    <div class="pt">
      <span class="pt-inner"></span>
    </div>
    
  </div>

  <div class="arrow arrow-right">
    <div class="pt">
      <span class="pt-inner"></span>
    </div>
  </div>
</div>
```
这里要强调的一点就是，虽然最后显示只有5张图片，但我插入了7个`img`标签，其中第一个和最后一张图一样，最后一个和第一个图一样。为什么要这么做呢？是因为要实现平滑的动画过渡效果，后面JS部分会提到。

# CSS样式
接下来就是CSS设置：
```css

.wrap {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  padding: 0;
  margin: 0;
  width: 600px;
  height: 500px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  overflow: hidden;
}
.wrap .loop-container {
  position: absolute;
  left: -600px;
  top: 0;
  width: 700%;
  height: 100%;
  margin: 0;
  padding: 0;
  animation: left .6s ease-out;
  font-size: 0;
}
.wrap:hover > .arrow {
  display: block;
}

.loop-container .loop-image {
  width: 600px;
  height: 500px;
  margin: 0;
}

```
这里我没有把全部的CSS代码贴出来，完整的代码我会在最后给出。

我只说几个要注意的地方：
- 首先外部容器要设置`overflow:hidden`，这样才能把多余的图片遮住；
- 其次由于img默认是inline元素，显示出来的特性是inline-block，所以img之间会有4px的空隙，即使设置了margin和padding为0也不能消除。为此我困惑了很久。。。后来上网找资料才发现解决方案。一般有几种解决方案，这里我用的是设置父元素的font-size为0，然后img的font-size设不设置根据需要，这样就可以消除空隙。更详尽的解决方案可以参考张大神的[博客](http://www.zhangxinxu.com/wordpress/2012/04/inline-block-space-remove-%E5%8E%BB%E9%99%A4%E9%97%B4%E8%B7%9D/)

其他css设置就根据样式慢慢调整了。

# JS部分
接下来就是重头戏，JS的实现了。我们一步一步来看。

我先创建了一个整体的“类”，里面有一些所需要用到的方法和属性：
```javascript
function Marquee() {
  this.timer = 0;
  this.index = 0; // 保存当前是第几张图
}

Marquee.prototype.animate = function (aimLeft) {  // 具体的动画实现
  var curLeft = parseInt(loopContainer.style.left) || -600, // 获取当前图片的left值
      speed = (aimLeft - curLeft) / 20, // 每次left移动的距离
      delay = 20,
      self = this;
  var time = setInterval(function () {  // 利用循环定时实现平滑移动的动画效果
    curLeft += speed;
    loopContainer.style.left = curLeft + 'px';
    if (curLeft === aimLeft) {  // 如果移动到了下一张图片的位置，则此次移动动画结束
      clearInterval(time);
      if (aimLeft <= -3600) {  // 特殊设置，如果是从最后一张图到第一张图，中间加一张图片实现动画过渡，当到达最后一张图后，立即设置left为第二个img的left。 
        loopContainer.style.left = '-600px';  // 第一张图实际的left为-600px，因为html里left为0的位置是最后一张图
      }
      if (aimLeft >= 0) { // 同理，当从第一张图过渡到最后一张图，先实现动画效果，当left为实际第一张图的位置0时，设置left为倒数第二个img的位置。
        loopContainer.style.left = '-3000px';
      }
      self.showCurrentDot();  // 动画结束后再改变小圆点的外观
    }
  }, delay);
};
Marquee.prototype.showCurrentDot = function () {  // 设置代表当前图片位置的小圆点class
  var dots = document.getElementsByTagName('span');
  for (var i = 0, len = dots.length; i < len; i++) {
    dots[i].className = '';
  }
  dots[this.index].className = 'on';
};

Marquee.prototype.changePhoto = function (offset) { // 自动改变图片的函数
  var left = loopContainer.style.left,
    newleft = left ? parseInt(left) + offset : offset - 600;  // 新的位置
  
  this.index = offset > 0 ? this.index - 1 : this.index + 1;  // 判断向左还是向右滑动
  if (this.index > 4) {
    this.index = 0;
  }
  if (this.index < 0) {
    this.index = 4;
  }
  // console.log(left);
  // console.log(newleft);
  // console.log('------');
  this.animate(newleft);
};

Marquee.prototype.gotoPhoto = function (count) {  // 跳转到第count个图片的函数
  var newleft = count * -600;
  // console.log(newleft);
  this.index = count - 1;
  this.animate(newleft);
}

var maq = new Marquee();
```
## 自动播放图片
轮播图，顾名思义就是轮流播放图片，所以首先要实现的功能就是自动轮流循环播放图片。 有了之前的类，我们要做的就是当页面加载完毕后开始循环播放：
```javascript
window.addEventListener('load', function() {
  var prevBtn = document.getElementsByClassName('arrow-left')[0],
      nextBtn = document.getElementsByClassName('arrow-right')[0],
      loopContainer = document.getElementsByClassName('loop-container')[0],
      btns = document.getElementsByClassName('buttons')[0],
      wrap = document.getElementsByClassName('wrap')[0];
  var stopFlag = 0;

  function startInterval() {  // 开始循环动画
    maq.timer = setTimeout(function () {
      // console.log(stopFlag);
      if (!stopFlag) {
        maq.changePhoto(-600);
        startInterval();
      } else {
        clearTimeout(maq.timer);
      }
    }, 4500);
  }
  startInterval();
});
```
这里我循环动画没有用setInterval函数，而是用的setTimeout。因为setInterval的机制是每隔一段时间就把事件加入到队列中去，但如果之前的事件还没执行完，就会容易造成队列堵塞。比如说setInterval里的函数执行时间要4秒，如果setInterval的间隔时间少于4秒，则会造成队列里的事件越来越多，而之前的事件却没执行完，这样可能就会使队列里的事件堵塞，最后一次性全部执行，而没有达到预期的间隔效果。

所以我用setTimeout来代替setInterval，每次要加入新的事件之前都先判断一下`stopFlag`是否为0。`stopFlag`的作用就是记录是否要停止动画，为0则不停止，为1则停止。

这里记住要clearTimeout，目的是把已经在队列里但还没有执行的事件清除，这样可以达到立即停止动画的效果。

## 鼠标悬停停止轮播动画，离开后开始动画
监听mouseover和mouseout事件来达到目的：
```javascript
wrap.addEventListener('mouseover', function () {
  stopFlag = 1;
  clearTimeout(maq.timer);
});
wrap.addEventListener('mouseout', function () {
  stopFlag = 0;
  startInterval();
});
```

## 左右切换图片
通过点击左右箭头按钮实现图片之间的滚动切换：
```javascript
prevBtn.addEventListener('click', function() {
    maq.changePhoto(600); // 向左滑
});
nextBtn.addEventListener('click', function () {
  maq.changePhoto(-600);  // 向右滑
});
```

## 点击小圆点跳转到对应图片
这里我用了事件代理，不用在每个小圆点上绑定click事件，提高dom性能：
```javascript
btns.addEventListener('click', function (event) {
  var count = parseInt(event.target.innerText);
  if (count < 6) {
    maq.gotoPhoto(count);
  }
});
```

## 离开当前页面动画停止
这里我开始没有想到，后来是当我每次切换到别的页面后再回到当前页面，发现动画效果出现问题了。经过一番debug才发现是因为chrome浏览器设置了离开当前页面后setInterval继续执行，如果setInterval的间隔时间小于100ms，则按100ms来执行，于是回来时动画的时间就发生错误了。

所以我们需要设置离开页面时动画停止，这样也节省了不少性能。这里利用的是`onvisibilitychange`事件：
```javascript
document.addEventListener('visibilitychange', function () {  // 离开当前页面后动画停止
  if (document.hidden) {
    stopFlag = 1;
    clearTimeout(maq.timer);
  } else {
    stopFlag = 0;
    startInterval();
  }
});
```
这里可以直接调用`document.hidden`API判断当前页面是否被隐藏，如果document.hidden为true则代表已经切换到别的页面，于是设置`stopFlag`为1，使动画停止。也可以用`document.visibilityState`,如果不为'visible'，则代表离开了当前页面。

这里其实可以不用setTimeout、setInterval来实现动画，而是用`requestAnimationFrame`，后者的优点是自动以浏览器支持的最小刷新间隔来实现重绘，使性能得到最大化提升，而且实现了离开页面重绘停止，大大节省性能。这里我就没有实现了，要想了解requestAnimationFrame，可参考[资料](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)。

# 总结
通过上述方法，最后实现了一个比较完善的轮播图，全部的动画效果都是用JS来实现的。其实要想实现自动轮播，也可以用CSS3的animation来实现，而且实现起来更快，这里我就不阐述了。

最后贴出实现的[demo](https://codepen.io/flyrk/full/brxQBm/)。
