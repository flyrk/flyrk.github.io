---
title: 实现基于codemirror的markdown编辑器(二)
date: 2018-12-20 18:39:16
toc: true
categories:
- JS相关
tags:
- 编辑器
- markdown
---

前面我们提到了如何加载`codemirror`组件和一些基本配置，以及如何实现实时预览。接下来才是重头戏，我们需要实现同屏滚动和添加自定义按钮。
<!--more-->

这篇我们先来着重讲讲同屏滚动。

# 同屏滚动
我们写文档时，除了想要实时预览，还想要能同步滚动，这样写到哪预览页面就滚动到哪，不用鼠标移来移去，方便不少。但是如何实现呢？我们一步一步来。

## 监听事件
首先，要想滚动，肯定得监听滚动事件`onscroll`，但是这里我没有直接监听`scroll`事件，因为我们有两个区域，左边是输入文本框，右边是预览区，我想实现这样的功能：当鼠标移到左边使滚动条滚动时，这个时候先监听左边的滚动事件计算出相应的滚动高度后再修改右边的滚动高度，这样右边是跟着左边的高度滚动而滚动的。相应的，当鼠标移到右边区域使滚动条滚动时，先监听右边的滚动事件计算出滚动高度后再修改左边的滚动高度。这样滚动才能实现对应关系，但是如果直接监听滚动事件，则会出现一个问题，当右边滚动时，左边滚动事件没有移除，则又会触发，计算高度后引发右边滚动，形成一个一直滚动的死循环，最后整个页面滚动位置都是乱的。
所以，这里我先监听`mouseover`和`mouseleave`事件，话不多说看代码：
```javascript
  //...
  codemirrorScroll = (doc) => () => {
    this.props.onScroll && this.props.onScroll(doc, this.editRoot);
  }

  codemirrorScrollHandler = () => {
    this.codeMirror.on('scroll', this.codemirrorScroll(this.codeMirror.doc));
  }

  codemirrorRemoveScroll = () => {
    this.codeMirror.off('scroll', this.codemirrorScroll(this.codeMirror.doc));
  }
  //...
  componentDidMount() {
    //...
    this.editRoot.addEventListener('mouseover', this.codemirrorScrollHandler);
    this.editRoot.addEventListener('mouseleave', this.codemirrorRemoveScroll);
  }
```
注意到，我在`mouseover`的时候开始监听`scroll`事件，`mouseleave`的时候移除之前的`scroll`事件，所以这里要用`codemirrorScroll`函数封装，以便移除时是同一个函数。
这里我把`scroll`事件的处理通过props抛给父组件，我们来看看父组件是怎么实现`scroll`事件的。

首先考虑到性能和滚动流畅度问题，使用`debounce`函数包装一下：
```js
  debounceContentScroll = debounce(this.handleScrollContent);
```
接下来就是如何去计算当前滚动的位置并使另一边的内容自动滚动到对应高度，这里我用一幅图解释一下：
![markdown-scroll](https://photos-1258216033.cos.ap-shanghai.myqcloud.com/markdown-scroll.png)
我们主要获取的就是这三个值：`child.offsetHeight`、`parent.offsetHeight`、`child.scrollTop`。所以我们需要在展示内容外面都用`div`包一层，代表`parent`元素。先看代码：
```js
  calcScrollScale = (scrollTopMax1, scrollTopMax2) => (scrollTopMax1 / scrollTopMax2);

  calcScrollTopMax = (parent, child) => Math.abs((child.offsetHeight || child.height) - parent.offsetHeight)

  updateScroll = (scrollTop, target, scrollTopMax1, scrollTopMax2) => {
    const scale = this.calcScrollScale(scrollTopMax1, scrollTopMax2);
    target.scrollTop = scrollTop / scale;
  }

  handleScrollContent = (doc, docParent) => {
    const mdPreview = this.mdPreview.current;
    const previewContent = this.previewContent.current;
    const scrollTopMaxFrom = this.calcScrollTopMax(docParent, doc);
    const scrollTopMaxTo = this.calcScrollTopMax(mdPreview, previewContent);
    this.updateScroll(doc.scrollTop, mdPreview, scrollTopMaxFrom, scrollTopMaxTo);
  }
```
这里有个很重要的参数：`scrollTopMax`。
`scrollTopMax = |(child.offsetHeight || child.height) - parent.offsetHeight|`。使用`child.height`是因为codemirror的`doc`对象内容高度可以直接通过`height`属性获得。为什么要这样计算？
我们算的是当前内容可滚动的最大高度，为什么要计算这个？因为我们知道因为渲染的原因，左边输入的时候是单纯的文本，右边渲染出来有标题、图片等等，实际高度会比左边高得多，通过计算两边相对父容器可滚动的最大高度，再计算这两个的比值：`calcScrollScale = (scrollTopMax1, scrollTopMax2) => (scrollTopMax1 / scrollTopMax2);`，可以得到一个`scrollScale`。
我们有这样一个公式：`left.scrollTop / right.scrollTop = left.scrollTopMax / right.scrollTopMax`，那么我们很容易得出右边的滚动高度：`right.scrollTop = left.scrollTop / scrollScale`。于是就可以使右边滚动到对应内容的高度了！

有了左边的范例，对右边预览的container同样使用一样的监听事件，这样就可以实现双向同步滚动绑定了！！！

# 总结
实现同步滚动的关键点其实有两个：一个是注意监听事件的变化，先监听`mouseover`和`mouseleave`，这样不会出现滚动死循环。另一个是理解公式：`left.scrollTop / right.scrollTop = left.scrollTopMax / right.scrollTopMax`。理解了这两个关键点，其实同步滚动就很容易了。
但要注意的是，这里的`debounce`设置延迟时间可能还需要好好斟酌，我默认使用20毫秒，实际滚动时还是会有点卡顿，可以适当改变一下数值。
还有就是，因为markdown语法原因，段落之间都必须有一个空行，不然渲染出来文本没换行，导致段落不一致。

> 参考资料：
> - [scrollTop](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollTop)
> - [原生JS控制多个滚动条同步跟随滚动](https://juejin.im/post/5a3bb40e5188252b145b38e3)
