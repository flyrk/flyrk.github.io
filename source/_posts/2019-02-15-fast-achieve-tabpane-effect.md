---
title: 快速实现标签条切换效果
date: 2019-02-15 10:57:57
categories:
- CSS相关
tags:
- CSS技巧
---

现在的单页应用中，我们经常需要通过切换不同的标签条显示相应的内容，一般想到的方法就是通过DOM操作给标签条添加点击事件然后加载对应内容，那么，有没有方法可以基本靠HTML和CSS就能快速实现标签条切换效果呢？我们来看一看。
<!-- more -->

# HTML结构
这里我先简单的把HTML结构展示出来，然后慢慢解释：
```html
<div class='pane-container'>
  <div class='pane-item'>
    <div class='content' id='tab1'>
      <p>Tab1 Content</p>
    </div>
    <a class='pane-btn' href='#tab1'>Tab1</a>
  </div>
  <div class='pane-item'>
    <div class='content' id='tab2'>
      <p>Tab2 Content</p>
    </div>
    <a class='pane-btn' href='#tab2'>Tab2</a>
  </div>
  <div class='pane-item'>
    <div class='content' id='tab3'>
      <p>Tab3 Content</p>
    </div>
    <a class='pane-btn' href='#tab3'>Tab3</a>
  </div>
</div>
```

我们可以看到，每一个标签条都是一个`a`标签，对应的内容为`.content`块。这里我为什么要用`a`标签来表示标签条呢？

原因就是，`a`标签的`href`属性可以设置`href='#myid'`，这样点击就能跳转到当前页面id为myid的元素。有人会说，但这里我们不需要跳转啊，别急，这样设置的目的是为了方便之后的CSS设置。

# 关键的CSS操作
## :target伪类
这里我们用到了一个关键的CSS选择器：`:target`伪类。`E:target`伪类选择的是匹配到URL的E元素，就是说如果我们设置了E元素的id为`eId`，并且当前URL后面有`#eId`，则E元素就被CSS选择器选中了，我们就可以设置E元素的CSS属性！来看代码：
```css
.content {
  position: absolute;
  top: 28px;
  left: 0;
  width: 500px;
  height: 200px;
  display: none;
  border: 1px solid #EDEDED;
  border-radius: 2px;
  text-align: left;
}
.content:target {
  display: block;
}
```
这里我们要选中的元素是class为`content`的元素，也就是要展示的内容。先对`content`设置`display:none;`，然后通过`.content:target`选择到当前选中的`content`元素，再让其可见，就达到了内容切换的目的！怎么样，是不是很简单？

## 修改tab选中样式
当然为了更加直观一点，我们还需要对选中的标签进行一点修饰，代表当前选中的是哪个标签，其主要也是运用了`:target`和兄弟选择器：`E~F`。这里要注意的是，`~`匹配的是E后面所有兄弟元素F，也就是说F的位置必须在E后面，我们来看代码：
```css
.pane-btn:hover {
  background-color: #F1F1F1;
}
.content:target ~ .pane-btn {
  border-top: 1px solid #6EAED8;
}
```
这样就能比较明显的显示当前选中的标签。

## 完整CSS代码
```css
.pane-container {
  position: relative;
  width: 100%;
  display: flex;
}
.pane-btn {
  background-color: #fff;
  border-style: none;
  font-size: 20px;
  padding: 0 5px;
  width: 50px;
  height: 24px;
  border-radius: 2px;
}
.pane-btn:hover {
  background-color: #F1F1F1;
}
a {
  text-decoration: none;
  color: #000;
}
.content:target {
  display: block;
}
.content:target ~ .pane-btn {
  border-top: 1px solid #6EAED8;
}

.content {
  position: absolute;
  top: 28px;
  left: 0;
  width: 500px;
  height: 200px;
  display: none;
  border: 1px solid #EDEDED;
  border-radius: 2px;
  text-align: left;
}
```

# 最后
你以为这样就OK了？还差最后一步。经过上面的HTML和CSS设置，我们虽然可以在标签之间进行切换并且显示对应内容，但是在页面初次加载进来后是没有显示任何内容的，因为此时没有任何标签被点击！所以我们还得用JS小小的设置一下：
```javascript
var tabPaneOne = document.getElementsByClassName('pane-btn')[0];  // 这里默认显示第一个标签
tabPaneOne.click();
```
这样就能完美的显示啦！完整效果请看<a href="https://codepen.io/flyrk/pen/aXaBOj" target="_blank">demo</a>。

# 总结
利用`:target`伪类，我们很好的实现了点击标签切换内容的效果。虽然这只是很简单的效果，但是在实际开发过程中我们也可以运用起来，在其基础上再加一些动画都是Ok的。这样实现的好处是会少很多DOM操作，从性能和代码简洁角度来说也是很大的一个提升。