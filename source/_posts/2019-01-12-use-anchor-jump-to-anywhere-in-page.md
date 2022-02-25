---
title: 如何实现页面不同内容之间的跳转
date: 2019-01-12 17:10:54
categories:
- JS相关
tags:
- JS技巧
---

我们常常会碰到这样的需求：点击某个标题、某段话，跳转到页面对应的位置，并且浏览器不会刷新，相当于在页面进行内部导航，这在浏览文章、或者是内容比较多的长屏单页应用上有着迫切的需求。那么，如何实现这个简单的功能呢？

<!--more-->

# 利用a标签的href值

我们知道，`<a>`标签的`href`属性支持绝对路径和相对路径，其实它还支持路由hash。打个比方，我们的浏览器地址是：`http://www.example.com`，HTML里有这样一个标签：`<a href="#start">click</a>`，点击该标签后，浏览器的URL地址后面就会多出一个hash值：`http://www.example.com#start`，如果HTML里有id为`start`的元素，则浏览器窗口会自动滚动到该元素所在的位置，也就是使该元素滚动到视窗最上面。

那么，很简单的我们就可以利用这个属性实现跳转到页面任何位置，只要在想要跳转的地方设置`id`值，再通过设置`a`标签的`href`为`#id`，我们就可以实现跳转。

为什么我们可以使用`#id`实现跳转呢？我们先来看看MDN的定义：

> **href**
>
> Contains a URL or a URL fragment that the hyperlink points to.
>
> A URL fragment is a name preceded by a hash mark (`#`), which specifies an internal target location (an `id` of an HTML element) within the current document.

什么意思呢？其实`href`就是给定一个超链接，它的值指向的其实是一个URL或者URL片段，而一个URL片段通常是以`#name`的形式，代表着当前文档页面内部target的位置（也就是一个HTML元素的id），所以我们就可以利用`id`来进行页面内部元素之间的跳转。

那么如果要跳转到页面顶部呢？有的人可能会说：用JS设置`scrollTop`的值不就好了，这当然可以，之前我也写过跳转到顶部的[文章](https://xmflyrk.com/2017/08/22/scroll-to-top-btn/)。但是，如果不用JS呢？很简单，我们直接这样设置：`<a href="#">click to top</a>`，点击就能直接跳转到页面顶部了！

这是为什么呢？我们知道`#`后面的name值代表的是某个元素的`id`，但如果没有这个name，那就代表所有元素，也就是整个页面，所以就把页面移动到视窗的最顶部了，是不是很方便呢。但是这样有一点不好，就是没有滚动动画，跳转比较生硬，对于UI动画要求不高的页面用这个非常合适。

# Input和Label的妙用

那么除了利用`a`标签，还有其他方法可以在不用JS的情况下实现跳转吗？答案就是利用`input`和`label`标签！

我们先在想要跳转的地方增加一个`input`标签：

```html
<body>
    <input id="anchor">	// 这是我们要跳转的位置
    <article>
    	<h1>This is a title</h1>
        <p>
            LoraeraweMefawef wefaewfaw fawe fwae faw
        </p>
        // 假设文章很长...
    </article>
</body>
```

然后设置`CSS`令其不可见：

```css
#anchor {
  content: '';
  font-size: 0;
  width: 0;
  height: 0;
  border: 0 transparent;
}
#anchor:focus {
  outline: 0;
}
```

接下来做什么呢？我们知道`label`标签有一个`for`属性，它的值代表着对应`input`的`id`，当设置了`for`值为某个`input`的`id`，我们点击`label`，则会令对应的`<input>`变为`focus`状态，而当某个元素为focus状态时，浏览器窗口会自动使其滚动到视窗内，这样我们就实现了点击跳转功能！

```html
<body>
    <input id="anchor">	// 这是我们要跳转的位置
    <article>
    	<h1>This is a title</h1>
        <p>
            LoraeraweMefawef wefaewfaw fawe fwae faw
        </p>
        // 假设文章很长...
    </article>
    <label for="anchor">点击跳转</label>
</body>
```

# 总结

这里我们用到了两种方法实现页面元素跳转，都没有用到JavaScript，在对滚动动画要求不高的条件下，用这两种方法能简单有效的实现跳转需求，值得一试！